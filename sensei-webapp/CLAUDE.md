# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Working Protocol

All development in this repo follows the protocol defined at:
`/Users/shantanu.govindjiwala/Documents/WORKING_PROTOCOL.md`

Read it before starting any task. Every feature must clear the full persona pipeline before being presented to the owner.

## Commands

```bash
npm run dev           # Development server on port 3000
npm run build         # Production build
npm run lint          # ESLint
npm test              # Run tests in watch mode (Vitest)
npm run test:ui       # Interactive Vitest UI
npm run test:coverage # Coverage report
npx vitest run        # Run all unit tests once (CI mode)
npx tsc --noEmit      # Type-check without emitting
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
npx playwright test tests/notebook.spec.ts  # Single spec file
```

## Architecture

**senSEi** is a Next.js 16 (App Router) full-stack SaaS application for sales operations — combining a hierarchical notebook, Kanban boards, and a pipeline view. All backend logic lives in Next.js API routes; there is no separate backend service.

### Directory Layout

- `app/api/` — 30+ REST API routes, organized by domain (boards, notebook, organizations, calendar, search, etc.)
- `app/` — Page layouts, `providers.tsx` (React Query + NextAuth), `login/`
- `components/` — All React UI components; `AppShell.tsx` is the main shell handling navigation and page routing
- `lib/` — Business logic and utilities:
  - `queries.ts` — All React Query hooks (60+ operations covering every domain)
  - `store.ts` — Zustand store for UI-only state (active page, active board, notebook state, onboarding tour)
  - `auth.ts` — NextAuth configuration (Okta, Google, Credentials providers)
  - `auth-helpers.ts` — `requireAuth()` middleware used in every API route
  - `prisma.ts` — Prisma client singleton
  - `encrypt.ts` — AES-256-GCM encryption for stored secrets
  - `calendar-sync.ts` / `gcal.ts` — Google Calendar integration
  - `litellm-client.ts` — Server-side LLM calls (used in API routes; see LiteLLM proxy note below)
  - `litellm-browser.ts` — Browser-side LLM calls (only when `NEXT_PUBLIC_LITELLM_BASE_URL` is defined)
  - `ai-guard.ts` — Prompt injection / jailbreak detection applied before AI calls
  - `audit.ts` — `logAudit()` helper for key mutations (attached to org + user)
  - `health-score.ts` — Deal health score calculation
  - `ai-jobs.ts` — Background AI job tracking (feeds the notification bell in the UI)
  - `logger.ts` — Error reporting to `/api/errors` with deduplication
- `prisma/schema.prisma` — Models covering auth, multi-tenancy, boards, notebook, features, integrations, AI chat history (`Conversation`, `ChatMessage`), background jobs (`AIJob`), error tracking (`ErrorLog`), and live session coaching (`LiveSession`, `AgentSuggestion`)
- `types/index.ts` — Core domain types
- `constants/index.ts` — Pipeline stages, meeting templates, priorities
- `middleware.ts` — NextAuth protection + extracts subdomain into `x-org-context` header

### State Management

Two-layer system:
1. **Server state** — React Query (`lib/queries.ts`). All data fetching uses hooks like `useBoards()`, `useNotebookTree()`, `useFeatures()`. Mutations call `invalidateQueries` on success. Default staleTime: 30s, gcTime: 60min.
2. **UI state** — Zustand (`lib/store.ts`), persisted to localStorage under key `sensei-ui`. Holds active page, active board ID, notebook sidebar widths, and tour progress.

### Multi-Tenancy

Every DB model scopes to `organizationId`. On first login, NextAuth creates a User, an Organization (slug from email + timestamp), and an OrganizationMember with role `admin`. The JWT carries `dbUserId`, `orgId`, `orgSlug`, and `role`. The middleware also injects subdomain as org context via headers.

### Notebook Data Model

The notebook is a tree of `NotebookNode` records with types: `account`, `opportunity`, `meeting`, `contact`, `note`, `free-folder`. Accounts have Opportunity children; Meetings have Action Item children. Properties are stored in `NodeProperty` (standard) and `NodeCustomProp` (user-defined) as key-value pairs. The client builds the tree via `buildNotebookTree()` in `lib/queries.ts`.

### API Pattern

Every API route calls `requireAuth()` from `lib/auth-helpers.ts`, which returns the session and enforces org membership. The client uses the `apiFetch()` helper (throws on non-OK responses). Responses are plain JSON or 204 No Content.

### Authentication

NextAuth v4 with three providers: **Okta** (enterprise OIDC, `goals.oktapreview.com`), **Google** (OAuth with calendar scopes), and **Credentials** (email/password).

**Note:** A new Okta tenant (`sen-sei.okta.com`) has been provisioned via Terraform (`terraform/okta/`) with a custom domain (`login.se-n-sei.com`, DNS pending). When ready to switch, update `OKTA_CLIENT_ID`, `OKTA_CLIENT_SECRET`, `OKTA_ISSUER` in Vercel env vars and restore the Okta-only `lib/auth.ts`.

### Styling

All styles live in a single file: `app/globals-redesign.css`. Do not create additional CSS files or CSS modules; add styles there.

### AI Services

Three AI services are in use:
- **LiteLLM proxy** (`llm.atko.ai`) — primary LLM calls; see the LiteLLM proxy note in the Deployment section for the server-vs-browser split
- **Groq** (`GROQ_API_KEY`) — Whisper transcription for live session audio
- **Tavily** — web search used by AI agents

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
LITELLM_API_KEY                # Internal Okta LiteLLM proxy key (server-side, unused from Vercel due to IP restriction)
LITELLM_BASE_URL               # https://llm.atko.ai
LITELLM_MODEL                  # claude-4-6-sonnet
NEXT_PUBLIC_LITELLM_API_KEY    # Vercel only — NOT in .env.local. Browser-side fallback for Vercel deployments.
NEXT_PUBLIC_LITELLM_BASE_URL   # Vercel only — NOT in .env.local. See LiteLLM note below.
NEXT_PUBLIC_LITELLM_MODEL      # Vercel only — NOT in .env.local.
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

The Okta internal LiteLLM proxy (`llm.atko.ai`) behaves differently depending on context:

| Context | How to call LiteLLM | Why |
|---|---|---|
| **localhost (dev)** | Server-side only | Server is on Okta network → 200. Browser gets 403 — Okta enterprise proxy blocks browser `Origin` header requests. Do NOT set `NEXT_PUBLIC_LITELLM_*` in `.env.local`. |
| **Vercel (production)** | Browser-side via `NEXT_PUBLIC_LITELLM_*` | Vercel server is outside Okta network → 403. User's browser IS on Okta network → 200. |

**Local dev:** `NEXT_PUBLIC_LITELLM_*` vars are intentionally absent from `.env.local`. The server-side route handles everything.

**Vercel:** `NEXT_PUBLIC_LITELLM_*` vars are set in Vercel env vars. `DeploymentGuide.tsx` detects them and calls LiteLLM from the browser. If the vars are not set, it falls back to the server route (which will fail with 403 on Vercel).

**Client-side LLM helper:** `lib/litellm-browser.ts` — used only when `NEXT_PUBLIC_LITELLM_BASE_URL` is defined.

### Agent cron jobs

The post-meeting agent (`/api/agent/post-meeting`) is triggered via cron. Configure in `vercel.json` or use an external cron service:

```json
{
  "crons": [{
    "path": "/api/agent/post-meeting",
    "schedule": "*/5 * * * *"
  }]
}
```

Set `AGENT_CRON_SECRET` in Vercel env vars. The endpoint validates this header.

---

## Context File Update Rule

**Every session must update these files before closing:**
- `CLAUDE.md` — if architecture, deployment, or env vars change
- `SESSION_NOTES.md` — summary of what was built, commits made, what's next
- `ROADMAP.md` — if features were completed or new ones scoped
