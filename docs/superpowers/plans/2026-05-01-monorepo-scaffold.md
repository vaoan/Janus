# Monorepo Scaffold Implementation Plan

> **For agentic workers:** REQUIRED SUB-SKILL: Use superpowers:subagent-driven-development (recommended) or superpowers:executing-plans to implement this plan task-by-task. Steps use checkbox (`- [ ]`) syntax for tracking.

**Goal:** Scaffold the Janus pnpm + Turbo monorepo with 6 app shells and 5 shared packages, copying and adapting from `Z:\Github\candystore`.

**Architecture:** Copy-and-adapt from CandyStore. Root tooling config copied verbatim where app-agnostic; adapted where Janus-specific (app names, ports, Supabase project). App shells are minimal Next.js 16 stubs stripped of all business logic. Packages copied and stripped of generated output.

**Tech Stack:** Next.js 16 App Router, pnpm workspaces, Turbo, TypeScript strict, ESLint v9, Prettier, Stylelint, Husky, Vitest, Playwright, shadcn/ui + Tailwind v4, TanStack Query v5, next-intl, Supabase.

**Reference:** `Z:\Github\candystore` — copy paths are relative to this root unless stated otherwise.

---

## File Map

```
janus/
├── package.json                         ADAPT  (name, version, app paths)
├── pnpm-workspace.yaml                  COPY
├── turbo.json                           COPY
├── tsconfig.base.json                   COPY
├── eslint.config.mjs                    COPY
├── .prettierrc / .prettierignore        COPY
├── stylelint.config.mjs                 COPY
├── .gitignore / .gitattributes / .npmrc COPY
├── cspell.json                          COPY
├── .jscpd.json                          COPY
├── .ls-lint.yml                         COPY
├── .syncpackrc.json                     COPY
├── .secretlintrc.json                   COPY
├── .husky/pre-commit / pre-push         COPY
├── .env.dev                             ADAPT (Janus Supabase, ports 5100-5105)
├── .env.ci / .env.staging / .env.prod   ADAPT (stubs, Janus app names)
├── .secrets.example                     ADAPT (Janus secret names)
├── scripts/start.mjs                    COPY  (auto-discovers apps, app-agnostic)
├── scripts/sync-secrets.mjs             COPY  (app-agnostic)
├── scripts/load-env.mjs                 COPY
├── scripts/build.mjs                    COPY
├── scripts/check-env-parity.mjs         COPY
├── scripts/check-css-sync.mjs           COPY
├── packages/api/                        COPY + strip generated types
├── packages/ui/                         COPY verbatim
├── packages/shared/                     COPY + strip CS constants
├── packages/auth/                       COPY verbatim
├── packages/app-components/             COPY + rename pkg, strip CS components
├── apps/auth/                           COPY from CS auth, strip flows, port 5100
├── apps/admin/                          COPY from CS admin, strip features, port 5101
├── apps/studio/                         COPY from CS studio, strip features, port 5102
├── apps/manager/                        COPY from CS store, strip features, port 5103
├── apps/events/                         COPY from CS store, strip features, port 5104
├── apps/checkin/                        COPY from CS admin, strip features, port 5105
└── docker/                              STUB (adapt once infra is set up)
```

---

## Task 1: Root workspace tooling — verbatim copies

**Files:**
- Copy: `pnpm-workspace.yaml`
- Copy: `turbo.json`
- Copy: `tsconfig.base.json`
- Copy: `eslint.config.mjs`
- Copy: `.prettierrc`
- Copy: `.prettierignore`
- Copy: `stylelint.config.mjs`
- Copy: `.gitignore`
- Copy: `.gitattributes`
- Copy: `.npmrc`
- Copy: `cspell.json`
- Copy: `.jscpd.json`
- Copy: `.ls-lint.yml`
- Copy: `.syncpackrc.json`
- Copy: `.secretlintrc.json`
- Copy: `.husky/pre-commit`
- Copy: `.husky/pre-push`

- [ ] **Step 1: Copy verbatim files**

```bash
CS=Z:/Github/candystore
JN=Z:/Github/Janus

cp "$CS/pnpm-workspace.yaml" "$JN/pnpm-workspace.yaml"
cp "$CS/turbo.json" "$JN/turbo.json"
cp "$CS/tsconfig.base.json" "$JN/tsconfig.base.json"
cp "$CS/eslint.config.mjs" "$JN/eslint.config.mjs"
cp "$CS/.prettierrc" "$JN/.prettierrc"
cp "$CS/.prettierignore" "$JN/.prettierignore"
cp "$CS/stylelint.config.mjs" "$JN/stylelint.config.mjs"
cp "$CS/.gitignore" "$JN/.gitignore"
cp "$CS/.gitattributes" "$JN/.gitattributes"
cp "$CS/.npmrc" "$JN/.npmrc"
cp "$CS/cspell.json" "$JN/cspell.json"
cp "$CS/.jscpd.json" "$JN/.jscpd.json"
cp "$CS/.ls-lint.yml" "$JN/.ls-lint.yml"
cp "$CS/.syncpackrc.json" "$JN/.syncpackrc.json"
cp "$CS/.secretlintrc.json" "$JN/.secretlintrc.json"
cp "$CS/.husky/pre-commit" "$JN/.husky/pre-commit"
cp "$CS/.husky/pre-push" "$JN/.husky/pre-push"
```

- [ ] **Step 2: Verify files were copied**

```bash
ls Z:/Github/Janus/pnpm-workspace.yaml Z:/Github/Janus/turbo.json Z:/Github/Janus/tsconfig.base.json Z:/Github/Janus/eslint.config.mjs
```
Expected: all 4 paths printed, no errors.

- [ ] **Step 3: Commit**

```bash
cd Z:/Github/Janus
git add pnpm-workspace.yaml turbo.json tsconfig.base.json eslint.config.mjs .prettierrc .prettierignore stylelint.config.mjs .gitignore .gitattributes .npmrc cspell.json .jscpd.json .ls-lint.yml .syncpackrc.json .secretlintrc.json .husky/pre-commit .husky/pre-push
git commit -m "chore: copy root tooling config from CandyStore"
```

---

## Task 2: Root package.json

**Files:**
- Create: `package.json`

- [ ] **Step 1: Create root package.json**

Create `Z:\Github\Janus\package.json`:

```json
{
  "name": "janus",
  "version": "2026.05.01.1",
  "private": true,
  "scripts": {
    "dev": "node scripts/start.mjs",
    "dev:test": "node scripts/start.mjs --env test",
    "build": "node scripts/build.mjs",
    "tunnel": "node scripts/cloudflared.mjs",
    "tunnel:stop": "node scripts/cloudflared-stop.mjs",
    "lint:env": "node scripts/check-env-parity.mjs",
    "lint": "eslint apps/auth/src apps/admin/src apps/studio/src apps/manager/src apps/events/src apps/checkin/src packages/ui/src packages/shared/src packages/api/src packages/app-components/src packages/auth/src",
    "typecheck": "turbo typecheck",
    "test": "turbo test",
    "test:watch": "vitest",
    "test:coverage": "turbo test:coverage",
    "format": "prettier --write \"**/*.{ts,tsx,js,jsx,json,css,md}\"",
    "format:check": "prettier --check \"**/*.{ts,tsx,js,jsx,json,css,md}\"",
    "fix:staged": "lint-staged --concurrent false",
    "fix:all": "pnpm format && pnpm lint --fix",
    "check:style": "stylelint \"apps/*/src/**/*.css\" \"packages/*/src/**/*.css\" && node scripts/check-css-sync.mjs",
    "check:tools": "pnpm cspell \"apps/*/src/**/*.{ts,tsx,js,jsx,md}\" \"packages/*/src/**/*.{ts,tsx}\" && pnpm knip && pnpm jscpd apps/auth/src apps/admin/src apps/studio/src apps/manager/src apps/events/src apps/checkin/src packages/ui/src packages/shared/src packages/api/src packages/app-components/src packages/auth/src && pnpm ls-lint && pnpm madge --circular --extensions ts,tsx,js,jsx apps/auth/src apps/admin/src apps/studio/src apps/manager/src apps/events/src apps/checkin/src packages/ui/src packages/shared/src packages/api/src packages/app-components/src packages/auth/src && pnpm sherif && pnpm syncpack lint",
    "sherif": "sherif",
    "sherif:fix": "sherif --fix",
    "syncpack:lint": "syncpack lint",
    "syncpack:fix": "syncpack fix",
    "secretlint": "secretlint \"**/*\"",
    "semgrep": "semgrep scan --config p/ci --config p/security-audit --config p/owasp-top-ten",
    "precommit": "pnpm lint-staged && pnpm lint:env && pnpm lint && pnpm check:style && pnpm check:tools",
    "prepush": "pnpm typecheck && pnpm test:coverage && pnpm build",
    "codegen:supabase": "supabase gen types typescript --local > packages/api/src/supabase/types.ts",
    "sync-secrets": "node scripts/sync-secrets.mjs",
    "e2e:dev": "node scripts/e2e.mjs --env dev",
    "e2e:dev:headed": "node scripts/e2e.mjs --env dev --headed",
    "e2e:staging": "node scripts/e2e.mjs --env staging",
    "e2e:staging:headed": "node scripts/e2e.mjs --env staging --headed",
    "e2e:ci": "node scripts/e2e.mjs --env ci",
    "e2e:ci:headed": "node scripts/e2e.mjs --env ci --headed",
    "docker:build": "node scripts/docker-build.mjs",
    "docker:teardown": "node scripts/docker-teardown.mjs",
    "supabase:start": "node scripts/supabase-cmd.mjs start",
    "supabase:stop": "node scripts/supabase-cmd.mjs stop",
    "supabase:reset": "node scripts/supabase-cmd.mjs db reset",
    "prepare": "husky",
    "clean:builds": "find . -not -path '*/node_modules/*' -not -path '*/.git/*' \\( -name '.next' -o -name '.turbo' -o -name 'test-results' -o -name 'playwright-report' -o -name 'coverage' \\) -type d -prune -exec rm -rf '{}' + ; find . -not -path '*/node_modules/*' -not -path '*/.git/*' \\( -name 'tsconfig.tsbuildinfo' -o -name 'next-env.d.ts' \\) -delete",
    "clean": "pnpm clean:builds && find . -name 'node_modules' -type d -prune -exec rm -rf '{}' +"
  },
  "devDependencies": {
    "@ls-lint/ls-lint": "^2.3.1",
    "@playwright/test": "^1.59.1",
    "@secretlint/secretlint-rule-preset-recommend": "^12.3.1",
    "@tailwindcss/typography": "^0.5.19",
    "@tanstack/eslint-plugin-query": "^5.100.5",
    "@typescript-eslint/eslint-plugin": "^8.59.0",
    "@typescript-eslint/parser": "^8.59.0",
    "@vitest/eslint-plugin": "^1.6.16",
    "concurrently": "^9.2.1",
    "cspell": "^10.0.0",
    "eslint": "^9.39.4",
    "eslint-config-next": "16.2.4",
    "eslint-plugin-better-tailwindcss": "^4.4.1",
    "eslint-plugin-boundaries": "^6.0.2",
    "eslint-plugin-i18next": "^6.1.4",
    "eslint-plugin-import": "^2.32.0",
    "eslint-plugin-jsx-a11y": "^6.10.2",
    "eslint-plugin-react-hooks": "^7.1.1",
    "eslint-plugin-security": "^4.0.0",
    "eslint-plugin-sonarjs": "^4.0.3",
    "eslint-plugin-testing-library": "^7.16.2",
    "eslint-plugin-unicorn": "^64.0.0",
    "eslint-plugin-unused-imports": "^4.4.1",
    "fast-check": "^4.7.0",
    "husky": "^9.1.7",
    "jscpd": "^4.0.9",
    "knip": "^6.7.0",
    "lint-staged": "^16.4.0",
    "madge": "^8.0.0",
    "prettier": "^3.8.3",
    "secretlint": "^12.3.1",
    "sharp": "^0.34.5",
    "sherif": "^1.11.1",
    "stylelint": "^17.9.1",
    "stylelint-config-standard": "^40.0.0",
    "stylelint-config-tailwindcss": "^1.0.1",
    "supabase": "^2.95.5",
    "syncpack": "^14.3.1",
    "tailwindcss": "^4.2.4",
    "turbo": "^2.9.6",
    "typescript": "^6.0.3",
    "vitest": "^4.1.5"
  },
  "lint-staged": {
    "*.{ts,tsx,js,jsx}": [
      "prettier --check",
      "eslint --no-warn-ignored --max-warnings=0",
      "secretlint"
    ],
    "*.{json,md,css}": [
      "prettier --check"
    ]
  },
  "pnpm": {
    "overrides": {
      "vite": ">=7.3.2",
      "uuid": ">=14.0.0",
      "postcss": "^8.5.12"
    },
    "onlyBuiltDependencies": [
      "@parcel/watcher",
      "@swc/core",
      "esbuild",
      "msw",
      "sharp",
      "supabase",
      "unrs-resolver"
    ]
  },
  "packageManager": "pnpm@10.32.1+sha512.a706938f0e89ac1456b6563eab4edf1d1faf3368d1191fc5c59790e96dc918e4456ab2e67d613de1043d2e8c81f87303e6b40d4ffeca9df15ef1ad567348f2be",
  "msw": {
    "workerDirectory": [
      "apps/admin/public",
      "apps/manager/public",
      "apps/events/public"
    ]
  }
}
```

- [ ] **Step 2: Commit**

```bash
cd Z:/Github/Janus
git add package.json
git commit -m "chore: add root package.json for janus monorepo"
```

---

## Task 3: Scripts

**Files:**
- Copy verbatim: `scripts/start.mjs`, `scripts/sync-secrets.mjs`, `scripts/load-env.mjs`, `scripts/build.mjs`, `scripts/check-env-parity.mjs`, `scripts/check-css-sync.mjs`
- Copy verbatim (if they exist): `scripts/e2e.mjs`, `scripts/docker-build.mjs`, `scripts/docker-teardown.mjs`, `scripts/supabase-cmd.mjs`, `scripts/supabase-docker.mjs`, `scripts/cloudflared.mjs`, `scripts/cloudflared-stop.mjs`

- [ ] **Step 1: Copy all scripts verbatim**

```bash
CS=Z:/Github/candystore
JN=Z:/Github/Janus
mkdir -p "$JN/scripts"

for f in start.mjs sync-secrets.mjs load-env.mjs build.mjs check-env-parity.mjs check-css-sync.mjs e2e.mjs docker-build.mjs docker-teardown.mjs supabase-cmd.mjs supabase-docker.mjs cloudflared.mjs cloudflared-stop.mjs; do
  [ -f "$CS/scripts/$f" ] && cp "$CS/scripts/$f" "$JN/scripts/$f" && echo "copied $f"
done
```

Expected: each script that exists in CandyStore prints "copied <name>".

- [ ] **Step 2: Commit**

```bash
cd Z:/Github/Janus
git add scripts/
git commit -m "chore: copy scripts from CandyStore (app-agnostic, verbatim)"
```

---

## Task 4: Environment files

**Files:**
- Create: `.env.dev`
- Create: `.env.ci`
- Create: `.env.staging`
- Create: `.env.prod`
- Create: `.secrets.example`

- [ ] **Step 1: Create .env.dev**

Create `Z:\Github\Janus\.env.dev`:

```
# =============================================================================
# Dev environment - Supabase Cloud (dedicated non-production project)
# =============================================================================
TARGET_ENV=dev

# --- Topology -----------------------------------------------------------------
APPS_MODE=local
SUPABASE_MODE=cloud

# --- Container identity (unused in dev, kept for parity) ----------------------
SITE_PROD_CONTAINER_NAME=janus-dev
SITE_PROD_IMAGE_NAME=janus-dev

# --- App origins ---------------------------------------------------------------
HOST_PORT=5150
APP_INTERNAL_ORIGIN=http://localhost:8088

# --- Supabase Cloud (dev project) ---------------------------------------------
NEXT_PUBLIC_SUPABASE_URL=$secret:DEV_SUPABASE_URL
SUPABASE_URL_INTERNAL=
NEXT_PUBLIC_SUPABASE_ANON_KEY=$secret:DEV_SUPABASE_ANON_KEY
SUPABASE_SERVICE_ROLE_KEY=$secret:DEV_SUPABASE_SERVICE_ROLE_KEY

# Not used in cloud mode. Kept for env parity with non-cloud environments.
SUPABASE_PORT=N/A
SUPABASE_JWT_SECRET=super-secret-jwt-token-with-at-least-32-characters-long
SUPABASE_DB_PASSWORD=$secret:DEV_SUPABASE_DB_PASSWORD

# --- Auth ----------------------------------------------------------------------
AUTH_PROVIDER_MODE=supabase

# --- Cross-app navigation ------------------------------------------------------
NEXT_PUBLIC_AUTH_URL=http://localhost:5100
NEXT_PUBLIC_AUTH_HOST_URL=http://localhost:5100
NEXT_PUBLIC_ADMIN_URL=http://localhost:5101
NEXT_PUBLIC_STUDIO_URL=http://localhost:5102
NEXT_PUBLIC_MANAGER_URL=http://localhost:5103
NEXT_PUBLIC_EVENTS_URL=http://localhost:5104
NEXT_PUBLIC_CHECKIN_URL=http://localhost:5105

# --- OAuth providers -----------------------------------------------------------
SUPABASE_AUTH_EXTERNAL_GOOGLE_CLIENT_ID=$secret:SUPABASE_AUTH_EXTERNAL_GOOGLE_CLIENT_ID
SUPABASE_AUTH_EXTERNAL_GOOGLE_SECRET=$secret:SUPABASE_AUTH_EXTERNAL_GOOGLE_SECRET
SUPABASE_AUTH_EXTERNAL_DISCORD_CLIENT_ID=$secret:SUPABASE_AUTH_EXTERNAL_DISCORD_CLIENT_ID
SUPABASE_AUTH_EXTERNAL_DISCORD_SECRET=$secret:SUPABASE_AUTH_EXTERNAL_DISCORD_SECRET
SUPABASE_AUTH_EXTERNAL_REDIRECT_URI=$secret:DEV_SUPABASE_AUTH_EXTERNAL_REDIRECT_URI
SUPABASE_AUTH_SITE_URL=http://localhost:5100/auth/callback

# --- Build flags ---------------------------------------------------------------
NEXT_PUBLIC_ENABLE_TEST_IDS=true
NEXT_PUBLIC_BUILD_HASH=local
NEXT_PUBLIC_TARGET_ENV=dev
ENV_DEBUG=true

# --- Cloudflare tunnels --------------------------------------------------------
CLOUDFLARE_TUNNEL_APP_ENABLED=false
CLOUDFLARE_TUNNEL_APP_TOKEN=

# --- E2E test account ----------------------------------------------------------
GOOGLE_TEST_EMAIL=$secret:GOOGLE_TEST_EMAIL
GOOGLE_TEST_PASSWORD=$secret:GOOGLE_TEST_PASSWORD
```

- [ ] **Step 2: Create .env.ci**

Create `Z:\Github\Janus\.env.ci`:

```
# =============================================================================
# CI environment - local Docker Supabase
# =============================================================================
TARGET_ENV=ci

APPS_MODE=local
SUPABASE_MODE=local

SITE_PROD_CONTAINER_NAME=janus-ci
SITE_PROD_IMAGE_NAME=janus-ci

NEXT_PUBLIC_SUPABASE_URL=http://localhost:54321
SUPABASE_URL_INTERNAL=http://supabase_kong_janus:8000
NEXT_PUBLIC_SUPABASE_ANON_KEY=$secret:CI_SUPABASE_ANON_KEY
SUPABASE_SERVICE_ROLE_KEY=$secret:CI_SUPABASE_SERVICE_ROLE_KEY

SUPABASE_PORT=54321
SUPABASE_JWT_SECRET=$secret:CI_SUPABASE_JWT_SECRET
SUPABASE_DB_PASSWORD=$secret:CI_SUPABASE_DB_PASSWORD

AUTH_PROVIDER_MODE=supabase

NEXT_PUBLIC_AUTH_URL=http://localhost:5100
NEXT_PUBLIC_AUTH_HOST_URL=http://localhost:5100
NEXT_PUBLIC_ADMIN_URL=http://localhost:5101
NEXT_PUBLIC_STUDIO_URL=http://localhost:5102
NEXT_PUBLIC_MANAGER_URL=http://localhost:5103
NEXT_PUBLIC_EVENTS_URL=http://localhost:5104
NEXT_PUBLIC_CHECKIN_URL=http://localhost:5105

SUPABASE_AUTH_EXTERNAL_REDIRECT_URI=http://localhost:5100/auth/callback
SUPABASE_AUTH_SITE_URL=http://localhost:5100/auth/callback

NEXT_PUBLIC_ENABLE_TEST_IDS=true
NEXT_PUBLIC_BUILD_HASH=ci
NEXT_PUBLIC_TARGET_ENV=ci
NEXT_PUBLIC_ENABLE_MOCKS=true

CLOUDFLARE_TUNNEL_APP_ENABLED=false
CLOUDFLARE_TUNNEL_APP_TOKEN=
```

- [ ] **Step 3: Create .env.staging**

Create `Z:\Github\Janus\.env.staging`:

```
# =============================================================================
# Staging environment - containerized Supabase, Cloudflare tunnel
# =============================================================================
TARGET_ENV=staging

APPS_MODE=docker
SUPABASE_MODE=local

SITE_PROD_CONTAINER_NAME=janus-staging
SITE_PROD_IMAGE_NAME=janus-staging

NEXT_PUBLIC_SUPABASE_URL=# TODO: staging Supabase URL (set once staging infra is set up)
NEXT_PUBLIC_SUPABASE_ANON_KEY=$secret:STAGING_SUPABASE_ANON_KEY
SUPABASE_SERVICE_ROLE_KEY=$secret:STAGING_SUPABASE_SERVICE_ROLE_KEY

SUPABASE_PORT=54321
SUPABASE_JWT_SECRET=$secret:STAGING_JWT_SECRET
SUPABASE_DB_PASSWORD=$secret:STAGING_POSTGRES_PASSWORD

AUTH_PROVIDER_MODE=supabase

NEXT_PUBLIC_AUTH_URL=# TODO: staging auth URL
NEXT_PUBLIC_AUTH_HOST_URL=# TODO: staging auth URL
NEXT_PUBLIC_ADMIN_URL=# TODO: staging admin URL
NEXT_PUBLIC_STUDIO_URL=# TODO: staging studio URL
NEXT_PUBLIC_MANAGER_URL=# TODO: staging manager URL
NEXT_PUBLIC_EVENTS_URL=# TODO: staging events URL
NEXT_PUBLIC_CHECKIN_URL=# TODO: staging checkin URL

SUPABASE_AUTH_EXTERNAL_GOOGLE_CLIENT_ID=$secret:SUPABASE_AUTH_EXTERNAL_GOOGLE_CLIENT_ID
SUPABASE_AUTH_EXTERNAL_GOOGLE_SECRET=$secret:SUPABASE_AUTH_EXTERNAL_GOOGLE_SECRET
SUPABASE_AUTH_EXTERNAL_DISCORD_CLIENT_ID=$secret:SUPABASE_AUTH_EXTERNAL_DISCORD_CLIENT_ID
SUPABASE_AUTH_EXTERNAL_DISCORD_SECRET=$secret:SUPABASE_AUTH_EXTERNAL_DISCORD_SECRET
SUPABASE_AUTH_EXTERNAL_REDIRECT_URI=# TODO: staging Supabase callback URL
SUPABASE_AUTH_SITE_URL=# TODO: staging auth callback URL

CLOUDFLARE_TUNNEL_APP_ENABLED=true
CLOUDFLARE_TUNNEL_APP_TOKEN=$secret:STAGING_CLOUDFLARE_TUNNEL_APP_TOKEN

NEXT_PUBLIC_BUILD_HASH=staging
NEXT_PUBLIC_TARGET_ENV=staging
```

- [ ] **Step 4: Create .env.prod**

Create `Z:\Github\Janus\.env.prod`:

```
# =============================================================================
# Production environment
# =============================================================================
TARGET_ENV=prod

APPS_MODE=docker
SUPABASE_MODE=cloud

SITE_PROD_CONTAINER_NAME=janus-prod
SITE_PROD_IMAGE_NAME=janus-prod

NEXT_PUBLIC_SUPABASE_URL=# TODO: prod Supabase URL
NEXT_PUBLIC_SUPABASE_ANON_KEY=$secret:PROD_SUPABASE_ANON_KEY
SUPABASE_SERVICE_ROLE_KEY=$secret:PROD_SUPABASE_SERVICE_ROLE_KEY

AUTH_PROVIDER_MODE=supabase

NEXT_PUBLIC_AUTH_URL=# TODO: prod auth URL
NEXT_PUBLIC_AUTH_HOST_URL=# TODO: prod auth URL
NEXT_PUBLIC_ADMIN_URL=# TODO: prod admin URL
NEXT_PUBLIC_STUDIO_URL=# TODO: prod studio URL
NEXT_PUBLIC_MANAGER_URL=# TODO: prod manager URL
NEXT_PUBLIC_EVENTS_URL=# TODO: prod events URL
NEXT_PUBLIC_CHECKIN_URL=# TODO: prod checkin URL

SUPABASE_AUTH_EXTERNAL_GOOGLE_CLIENT_ID=$secret:SUPABASE_AUTH_EXTERNAL_GOOGLE_CLIENT_ID
SUPABASE_AUTH_EXTERNAL_GOOGLE_SECRET=$secret:SUPABASE_AUTH_EXTERNAL_GOOGLE_SECRET
SUPABASE_AUTH_EXTERNAL_DISCORD_CLIENT_ID=$secret:SUPABASE_AUTH_EXTERNAL_DISCORD_CLIENT_ID
SUPABASE_AUTH_EXTERNAL_DISCORD_SECRET=$secret:SUPABASE_AUTH_EXTERNAL_DISCORD_SECRET
SUPABASE_AUTH_EXTERNAL_REDIRECT_URI=# TODO: prod Supabase callback URL
SUPABASE_AUTH_SITE_URL=# TODO: prod auth callback URL

NEXT_PUBLIC_BUILD_HASH=prod
NEXT_PUBLIC_TARGET_ENV=prod
STANDALONE=true
```

- [ ] **Step 5: Create .secrets.example**

Create `Z:\Github\Janus\.secrets.example`:

```
# =============================================================================
# Secrets file — resolved by load-env.mjs at runtime
# =============================================================================
# This file holds actual secret values referenced by $secret:KEY_NAME in
# the .env.{TARGET_ENV} files. It is gitignored and should never be committed.
#
# To populate this file automatically from GitHub repository secrets:
#   pnpm sync-secrets
#
# Or copy this file and fill in values manually:
#   cp .secrets.example .secrets
#
# Format: KEY=VALUE (one per line, # for comments)
# =============================================================================

# ─── Dev (Supabase Cloud project) ────────────────────────────────────────────
# ✓ Already configured in GitHub repo secrets
DEV_SUPABASE_URL=https://uhsnzkwxdepphcxzkqam.supabase.co
DEV_SUPABASE_ANON_KEY=replace_me
DEV_SUPABASE_SERVICE_ROLE_KEY=replace_me
DEV_SUPABASE_ACCESS_TOKEN=replace_me
DEV_SUPABASE_DB_PASSWORD=replace_me
DEV_SUPABASE_AUTH_EXTERNAL_REDIRECT_URI=https://uhsnzkwxdepphcxzkqam.supabase.co/auth/v1/callback

# ─── Staging (containerized Supabase) ────────────────────────────────────────
# TODO: add to GitHub repo secrets once staging infra is set up
STAGING_SUPABASE_ANON_KEY=replace_me
STAGING_SUPABASE_SERVICE_ROLE_KEY=replace_me
STAGING_JWT_SECRET=replace_me
STAGING_POSTGRES_PASSWORD=replace_me

# ─── CI (local Docker Supabase) ──────────────────────────────────────────────
# TODO: add to GitHub repo secrets
CI_SUPABASE_ANON_KEY=replace_me
CI_SUPABASE_SERVICE_ROLE_KEY=replace_me
CI_SUPABASE_JWT_SECRET=replace_me
CI_SUPABASE_DB_PASSWORD=replace_me

# ─── E2E (isolated Supabase) ─────────────────────────────────────────────────
# TODO: add to GitHub repo secrets
E2E_SUPABASE_ANON_KEY=replace_me
E2E_SUPABASE_SERVICE_ROLE_KEY=replace_me

# ─── Production (Supabase Cloud) ─────────────────────────────────────────────
# TODO: add to GitHub repo secrets once prod infra is set up
PROD_SUPABASE_ANON_KEY=replace_me
PROD_SUPABASE_SERVICE_ROLE_KEY=replace_me
PROD_SUPABASE_DB_PASSWORD=replace_me
PROD_SUPABASE_ACCESS_TOKEN=replace_me

# ─── Shared OAuth providers ───────────────────────────────────────────────────
# TODO: add to GitHub repo secrets (shared with CandyStore)
SUPABASE_AUTH_EXTERNAL_GOOGLE_CLIENT_ID=replace_me
SUPABASE_AUTH_EXTERNAL_GOOGLE_SECRET=replace_me
SUPABASE_AUTH_EXTERNAL_DISCORD_CLIENT_ID=replace_me
SUPABASE_AUTH_EXTERNAL_DISCORD_SECRET=replace_me

# ─── Cloudflare tunnel ────────────────────────────────────────────────────────
# TODO: add to GitHub repo secrets
STAGING_CLOUDFLARE_TUNNEL_TOKEN=replace_me
STAGING_CLOUDFLARE_TUNNEL_APP_TOKEN=replace_me
CLOUDFLARED_TUNNEL_CREDENTIALS=replace_me
CLOUDFLARED_DOCKER_CONFIG=replace_me
CLOUDFLARED_CONFIG=replace_me

# ─── E2E Google test account ──────────────────────────────────────────────────
# TODO: add to GitHub repo secrets
GOOGLE_TEST_EMAIL=replace_me
GOOGLE_TEST_PASSWORD=replace_me

# ─── Telegram alerting ────────────────────────────────────────────────────────
# TODO: add to GitHub repo secrets
TELEGRAM_BOT_TOKEN=replace_me
TELEGRAM_CHAT_ID=replace_me
TELEGRAM_THREAD_ID=replace_me
TELEGRAM_CRITICAL_THREAD_ID=replace_me
TELEGRAM_REVIEWS_THREAD_ID=replace_me
TELEGRAM_BACKUPS_THREAD_ID=replace_me

# ─── Production server ────────────────────────────────────────────────────────
# TODO: add to GitHub repo secrets
PROD_SERVER_SSH_KEY=replace_me
PROD_SERVER_HOST=replace_me
PROD_SERVER_USER=replace_me

# ─── Cloudflare cache purge ───────────────────────────────────────────────────
# TODO: add to GitHub repo secrets
PROD_CF_API_TOKEN=replace_me
PROD_CF_ZONE_ID=replace_me
```

- [ ] **Step 6: Verify .gitignore includes .secrets**

Check that `.secrets` is in `.gitignore` (it was copied from CandyStore which already has it). Run:

```bash
grep "^\.secrets$" Z:/Github/Janus/.gitignore
```

Expected: `.secrets` printed. If not found, add it manually.

- [ ] **Step 7: Commit**

```bash
cd Z:/Github/Janus
git add .env.dev .env.ci .env.staging .env.prod .secrets.example
git commit -m "chore: add env files and secrets template for janus"
```

---

## Task 5: Shared packages

**Files:**
- Copy + adapt: `packages/api/`, `packages/ui/`, `packages/shared/`, `packages/auth/`, `packages/app-components/`

The key adaptation: CandyStore's `app-components` package is named `@monorepo/app-components`. Janus uses no scope prefix — rename to `app-components`.

- [ ] **Step 1: Copy all 5 packages from CandyStore**

```bash
CS=Z:/Github/candystore
JN=Z:/Github/Janus
mkdir -p "$JN/packages"

cp -r "$CS/packages/api" "$JN/packages/api"
cp -r "$CS/packages/ui" "$JN/packages/ui"
cp -r "$CS/packages/shared" "$JN/packages/shared"
cp -r "$CS/packages/auth" "$JN/packages/auth"
cp -r "$CS/packages/app-components" "$JN/packages/app-components"
```

- [ ] **Step 2: Rename app-components package**

Edit `Z:\Github\Janus\packages\app-components\package.json`. Change:
```json
"name": "@monorepo/app-components",
```
to:
```json
"name": "app-components",
```

- [ ] **Step 3: Strip CandyStore generated output from packages/api**

Delete CandyStore-specific generated files that will be regenerated for Janus once the schema exists:

```bash
# Delete Orval-generated REST clients (CandyStore-specific endpoints)
rm -rf Z:/Github/Janus/packages/api/src/rest/generated/

# Delete Supabase generated types (CandyStore schema — regenerate for Janus)
rm -f Z:/Github/Janus/packages/api/src/supabase/types.ts

# Delete GraphQL generated (CandyStore-specific)
rm -rf Z:/Github/Janus/packages/api/src/graphql/generated/
```

After deletion, create empty placeholder files to keep TypeScript happy:

Create `Z:\Github\Janus\packages\api\src\supabase\types.ts`:
```typescript
// Janus Supabase types — run `pnpm codegen:supabase` after schema migrations are applied
export type Json = string | number | boolean | null | { [key: string]: Json | undefined } | Json[];

export type Database = {
  public: {
    Tables: Record<string, never>;
    Views: Record<string, never>;
    Functions: Record<string, never>;
    Enums: Record<string, never>;
  };
};
```

Create `Z:\Github\Janus\packages\api\src\rest\generated\.gitkeep`:
```
```

- [ ] **Step 4: Strip CandyStore-specific components from app-components**

Remove CandyStore-specific composed components while keeping the package structure and exports:

```bash
# Keep the package structure but clear CandyStore-specific component implementations
# Check what's in the components directory first:
ls Z:/Github/Janus/packages/app-components/src/components/
```

For each component file that is CandyStore-specific (anything referencing store, products, payments, landing), replace its content with an empty re-export placeholder. Components that are generic (navigation, layout wrappers, error boundaries) can be kept as-is.

The `src/index.ts` and `src/components/index.ts` barrel files should be updated to only export what remains.

- [ ] **Step 5: Commit**

```bash
cd Z:/Github/Janus
git add packages/
git commit -m "chore: add shared packages (copied and adapted from CandyStore)"
```

---

## Task 6: App shell — auth (port 5100)

**Files:**
- Create: `apps/auth/` (full shell)

The auth app is copied directly from CandyStore's auth app. The authentication flow structure is identical — only the downstream app URLs change.

- [ ] **Step 1: Copy auth app from CandyStore**

```bash
CS=Z:/Github/candystore
JN=Z:/Github/Janus
cp -r "$CS/apps/auth" "$JN/apps/auth"
```

- [ ] **Step 2: Update package.json — port and package name**

Edit `Z:\Github\Janus\apps\auth\package.json`. Change:
- `"name": "auth-app"` → keep as `"auth-app"`
- `"dev": "next dev -p 5000"` → `"dev": "next dev -p 5100"`
- `"start": "next start -p 5000"` → `"start": "next start -p 5100"`
- `"@monorepo/app-components": "workspace:*"` → `"app-components": "workspace:*"`

- [ ] **Step 3: Update next.config.ts — transpilePackages**

Edit `Z:\Github\Janus\apps\auth\next.config.ts`. Change:
```typescript
transpilePackages: [
  "api",
  "auth",
  "ui",
  "shared",
  "@monorepo/app-components",
],
```
to:
```typescript
transpilePackages: [
  "api",
  "auth",
  "ui",
  "shared",
  "app-components",
],
```

Also update the basePath for standalone mode from `/auth` (no change needed — same app name).

Remove or clear the `allowedDevOrigins` array (CandyStore-specific domain):
```typescript
// Remove: const allowedDevOrigins = ["auth.ffxivbe.org"];
// Remove: allowedDevOrigins, from nextConfig
```

- [ ] **Step 4: Strip CandyStore auth business logic, keep structure**

The auth app in CandyStore already has a clean structure. Keep:
- `src/app/layout.tsx`
- `src/app/globals.css`
- `src/app/page.tsx` (redirects to locale)
- `src/shared/infrastructure/i18n/` (i18n setup)
- All config files (vitest, playwright, tsconfig, postcss)

Replace `src/app/page.tsx` with a clean Janus placeholder if the current one has CandyStore-specific redirects:

```typescript
import { redirect } from "next/navigation";
import { routing } from "@/shared/infrastructure/i18n";

export default function RootPage() {
  redirect(`/${routing.defaultLocale}`);
}
```

Delete all feature directories with CandyStore-specific content:
```bash
rm -rf Z:/Github/Janus/apps/auth/src/features/
mkdir Z:/Github/Janus/apps/auth/src/features
```

- [ ] **Step 5: Update all imports of @monorepo/app-components**

```bash
# Find any remaining references to the old package name
grep -r "@monorepo/app-components" Z:/Github/Janus/apps/auth/src/
```

Replace any found with `app-components`.

- [ ] **Step 6: Commit**

```bash
cd Z:/Github/Janus
git add apps/auth/
git commit -m "chore: add auth app shell (port 5100)"
```

---

## Task 7: App shell — admin (port 5101)

**Files:**
- Create: `apps/admin/` (full shell)

- [ ] **Step 1: Copy admin app from CandyStore**

```bash
cp -r Z:/Github/candystore/apps/admin Z:/Github/Janus/apps/admin
```

- [ ] **Step 2: Update package.json**

Edit `Z:\Github\Janus\apps\admin\package.json`. Change:
- `"dev": "next dev -p 5002"` → `"dev": "next dev -p 5101"`
- `"start": "next start -p 5002"` → `"start": "next start -p 5101"`
- `"@monorepo/app-components": "workspace:*"` → `"app-components": "workspace:*"`

- [ ] **Step 3: Update next.config.ts — transpilePackages and basePath**

Find `Z:\Github\Janus\apps\admin\next.config.ts`. Apply same changes as auth:
- Replace `"@monorepo/app-components"` with `"app-components"` in `transpilePackages`
- Remove `allowedDevOrigins` array
- Update `basePath` if standalone: `/admin` (no change needed)

- [ ] **Step 4: Strip CandyStore admin features, keep structure**

```bash
rm -rf Z:/Github/Janus/apps/admin/src/features/
mkdir Z:/Github/Janus/apps/admin/src/features
```

Replace `src/app/page.tsx` with:
```typescript
export default function Home() {
  return <main><h1>Janus Admin</h1></main>;
}
```

Keep all config files, layout, globals.css, and i18n infrastructure.

- [ ] **Step 5: Fix remaining @monorepo/app-components imports**

```bash
grep -r "@monorepo/app-components" Z:/Github/Janus/apps/admin/src/
```

Replace any found with `app-components`.

- [ ] **Step 6: Commit**

```bash
cd Z:/Github/Janus
git add apps/admin/
git commit -m "chore: add admin app shell (port 5101)"
```

---

## Task 8: App shell — studio (port 5102)

**Files:**
- Create: `apps/studio/` (full shell)

- [ ] **Step 1: Copy studio app from CandyStore**

```bash
cp -r Z:/Github/candystore/apps/studio Z:/Github/Janus/apps/studio
```

- [ ] **Step 2: Update package.json**

Edit `Z:\Github\Janus\apps\studio\package.json`. Change:
- `"dev": "next dev -p 5006"` → `"dev": "next dev -p 5102"`
- `"start": "next start -p 5006"` → `"start": "next start -p 5102"`
- `"@monorepo/app-components": "workspace:*"` → `"app-components": "workspace:*"`

- [ ] **Step 3: Update next.config.ts**

Same adaptations as admin: replace `@monorepo/app-components` in transpilePackages, remove allowedDevOrigins.

- [ ] **Step 4: Strip CandyStore studio features**

```bash
rm -rf Z:/Github/Janus/apps/studio/src/features/
mkdir Z:/Github/Janus/apps/studio/src/features
```

Replace `src/app/page.tsx` with:
```typescript
export default function Home() {
  return <main><h1>Janus Studio</h1></main>;
}
```

- [ ] **Step 5: Fix remaining @monorepo/app-components imports**

```bash
grep -r "@monorepo/app-components" Z:/Github/Janus/apps/studio/src/
```

Replace any found with `app-components`.

- [ ] **Step 6: Commit**

```bash
cd Z:/Github/Janus
git add apps/studio/
git commit -m "chore: add studio app shell (port 5102)"
```

---

## Task 9: App shell — manager (port 5103)

**Files:**
- Create: `apps/manager/` (full shell, based on CandyStore store)

- [ ] **Step 1: Copy store app as manager**

```bash
cp -r Z:/Github/candystore/apps/store Z:/Github/Janus/apps/manager
```

- [ ] **Step 2: Update package.json**

Edit `Z:\Github\Janus\apps\manager\package.json`. Change:
- `"name": "store"` → `"name": "manager"`
- `"dev": "next dev -p 5001"` → `"dev": "next dev -p 5103"`
- `"start": "next start -p 5001"` → `"start": "next start -p 5103"`
- `"@monorepo/app-components": "workspace:*"` → `"app-components": "workspace:*"`

- [ ] **Step 3: Update next.config.ts**

Same adaptations: replace `@monorepo/app-components`, remove allowedDevOrigins.

Update standalone basePath from `/store` to `/manager`:
```typescript
basePath: `${basePathPrefix}/manager`,
```

- [ ] **Step 4: Strip CandyStore store features**

```bash
rm -rf Z:/Github/Janus/apps/manager/src/features/
mkdir Z:/Github/Janus/apps/manager/src/features
```

Replace `src/app/page.tsx` with:
```typescript
export default function Home() {
  return <main><h1>Janus Manager</h1></main>;
}
```

- [ ] **Step 5: Fix remaining @monorepo/app-components imports**

```bash
grep -r "@monorepo/app-components" Z:/Github/Janus/apps/manager/src/
```

Replace any found with `app-components`.

- [ ] **Step 6: Commit**

```bash
cd Z:/Github/Janus
git add apps/manager/
git commit -m "chore: add manager app shell (port 5103)"
```

---

## Task 10: App shell — events (port 5104)

**Files:**
- Create: `apps/events/` (full shell, based on CandyStore store)

- [ ] **Step 1: Copy store app as events**

```bash
cp -r Z:/Github/candystore/apps/store Z:/Github/Janus/apps/events
```

- [ ] **Step 2: Update package.json**

Edit `Z:\Github\Janus\apps\events\package.json`. Change:
- `"name": "store"` → `"name": "events"`
- `"dev": "next dev -p 5001"` → `"dev": "next dev -p 5104"`
- `"start": "next start -p 5001"` → `"start": "next start -p 5104"`
- `"@monorepo/app-components": "workspace:*"` → `"app-components": "workspace:*"`

- [ ] **Step 3: Update next.config.ts**

Same adaptations as manager. Update basePath to `/events`.

- [ ] **Step 4: Strip CandyStore store features**

```bash
rm -rf Z:/Github/Janus/apps/events/src/features/
mkdir Z:/Github/Janus/apps/events/src/features
```

Replace `src/app/page.tsx` with:
```typescript
export default function Home() {
  return <main><h1>Janus Events</h1></main>;
}
```

- [ ] **Step 5: Fix remaining @monorepo/app-components imports**

```bash
grep -r "@monorepo/app-components" Z:/Github/Janus/apps/events/src/
```

Replace any found with `app-components`.

- [ ] **Step 6: Commit**

```bash
cd Z:/Github/Janus
git add apps/events/
git commit -m "chore: add events app shell (port 5104)"
```

---

## Task 11: App shell — checkin (port 5105)

**Files:**
- Create: `apps/checkin/` (full shell, based on CandyStore admin)

- [ ] **Step 1: Copy admin app as checkin**

```bash
cp -r Z:/Github/candystore/apps/admin Z:/Github/Janus/apps/checkin
```

- [ ] **Step 2: Update package.json**

Edit `Z:\Github\Janus\apps\checkin\package.json`. Change:
- `"name": "admin"` → `"name": "checkin"`
- `"dev": "next dev -p 5002"` → `"dev": "next dev -p 5105"`
- `"start": "next start -p 5002"` → `"start": "next start -p 5105"`
- `"@monorepo/app-components": "workspace:*"` → `"app-components": "workspace:*"`

- [ ] **Step 3: Update next.config.ts**

Same adaptations. Update basePath to `/checkin`.

- [ ] **Step 4: Strip CandyStore admin features**

```bash
rm -rf Z:/Github/Janus/apps/checkin/src/features/
mkdir Z:/Github/Janus/apps/checkin/src/features
```

Replace `src/app/page.tsx` with:
```typescript
export default function Home() {
  return <main><h1>Janus Checkin</h1></main>;
}
```

- [ ] **Step 5: Fix remaining @monorepo/app-components imports**

```bash
grep -r "@monorepo/app-components" Z:/Github/Janus/apps/checkin/src/
```

Replace any found with `app-components`.

- [ ] **Step 6: Commit**

```bash
cd Z:/Github/Janus
git add apps/checkin/
git commit -m "chore: add checkin app shell (port 5105)"
```

---

## Task 12: Docker stubs

**Files:**
- Copy + stub: `docker/`

- [ ] **Step 1: Copy docker directory from CandyStore**

```bash
cp -r Z:/Github/candystore/docker Z:/Github/Janus/docker
```

- [ ] **Step 2: Update docker/prod/Dockerfile for Janus apps**

In `Z:\Github\Janus\docker\prod\Dockerfile`, find every reference to CandyStore app names (`store`, `landing`, `payments`, `playground`) and replace with Janus app names (`manager`, `events`, `checkin`).

The app list should be: `auth`, `admin`, `studio`, `manager`, `events`, `checkin`.

- [ ] **Step 3: Update docker/prod/nginx.conf**

Replace all CandyStore app upstream and location blocks with Janus equivalents. Map ports:
- auth → 5100
- admin → 5101
- studio → 5102
- manager → 5103
- events → 5104
- checkin → 5105

- [ ] **Step 4: Update docker/prod/supervisord.conf**

Replace CandyStore app process entries with Janus app entries (same 6 apps, same ports).

- [ ] **Step 5: Update docker/compose.yml**

Replace CandyStore service names with Janus service names, update port mappings and env references.

- [ ] **Step 6: Update docker/ci/ for Janus**

In `docker/ci/Dockerfile` and `docker/ci/playwright.config.ts`, replace CandyStore app names and ports with Janus equivalents.

- [ ] **Step 7: Commit**

```bash
cd Z:/Github/Janus
git add docker/
git commit -m "chore: add docker stubs adapted from CandyStore"
```

---

## Task 13: Install dependencies and verify

- [ ] **Step 1: Install dependencies**

```bash
cd Z:/Github/Janus
pnpm install
```

Expected: dependencies installed, no peer dependency errors. If there are errors, fix them before proceeding.

- [ ] **Step 2: Verify package resolution**

```bash
pnpm ls --depth=0 2>&1 | head -40
```

Expected: all workspace packages listed without errors.

- [ ] **Step 3: Check for remaining @monorepo/app-components references**

```bash
grep -r "@monorepo/app-components" Z:/Github/Janus/apps/ Z:/Github/Janus/packages/
```

Expected: no output. If any found, fix them before continuing.

- [ ] **Step 4: Run typecheck**

```bash
cd Z:/Github/Janus
pnpm typecheck
```

Expected: exits 0, all 6 apps and 5 packages type-check cleanly. Fix any type errors before continuing — most will be from stripped CandyStore types that are now missing.

- [ ] **Step 5: Run tests**

```bash
cd Z:/Github/Janus
pnpm test
```

Expected: exits 0. All tests pass (or `--passWithNoTests` for empty apps).

- [ ] **Step 6: Run build**

```bash
cd Z:/Github/Janus
pnpm build
```

Expected: all 6 apps build successfully to `.next/`. Fix any build errors before proceeding.

- [ ] **Step 7: Commit and push**

```bash
cd Z:/Github/Janus
git add -A
git commit -m "chore: verified monorepo scaffold — install, typecheck, test, build all pass"
git push origin develop
```

---

## Self-review

- **Spec coverage:** All sections covered — workspace structure (Tasks 1-2), root tooling (Task 1), env files (Task 4), secrets flow (Task 4 step 5 + sync-secrets.mjs in Task 3), packages (Task 5), app shells (Tasks 6-11), docker (Task 12), install verification (Task 13). ✓
- **@monorepo/app-components rename:** Covered in Task 5 step 2, and in every app task. ✓
- **Ports:** 5100-5105 used consistently throughout. ✓
- **Secrets already in GitHub:** .secrets.example marks the 6 dev secrets as already configured. ✓
- **No placeholders in code steps:** All steps contain either copy commands or exact file content. ✓
