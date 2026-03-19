# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Working Protocol

All development in this repo follows the protocol defined at:
`/Users/shantanu.govindjiwala/Documents/WORKING_PROTOCOL.md`

Read it before starting any task. Every feature must clear the full persona pipeline before being presented to the owner.

## Commands

```bash
npm run dev          # Development server on port 3000
npm run build        # Production build
npm run lint         # ESLint
npm test             # Run tests (Vitest)
npm run test:ui      # Interactive Vitest UI
npm run test:coverage # Coverage report
```

Run a single test file:
```bash
npx vitest run __tests__/path/to/file.test.ts
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
- `prisma/schema.prisma` — 14 models covering auth, multi-tenancy, boards, notebook, features, integrations
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

**Okta-only.** NextAuth v4 with a single Okta OIDC provider. Credentials and Google providers removed.

- Okta tenant: `sen-sei.okta.com` (custom domain pending: `login.se-n-sei.com`)
- JIT provisioning: users upserted on first Okta sign-in
- Mobile: PKCE flow via `POST /api/auth/mobile-okta` (accepts Okta ID token, returns sensei JWT)
- Terraform config: `terraform/okta/` — manages app, users, domain, brand
- Cross-app access (CAA): add `token_exchange` grant to app + custom auth server when needed (no rework required)

### Testing

Vitest with `happy-dom` environment and MSW for API mocking. Global mocks for NextAuth and IDPWizard are in `vitest.setup.ts`. Test files live under `__tests__/`.

### Path Alias

`@/` maps to the repo root (configured in `tsconfig.json`). Use `@/lib/...`, `@/components/...`, etc. for all internal imports.

### Key Environment Variables

```
DATABASE_URL          # Pooled PostgreSQL connection (pgbouncer)
DIRECT_URL            # Direct PostgreSQL connection (for migrations)
NEXTAUTH_URL          # https://okta.se-n-sei.com (production)
NEXTAUTH_SECRET
OKTA_CLIENT_ID, OKTA_CLIENT_SECRET, OKTA_ISSUER
GOOGLE_CLIENT_ID, GOOGLE_CLIENT_SECRET
RESEND_API_KEY        # Email delivery (password reset, verification)
EMAIL_FROM            # e.g. "senSEi <hello@se-n-sei.com>"
LITELLM_API_KEY       # Internal Okta LiteLLM proxy key
LITELLM_BASE_URL      # https://llm.atko.ai
LITELLM_MODEL         # claude-4-6-sonnet
GROQ_API_KEY          # Whisper transcription
ENCRYPTION_KEY        # 64-char hex, AES-256-GCM for stored secrets
AGENT_CRON_SECRET     # Auth token for cron-triggered agent endpoints
```

All secrets live in `.env.local` on the EC2 instance — never in git.

---

## Deployment — EC2 (Production)

**This is an internal Okta enterprise app. All data belongs to Okta.**

### Architecture

| Component | Detail |
|---|---|
| Server | AWS EC2 t3.medium, us-east-1 |
| OS | Ubuntu 22.04 LTS |
| Process manager | PM2 (`ecosystem.config.js`) |
| Reverse proxy | Nginx → localhost:3000 |
| SSL | Certbot (Let's Encrypt) |
| Domain | okta.se-n-sei.com |
| Database | Supabase PostgreSQL (pooled via pgbouncer) |
| CI/CD | GitHub Actions → SSH deploy on push to main |

### Deploy flow

```
git push main
→ GitHub Actions (.github/workflows/deploy.yml)
→ SSH into EC2 (EC2_HOST + EC2_SSH_KEY secrets)
→ git pull → npm ci → npm run build → pm2 restart sensei-webapp
```

### On-server secrets

`.env.local` at `/var/www/sensei-webapp/.env.local` holds all secrets.
PM2 inherits the environment — never put secrets in `ecosystem.config.js`.

### Docker
Docker is available on the EC2 instance. A `Dockerfile` exists for containerised deployment (multi-stage, Node 22 slim, non-root user). Can be used for local dev parity or future container orchestration.

### Useful commands on EC2

```bash
pm2 status                    # Check app status
pm2 logs sensei-webapp        # Tail logs
pm2 restart sensei-webapp     # Restart
sudo systemctl status nginx   # Check Nginx
sudo certbot renew            # Renew SSL cert
```

### Agent cron jobs

The post-meeting agent (`/api/agent/post-meeting`) should be triggered
by a systemd timer or cron job on EC2, not Vercel cron.
Set `AGENT_CRON_SECRET` in `.env.local` and add a cron entry:
```bash
*/5 * * * * curl -s -X POST https://okta.se-n-sei.com/api/agent/post-meeting \
  -H "Authorization: Bearer $AGENT_CRON_SECRET"
```

---

## Context File Update Rule

**Every session must update these files before closing:**
- `CLAUDE.md` — if architecture, deployment, or env vars change
- `SESSION_NOTES.md` — summary of what was built, commits made, what's next
- `ROADMAP.md` — if features were completed or new ones scoped
