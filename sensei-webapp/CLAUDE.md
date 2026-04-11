# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Working Protocol

All development in this repo follows the protocol defined at:
`/Users/shantanu.govindjiwala/Documents/WORKING_PROTOCOL.md`

Read it before starting any task. Every feature must clear the full persona pipeline before being presented to the owner.

At the start of each session, also read:
- `SESSION_NOTES.md` — prior session history, recent commits, and what's next
- `ROADMAP.md` — feature status and upcoming scope

## Current State
_Update this section at the end of every session._

**Last session:** April 11, 2026
**Tests:** 2319 passing · 0 failing (192 files)
**TypeScript:** Clean
**E2E specs on hold:** 28 spec files / 664+ tests written. Last subset run: 85 passed, 43 skipped, 0 failed.
**Repo:** Migrated to `atko-presales/sensei` on Okta GitHub (via Terminus/OCM). OCM install + git config needed to push.
**Pilot status:** Sean Newell (presales strategy, works with Joel + Eve) engaged. Rick (SE Manager) supporting. TDI meeting planned for infrastructure/API access asks.
**Blocked on (pilot):** Moving to Okta AWS resolves LiteLLM network issue. TDI ask submitted via `/prod-readiness` page (local only — not in git).

## Custom Skills

Four project-specific skills are available in `.claude/skills/`:

| Skill | Purpose |
|---|---|
| `/new-agent` | Scaffold a new AI agent (API route + LiteLLM integration + React Query hooks) |
| `/add-query` | Add React Query hooks to `lib/queries.ts` following existing conventions |
| `/new-route` | Scaffold a new Next.js API route with `requireAuth()`, Zod validation, and Prisma |
| `/prisma-migrate` | Create a safe Prisma migration (generates, reviews, applies to Supabase) |

Use these instead of writing boilerplate from scratch.

## Commands

```bash
npm run dev           # Development server on port 3000
npm run build         # Production build
npm run lint          # ESLint
npm test              # Run tests in watch mode (Vitest)
npm run test:ui       # Interactive Vitest UI
npm run test:coverage # Coverage report
npx vitest run        # Run all unit tests once (CI mode)
npx tsc --noEmit      # Type-check without emitting (TypeScript errors are intentionally suppressed in next.config.ts; CI runs this separately)
```

Run a single test file:
```bash
npx vitest run __tests__/path/to/file.test.ts
```

Run E2E / Playwright tests:
```bash
npx playwright test                   # All projects (setup + e2e + smoke)
npx playwright test --project=smoke   # Unauthenticated smoke tests only
npx playwright test --project=e2e     # Authenticated E2E tests
npx playwright test __tests__/e2e/notebook.spec.ts  # Single spec file
```

## Architecture

**senSEi** is a Next.js 16 (App Router) full-stack SaaS application for sales operations — combining a hierarchical notebook, Kanban boards, and a pipeline view. All backend logic lives in Next.js API routes; there is no separate backend service.

### Directory Layout

- `app/api/` — 35+ REST API routes, organized by domain (boards, notebook, organizations, calendar, search, etc.). Key AI routes: `notebook/[id]/analyze`, `analyze-worker`, `analyze-com`, `analyze-state`, `analyze-presales`, `analyze-bv` (BV slides), `poc/extract` (Gemini).
- `app/api/seed-demo-data/route.ts` — POST seeds Waters OCI + BMC real deal data (all AI fields pre-populated); DELETE removes all `isDemoData="true"` tagged nodes. Source data in `app/api/seed-demo-data/data.ts` (auto-generated from DB export).
- `app/demos/page.tsx` — Demo library with liquid glass UI. Videos at `public/demos/` named `01-the-pitch.mp4` → `09-the-business-case.mp4` in narrative order + `sensei-full-demo.mp4`.
- `app/execmeeting2/page.tsx` — Public executive deck for leadership meetings (no auth). 9 slides with embedded full demo video. URL: `/execmeeting2`.
- `app/poc-guide/[opportunityId]/` — Standalone printable POC guide page (PDF export)
- `app/` — Page layouts, `providers.tsx` (React Query + NextAuth), `login/`
- `components/` — All React UI components; `AppShell.tsx` is the main shell. `OpportunityDetail.tsx` has two main tabs: **Sales** (CoM fields, Mantra, BV Slides, Signals) and **Presales** (Presales fields, State Analysis, TechQual, POC Guide). `OpportunityAIPanel.tsx` handles all AI generation buttons.
- `lib/` — Business logic and utilities:
  - `queries.ts` — All React Query hooks (65+ operations covering every domain)
  - `store.ts` — Zustand store for UI-only state (active page, active board, notebook state, onboarding tour)
  - `auth.ts` — NextAuth configuration (Okta, Google, Credentials providers)
  - `auth-helpers.ts` — `requireAuth()` middleware used in every API route
  - `prisma.ts` — Prisma client singleton
  - `prompt-defaults.ts` — All AI prompt definitions (customizable per-org via Prompt Manager). Calibrated for SE/Sales leadership output.
  - `poc-merge.ts` — POC draft utilities: `PocDraft` type, `pocDraftToUpserts` for DB writes
  - `com-constants.ts` — Single source of truth for CoM field keys
  - `litellm-client.ts` — Shared `callLiteLLM()` helper used by newer agent routes
  - `ai-guard.ts` — Prompt injection / jailbreak detection applied before AI calls
  - `audit.ts` — `logAudit()` helper for key mutations (attached to org + user)
  - `health-score.ts` — Deal health score calculation
  - `ai-jobs.ts` — Background AI job tracking (feeds the notification bell in the UI)
  - `logger.ts` — Error reporting to `/api/errors` with deduplication
- `prisma/schema.prisma` — Models covering auth, multi-tenancy, boards, notebook, features, integrations, AI chat history (`Conversation` + `ChatMessage`), background jobs (`AIJob`), error tracking (`ErrorLog`), and live session coaching (`LiveSession`, `AgentSuggestion`)
- `types/index.ts` — Core domain types
- `constants/index.ts` — Pipeline stages, meeting templates, priorities
- `middleware.ts` — NextAuth protection + extracts subdomain into `x-org-context` header

### State Management

Two-layer system:
1. **Server state** — React Query (`lib/queries.ts`). All data fetching uses hooks like `useBoards()`, `useNotebookTree()`, `useFeatures()`. Mutations call `invalidateQueries` on success. Default staleTime: 30s, gcTime: 60min.
2. **UI state** — Zustand (`lib/store.ts`), persisted to localStorage under key `sensei-ui`. Holds active page, active board ID, notebook sidebar widths, tour progress, and `opportunityTabs` (per-opportunity tab memory: `Record<nodeId, { activeTab, salesSubTab, presalesSubTab }>`). Tab state restored on refresh so users return to the exact tab they were on.

### Multi-Tenancy

Every DB model scopes to `organizationId`. On first login, NextAuth creates a User, an Organization (slug from email + timestamp), and an OrganizationMember with role `admin`. The JWT carries `dbUserId`, `orgId`, `orgSlug`, and `role`. The middleware also injects subdomain as org context via headers.

### Notebook Data Model

The notebook is a tree of `NotebookNode` records with types: `account`, `opportunity`, `meeting`, `contact`, `note`, `free-folder`. Accounts have Opportunity children; Meetings have Action Item children. Properties are stored in `NodeProperty` (standard) and `NodeCustomProp` (user-defined) as key-value pairs. The client builds the tree via `buildNotebookTree()` in `lib/queries.ts`.

**POC Guide:** Opportunities with POC data render a dedicated printable guide (`app/poc-guide/[opportunityId]/page.tsx`). POC data is stored as `NodeProperty` keys prefixed with `poc_` (e.g. `poc_use_cases`, `poc_environment`, `poc_milestones`, `poc_success_criteria`). The extraction pipeline uses a **single Gemini call** (full untruncated context, 1M window) that extracts all fields including narrative sections in one pass. Fallback: Claude full context (no chunking). Build steps are a separate on-demand route (`poc/build-steps`) triggered by a "Generate Build Steps" button in the AI panel (only shown when use cases exist).

### API Pattern

Every API route calls `requireAuth()` from `lib/auth-helpers.ts`, which returns the session and enforces org membership. The client uses the `apiFetch()` helper (throws on non-OK responses). Responses are plain JSON or 204 No Content.

### Authentication

NextAuth v4 with three providers: **Okta** (enterprise OIDC, `goals.oktapreview.com`), **Google** (OAuth with calendar scopes), and **Credentials** (email/password).

**Note:** A new Okta tenant (`sen-sei.okta.com`) has been provisioned via Terraform (`terraform/okta/`) with a custom domain (`login.se-n-sei.com`, DNS pending). To switch tenants: update settings in Admin → SSO/IDP (takes effect immediately, no redeploy). Env vars remain as fallback if DB config is cleared. `lib/auth-config-cache.ts` manages 60s TTL cache; `buildAuthOptions()` in `lib/auth.ts` is used by the dynamic NextAuth route handler.

### Styling

All styles live in a single file: `app/globals-redesign.css`. Do not create additional CSS files or CSS modules; add styles there.

### AI Services

All LLM calls are **server-side only** via the `llm.atko.ai` proxy (IP-restricted to Okta corporate network). Two API keys in use:

- **`LITELLM_API_KEY`** (`claude-4-6-opus`) — used by all analysis routes: meeting analysis, CoM synthesis, state analysis, presales, stakeholder map, BV slides, chat, and all agents.
- **`LITELLM_GEMINI_KEY`** (`gemini-2.5-pro`) — used exclusively by POC extraction (`app/api/notebook/[id]/poc/extract/route.ts`). The 1M context window handles all meetings + attachments untruncated in a single call. Falls back to Claude full context (no chunking) if the key is absent or call fails.
- **Groq** (`GROQ_API_KEY`) — Whisper transcription for live session audio (publicly accessible, no IP restriction)
- **Tavily** — web search used by AI agents

**AI route timeouts:** Each route has `maxDuration` (Vercel) and an AbortController timeout per endpoint (typically 90s × 2 endpoints = 3 min max failure time). POC extraction uses 180s timeout with `after()` fire-and-forget.

**`parseLiteLLMJson` in `lib/litellm-client.ts`:** Uses a balanced-brace walker (not a greedy regex) to extract JSON from LLM responses — handles Claude responses that append trailing text after the closing brace. Also strips all markdown code fences. Use this for all JSON parsing from LLM output.

**Prompt system:** All AI prompts are defined in `lib/prompt-defaults.ts` and can be customized per-org via the Prompt Manager UI (Admin → Prompts). Prompts are calibrated for SE/Sales leadership output — deal intelligence, not data extraction. The `bv_slides` prompt generates executive-ready Business Value slide content.

### Testing

**Unit/API tests:** Vitest with `happy-dom` environment and MSW for API mocking. Global mocks for NextAuth and IDPWizard are in `vitest.setup.ts`. Test files live under `__tests__/`.

**E2E tests:** Playwright with three projects — `setup` (auth state), `e2e` (authenticated flows), `smoke` (unauthenticated). Auth state is reused across e2e tests via `playwright/.auth/`. The CI workflow runs smoke tests only after every Vercel deployment; full e2e runs on a daily schedule.

### Path Alias

`@/` maps to the repo root (configured in `tsconfig.json`). Use `@/lib/...`, `@/components/...`, etc. for all internal imports.

### Key Environment Variables

```
DATABASE_URL                   # Pooled PostgreSQL connection (pgbouncer)
DIRECT_URL                     # Direct PostgreSQL connection (for migrations)
NEXTAUTH_URL                   # https://sensei-webapp-eta.vercel.app (production)
NEXTAUTH_SECRET
OKTA_CLIENT_ID, OKTA_CLIENT_SECRET, OKTA_ISSUER
GOOGLE_CLIENT_ID, GOOGLE_CLIENT_SECRET
RESEND_API_KEY                 # Email delivery (password reset, verification)
EMAIL_FROM                     # e.g. "senSEi <hello@se-n-sei.com>"
LITELLM_API_KEY                # Internal Okta LiteLLM proxy key — claude-4-6-opus (server-side only; 403s on Vercel — see Deployment)
LITELLM_BASE_URL               # https://llm.atko.ai
LITELLM_MODEL                  # claude-4-6-opus
LITELLM_GEMINI_KEY             # Gemini-only proxy key — gemini-2.5-pro (used exclusively by POC extraction)
LITELLM_GEMINI_MODEL           # gemini-2.5-pro
GROQ_API_KEY                   # Whisper transcription
ENCRYPTION_KEY                 # 64-char hex, AES-256-GCM for stored secrets
AGENT_CRON_SECRET              # Auth token for cron-triggered agent endpoints
SUPER_ADMIN_EMAIL              # Email address of the platform super admin (gates /admin section)
```

All secrets are stored as Vercel environment variables — never in git.

---

## Deployment — Vercel (Production)

**This is an internal Okta enterprise app. All data belongs to Okta.**

### Architecture

| Component | Detail |
|---|---|
| Hosting | Vercel — `okta-solutions-engineering` team |
| Project | `sensei-webapp` → `sensei-webapp-eta.vercel.app` |
| Domain | `okta.se-n-sei.com` (DNS: Vercel nameservers, A record pending attachment) |
| Database | Supabase PostgreSQL (pooled via pgbouncer) |
| CI/CD | Vercel auto-deploys on push to `main` |

### Deploy flow

Push to `main` → Vercel builds and deploys automatically.

```bash
git push origin main   # triggers Vercel build
```

To add or update environment variables:
```bash
vercel env add VAR_NAME production --scope okta-solutions-engineering
```

### Schema changes

When the Prisma schema changes, run `prisma db push` locally to apply to the shared Supabase database:

```bash
npx prisma db push
npx prisma generate
```

Vercel's build runs `prisma generate` automatically during deployment.

### AI features — LiteLLM proxy

The Okta internal LiteLLM proxy (`llm.atko.ai`) only accepts requests from inside the Okta corporate network. **All LiteLLM calls are server-side** — `lib/litellm-client.ts` is used in every AI route; no browser-side calls exist.

| Context | Status |
|---|---|
| **localhost (dev)** | ✅ Works — dev machine is on Okta network |
| **Vercel (production)** | ❌ 403 — Vercel servers are not on Okta network |

**This is the pilot blocker.** Three options evaluated (decision pending):

| Option | How | Pro | Con |
|---|---|---|---|
| **1. Whitelist Vercel IPs** | Okta IT adds Vercel's outbound CIDR to `llm.atko.ai` allowlist | No code changes | Large/changing IP range; needs IT ticket |
| **2. Vercel Static Outbound IPs** | Enable on Vercel Pro — one fixed IP to whitelist | Clean, one IP, fully server-side | ~$50/mo |
| **3. Relay proxy on Okta network** | Small server on Okta network forwarding `/v1/chat/completions` → `llm.atko.ai` | Works today | Extra infra to maintain |

### Agent cron jobs

Seven crons are configured in `vercel.json` (all UTC):

| Route | Schedule |
|---|---|
| `/api/agent/post-meeting` | Daily 6am |
| `/api/agent/digest-email` | Daily 7am |
| `/api/agent/deal-monitor` | Daily 7:30am |
| `/api/agent/meeting-prep` | Every 30 min |
| `/api/agent/followup` | Hourly |
| `/api/agent/weekly-digest` | Friday 4pm |
| `/api/docs-cache/refresh` | Daily 6am |

All cron endpoints validate the `AGENT_CRON_SECRET` header.

---

## Context File Update Rule

**Every session must update these files before closing:**
- `CLAUDE.md` — if architecture, deployment, or env vars change
- `SESSION_NOTES.md` — summary of what was built, commits made, what's next
- `ROADMAP.md` — if features were completed or new ones scoped
