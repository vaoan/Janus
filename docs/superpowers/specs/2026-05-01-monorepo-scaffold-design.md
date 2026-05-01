# Janus — Monorepo Scaffold Design

**Date:** 2026-05-01
**Status:** Approved — ready for implementation planning

---

## Overview

Scaffold the Janus monorepo using CandyStore (`Z:\Github\candystore`) as the copy-and-adapt template. The tech stack is identical; the difference is the set of apps, port assignments, and Supabase project. No new tooling decisions are made — everything follows CandyStore's established conventions.

---

## Approach

**Copy-and-adapt from CandyStore.** Root tooling config files are copied verbatim where they contain no CandyStore-specific values. Files that reference app names, ports, or credentials are adapted for Janus. App shells are copied from the closest CandyStore equivalent, stripped of all business logic, and left as minimal Next.js 16 App Router stubs.

---

## Repository & Workspace Structure

```
janus/
├── apps/
│   ├── auth/          ← port 5100  (copied from candystore/apps/auth)
│   ├── admin/         ← port 5101  (copied from candystore/apps/admin)
│   ├── studio/        ← port 5102  (copied from candystore/apps/studio)
│   ├── manager/       ← port 5103  (copied from candystore/apps/store)
│   ├── events/        ← port 5104  (copied from candystore/apps/store)
│   └── checkin/       ← port 5105  (copied from candystore/apps/admin)
├── packages/
│   ├── api/
│   ├── ui/
│   ├── shared/
│   ├── auth/
│   └── app-components/
├── docker/
│   ├── ci/
│   ├── prod/
│   └── infra/supabase/
├── scripts/
├── docs/superpowers/specs/
└── .github/workflows/
```

**Root package:** name `"janus"`, version `"2026.05.01.1"`, private.

**Workspace:** `pnpm-workspace.yaml` — `apps/*`, `packages/*` (identical to CandyStore).

**Package names:** no scope prefix. Packages are named `api`, `ui`, `shared`, `auth`, `app-components` — referenced as workspace dependencies without `@org/` prefix.

---

## Root Tooling Config

### Copied verbatim (no Janus-specific content)

| File | Notes |
|---|---|
| `turbo.json` | Pipeline is app-agnostic |
| `tsconfig.base.json` | Compiler options identical |
| `eslint.config.mjs` | Same plugin set |
| `.prettierrc` / `.prettierignore` | Identical |
| `stylelint.config.mjs` | Identical |
| `.husky/pre-commit` / `pre-push` | Same hook scripts |
| `cspell.json` | Same dictionary |
| `.jscpd.json` | Same duplicate detection config |
| `.ls-lint.yml` | Same naming conventions |
| `.syncpackrc.json` | Same version alignment rules |
| `.secretlintrc.json` | Same secret scanning rules |
| `pnpm-workspace.yaml` | `apps/*`, `packages/*` pattern identical |
| `.gitignore` / `.gitattributes` / `.npmrc` | Identical |

### Adapted (same structure, Janus-specific values)

**`package.json` (root):**
- `name`: `"janus"`
- `version`: `"2026.05.01.1"`
- Dev scripts updated for 6 Janus apps (no store, landing, payments, playground)
- `NEXT_PUBLIC_*` URL env vars updated to Janus ports

**`.env.dev`:**
- Supabase URL: `https://uhsnzkwxdepphcxzkqam.supabase.co`
- Anon key, service role key: Janus dev project values
- App URLs: ports 5100–5105
- CandyStore-only vars removed: `NEXT_PUBLIC_STORE_URL`, `NEXT_PUBLIC_LANDING_URL`, `NEXT_PUBLIC_PAYMENTS_URL`, `NEXT_PUBLIC_PLAYGROUND_URL`
- Janus app URLs added: `NEXT_PUBLIC_AUTH_URL`, `NEXT_PUBLIC_ADMIN_URL`, `NEXT_PUBLIC_STUDIO_URL`, `NEXT_PUBLIC_MANAGER_URL`, `NEXT_PUBLIC_EVENTS_URL`, `NEXT_PUBLIC_CHECKIN_URL`

**`.env.ci` / `.env.staging` / `.env.prod`:** same structure as `.env.dev`, placeholder values for secrets not yet configured (CI, staging, prod Supabase not set up yet).

**`docker/prod/Dockerfile` / `nginx.conf` / `supervisord.conf`:** updated for 6 Janus apps instead of 7 CandyStore apps.

**`docker/ci/Dockerfile` / `playwright.config.ts`:** updated for Janus app names and ports.

### Not copied

- `.secrets` — live secrets file, never in git
- `.ssh-credentials.json` — environment-specific
- Any CandyStore app business logic

---

## App Shells

Each app is a minimal Next.js 16 App Router stub — enough to install, build, and pass CI. No features, no routes beyond a placeholder home page.

**Per-app file set:**
```
apps/<name>/
├── package.json          ← name: "<name>-app", port in dev script
├── next.config.ts
├── tsconfig.json
├── postcss.config.mjs
├── src/
│   ├── app/
│   │   ├── layout.tsx    ← root layout, i18n wrapper
│   │   ├── page.tsx      ← placeholder "coming soon" page
│   │   └── globals.css
│   ├── features/         ← empty, structure only
│   ├── shared/           ← empty, structure only
│   └── test/
│       └── setup.ts
├── vitest.config.ts
└── playwright.config.ts  ← (auth, events apps only — public-facing)
```

**Copy source per app:**

| Janus app | Copy from | Strip |
|---|---|---|
| `auth` | `candystore/apps/auth` | CandyStore auth flows, keep structure |
| `admin` | `candystore/apps/admin` | All CandyStore admin features |
| `studio` | `candystore/apps/studio` | All CandyStore studio features |
| `manager` | `candystore/apps/store` | All store logic |
| `events` | `candystore/apps/store` | All store logic |
| `checkin` | `candystore/apps/admin` | All admin logic |

---

## Shared Packages

Packages are copied from CandyStore and stripped of CandyStore-specific generated output. Structure, exports, and config are preserved intact.

| Package | Copy from | Strip |
|---|---|---|
| `api` | `candystore/packages/api` | Orval-generated clients, Supabase generated types (regenerated once Janus schema exists) |
| `ui` | `candystore/packages/ui` | Nothing — purely primitive components |
| `shared` | `candystore/packages/shared` | CandyStore-specific constants, URLs |
| `auth` | `candystore/packages/auth` | Nothing — structure is app-agnostic |
| `app-components` | `candystore/packages/app-components` | CandyStore-specific composed components |

Package names use no scope prefix: `"name": "api"`, `"name": "ui"`, etc.

---

## Secrets Flow

Identical to CandyStore. The sync-secrets system allows any developer to pull all GitHub repo secrets to a local `.secrets` file in one command.

**Files:**

- **`.secrets.example`** — lists every secret the Janus repo uses, by name. Secrets already configured in GitHub are noted as ready; secrets not yet configured are marked `# TODO: add to GitHub repo secrets`. On day one, the 6 dev Supabase secrets are ready.
- **`.secrets`** — git-ignored. Populated by running `node scripts/sync-secrets.mjs`.
- **`scripts/sync-secrets.mjs`** — adapted from CandyStore. Triggers `sync-secrets.yml` workflow, downloads and decrypts the artifact, writes `.secrets` locally.
- **`.github/workflows/sync-secrets.yml`** — already in place, secret list updated to Janus's set.

**Day-one secrets (already in GitHub):**

| Secret | Status |
|---|---|
| `DEV_SUPABASE_URL` | ✓ configured |
| `DEV_SUPABASE_ANON_KEY` | ✓ configured |
| `DEV_SUPABASE_SERVICE_ROLE_KEY` | ✓ configured |
| `DEV_SUPABASE_ACCESS_TOKEN` | ✓ configured |
| `DEV_SUPABASE_DB_PASSWORD` | ✓ configured |
| `DEV_SUPABASE_AUTH_EXTERNAL_REDIRECT_URI` | ✓ configured |
| All others | TODO — placeholder in `.secrets.example` |

---

## Docker

Copied and adapted from CandyStore:

- **`docker/prod/`** — multi-stage Dockerfile, nginx reverse proxy, supervisord. Updated for 6 Janus apps.
- **`docker/ci/`** — CI Dockerfile for E2E test runs. Updated for Janus apps and ports.
- **`docker/infra/supabase/`** — local Supabase stack for staging/CI. Copied verbatim (config is project-agnostic).
- **`docker/compose.yml`** — updated for Janus services.

Docker is a stub on day one — the configs are in place but not tested until the staging environment is set up.

---

## Out of Scope

- No business logic in any app or package on day one
- No Supabase migrations (separate task once schema finalized)
- No Orval codegen (requires Supabase types to be generated first)
- No E2E test suites (separate task per app)
- No CI/staging/prod secrets (separate task, tracked in issue #1)
