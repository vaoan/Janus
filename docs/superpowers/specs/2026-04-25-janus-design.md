# Janus — Event Ticketing Platform Design

**Date:** 2026-04-25
**Status:** In progress — architecture, data model, and entitlement system defined. Interpreter detail, claim flow detail, and studio creative tools to be fleshed out in follow-up sessions.

---

## Overview

Janus is an event ticketing and check-in management platform. It is **not** a payment processor — tickets are sold elsewhere (primarily CandyStore). Janus handles everything that happens after payment: importing attendee data, distributing ticket claims, managing entitlements, running check-in at the event, and auditing every human action.

Janus is a sister project to CandyStore. Both repos share the same toolchain, conventions, code quality rules, CI/CD discipline, and architecture patterns. They evolve in lockstep. Improvements to one should be evaluated for the other.

---

## Tech Stack

Identical to CandyStore:

- **Framework**: Next.js 16 (App Router, server components by default)
- **Backend**: Supabase (PostgreSQL, RLS, Auth, Storage, Realtime)
- **Monorepo**: pnpm workspaces + Turbo
- **State**: TanStack Query v5
- **Forms**: React Hook Form + Zod
- **UI**: shadcn/ui + Radix + Tailwind v4
- **i18n**: next-intl (EN/ES, bilingual from day one)
- **Codegen**: Orval (typed API clients from OpenAPI)
- **Testing**: Playwright E2E, Vitest unit, Testing Library components
- **Quality**: ESLint (same plugin set), Prettier, Stylelint, Husky pre-commit/pre-push, TypeScript strict
- **Auth**: Supabase social login only — Google and Discord. No email/password auth.
- **Deployment**: Docker Compose staging, Cloudflare tunnel, same environment system as CandyStore

---

## Applications

6 apps in the monorepo:

| App | Audience | Purpose |
|---|---|---|
| `auth` | Everyone | Social login (Google, Discord), OAuth callback |
| `admin` | Superadmins | Interpreter management, system config, global permissions, event creation |
| `studio` | Designers / event creators | Event page design, QR canvas/branding, creative tools (event must exist first) |
| `manager` | Event organizers + assigned staff | Import data, manage attendees, configure entitlements, staff assignment, live check-in dashboard |
| `events` | Attendees (public) | Browse events, claim ticket, pre-register, view QR, My Ticket page, attendee list |
| `checkin` | Door staff | Gate-mode QR scanner, fast allow/deny feedback per entitlement |

**Shared packages** (same as CandyStore): `packages/api`, `packages/ui`, `packages/shared`, `packages/auth`, `packages/app-components`

**Clean architecture** enforced per feature in every app: `domain → application → infrastructure → presentation`

---

## Data Model

### `events`
Core event record.

| Field | Type | Notes |
|---|---|---|
| `id` | uuid | |
| `slug` | text | URL-safe identifier |
| `name_en` / `name_es` | text | Bilingual |
| `description_en` / `description_es` | text | Bilingual |
| `location` | text | |
| `date_start` / `date_end` | timestamptz | |
| `capacity` | int | Nullable = unlimited |
| `status` | enum | `upcoming \| active \| ended` |
| `lock_date` | timestamptz | After this, claimed tickets cannot be revoked by the buyer |
| `cover_image_url` | text | |
| `created_by` | uuid | User who created the event |
| `created_at` | timestamptz | |

### `interpreters`
Reusable import schema. Defines how an external file maps to Janus's data model. Created by superadmins, reused across any event.

| Field | Type | Notes |
|---|---|---|
| `id` | uuid | |
| `name` | text | e.g. "CandyStore Export v1", "Conspace CSV" |
| `source` | text | e.g. `candystore`, `conspace`, `custom` |
| `type` | enum | `json \| csv \| xls \| xlsx` |
| `field_mappings` | jsonb | Maps incoming fields to Janus fields. Each entry has a strategy: `map`, `link_url`, `image_skip`, `constant` |
| `created_by` | uuid | |
| `created_at` | timestamptz | |

### `imports`
A single file import run. Ties a file to an event using a chosen interpreter.

| Field | Type | Notes |
|---|---|---|
| `id` | uuid | |
| `event_id` | uuid | |
| `interpreter_id` | uuid | |
| `filename` | text | |
| `status` | enum | `pending \| processing \| completed \| failed` |
| `row_count` | int | |
| `error_log` | jsonb | Per-row errors |
| `imported_by` | uuid | |
| `imported_at` | timestamptz | |

### `orders`
A buyer record created by an import. Represents one purchase that may include multiple tickets.

| Field | Type | Notes |
|---|---|---|
| `id` | uuid | |
| `event_id` | uuid | |
| `import_id` | uuid | |
| `buyer_email` | text | From import — may differ from any Janus account |
| `buyer_name` | text | Used for identity verification / fail-safes |
| `ticket_count` | int | Number of ticket slots to generate |
| `claim_secret_hash` | text | Nullable. bcrypt hash of buyer's optional password for claiming slots |
| `created_at` | timestamptz | |

### `ticket_slots`
One row per individual ticket. The unit of ownership.

| Field | Type | Notes |
|---|---|---|
| `id` | uuid | |
| `order_id` | uuid | |
| `token` | uuid | The value encoded in the QR code. Never the slot ID. |
| `status` | enum | `unclaimed \| claimed \| checked_in` |
| `claimed_at` | timestamptz | |
| `checked_in_at` | timestamptz | |
| `invalidated_at` | timestamptz | Set when a QR is voided and regenerated |
| `invalidated_by` | uuid | Staff member who invalidated |
| `previous_token` | uuid | The old token, kept for audit trail |

### `attendees`
A claimed slot linked to a real Janus user account.

| Field | Type | Notes |
|---|---|---|
| `id` | uuid | |
| `ticket_slot_id` | uuid | |
| `user_id` | uuid | Supabase auth user |
| `display_name` | text | |
| `avatar_url` | text | |
| `claimed_at` | timestamptz | |

### `attendee_metadata`
Flexible jsonb store for event-specific attendee fields that can't be predicted upfront (dietary needs, shirt size, accessibility requirements, etc.).

| Field | Type |
|---|---|
| `id` | uuid |
| `attendee_id` | uuid |
| `event_id` | uuid |
| `data` | jsonb |
| `updated_at` | timestamptz |

### `event_entitlements`
Defines the named access rights an event offers, each with its own rule engine.

| Field | Type | Notes |
|---|---|---|
| `id` | uuid | |
| `event_id` | uuid | |
| `name_en` / `name_es` | text | |
| `icon` | text | |
| `color` | text | |
| `sort_order` | int | |
| `capacity` | int | Nullable = unlimited |
| `rule` | jsonb | Access rule — see Rule Engine below |
| `created_by` | uuid | |

**Rule Engine** — `rule` jsonb shapes:

```json
{ "type": "total", "max": 1 }
{ "type": "daily", "max": 1 }
{ "type": "interval", "hours": 6, "max": 1 }
{ "type": "window", "from": "2026-08-02T20:00Z", "to": "2026-08-03T02:00Z", "max": 1 }
{ "type": "dates", "dates": ["2026-08-01", "2026-08-02"], "max": 1 }
{ "type": "unlimited" }
```

### `ticket_entitlements`
Links a ticket slot to the entitlements it grants. Populated at import time via interpreter mapping.

| Field | Type |
|---|---|
| `id` | uuid |
| `ticket_slot_id` | uuid |
| `event_entitlement_id` | uuid |

### `entitlement_uses`
Immutable log of every scan attempt against any entitlement. Replaces a simple check-ins table.

| Field | Type | Notes |
|---|---|---|
| `id` | uuid | |
| `ticket_entitlement_id` | uuid | |
| `gate_config_id` | uuid | Which gate performed the scan |
| `scanned_by` | uuid | Staff user |
| `scanned_at` | timestamptz | |
| `result` | enum | `allowed \| denied_rule \| denied_invalid \| already_used` |
| `device_info` | jsonb | |

### `gate_configs`
Configures what a checkin app instance scans for at a specific physical location.

| Field | Type | Notes |
|---|---|---|
| `id` | uuid | |
| `event_id` | uuid | |
| `event_entitlement_id` | uuid | Which entitlement this gate checks |
| `name` | text | e.g. "Main entrance", "Meal hall B", "Shuttle pickup" |
| `created_by` | uuid | |

### `staff_assignments`
Records who is assigned to an event. What they can *do* is determined by their permissions, not by this table.

| Field | Type |
|---|---|
| `id` | uuid |
| `event_id` | uuid |
| `user_id` | uuid |
| `assigned_by` | uuid |
| `assigned_at` | timestamptz |

### `qr_designs`
Studio-created QR canvas designs for an event. The active design is used when generating attendee QR codes.

| Field | Type | Notes |
|---|---|---|
| `id` | uuid | |
| `event_id` | uuid | |
| `name` | text | |
| `canvas_data` | jsonb | Full canvas definition |
| `is_active` | bool | Only one active per event |
| `created_by` | uuid | |
| `created_at` | timestamptz | |

### `audit.logged_actions`
Same immutable trigger-based audit system as CandyStore. Covers every tracked table. No human action goes unrecorded — permission grants, revocations, check-ins, QR invalidations, slot reassignments, import runs, event edits, staff assignments, entitlement changes. Before/after state captured on every write.

---

## Permission System

Same three-table model as CandyStore: `permissions`, `resource_permissions`, `user_permissions` with `grant | deny` modes, expiry dates, and the `has_permission()` function. RLS policies enforce it at the database level.

Permissions are scoped to events via `resource_type: 'event'`, `resource_id: <event_id>`. A user can have `attendees.read` on Event X without any access to Event Y.

### Permission Keys

**Event-scoped** (assigned per event via `resource_type: 'event'`):

| Key | What it allows |
|---|---|
| `event.edit` | Edit event details |
| `event_staff.manage` | Assign/revoke any permission on this event — **delegatable** |
| `event_staff.assign` | Assign staff (non-manager permissions only, non-delegatable) |
| `imports.create` | Run a file import against this event |
| `imports.read` | View import results and error logs |
| `attendees.read` | View the attendee list |
| `attendees.manage` | Edit attendee data, reassign slots, handle fail-safes |
| `ticket_slots.reassign` | Move a slot to a different user account |
| `ticket_slots.invalidate` | Void a QR and generate a new token |
| `check_ins.create` | Scan QR codes to check in via a gate |
| `check_ins.manual` | Check in by name/order lookup without a QR |
| `badges.print` | Print physical badges |
| `entitlements.manage` | Create and edit event entitlements and gate configs |
**System-scoped** (global, not tied to a specific event):

| Key | What it allows |
|---|---|
| `interpreters.create` | Create reusable interpreter schemas |
| `interpreters.read` | View interpreters |
| `interpreters.update` | Edit interpreters |
| `interpreters.delete` | Delete interpreters |
| `user_permissions.create` | Grant permissions to users |
| `user_permissions.read` | View user permissions |
| `user_permissions.update` | Modify permissions |
| `user_permissions.delete` | Revoke permissions |

### Role Templates (not enforced — pre-bundled permission sets for convenience)

| Template | Permissions |
|---|---|
| Event manager | All event-scoped keys + `event_staff.manage` |
| Supervisor | All event-scoped keys except `event_staff.manage` |
| Data staff | `imports.create`, `imports.read`, `attendees.read` |
| Door staff / ticketer | `check_ins.create` |
| Fail-safe staff | `ticket_slots.reassign`, `ticket_slots.invalidate`, `check_ins.manual`, `attendees.manage` |
| Badge printer | `badges.print`, `attendees.read` |
| Attendee | Own ticket slot data only (enforced via RLS, no explicit permission needed) |

### Permission Delegation

The event creator (studio user) automatically holds full permissions on their event. They grant `event_staff.manage` to the operational manager. The manager can then assign any permission — including `event_staff.manage` — to others. No user can grant a permission they do not themselves hold.

---

## Ticket Claim Flow

1. Admin imports a file → system creates `orders` (buyer email + ticket count) and `ticket_slots` (one per ticket, each with a unique `token` UUID)
2. System emails the buyer N claim links — one per slot
3. Buyer distributes links to attendees. Each link can optionally require a claim secret (password) that only the buyer knows. The buyer sets this password and can change it at any time before the lock date.
4. Each recipient opens their link, logs in with social auth, fills in their info, and claims the slot. The slot `status` moves from `unclaimed` to `claimed` and an `attendee` record is created.
5. Buyer can revoke and reassign any unclaimed slot, or any claimed slot, until the event's `lock_date`. After the lock date, all claimed tickets are frozen — no revocations.
6. Once claimed, the attendee's QR code is available in their My Ticket page. The QR encodes the slot's `token` UUID, not the attendee ID.
7. Unclaimed slots remain in the system. Staff with `check_ins.manual` can check in a person by matching their name or buyer email against the order data (e.g., cross-referencing a bank transaction record with imported data).

### Buyer Dashboard (events app)
The buyer has a dedicated page showing all their orders across events: each slot, who claimed it, claim status, and the ability to revoke/reassign before the lock date.

---

## QR Security

- Each slot token is a random UUID generated at import time. It does not encode any attendee information.
- On scan, the checkin app calls an API: "check entitlement X for token Y at gate Z." The system evaluates the entitlement rule against `entitlement_uses`, returns allow or deny with a reason, and records the attempt.
- Double-scan returns `already_used` with the timestamp of the first scan.
- If a QR is lost or stolen, staff with `ticket_slots.invalidate` can void the old token. A new token is generated. The old token is stored in `previous_token` and can never be used again. The full action is audited.
- Offline scanning (for events with unreliable connectivity) is noted as a future enhancement using HMAC-signed tokens verifiable without a DB call.

---

## Attendee Visibility

- While `event.status = 'upcoming'`, the attendee list is not visible to anyone except authorized staff.
- When `event.status = 'active'` (event has started), the attendee list becomes visible to all claimed attendees of that event. Only `display_name` and `avatar_url` are shown — no emails or personal details.
- Visibility persists while `status = 'ended'`.
- Staff with `attendees.read` can see the full list at any time.

---

## Studio (Creative)

The studio app is purely a creative tool. It does not handle operational event management.

- **Event page builder**: visual editor for the public event page — layout, images, descriptions, branding.
- **QR canvas designer**: event-themed QR codes. A canvas editor where designers place the QR matrix within custom artwork (event motifs, logos, colors). The active `qr_designs` record for an event is used when attendees view their QR on the events app.
- The event creator holds full permissions on their event and is responsible for delegating operational access to a manager before the event.

---

## Open Items (to design in follow-up sessions)

- **Interpreter field mapping UI**: how admins build and test a JSON/CSV/XLS/XLSX mapping schema in the admin app. Includes handling of image cells in spreadsheets (current plan: flag as `image_skip`, store a note for manual review — no OCR in v1).
- **On-site registration**: walk-in attendees who were not in any import. Creating a slot manually at the event.
- **Badge printing**: integration with physical badge printers, badge template design.
- **Sync protocol**: future real-time or webhook-based sync between CandyStore and Janus (replacing the current manual JSON export/import).
- **Checkin app offline mode**: HMAC-signed tokens for low-connectivity environments.
- **Entitlement capacity enforcement**: how the system handles an entitlement that's reached its capacity limit (e.g., shuttle is full).
- **Notifications**: email/push to buyers when a slot is claimed, to attendees before the event. Email provider not yet chosen — claim link delivery depends on this.
- **Event creation flow**: events are created in admin, then the creator gains full permissions on that event and can open it in studio for design. The exact handoff UX is to be defined.
- **Claim link URL structure**: exact format (e.g. `/claim/<order_token>/<slot_index>`) to be defined during implementation planning.
