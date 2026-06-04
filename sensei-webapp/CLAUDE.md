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

**Last session:** Jun 3, 2026 (afternoon) — Google Tasks full integration (Card.googleTaskId/googleTaskListId, push from CardModal+ActionItemModal, Google logo badges, linked-first Kanban sort); SearchableSelect sweep across all assignee pickers; action items scoped to userId (was leaking org-wide); auto-trigger cascade on manual transcript save; due date synced to Google Tasks; calendar event matching fully rewritten (4-signal priority: contact 97, domain Levenshtein 60–95, token overlap 58–80/48–62; INTERNAL_TITLE_RE guard; 673 stale entries cleared; match-helpers.ts shared lib); digest "Meetings Today" reads GoogleCalendarEventCache. **Prior session (Jun 3 morning):** AI intelligence pipeline — meeting signals, Analysis DAG, pgvector RAG (TranscriptChunk + HNSW, 2684 chunks), LLM cascade 70–80% reduction, Meeting Intelligence dashboard. **Prior:** Google Workspace MCP + persisted calendar/tasks/SFDC sync; LiteLLM NAT gateway whitelisted on ECS.
**Tests:** Unit/API: 3116 passing · 7 failing (same 7 pre-existing transcript-validation failures). TypeScript: clean.
**TypeScript:** Clean
**Data models:** Contact now dedicated Prisma model (166 NotebookNodes → 74 Contact records), manualRole flag prevents agent overwrites
**Asset hosting:** s3://sensei-assets created; 14 demo videos + Hasbro BV video (103.2s) uploaded; presigned URL + delete endpoints live
**Hasbro BV video:** Custom Remotion components (HBVSlide, HBVRenderer), hosted at s3://sensei-assets/videos/hasbro-bv.mp4, displayed in Opportunity → Sales → BV Slides
**Schema fixes applied (Supabase + RDS):** dropped `isAdmin` from `OrganizationMember`, `NodeCustomProp @@unique`, `Board → Organization CASCADE`, `NotebookNode → Organization CASCADE`, `Contact @@index(name, accountId)`, `Feature @@index(userId)`. Both DBs in sync with `prisma/schema.prisma`.
**DB access:** RDS via SSM tunnel — run `~/tunnel-rds.sh` first, then `localhost:5433` routes to RDS. Supabase URLs are commented out in `.env.local`.
**E2E specs:** 28 spec files / 519 passing / 203 skipped / 0 real failures. Full suite stable as of April 19, 2026.
**Repo:** Migrated to `atko-presales/sensei` on Okta GitHub (via Terminus/OCM). OCM install + git config needed to push.
**Pilot status:** Sean Newell (presales strategy, works with Joel + Eve) engaged. Rick (SE Manager) supporting. TDI meeting planned for infrastructure/API access asks.
**Attachment AI Analysis:** `analyzeAttachmentPDF()` + `analyzeAttachmentImage()` in `lib/attachment-vision.ts`; `POST /api/notebook/[id]/attachments/[attachmentId]/analyze` route with `after()` async job; `✦ Analyze` button in AttachmentsPanel. Blocked in prod until TDI network unblock (same dependency as Re-extract).
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
  - `contact-helpers.ts` — Shared contact management: `isInternalContact()` (Okta/vendor filter), `matchContact()` (email-first multi-signal dedup), `getAccountContacts()`, `findOrCreateContact()`. All agents that create contacts must use this — contacts always anchor to the account node, never the opportunity.
  - `ai-guard.ts` — Prompt injection / jailbreak detection applied before AI calls
  - `audit.ts` — `logAudit()` helper for key mutations (attached to org + user)
  - `health-score.ts` — Deal health score calculation
  - `ai-jobs.ts` — Background AI job tracking (feeds the notification bell in the UI)
  - `logger.ts` — Error reporting to `/api/errors` with deduplication
  - `database-schema.ts` — Shared types (`SchemaTable`, `SchemaForeignKey`, `SchemaResponse`) and `buildMermaidErDiagram()` / `filterVisibleTables()` utilities. Imported by both `app/api/admin/database/schema/route.ts` (server) and `app/admin/database/page.tsx` (client).
- `app/admin/database/page.tsx` — Database ER diagram page (superadmin-only). Queries live schema from `information_schema`, renders Mermaid `erDiagram` with table/FK filter. Nav entry: Admin → Database.
- `app/api/admin/database/schema/route.ts` — `GET` superadmin-only; queries `information_schema.columns` + FK constraints via `prisma.$queryRaw`; returns `{ tables, foreign_keys, mermaid }`.
- `prisma/schema.prisma` — Models covering auth, multi-tenancy, boards, notebook, features, integrations, AI chat history (`Conversation` + `ChatMessage`), background jobs (`AIJob`), error tracking (`ErrorLog`), and live session coaching (`LiveSession`, `AgentSuggestion`)
- `types/index.ts` — Core domain types
- `constants/index.ts` — Pipeline stages, meeting templates, priorities
- `middleware.ts` — NextAuth protection + extracts subdomain into `x-org-context` header

### State Management

Two-layer system:
1. **Server state** — React Query (`lib/queries.ts`). Hooks: `useBoards()`, `useNotebookTree()`, `useFeatures()`, etc. Mutations call `invalidateQueries` on success. Default staleTime: 30s, gcTime: 60min.
2. **UI state** — Zustand (`lib/store.ts`), persisted to localStorage under key `sensei-ui`. Holds active page, active board ID, notebook sidebar widths, tour progress, and `opportunityTabs` (per-opportunity tab memory). Tab state restored on refresh.

### Multi-Tenancy

Every DB model scopes to `organizationId`. On first login, NextAuth creates a User, an Organization (slug from email + timestamp), and an OrganizationMember with role `admin`. The JWT carries `dbUserId`, `orgId`, `orgSlug`, and `role`.

### Notebook Data Model

Tree of `NotebookNode` records: `account` → `opportunity` → `meeting` → `note/contact/free-folder`. Properties stored in `NodeProperty` (standard) and `NodeCustomProp` (user-defined). Built client-side via `buildNotebookTree()` in `lib/queries.ts`.

**POC Guide:** Opportunities with POC data render a dedicated printable guide (`app/poc-guide/[opportunityId]/page.tsx`). POC data stored as `NodeProperty` keys prefixed with `poc_`. Extraction uses a single Gemini call (1M context, no chunking). Fallback: Claude full context.

### API Pattern

Every API route calls `requireAuth()` from `lib/auth-helpers.ts`. Client uses `apiFetch()` helper (throws on non-OK). Responses are plain JSON or 204 No Content. All inputs validated with Zod.

### Authentication

NextAuth v4: **Okta** (demo.okta.com Auth0 tenant — `auth.demo.okta.com`, replaces `goals.oktapreview.com`), **Google** (OAuth + calendar scopes), **Credentials** (email/password).

### Styling

All styles in `app/globals-redesign.css`. Do not create additional CSS files or CSS modules.

### AI Services

> **LiteLLM access:** ECS NAT gateway (`52.206.25.250`) is whitelisted at `llm.atko.ai` as of Jun 2026. Server-side LLM calls work from ECS. All AI analysis routes call LiteLLM directly. MCP tool handlers (`lib/mcp-tools.ts`) should still remain data/tool-only — keep LLM calls in dedicated API routes, not inside MCP handlers.

All LLM calls are **server-side only** via `llm.atko.ai` (IP-restricted to Okta network). Full infra/proxy options: see `DEPLOY.md`.

- **`LITELLM_API_KEY`** (`claude-4-6-opus`) — all analysis routes: meeting analysis, CoM synthesis, state analysis, presales, stakeholder map, BV slides, chat, agents.
- **`LITELLM_GEMINI_KEY`** (`gemini-2.5-pro`) — POC extraction only (`app/api/notebook/[id]/poc/extract/route.ts`). Falls back to Claude if absent.
- **Groq** (`GROQ_API_KEY`) — Whisper transcription (publicly accessible, no IP restriction)
- **Tavily** — web search used by AI agents

**AI route timeouts:** AbortController timeouts removed — ECS Fargate has no function timeout limit. Routes use `after()` fire-and-forget. `analyze-worker` has `maxDuration = 600`.

**`parseLiteLLMJson` in `lib/litellm-client.ts`:** Balanced-brace walker extracts JSON from LLM responses — handles trailing text after closing brace. Strips markdown code fences. Use for all JSON parsing from LLM output.

**Prompt system:** Defined in `lib/prompt-defaults.ts`, customizable per-org via Prompt Manager UI (Admin → Prompts).

### Testing

**Unit/API tests:** Vitest + `happy-dom` + MSW. Global mocks for NextAuth and IDPWizard in `vitest.setup.ts`. Test files under `__tests__/`.

**E2E tests:** Playwright — `setup` (auth state), `e2e` (authenticated), `smoke` (unauthenticated). CI runs smoke after every Vercel deploy; full e2e on daily schedule.

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
LITELLM_API_KEY                # Internal Okta LiteLLM proxy key — claude-4-6-opus (server-side only; 403s on Vercel — see DEPLOY.md)
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

## Deployment

Production: AWS ECS Fargate — `sensei.oktademo.app`. Deploy: push to `main` → GitHub Actions → ECR → force ECS redeploy.
Static IP: `52.206.25.250` (NAT Gateway, whitelisted at llm.atko.ai). Full infra IDs, cron schedules, and proxy options: see `DEPLOY.md`.

---

## Context File Update Rule

**Every session must update these files before closing:**
- `CLAUDE.md` — if architecture, deployment, or env vars change
- `SESSION_NOTES.md` — summary of what was built, commits made, what's next
- `ROADMAP.md` — if features were completed or new ones scoped
