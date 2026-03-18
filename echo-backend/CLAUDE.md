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
npm run test:e2e     # Playwright E2E tests
```

Run a single test file:
```bash
npx vitest run __tests__/path/to/file.test.ts
```

## Architecture

**echo** is a Next.js 16 (App Router) full-stack SaaS application for private AI thought journaling. All backend logic lives in Next.js API routes; there is no separate backend service.

Core loop: **Record → Transcribe → Reflect → Store (encrypted)**

### Directory Layout

- `app/api/` — REST API routes organised by domain (live-sessions, agent, billing, auth, etc.)
- `app/` — Page layouts, `providers.tsx` (React Query + NextAuth), auth pages
- `components/` — React UI components; `AppShell.tsx` is the main shell
- `lib/` — Business logic and utilities:
  - `queries.ts` — All React Query hooks
  - `store.ts` — Zustand UI state (persisted to localStorage under key `echo-ui`)
  - `auth.ts` — NextAuth configuration (Okta, Google, Credentials providers)
  - `auth-helpers.ts` — `requireAuth()` middleware used in every API route
  - `prisma.ts` — Prisma client singleton
  - `encrypt.ts` — AES-256-GCM encryption for stored secrets
  - `user-encrypt.ts` — Per-user AES-256-GCM encryption for utterances and reflections
  - `billing.ts` — Subscription status calculation
- `prisma/schema.prisma` — DB models covering auth, sessions, reflections, billing
- `middleware.ts` — NextAuth protection

### State Management

Two-layer system:
1. **Server state** — React Query (`lib/queries.ts`). All data fetching uses hooks. Mutations call `invalidateQueries` on success. Default staleTime: 30s.
2. **UI state** — Zustand (`lib/store.ts`), persisted to localStorage under key **`echo-ui`**.

### Data Model

- `LiveSession` — a recording session (status: recording | completed)
- `Utterance` — a transcribed speech segment, encrypted per user
- `SessionReflection` — AI-generated output (summary, actionItems, keyIdeas, followUpQuestion), encrypted per user
- `OrganizationBilling` — subscription status, mobileEntitlement from RevenueCat

### API Pattern

Every API route calls `requireAuth()` from `lib/auth-helpers.ts`. Responses are plain JSON or 204 No Content.

### Authentication

NextAuth v4 with three providers: **Okta** (enterprise OIDC), **Google** (OAuth), and **Credentials** (email/password).

### Per-User Encryption

All utterances and reflections are encrypted at rest using per-user AES-256-GCM keys stored in `UserIntegration`. Use `ensureUserEncryptionKey()` and `safeDecrypt()` from `lib/user-encrypt.ts`.

### Testing

Vitest with `happy-dom` environment and MSW for API mocking. Playwright for E2E. Test files live under `__tests__/`.

### Path Alias

`@/` maps to the repo root. Use `@/lib/...`, `@/components/...`, etc.

### Key Environment Variables

```
DATABASE_URL          # Pooled PostgreSQL connection
DIRECT_URL            # Direct PostgreSQL connection (for migrations)
NEXTAUTH_URL
NEXTAUTH_SECRET
ENCRYPTION_KEY        # 64-char hex = 32 bytes, for integration secrets
OKTA_CLIENT_ID, OKTA_CLIENT_SECRET, OKTA_ISSUER
GOOGLE_CLIENT_ID, GOOGLE_CLIENT_SECRET
RESEND_API_KEY
EMAIL_FROM            # e.g. "echo <hello@echo.app>"
GROQ_API_KEY          # Audio transcription (Whisper)
LITELLM_API_KEY, LITELLM_BASE_URL, LITELLM_MODEL   # AI reflection
STRIPE_SECRET_KEY, STRIPE_WEBHOOK_SECRET
STRIPE_MONTHLY_PRICE_ID, STRIPE_YEARLY_PRICE_ID
REVENUECAT_WEBHOOK_SECRET
DISABLE_BILLING       # Set to "true" to bypass all billing checks (testing only)
```

### Billing & Tiers

See `PRICING_ADVISOR.md` for full tier architecture and pricing strategy.

| Tier | Price | Hours/month |
|------|-------|-------------|
| Free | $0 | 3 hrs |
| Personal | $7.99/mo | 10 hrs |
| Thinker | $14.99/mo | 20 hrs |
| Pro | $24.99/mo | 50 hrs |

- Mobile IAP: RevenueCat (entitlement: `echo_pro`)
- Web: Stripe Checkout
- `DISABLE_BILLING=true` treats all users as Pro (for testing)
- Webhook endpoint: `POST /api/billing/revenuecat-webhook` (mobile) and `POST /api/billing/webhook` (Stripe web)
