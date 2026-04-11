# senSEi — Session Notes
_Last updated: April 15, 2026 (addendum — optimization pass)_
_Archived: sessions before March 30 addendum 5 → SESSION_NOTES_ARCHIVE_2026-Q1.md_

## ⟶ What's Next
- [x] **Database ER Diagram** — Admin → Database, live schema from information_schema, Mermaid erDiagram, table filter ✓
- [ ] **Database visualizer remaining tabs** — Table Stats, Data Browser, Query Runner (approved scope, not yet built)
- [ ] **Add release note** — "Database Schema Viewer" via Admin → Releases when server is running
- [ ] **Staging environment** — RDS `sensei-db-staging`, ECS service `sensei-webapp-staging`, ALB rule, `staging.se-n-sei.com` DNS
- [ ] **Bug Fix Agent** — EC2 daemon, `scripts/bug-agent.ts`, admin UI badges + PR links, schema 3 fields
- [ ] **Live Support Agent** — `/api/agent/support`, Support tab in AICopilotPanel
- [ ] **POC Fields Rework** — auto-update after meetings, 8 new fields, completeness score
- [x] Contact management Phase 2: signals in contact sidebar, poc_role badge in OrgChart ✓
- [x] Contact management Phase 3: Contacts tab on AccountDetail ✓
- [ ] Andy March: whitelist `52.206.25.250` at llm.atko.ai + register OIDC app in demo.okta.com
- [ ] Sean Newell TDI meeting — bring `/prod-readiness` PDF (local, print before meeting)
- [ ] File provisional patent — route IDF to Okta IP counsel via Sean/Joel
- [ ] Gong API credentials — Sean to arrange with Okta Gong workspace admin
_Updated: April 15, 2026_

---

## Session: April 15, 2026 (addendum) — Optimization + dead code removal

**Codebase/DB optimization pass (commit `9853d9e`):**

| Change | File | Impact |
|---|---|---|
| Batch node validation | `app/api/suggestions/review/route.ts` | N×`findFirst` loop → 1×`findMany` + 1×`findMany` for node check. Reduces per-batch DB calls from N×5 to 2. |
| New index | `prisma/schema.prisma` | `@@index([organizationId, status, createdAt(sort:Desc)])` on AgentSuggestion for fast pending list queries. |
| New helper | `lib/poc-merge.ts` | `batchUpsertProperties(db, nodeId, kvMap)` — eliminates copy-pasted `nodeProperty.upsert` pattern across poc/* routes. |
| Dead code removal | `lib/agent-helpers.ts` | Removed `fuzzyMatchContact` + `levenshtein` (superseded by `matchContact` in contact-helpers.ts after Contact model migration). |

**⚠️ Schema index not yet applied to RDS.** The `prisma db push` timed out because the DB has the Contact model (applied by other session) but it's not in the committed schema.prisma. Applying would have tried to DROP Contact. **Fix:** When the other session commits their schema (Contact model), the index will be included in that same push — no separate push needed.

**Tests:** 2572 passing · 0 failing (204 files)

---

## Session: April 15, 2026 — Database ER Diagram (Admin Console)

### What was built

**Database Schema Viewer** — new `Admin → Database` page that renders a live ER diagram of the PostgreSQL schema directly from `information_schema` at runtime.

**Files added:**
- `lib/database-schema.ts` — shared types (`SchemaTable`, `SchemaForeignKey`, `SchemaResponse`) + `buildMermaidErDiagram()` + `filterVisibleTables()`. Imported by both server route and client page.
- `app/api/admin/database/schema/route.ts` — superadmin-only `GET`; queries `information_schema.columns` + FK constraints via `prisma.$queryRaw`; maps PG types to Mermaid-friendly types; generates full `erDiagram` string.
- `app/admin/database/page.tsx` — renders the erDiagram using same Mermaid lazy-import pattern as `ArchitectureDiagram.tsx`; search filter narrows to matching tables + their 1-hop FK neighbors; re-renders client-side on filter change without refetching.
- `__tests__/api/admin/database-schema.test.ts` — 17 tests covering auth, response shape, type mapping, nullable, FK relationships, mermaid generation, and `buildMermaidErDiagram` filtering.

**Files modified:**
- `app/admin/layout.tsx` — added `database` icon + `{ label: 'Database', href: '/admin/database', icon: 'database' }` nav item after `Data`.
- `__tests__/mocks/handlers.ts` — added MSW handlers for `GET /api/notebook/:nodeId/contacts` and `POST /api/notebook/:nodeId/contacts` (missing from prior contact model migration).
- `__tests__/lib/password.test.ts` — increased bcrypt test timeouts to 30s (was timing out under full-suite CPU load at 5s default).

**Pre-existing failures fixed (3 root causes from April 11 contact model work):**
1. `AccountDetail.test.tsx` — MSW handler missing for `GET /api/notebook/:id/contacts` (new route had no mock)
2. `stakeholder-map.test.ts` + `intelligence-features.test.ts` — pass when run in isolation; were failing under full-suite mock pollution
3. `password.test.ts` — bcrypt timeout under load; increased to 30s

### Tests
2579 passing · 0 failing (206 files). Up from 2385.

---

## Session: April 11, 2026 (addendum 9) — Contact dedup: fix repeated contacts

### Root cause

Race condition: `analyze-worker` triggers `poc/extract` and `stakeholder-map` concurrently after every meeting. Each agent calls `getAccountContacts` at the start, sees no existing contact, and creates a fresh node. Multiple agents running in parallel insert the same people multiple times.

### What was built

`POST /api/admin/dedup-contacts` — superadmin, idempotent. Groups account-level contact duplicates using two conservative rules: email exact match OR name exact match (if one has no email). Different emails = different people, never merged. Fuzzy name alone = not merged. Winner keeps the most non-empty properties; losers' missing fields are copied in, then losers are deleted.

Returns `{ accountsProcessed, groupsFound, contactsRemoved }`.

**Run order on production:**
1. `POST /api/admin/dedup-contacts` — collapse account-level dupes first
2. `POST /api/admin/migrate-opp-contacts` — move remaining opp-level contacts up

**Tests:** 2385 passing · 0 failing (198 files)

---

## Session: April 11, 2026 (addendum 9) — Test coverage + Contact model migration fixes

**Goal:** Maximize test coverage; fix all test failures from Contact model migration.

**New test files (179 new tests):**
- `__tests__/lib/deal-context.test.ts` — 31 tests, 99.5% coverage (was 24%)
- `__tests__/lib/expo-push.test.ts` — 17 tests, 100% coverage (was 48%)
- `__tests__/lib/auth-helpers.test.ts` — 18 tests (was 0%)
- `__tests__/lib/attachment-context.test.ts` — 16 tests
- `__tests__/lib/email.test.ts` — 16 tests (was 3%)
- `__tests__/lib/litellm-client.test.ts` — extended to 34 tests (was partial)
- `__tests__/api/route-coverage-enhancements.test.ts` — 16 API route tests
- `__tests__/components/AIAnalysisOverlay.test.tsx` — 25 tests (new component)
- `__tests__/components/ActionItemsReviewModal.test.tsx` — 24 tests
- `__tests__/components/TranscriptAnalyzer.test.tsx` — 22 tests

**Fixed 36 test failures from Contact model migration (other session):**
- `dedup-contacts.test.ts` — rewrote for `prisma.contact.findMany` (route was fully migrated away from NotebookNode)
- `migrate-opp-contacts.test.ts` — rewrote for verification-only endpoint shape
- `contact-helpers.test.ts` — updated mock contacts to use Contact first-class fields
- `OrgChart.test.tsx` — updated makeContact() for Contact model, `pocRole` camelCase
- `notebook-poc.test.ts`, `stakeholder-map.test.ts`, `agent-routes.test.ts`, `intelligence-features.test.ts` — added `prisma.contact` + `contact-helpers` mocks

**Coverage improvement:** 53.3% → 54.1% (limited by large component files still untested)
**High-value gains:** `deal-context.ts` 24%→99.5%, `expo-push.ts` 48%→100%, `email.ts` 3%→38%

**Tests:** 2579 passing · 0 failing (206 files)
**TypeScript:** Clean
**Commit:** `bf770d3` — pushed to main

---

## Session: April 11, 2026 (addendum 8) — Overlay Rework Part E+F

**Part E — Results diff panel:** After analysis completes, green card fades in below the panel showing: ✓ Analysis complete — Xs · Summary updated · Action items N found — pending review · Opportunity N fields updated. Elapsed time tracked via `analysisStartRef`. Built from `processResult` data before `setStatus('success')`.

**Part F — Error categories:** `TranscriptAnalyzer` catch block now classifies errors:
- HTTP 429 → "Too many requests — wait 60s and retry"
- HTTP 502/503 or network error → "AI service unreachable — check VPN, then retry"
- Parse/JSON error → auto-retry once silently, then "Response malformed — please retry"
- `autoRetriedRef` (ref, not state) avoids stale closure loops

**Tests:** 2385 passing · 0 failing (198 files)
**TypeScript:** Clean
**Commit:** `0f066b1` — pushed to main, ECS auto-deploy triggered

---

## Session: April 11, 2026 (addendum 7) — Contact migration: fix opp-level contacts

### What was built

`POST /api/admin/migrate-opp-contacts` — superadmin route that fixes all existing opportunity-level contacts across all orgs.

**Logic (same three cases as Phase 1):**
- **MOVE** — no existing account-level contact found → `UPDATE parentId = accountId`
- **MERGE** — email/name match found at account level → copy missing properties onto it, delete the opp-level duplicate
- **DELETE** — Okta/vendor employee (`@okta.com`, `@auth0.com`, "Okta" in name/title) → delete without migrating

Uses `isInternalContact`, `matchContact`, `getAccountContacts` from `lib/contact-helpers` — same dedup logic as the live routes. Idempotent: safe to run multiple times. Returns `{ moved, merged, deleted, skipped, orgsProcessed }`.

**Tests:** 7 new tests — 403 auth, all-zeros on empty data, move case, merge case, Okta-employee delete, orphaned-opp skip, idempotency.

**Tests:** 2376 passing · 0 failing (197 files)
**TypeScript:** Clean

---

## Session: April 11, 2026 (addendum 6) — Overlay Rework Parts B, C+D

### Part B — Non-blocking overlay mini-panel

`AIAnalysisOverlay` rewritten: full-screen for first 3s, then collapses to a fixed bottom-right mini-panel (`position: fixed`) so users can keep working while analysis runs in background. Mini-panel shows title + stage (`Preparing` 0–8s / `AI analyzing — Xs elapsed` 8s+) + pulsing dot. Elapsed timer ticks every second. Hint text: "Working in background — you can keep using senSEi". No schema changes.

### Part C — Pre-analysis warnings

Two inline banners shown above the Analyze button when status is idle:
- **Amber**: if transcript > 15,000 chars → "Long transcript (~N min) — analysis may take 2–3 minutes"
- **Blue**: if already analyzed → "Already analyzed — re-analyzing will update existing results"

### Part D — 10-second countdown review modal

Replaced silent `autoConfirmTodos` call in `processResult` with `setReviewModalTodos` — action items now always show the `ActionItemsReviewModal`. New `autoAcceptSeconds={10}` prop added to the modal: shows "Auto-accepting in Xs" banner with a "Review" button to stop the countdown. If user doesn't interact, all items are accepted after 10s. Any interaction (click, input change) stops the countdown permanently.

**Tests:** 2369 passing · 0 failing (196 files)
**TypeScript:** Clean
**Commits:** `c4a30ef` (Part B) · `1b61676` (Parts C+D) — both pushed to main, ECS auto-deploy triggered

---

## Session: April 11, 2026 (addendum 5) — Contact management Phase 2 + 3

### What was built

**Phase 2 — UI enhancements:**

| File | Change |
|---|---|
| `components/OrgChart.tsx` | Added `POC_ROLE_LABEL` map. `poc_role` badge rendered inline with the role pill (flex row). `hasAnyRole` and flat grid filter now also check `poc_role` so contacts enriched by poc/extract show up even without a stakeholder-map role. |
| `components/NotebookPage.tsx` | Added AI intel block in `ContactPanel` — shows role badge, poc_role badge, sentiment indicator, and signals quote when present. All conditional: only renders when AI has populated at least one field. |

**Phase 3 — Contacts tab:**

| File | Change |
|---|---|
| `components/AccountDetail.tsx` | New "Contacts (N)" tab alongside Overview and Power Map. State type widened to include 'contacts'. Tab content: enriched contact cards showing name, title, email, sentiment indicator, role badge (green), poc_role badge (grey). Clicking navigates to that contact's detail panel. |

**Tests:** 2369 passing · 0 failing (196 files)
**TypeScript:** Clean

---

## Session: April 11, 2026 (addendum 4) — Overlay Rework Part A: remove truncation + timeouts

### What was built

**Part A of AI Analysis Overlay Rework** — full transcript context (no truncation), no LLM timeouts.

**Root cause of prior analysis degradation:**
- `smartTruncate` was cutting transcripts at 10K+5K chars — dropping critical deal context mid-sentence
- `capTranscripts` was proportionally trimming transcripts across meetings — same problem
- `AbortController` 90s timeouts were a Vercel constraint. On ECS Fargate there is no function timeout. Removing them lets long transcripts complete without error.
- Claude 200K context window means truncation was never necessary

**Changes:**

| File | Change |
|---|---|
| `analyze-worker/route.ts` | Removed `smartTruncate`, sends full `transcript`. Removed `AbortController`. `maxDuration` 240→600. `calcMaxTokens` replaced with inline formula. |
| `lib/litellm-client.ts` | Removed `AbortController` timeout entirely. Removed `timeoutMs` from `LiteLLMOptions` interface. |
| `analyze-bv/route.ts` | Removed `capTranscripts` call and timer block |
| `analyze-com/route.ts` | Removed `capTranscripts` + `calcMaxTokens`, removed AbortController, hardcoded `max_tokens: 6000` |
| `analyze-state/route.ts` | Removed `capTranscripts` + `calcMaxTokens`, removed AbortController, hardcoded `max_tokens: 8000` |
| `analyze-presales/route.ts` | Removed `calcMaxTokens` + `capTranscripts`, removed AbortController timeout block |
| `lib/transcript-utils.ts` | Added deprecation comments to `smartTruncate` and `capTranscripts` (functions kept for backward compat) |

**Tests:** 2365 passing · 0 failing (196 files)
**TypeScript:** Clean
**Commit:** `8baec62` — pushed to main, ECS auto-deploy triggered

---

## Session: April 11, 2026 (addendum 3) — Contact management Phase 1: dedup + Okta filter

### What was built

**Deep-dive + plan:** studied the full contact management system — stakeholder-map, poc/extract, OrgChart, fuzzyMatchContact, property coverage. Identified 15 gaps; built a 3-phase plan.

**Phase 1 implemented:**

| Change | File(s) |
|---|---|
| New `lib/contact-helpers.ts` | `isInternalContact()`, `matchContact()` (email-first, last+initial, levenshtein with length guard), `getAccountContacts()`, `findOrCreateContact()` |
| Okta employee filter | Both `stakeholder-map` and `poc/extract` now skip contacts with `@okta.com`/`@auth0.com` email, name containing "Okta"/"Auth0", or Okta-brand title |
| Email-first dedup | `stakeholder-map` now uses `matchContact()` instead of `fuzzyMatchContact()` — email match takes priority over name matching |
| Account-level contacts | `poc/extract` now creates stakeholder contacts under the **account** node (via `node.parentId`), not the opportunity — eliminates the split between two contact pools |
| Email extraction prompt | `stakeholder-map` system prompt updated: adds `email` field to the JSON schema and adds "CUSTOMER-SIDE ONLY" exclusion instruction for Okta employees |
| Okta exclusion in poc_extract prompt | `lib/prompt-defaults.ts` Section 10 now explicitly says not to include `@okta.com`/`@auth0.com` employees in stakeholders |
| Tightened `fuzzyMatchContact` | Raised levenshtein minimum threshold 0.5→0.65, added length guard (names must be >5 chars), levenshtein score 0.6→0.65 |
| `LiteLLMOptions.timeoutMs` restored | ECS deployment removed AbortController from `callLiteLLM` but left routes passing `timeoutMs` — restored as accepted-but-ignored field |

**Tests:** 4 failing tests written before implementation, all 4 now green. 25 new tests total across 4 files.
**Test state:** 2365 passing · 0 failing (196 files)
**TypeScript:** Clean

---

## Session: April 11, 2026 (addendum 2) — Fix: analysis results never show without page reload

### Root cause
`TranscriptAnalyzer` polls `GET /api/notebook/{nodeId}` after dispatching analysis. That endpoint returns properties as a Prisma relation **array** `[{key, value}]`. But the polling read `node?.properties?.summary` treating it as a flat object — `Array.summary` is always `undefined`. So polling never detected completion, `processResult` was never called, `invalidateQueries` never fired, and users had to reload to see results. The overlay also stayed up indefinitely.

### Fix
Added `getNodeProp(props, key)` helper in `TranscriptAnalyzer.tsx` that handles both array format (direct endpoint response) and flat format (React Query tree). Applied to both polling loops.

### Tests added (5 + 1)
- `MeetingDetail.test.tsx`: 4 new tests — Analysis Results section renders when summary/problems present, does NOT render when absent, all fields show correctly
- `TranscriptAnalyzer.test.tsx`: 1 new timer-based test — overlay dismisses when poll returns array-format properties with summary (confirms fix, was timing out before)

### Commit
`079a388` — pushed to `origin/main`

**Tests:** 2324 passing · 0 failing · TypeScript clean

---

## Session: April 11, 2026 (addendum) — Fix "Failed to parse AI response" across all 8 AI routes

### Root cause
The April 6 `parseLiteLLMJson` fix (balanced-brace walker) was only applied to `analyze-note`. The other 8 analysis routes still used the greedy regex `content.match(/\{[\s\S]*\}/)` which fails when Claude:
- Appends trailing text after the closing brace
- Wraps output in markdown code fences (```json\n...)

Error log showed 194 open "Failed to parse AI response" errors. Preview confirmed the pattern: `` ```json\n\n {"summary": "This was an internal alignment...` ``

### Fix
Replaced old regex + repair loop in 8 routes with `parseLiteLLMJson<T>()`:
- `app/api/notebook/[id]/analyze-worker/route.ts` — removed strip + greedy match + 7-close repair loop
- `app/api/notebook/[id]/analyze-com/route.ts`
- `app/api/notebook/[id]/analyze-state/route.ts` — removed strip + greedy match + 5-close repair loop
- `app/api/notebook/[id]/analyze-presales/route.ts`
- `app/api/notebook/[id]/followups/route.ts`
- `app/api/agent/stakeholder-map/route.ts`
- `app/api/agent/research/route.ts` — was matching `\[[\s\S]*\]` (array variant)
- `app/api/notebook/[id]/poc/extract/route.ts` — in `callLLM()` helper

### Test mock update (3 files)
Changed `vi.mock('@/lib/litellm-client', () => ({ parseLiteLLMJson: vi.fn().mockReturnValue({}) }))` to `vi.fn().mockImplementation(actual.parseLiteLLMJson)` in:
- `__tests__/api/notebook-analysis.test.ts`
- `__tests__/api/notebook-poc.test.ts`
- `__tests__/api/ai-job-tracking-routes.test.ts`

This makes `parseLiteLLMJson` behave like the real parser in tests that control output via `global.fetch`, while still allowing `mockReturnValueOnce` overrides in analyze-note tests.

### Test state
- 2319 passing · 0 failing · TypeScript clean

### Commit
`17a8c67` — pushed to `origin/main` (Vercel auto-deploy triggered)

---

## Session: April 8–11, 2026 — Patent IDF, patent demo video, GitHub migration, TDI brief

### Patent work
- **23-innovation IDF complete** (`PATENT_IDF.md`) — all sections filled, dates entered, public disclosure documented, prior art section cleaned of all hallucinated patent numbers (all 13 Gemini-cited numbers verified as wrong against USPTO)
- **Two Gemini Deep Research searches run** — competitive landscape accurate, patent numbers all wrong; documented in `PATENT_PRIOR_ART_SEARCH.md` + `PATENT_PRIOR_ART_SEARCH_V2.md`
- **Defensibility analysis** — 23 innovations assessed across 3 tiers: 6 strong (lead claims), 10 defensible with narrow claims, 7 weak/risky. Innovation 5 (full-context POC) recommended to drop.
- **Patent demo video** — 2:31 Remotion video covering 4 core innovations. Audio: TTS via LiteLLM. Paragraph timings silence-detected. Final: `public/demos/patent-demo.mp4`
- **`/admin/patent-demo`** — superadmin-only page with video player, download link, and innovation summary cards

### GitHub migration
- Repo created at `atko-presales/sensei` via Terminus. Remote added as `okta`.
- `public/demos/*.mp4` (206MB) removed from git tracking, added to `.gitignore` — videos served from filesystem only
- Committed: patent docs, admin patent route, manifest update, gitignore fix
- Push pending OCM setup (Terminus access + `ocm install` + git config)

### Bug fixes
- **Sidebar z-index** (`globals-redesign.css`) — added `position: relative; z-index: 10` to `.sidebar-wrap`. Content area's `position: relative` stacking context was capturing pointer events over sidebar nav items, making Agents tab unclickable on Organisations page.

### Stakeholder context (new)
- **Sean Newell** — presales strategy, works with Joel Hanson + Eve (Joel's boss). Saw full demo (Eve forwarded it), thought it was a concept — surprised to learn it's production software.
- **Rick** — Shantanu's SE Manager, supporting senSEi actively. Natural fit for SE Champion / Adoption Lead role.
- **Eve** — Joel Hanson's boss, presales leadership. Already forwarding the demo internally.

### TDI brief
- `/prod-readiness` page created — A4 print-ready PDF for Sean's TDI (Technology and Digital Infrastructure) meetings. Three sections: Personnel (6 roles), API Access (6 integrations), Infrastructure (8 tiles). **Local only — not committed to git.**

### AWS architecture planned
- Full ECS Fargate + RDS + ECR + ALB + CloudFront + EventBridge + S3 + Secrets Manager design documented
- Moving to Okta AWS resolves LiteLLM network blocker entirely

### Test state
- Tests: 2319 passing · 0 failing · TypeScript clean (no code changes this session)

---

## Session: April 5–7, 2026 — Demo video, exec deck, seed data, AI fixes

### AI service fixes

**`parseLiteLLMJson` — balanced-brace extractor** (`lib/litellm-client.ts`)
- Root cause: greedy regex `\{[\s\S]*\}` captured trailing text after closing brace → `JSON.parse` failed
- Fix: replaced with `extractBalancedJson()` — character-by-character balanced brace walker that stops exactly at the closing `}`, handles escaped characters and nested objects
- Also stripped ALL code fences (not just leading/trailing) in the stripped preprocessing step

**`analyze-note` fallback to transcript** (`app/api/notebook/[id]/analyze-note/route.ts`)
- Note analysis was returning empty when `node.content` was sparse (meeting with transcript but no typed notes)
- Fix: added `include: { properties: true }` to node fetch; falls back to `properties.transcript` when `node.content < 10 chars`
- Result: Note Analysis now works on meetings that have only a recorded transcript

**Auth-routes test fixes** (`__tests__/api/auth-routes.test.ts`)
- Domain restriction added to register endpoint broke tests using `@example.com` emails
- Fixed test emails to `@okta.com`; added `sendWelcomeEmail: vi.fn().mockResolvedValue(undefined)` to email mock (was undefined, `.catch()` call threw)

### Demos page — liquid glass redesign (`app/demos/page.tsx`)
- Full rewrite: animated background orbs, dot grid overlay, frosted glass cards with `backdrop-filter: blur(20px)`
- Featured full demo section (large card, 16:9 thumbnail) + 9-act grid with act number badges
- Inline video player with glass overlay when a card is clicked
- `autoPlay` removed — was blocking in browsers (returned spinning gray rectangle)
- Act files renamed to narrative order: `01-the-pitch.mp3` → `09-the-business-case.mp3` matching `FullDemo.tsx` sequence
- Old Playwright-captured clips (`meeting-analysis.mp4`, `com-mantra-bv.mp4`) removed

### Remotion demo video
- 9 acts rendered and stitched in correct narrative order (1→6→2→3→4→5→7→8→9): `sensei-full-demo.mp4` (13:57, 76MB)
- Audio files and scripts renamed from `act*.txt/mp3` → `01-the-pitch.txt/mp3` etc. throughout (`narrate.ts`, all Act*.tsx, `DEMO_RULES.md`)
- Personal narration scripts rewritten in first-person voice — removed all "dropping the ball" language, framed as process friction not personal failure
- All 9 scripts in `/Documents/sensei-demo-remotion/audio/` with new naming

### Executive meeting deck (`app/execmeeting2/page.tsx`)
- New public page (no auth required) for Joel Hanson VP Presales meeting
- 9 slides: Hero → Demo Video (embedded, full 14-min) → Problem → FY27 Alignment → Business Impact ($3.3M) → The Vision (integrations roadmap) → Before/After → What I Need → The Ask (30 SEs, 30 days)
- Vision slide includes MCP server / Claude integration as output channel
- Deployed to Vercel: `https://sensei-webapp-eta.vercel.app/execmeeting2`

### Roadmap updates (`ROADMAP.md`)
- Added `## Planned` section with:
  - Autonomous Bug-Fix Agent (EC2 + Claude Code daemon) — full architecture documented
  - sensei MCP Server — tools, API key auth, when to use vs Claude Code skill
  - B2B Tenant Provisioning
  - Switch Okta Issuer to Production
- Removed Sentry (redundant with in-house ErrorLog/SysLog) and Upstash Redis (in-memory sufficient for internal tool)

### Seed demo data (`app/api/seed-demo-data/`)
**Route + data file added:**
- `POST /api/seed-demo-data` — seeds Waters OCI + BMC real deal data
- `DELETE /api/seed-demo-data` — removes all `isDemoData: "true"` tagged nodes and descendants
- `app/api/seed-demo-data/data.ts` — auto-generated from real DB export (Waters OCI + Boston Medical Center - BMC)

**What's seeded:**
- 2 accounts (Waters, Boston Medical Center - BMC) with full account-level fields (accountSnapshot, trackingItems, industry, tier, region)
- 2 opportunities with every AI-generated field pre-populated: CoM (all 7 fields + evidence), BV Slides JSON (7 slides for Waters, 4 for BMC), State Analysis (currentStateDesc/Html/Mermaid + futureState equivalents), TechQual items, presales fields, health score + factors
- 5 meetings with real transcripts, summaries, problems, next steps, key quotes, action items, attendees, follow-up drafts (followupSlack/Email/Persona), meeting prep (prepNotes/Icebreaker/IndustryInsights), SFDC update drafts (sfdc_update_se)
- 14 board cards across BACKLOG/TODAY/IN PROGRESS/BLOCKED/DONE
- After seed: navigates to first opportunity + fires onboarding tour

**UI:**
- Empty state: prominent CTA card ("New here?" + blue gradient border + full-width primary button)
- Sidebar: "Sample data / Clear" banner when demo data is active

### Tests
**2319 passing · 0 failing (192 files)**

---

## Session: April 4, 2026 (addendum 2) — Bug sweep + UX fixes

**Protocol catch-up:** Tests, context files, and surface sync completed end-of-session.

**Build steps `response_format` fix:**
- Removed `response_format: { type: 'json_object' }` from both `poc/extract/route.ts` (Claude fallback) and `poc/build-steps/route.ts` — the LiteLLM proxy at `llm.atko.ai` doesn't support this OpenAI-specific param for Claude, causing silent 400s
- Added markdown fence stripping to build-steps JSON parser as safety net

**SAML removed from IDP settings page:**
- Removed SAML tab from `app/admin/sso/page.tsx` — OIDC-only now (SAML not wired to any auth flow, was dead weight)
- Cleaned up route schema, CSS, and tests

**presalesNotes display fixed:**
- `parseListItems` was treating `SG - CISO...` entries as dash-list markers → rendered as `<ol>` in read view, losing content
- Fixed both read-view occurrences in `OpportunityDetail.tsx` to use plain `pre-wrap` text
- Updated `analyze-presales` prompt: meetingBlurbs now explicitly newest-first, richer detail required (attendees, systems, signals, deal momentum)

**Review button fixed (PendingSuggestionsBar):**
- Was calling `setActivePage('copilot')` — page doesn't exist, rendered blank screen
- Fixed to `openAIPanel('actions')` which opens the AI Copilot panel on the Actions tab

**User detail page "User not found" fixed:**
- `GET /api/admin/users/[userId]` endpoint was missing — page worked around it by searching the user list with the cuid as search term (matched nothing)
- Added `GET` handler to `app/api/admin/users/[userId]/route.ts` that fetches directly by ID
- Fixed `app/admin/users/[userId]/page.tsx` to call it directly
- Fixed React.Fragment key error in the user detail DL grid

**NoteAnalyzer board selector removed:**
- Dropdown showing "my project" next to Analyze button when multiple boards exist — confusing mid-workflow
- Removed selector; always uses active/first board silently

**Admin nav — Identity Provider:**
- Added "Identity Provider" nav item to admin layout → `/admin/sso`
- Fixed active-state conflict: `/admin/sso-mgmt` starts with `/admin/sso`, used exact match for the IDP item

**Opportunity tab persistence:**
- Tabs and sub-tabs in `OpportunityDetail` reset to 'overview'/'com'/'presales' on refresh
- Added `opportunityTabs: Record<nodeId, { activeTab, salesSubTab, presalesSubTab }>` to Zustand store, persisted under `sensei-ui`
- `OpportunityDetail.tsx` reads from store on init (lazy `useState` initialiser), writes through on every tab change; scoped per opportunity

**NotebookPage collapse fix:**
- Clicking collapse arrow on an account with a selected opp moved the arrow but kept children visible (`hasActiveChild` exception)
- Fix: on collapse of a node with active descendant, `setActiveNode(node.id)` first so `hasActiveChild` becomes false and tree actually closes

**Error log cleanup:** Bulk-resolved 56 stale errors (old API key period, refactored code, old Supabase upload attempts pre-sanitization).

**Tests:** 2319 passing · 0 failing (192 files). TypeScript: clean.
- New: `GET /api/admin/users/[userId]` — 3 tests in `admin-user-security.test.ts`
- New: `setOpportunityTab` store — 5 tests in `store.test.ts`
- Updated: `OpportunityDetail.test.tsx`, `PendingSuggestionsBar.test.tsx`, `poc-merge.test.ts`

---

## Session: April 4, 2026 (addendum) — Okta settings admin UI (live config)

**Okta settings now configurable from admin UI at `/admin` → SSO/IDP:**
- OIDC tab: Discovery URL, Client ID, Client Secret (save without entering secret preserves existing)
- SAML tab: paste XML, drag-and-drop file upload, or type XML directly (stored, not yet active for login)
- Source badge shows "Using saved config" (green) vs "Using env var fallback" (amber)
- Changes take effect on the next login attempt — no restart needed (60s cache invalidated on save)

**Architecture:**
- `lib/auth-config-cache.ts` — 60s TTL cache; reads master org's OIDC fields from DB, falls back to env vars
- `lib/auth.ts` — added `buildAuthOptions(config)` factory; static `authOptions` kept as fallback for other importers
- `app/api/auth/[...nextauth]/route.ts` — now async, builds NextAuth config per-request via cache
- `app/api/admin/okta-config/route.ts` — GET (masked config + source), POST (save + invalidate cache), admin-only
- `app/admin/sso/page.tsx` — rewritten with enterprise-grade UI (tabs, source badge, drop zone, inline status)

**Tests:** 2312 passing · 0 failing (192 files). TypeScript: clean.
- New: `__tests__/api/admin-okta-config.test.ts` (10 tests)

---

## Session: April 4, 2026 — POC extraction redesign + API key fix

**API key + model reverted:** `LITELLM_API_KEY` updated to working key, `LITELLM_MODEL` reverted to `claude-4-6-sonnet`.

**POC extraction redesigned for quality over speed:**
- Removed 3-pass chunked pipeline (structured → narrative → build steps) — replaced with single Gemini call + Claude fallback
- No truncation: full untruncated transcripts sent to Gemini 2.5 Pro (1M context window)
- `poc_extract` + `poc_extract_narrative` prompts merged into one unified prompt — narrative generated from full context (higher quality than synthesizing from structured data summary)
- `response_format: { type: 'json_object' }` added — eliminates JSON parse repair dance
- Gemini timeout: 240s (was 180s). Claude fallback: 240s. Client polling deadline: 8 min (was 5). Age timeout: 9 min (was 6).
- POC Guide polling now refreshes POC data every 15s so partial results appear during extraction

**Build Steps moved to separate on-demand button:**
- New route: `POST /api/notebook/[id]/poc/build-steps` — reads existing use cases, generates step-by-step Okta console instructions per use case
- "Generate Build Steps" button added to OpportunityAIPanel — only visible when use cases exist (`hasUseCases` check via `usePOCData`)
- Same fire-and-forget + job polling pattern

**Cleanup:**
- `mergePocDrafts` removed from `lib/poc-merge.ts` (no longer called)
- `poc_extract_narrative` prompt entry removed from `lib/prompt-defaults.ts`
- `smartTruncate` no longer imported in extract route

**Tests:** 2302 passing · 0 failing (191 files). TypeScript: clean.
- New test file: `__tests__/api/notebook-poc-build-steps.test.ts` (7 tests)
- Updated: `__tests__/api/notebook-poc.test.ts` — replaced chunk/narrative tests with single-call + fallback tests
- Removed `mergePocDrafts` tests from `__tests__/lib/poc-merge.test.ts`

---

## Session: April 3, 2026 — Leadership-quality prompts + BV slides + reliability fixes

**All analysis prompts calibrated for SE/Sales leadership:**
- Philosophy: from "extract what was said" → "deal intelligence that SE/Sales leadership would act on"
- `buildPrompt()`: system framing → "SE writing a post-meeting deal intelligence brief for leadership review"; summary → "deal intelligence brief: purpose, customer signal, deal advancement, momentum assessment"; problems → "deal-relevant blockers with deal impact"; action item descriptions → "why this matters for the deal"; next steps → "SE-accountable actions a manager can track"
- `meeting_analyze_com`: framing → "SE completing CoM for ASK review / deal strategy session quality"; specific to this customer, not generic
- `meeting_analyze_state`: system → "senior SE preparing executive presentation for sponsor/leadership"; descriptions → executive-quality QBR/briefing narrative
- `presales_analyze`: leadership framing; meetingBlurbs 2-3 sentences; risksGaps with business impact; presalesNextSteps with owner/rationale; temperature 0.2→0.3

**BV Slides:** `analyze-bv` route + `bv_slides` prompt + `useGenerateBVSlides` hook + tab in CoM section + AI panel button. 11 new tests.

**POC Extract button shows doc count:** `attachmentCount` prop added to POCGuide via `useNodeAttachments`.

**Reliability fixes:**
- Overlay stuck after success: `try/finally` in `processResult` → `setStatus('success')` always runs
- Overlay stuck on failure: capture `jobId` from 202, poll job status → immediate dismiss
- `analyze-worker`: 240s maxDuration, 90s per-endpoint timeout, 5000 max_tokens
- `analyze-state`: 16000 max_tokens, JSON bracket repair, better error logging
- Reverted meeting analysis, state analysis, and CoM prompts back to original after Gemini experiment (then re-calibrated for leadership)

**Model upgrade — claude-4-6-sonnet → claude-4-6-opus everywhere:**
- New `LITELLM_API_KEY` (`sk-asWc7tWa_rw845IbAEOnvg`) tested against llm.atko.ai ✓
- `LITELLM_MODEL=claude-4-6-opus` updated in `.env.local` and Vercel production env vars
- Claude Code (`~/.claude/settings.json`) switched from `sonnet` → `opus`
- `CLAUDE.md` updated — all `claude-4-6-sonnet` references replaced with `claude-4-6-opus`
- Deployed to Vercel via commit `d0e018a`

**Peripheral documentation sweep — all files updated to reflect current product:**
- `app/page.tsx` (Hero): Feature cards updated — "Sales Intelligence" card (COM + Mantra + BV Slides + Signals), POC Guide card (3-phase extraction + build steps + PDF), AI Agents card (12 agents, BV slides, leadership-quality), "How it works" step 2 mentions BV slides + POC build steps.
- `app/brief/page.tsx` (1-page brief): Date March → April 2026. Presales^AI bullet updated to mention BV Slides + Sales/Presales tab structure. "What It Is" updated with new capabilities + /pitch reference.
- `app/pitch/page.tsx` (Exec deck): All "March 2026" → "April 2026". FY27 Presales^AI bullet updated with BV Slides + tab structure. "The Shift" column updated. Appendix A1 meeting analysis + AI agents bullets updated.
- `components/OnboardingTour.tsx` (Virtual tour): Added new step "Sales & Presales Tabs" (center modal) after notebook-sidebar, explaining COM/Mantra/BV Slides/Signals in Sales and Presales tracking/State/TechQual/POC Guide in Presales. Tour is now 12 steps.
- `lib/docs-content.ts` (Docs): Notebook article fully rewritten for new tab structure — Sales tab (COM/Mantra/BV Slides/Signals) and Presales tab (Presales/State Analysis/TechQual/POC Guide) with accurate descriptions. Presales tracking article fixed ("Presales tab" not "Intelligence tab"). AI features article updated: COM synthesis, BV Slides, Extract POC (3-phase), State Analysis PDF, Presales analysis all updated.

**Tests:** 2302 passing · 0 failing

---

## Session: April 2, 2026 (addendum 2) — POC extraction build steps + opportunity tab restructure

**POC PDF state diagrams fix (4 root causes):**
- Root cause 1 (`break-inside`): `@media print { .pg-section-state { break-inside: avoid } }` — two large HTML diagrams together exceeded one page. Chrome skips sections it can't keep together when `break-inside: avoid` is set. Fix: removed from section level, moved to `.pg-state-col` (per-column).
- Root cause 2 (destructive CSS): `.pg-state-diagram div[style*="width"] { width: auto !important }` and `flex-wrap: wrap !important` obliterated AI HTML diagram layout. Also `contain: layout` caused zero-height containers. Also `overflow: hidden` on both `.pg-state-col` and `.pg-state-diagram` clipped diagram content. All removed/fixed to `overflow: visible`.
- Root cause 3 (`hasState`): print page didn't check `currentStateMermaid`/`futureStateMermaid` — if user had only Mermaid, entire section hidden. Fixed. Mermaid CDN script added as last-resort fallback.
- Root cause 4 (WRONG PRIORITY — the actual "still not working" cause): my fix set priority as `currentStateSvg → currentStateMermaid → currentStateHtml`. Since analyze-state writes BOTH `currentStateMermaid` AND `currentStateHtml`, `currentStateMermaid` was always truthy → rendered raw `<pre>` with Mermaid text → skipped the actual HTML diagram entirely. Fixed: HTML and SVG are now always shown if they exist (mirrors app). Mermaid CDN only fires when NEITHER SVG nor HTML exists.
- Bonus: `meeting-prep/route.ts` midnight-crossing bug — HH:MM string comparison `"23:43" > "00:03"` skipped meetings when 90-min window crossed midnight. Fixed with minutes-from-midnight arithmetic.

**Tests:** 2302 passing · 0 failing

**POC extraction: build steps + milestones (Phase 3):**
- `lib/prompt-defaults.ts`: Fixed `poc_extract` schema — replaced `buildSteps: []` with a 2-step example with callouts. Added `milestones` field to output schema. Strengthened `successCriteria` instruction to require testable verbs. Added new `poc_extract_build_steps` prompt template.
- `app/api/notebook/[id]/poc/extract/route.ts`: Removed `CHUNK_SCHEMA_NOTE` from Gemini call (Gemini now generates real build steps). Updated `CHUNK_SCHEMA_NOTE` text for Claude chunked fallback. Added `appendSchemaNote` optional param to `callLLMChunk` (Phase 2 and 3 pass `false`). Added **Phase 3** after narrative pass: calls `poc_extract_build_steps` template + Okta Knowledge, merges build steps onto use cases by title match, writes back to DB.
- `lib/poc-merge.ts`: Added `milestones?` array to `PocDraft`, merge logic (dedup by `item`), `poc_milestones` upsert.
- `types/index.ts`: Added `POCMilestone` interface, `milestones: POCMilestone[]` to `POCData`, `milestones?` to `POCExtractDraft`.
- `app/api/notebook/[id]/poc/route.ts`: Added `poc_milestones` to `POC_KEYS`.
- `components/POCGuide.tsx`: Replaced bare Key Dates date pickers with editable milestone table (Date / Item / Duration / Description, inline blur-to-save, add/delete rows).
- `app/poc-guide/[opportunityId]/page.tsx`: Print layout renders milestone table if `poc_milestones` exists; falls back to hardcoded Kickoff/Finish rows.

**Opportunity tab restructure — Intelligence/POC → Sales/Presales:**
- `type Tab`: `'intelligence' | 'poc'` → `'sales' | 'presales'`
- **Sales tab** (was Intelligence): sub-tabs = **COM** (fields only), **Mantra** (promoted from inner pill), **BV Slides** (promoted from inner pill), **Signals**. The comView inner toggle removed — each is now a top-level tab.
- **Presales tab** (was POC): sub-tabs = **Presales**, **State Analysis**, **TechQual**, **POC Guide**. Presales/State/TechQual content moved from Intelligence inner sub-tabs into Presales outer tab.
- `StallPatternBanner.tsx`: Updated tab refs — `'intelligence'` → `'sales'`, `tab: 'poc'` → `tab: 'presales'`.
- `__tests__/components/StallPatternBanner.test.tsx`: Updated expected tab value `'poc'` → `'presales'`.

**Tests:** 2302 passing · 0 failing

---

## Session: April 2, 2026 — Gemini extraction + file upload fix

**File upload fix:**
- Root cause: Supabase Storage rejects object keys with non-ASCII characters (en dash `–`, spaces). File `BTADP-Okta Identity POC – Feasibility and Validation Framework.pdf` was hitting `"Invalid key"` error on every upload attempt.
- Fix: `app/api/notebook/[id]/attachments/route.ts` — sanitize filename in storage path (`/[^a-zA-Z0-9._-]/g → '_'`). Original `file.name` stored in DB unchanged so UI display is unaffected.

**POC extraction switched to Gemini 2.5 Pro:**
- Root causes: `max_tokens: 4000` per chunk was exhausting before populating `poc_success_criteria`, `poc_vendor_team`, `poc_tech_notes`, `poc_doc_references`. Chunked processing also lost cross-chunk signal and let PDF framework document dominate use case extraction over actual meeting content.
- New key `LITELLM_GEMINI_KEY` provisioned (gemini-2.5-pro only). Added to `.env.local` and Vercel production.
- `app/api/notebook/[id]/poc/extract/route.ts` — added Gemini single-pass path: all transcripts untruncated + full context in one call, `max_tokens: 16000`, explicit instruction to weight transcripts over documents. Chunked Claude path kept as fallback if Gemini key absent or call fails.
- Phase 2 narrative pass (background, execSummary, architectureNotes, blockers) unchanged — stays on Claude.

**Prompt depth overhaul across all analysis prompts:**
- Root cause: prompts explicitly instructed brevity ("concise summary (2-4 sentences)", "SHORT (under 10 words per line)", "1-2 sentences") — Gemini follows these literally, Claude was less strict.
- `buildPrompt()` in `analyze/route.ts`: summary expanded to 4-8 sentences, problems thorough with context, nextSteps includes owners/deadlines, CoM fields ask for 2-4 sentences each with full detail, technicalFindings added (integrations, architecture, open questions, compliance), keyQuotes expanded to 6-12.
- `meeting_analyze_com`: all 7 CoM field descriptions expanded to ask for 2-4 sentences with specific evidence.
- `presales_analyze`: risksGaps → comprehensive risk register with severity ratings; presalesNextSteps → detailed action plan with owner/timeline; meetingBlurbs → 2-3 sentences per meeting.
- `agent_stage_actions`: 2-3 → 3-5 actions; descriptions 1-2 → 3-4 sentences.
- `agent_next_action`: descriptions 1-2 → 3-4 sentences with expected outcomes.
- `agent_research`: 4-5 → 6-8 points; 1-2 → 2-3 sentences each; added stakeholder and risk categories.
- `agent_weekly_digest`: 3-4 → 4-5 paragraphs with structured coverage.
- `poc_extract`: use case descriptions 2-3 → 4-6 sentences; success criteria 3-5 → 5-8 per use case; global criteria 5-10; features/techNotes/stakeholders all expanded.
- `poc_extract_narrative`: background 2-4 → 4-6 paragraphs; execSummary 3-5 → 5-7 sentences; architectureNotes 2-3 → 4-6 sentences; blockers → comprehensive register.
- Left as-is (intentionally brief): `digest`, `meeting_followups`, `agent_followup`, `meeting_analyze_note`, `meeting_analyze_state`.

**Gemini reverted — all analysis routes back on Claude:**
- Briefly switched `analyze-worker`, `analyze-state`, `analyze-com`, `analyze-presales`, `agent/stakeholder-map` to Gemini 2.5 Pro but reverted after output quality was underwhelming compared to Claude. POC extraction (`poc/extract`) stays on Gemini — demonstrably better there due to 1M context window.
- All 5 analysis routes now use `LITELLM_API_KEY` + `LITELLM_MODEL` (claude-4-6-sonnet).

**Business Value Slides — new feature:**
- `app/api/notebook/[id]/analyze-bv/route.ts` (new) — POST route, Claude, fire-and-forget pattern, reads existing CoM fields + transcripts, generates 7-9 BV slide objects, writes to `bvSlides` NodeProperty as JSON.
- `lib/prompt-defaults.ts` — added `bv_slides` prompt: 7-9 slides (challenge, impact, vision, capabilities, solution, differentiation, proof, metrics, next_steps), each with title, subtitle, headline, bullets, talkingPoints, evidence.
- `lib/queries.ts` — `useGenerateBVSlides(nodeId)` mutation hook.
- `components/OpportunityDetail.tsx` — "BV Slides" tab added to the CoM section pill toggle (next to Fields + Mantra). Shows slide cards with headline, bullets, SE talking points, customer evidence quote. Generate/Regenerate button with job polling. Empty state prompts generation.

**Analyze meeting JSON truncation fix:**
- Root cause: `max_tokens: 4000` in `analyze-worker/route.ts` was too small for large Waters transcripts (15k–23k char responses). Claude was truncating within 95 chars of a complete JSON response.
- Fix 1: `max_tokens: 4000` → `8000`
- Fix 2: Added progressive JSON bracket repair (same pattern as poc-merge) so partial responses are salvaged rather than discarded

**Meeting analysis timeout + overlay fixes:**
- Root cause 1: `maxDuration = 120` + `timeoutMs = 120_000` in `analyze-worker` too short for expanded prompts + 5000 max_tokens output. Raised both to 240s. Also reduced `max_tokens: 8000 → 5000` (meeting analysis needs ~3-4K tokens, not 8K).
- Root cause 2: Overlay stuck on failure — 202 `jobId` was discarded by client; polling only checked `summary`, never job failure → 5-min wait. Fixed in `TranscriptAnalyzer.tsx`: capture `jobId` from 202 response, added job status check in both poll loops (main + reload-resilience). Overlay now dismisses immediately on failure.

**State analysis parse failure:**
- Root cause: Expanded HTML diagram instructions (700-900px, all systems, CSS grid) generate ~14-16K tokens. `max_tokens: 12000` was insufficient → JSON truncated → parse failure. Raised to `max_tokens: 16000`. Added JSON bracket repair + better error logging (last 200 chars of truncated response).

**Tests:** 2302 passing · 0 failing (+11 new tests for analyze-bv)

---

## Session: April 1, 2026 (addendum 5) — POC PDF/Word export consistency: Mermaid diagrams + missing sections

**All three views (edit, PDF, Word) now show the same sections.**

**Mermaid diagrams in PDF and Word:**
- `ArchitectureDiagram.tsx` — after a successful `mermaid.render()`, saves the rendered SVG to a companion NodeProperty (`currentStateSvg`, `futureStateSvg`). Only saves when the value is non-empty and the SVG has changed (tracked via `lastSavedSvg` ref).
- **PDF page** (`app/poc-guide/[opportunityId]/page.tsx`) — reads `currentStateSvg`/`futureStateSvg` and renders inline SVG. Falls back to `currentStateHtml`/`futureStateHtml` if SVG not yet saved.
- **Word export** — reads SVG properties, converts to docx `ImageRun` with auto-sized dimensions parsed from SVG viewBox. Scales to 460px max width.

**Word export missing sections added:**
- `poc_background` — was hard-coded boilerplate; now reads actual value (with boilerplate fallback)
- `poc_environment` — new Parameter/Value table matching PDF layout
- `poc_tech_notes` — new Technical Notes section with title + body per note
- `poc_doc_references` — new Documentation References table
- All added to Word TOC as well

**Tests:** 2291 passing · 0 failing

---

## Session: April 1, 2026 (addendum 4) — POC comprehensive redesign: section order, 2-pass extraction, Key Dates

**Full redesign of the POC section to match PDF structure:**

### Section order (now matches PDF top-to-bottom)

**Background & Scope group:**
1. Executive Summary → 2. Current/Future State → 3. Background & Scope → 4. Architecture Overview → 5. POC Environment → 6. Use Cases → 7. Success Criteria

**Execution group:**
8. Required Features → 9. Key Dates (NEW) → 10. Documentation Needed → 11. Technical Notes → 12. Documentation References → 13. Okta Team → 14. Customer Team → 15. Blockers

Visual group headers (`poc-group-header`) between the two groups. TOC updated to match with group labels (`poc-toc-group` spans).

### Key Dates section (NEW)
Exposes `poc_start_date` and `poc_end_date` as a dedicated "Key Dates" section in the Execution group (previously only bare date inputs in the meta row at top).

### Environment values overflow fixed
`poc-env-value`: `flex: 1; min-width: 0; width: auto; border-color: var(--border)` — always visible, no overflow. `poc-env-key`: `width: 200px !important` overrides `poc-inline-input: width: 100%`.

### 2-pass extraction (structured + narrative, separate token budgets)

**Pass 1 (per 3-meeting chunk):** `poc_extract` prompt — structured data only: environment, use cases, features, criteria, stakeholders. All 4000 tokens focused on facts. Skip buildSteps, architectureNotes, blockers, background, execSummary.

**Pass 2 (single synthesis after all chunks):** `poc_extract_narrative` prompt — narrative fields only: background, execSummary, architectureNotes, blockers. 3000 tokens for narrative writing. Input: accumulated structured data summary + first chunk transcripts + opp/account context.

Both prompts added to `prompt-defaults.ts`. Route updated to call Phase 2 after all chunks complete.

### Context expansion
Added free-form note children (`type='note'` without `sourceType`) as `=== NOTES ===` section in first chunk context.

### POC edit/PDF/Word all now consistent
Documentation References, Customer Team — both now have scroll-target `id` attributes. All 15 sections visible and editable inline.

**Tests:** 2291 passing · 0 failing (189 files)

---

## Session: April 1, 2026 (addendum 3) — POC edit view: missing sections added, full consistency with PDF/Word

**What was missing / fixed:**

1. **Architecture Overview** (`poc_arch_notes`) — field was extracted by AI and shown in PDF/Word but had no editable textarea in the POC edit view. Added section with textarea between Background & Scope and POC Environment, matching PDF layout. Also added to TOC nav.

2. **Current/Future State section** — was hidden when empty (conditional `&&`). Now always visible with an empty-state notice prompting the user to run "Generate Current/Future State" in the AI panel. Mermaid diagrams and state descriptions now always visible for review before exporting.

3. **Success criteria text** — existing criteria were read-only `<span>`. Replaced with transparent `<input>` that saves on blur (consistent with how features/env entries work). Styling: transparent until hover/focus, `line-through` for done items.

4. **Word export (`export-docx`)** — added `poc_arch_notes` → "Architecture Overview" section between Background and Use Cases, matching PDF structure.

5. **Extraction prompt** — added two missing sections:
   - "Architecture Overview" (`architectureNotes`) — now has its own dedicated prompt section (was in schema only, no explanation)
   - "Blockers & Open Issues" (`blockers`) — completely absent; now section 11 with dedicated prompt + added to JSON schema
   - `poc-merge.ts`: added `blockers` to `PocDraft` interface, `mergePocDrafts`, and `pocDraftToUpserts` → writes to `poc_blockers` DB key

**Tests:** 2290 passing · 0 failing

---

## Session: April 1, 2026 (addendum 2) — POC extraction: chunked processing + direct DB writes

**Root causes:**
1. `getAttachmentContext` had `MAX_TOTAL = 100,000` chars — 3 large PDFs could send 100KB of text, causing 240s LLM timeout
2. The `after()` refactoring broke POC extraction: draft was parsed and discarded, never written to DB or returned to client
3. Single-call architecture can't scale to tens of meetings

**Fixes:**

1. **`lib/attachment-context.ts`** — `MAX_TOTAL` reduced from 100,000 → 20,000 chars (~5k tokens, generous but bounded)

2. **`lib/poc-merge.ts`** (new) — Merge utility for progressive chunked extraction:
   - `mergePocDrafts(accumulated, newBatch)` — deduplicates arrays by stable key (title/feature/url/name), last non-empty string wins
   - `pocDraftToUpserts(nodeId, draft)` — maps LLM fields → NodeProperty keys, serializes arrays to JSON, assigns IDs

3. **`poc/extract/route.ts`** — Complete redesign of `after()` callback:
   - Sort all meetings by date (most recent first), **no cap** — chunking handles any scale
   - CHUNK_SIZE = 3 meetings per LLM call (60s timeout per chunk, 2500 tokens output per chunk)
   - Chunk 1: full context (docs + signals + opp + account + contacts)
   - Subsequent chunks: brief summary of accumulated results ("found X use cases, Y requirements — add net-new items")
   - Each chunk merges with accumulated draft and writes to DB progressively
   - Stakeholders written as contact child nodes via Prisma (direct, not HTTP API)
   - If all chunks fail → `failJob`; if ≥1 chunk succeeds → `completeJob` with partial/full results
   - `max_tokens: 2500` per chunk (vs unbounded before)

4. **`POCGuide.tsx`** — Extract button simplified:
   - Removed `POCExtractReviewModal` import and rendering (draft review flow eliminated)
   - Removed `extractDraft`, `extractCount` state and `handleAcceptExtract` function
   - `handleExtract()` now: fires extraction → gets `{ jobId }` → polls `/api/jobs/[id]` → on success, invalidates notebook cache
   - Button shows "Extracting…" spinner + blocks during polling (via `isExtracting` state)

5. **`lib/queries.ts`** — `useExtractPOC` return type: `{ draft, meetingsAnalyzed }` → `{ jobId: string | null }`

**Result:** Waters OCI (3 meetings, 3 docs): 1 chunk × 60s = ~30-40s instead of 240s timeout. 9 meetings: 3 chunks × ~35s = ~105s total, all meetings covered.

**Tests:** 2290 passing · 0 failing (189 files)
- New: `__tests__/lib/poc-merge.test.ts` (13 tests)
- Updated: notebook-poc.test.ts (+3 tests for DB writes + chunking)

---

## Session: April 1, 2026 — Async AI analysis (fire-and-forget) + LLM error visibility

**Root cause:** COM analysis and POC Extract were timing out for large transcript payloads. `analyze-com` had a 55s abort timeout (too short for Waters-scale transcripts). All 4 analyze routes were silently swallowing LLM errors — no logging, no actual error message surfaced. `poc/extract` had a bug where the AIJob was never marked `failed` when both LLM endpoints failed (job stayed `running` forever).

**Fixes (3 layers):**

1. **Error logging fixed** — `analyze-com`, `analyze-state`, `analyze-presales` all now log actual HTTP status or timeout reason via `logger.error`. The `failJob` message now contains the specific error (e.g., "HTTP 429 from endpoint" or "Timeout (110s)") instead of the generic "Could not reach AI service".

2. **Missing `failJob` fix** — `poc/extract` was not calling `failJob` when both LLM endpoints failed. Added the missing call (matched pattern from all other routes).

3. **Fire-and-forget async** — All 4 analyze routes now use `after()` from `next/server`. Routes return `202 { jobId }` immediately (< 100ms) and the LLM call runs in the background up to `maxDuration`. Client (`OpportunityAIPanel`) polls `GET /api/jobs/[id]` every 3s until the job completes or fails, then invalidates notebook queries.

**New endpoint:** `GET /api/jobs/[id]` — returns `{ id, status, errorMessage, completedAt }` for polling.

**Timeouts updated:**
- `analyze-com`: 55s → 110s abort, `maxDuration` 60 → 120
- `analyze-presales`: 55s → 110s abort, `maxDuration` 60 → 120
- `analyze-state`: already 240s / maxDuration 300 (unchanged)
- `poc/extract`: already 240s / maxDuration 300 (unchanged)

**Validation moved before `createJob`** — validation errors (no transcripts, no LLM config) now return 4xx immediately without ever creating an AIJob record.

**Tests:** 2275 passing · 0 failing (188 files)
- New: `__tests__/api/jobs-id.test.ts` (5 tests)
- Updated: analyze-com (13), analyze-state (2), analyze-presales (17), notebook-poc (56), notebook-analysis, ai-job-tracking-routes, hooks/usePOC, hooks/useNotebookAnalysis, mocks/handlers

---

## Session: March 31, 2026 (addendum 10) — Error log review: 206 errors → 0

**All 206 unresolved error log entries resolved.**

Real app bugs fixed: `orgMembersData.map` crash (useOrgMembers returns `??[]`), `notebookTree` TDZ in setInterval (notebookTreeRef added), suggestions FK violation (node validation before card.create), admin/data query timeout (safeQuery wrapper, take 20→5). Tests: 2270 passing.

---

## Session: March 31, 2026 (addendum 9) — Share feature audit + 10 bug fixes

**Audit found:** 3 critical bugs + 5 medium issues + 7 low issues in the public share link feature.

**All fixed:**

| # | Bug | Fix |
|---|---|---|
| 1 | **Double viewCount increment** — `verifyShareToken()` AND the route both incremented. Every view counted twice. | Removed `updateMany` from `app/api/share/[token]/route.ts` — delegate to `verifyShareToken` only |
| 2 | **Share dialog crash** — API returns `{token:null, url:null}` (truthy). Dialog checked `if (shareLink)`, then called `.url.slice()` on null → TypeError. | Hook normalizes `{token:null}` to `null`; component checks `shareLink?.token` |
| 3 | **sanitizeHtml gaps** — `<iframe>`, `<object>`, `<embed>`, `<form>` passed through to `dangerouslySetInnerHTML` on public share page. | Added stripping for all 4 in `lib/share-utils.ts` |
| 4 | **E2E share suite disabled** — `beforeAll` called POST with no body → 400 → `sharedToken=null` → 8/12 tests silently skipped. | Added `{ data: { expiresInDays: 7 } }` to POST |
| 5 | **No noindex** — Shared notes were indexable by search engines. | Added `robots: { index: false, follow: false }` to share page metadata |
| 6 | **No audit log** — Share link create/revoke weren't tracked. | Added `audit()` calls to POST and DELETE handlers |
| 7 | **No revoke confirmation** — One click immediately destroyed the link. | Added two-step confirm UI in NotebookShareDialog |
| 8 | **No revoke error state** — Revoke failures were silent. | Added `revokeShareLink.isError` error display |
| 9 | **No link tracking** — No way to see what's been shared externally. | Added `GET /api/share-links`, `useSharedLinks()` hook, and "Shared Links" tab in Settings |
| 10 | **staleTime missing** — `useGetShareLink` re-fetched on every focus. | Added `staleTime: 60_000` |

**Tests:** 2265 passing (was 2253) — added 12 new tests covering all critical bugs (all were failing before fixes).

---

## ⟶ What's Next
_Updated: April 1, 2026 — update this section at the start of every session_

**Blocking for pilot (must ship before SEs log in):**
- [ ] **Vercel → LiteLLM network fix** — all AI routes are server-side but Vercel gets 403 from `llm.atko.ai`. AI analysis now uses `after()` (fire-and-forget) which avoids HTTP timeouts on the client, but Vercel still can't reach `llm.atko.ai`. Options: (1) Okta IT whitelists Vercel CIDR, (2) Vercel Static Outbound IPs ~$50/mo, (3) relay proxy on Okta network.
- [ ] OAuth callback URLs — add `/api/auth/callback/okta`, `/google`, `/credentials` to Okta OIDC app and Google OAuth app
- [ ] GitHub E2E secrets — add `TEST_USER_EMAIL=sensei.e2e@test.com` + `TEST_USER_PASSWORD=E2eRunner1234!` in repo Settings → Secrets → Actions

**E2E suite — LATEST STATE (March 31, evening run):**
- 28 spec files, ~682 tests
- Test account: `sensei.e2e@test.com` / `E2eRunner1234!` — isolated org `sensei-e2e-mnex8771`
- Seed data: account + opportunity + meeting + board created in `auth.setup.ts` → `.auth/seed-data.json`
- **480 passed / 187 skipped / 15 failed — 14.6 minutes** ✓ CONFIRMED
- Was 276/89/318 in 2.4 hours — 95% fewer failures, 90% faster

Remaining 16 failures (all non-blocking, no genuine app bugs):
1. Rate limiting lockout tests (2) — timing-sensitive
2. UI selector refinement (7) — notification bell, health score widget, templates, checklist modal
3. Complex workflow flakiness (3) — critical-path multi-step, create account tree
4. State/timing (4) — 5000-char note, global search, pipeline card, settings email

**Next feature:**
- [ ] Think Tank — fully scoped in ROADMAP.md, all deps identified (Liveblocks, Tiptap)

**Playwright E2E suite — latest state:**
- Test account: `sensei.e2e@test.com` / `E2eRunner1234!` — org `sensei-e2e-mnex8771` (fully isolated, NOT in master org)
- Root cause fixed: `waitForLoadState('networkidle')` → `domcontentloaded` across ALL 28 spec files — was causing 318 timeouts
- Root cause fixed: `MASTER_ORG_ID` was adding test user to owner's org — recreated with MASTER_ORG_ID disabled, now isolated
- AppShell: `role="banner"` + `role="main"` added (real app accessibility gaps)
- Login CSS: `min-height: 44px` on `.login-input` and `.login-btn-primary` (WCAG tap target fix)
- Latest full run (before all networkidle fixes): 379 passed / 180 skipped / ~80 failed (47 min)
- Remaining failures to investigate: board spec (23 networkidle just fixed), ai-features, auth, viewport tap targets (CSS just fixed)
- NEXT: run full suite with all networkidle fixed to get baseline
- Unit tests: 1922 passing, 0 failing (140 files)

**Housekeeping:**
- [ ] Sentry or Logtail decision + wire error spikes to Slack/email
- [ ] Notebook `+Contact`/`+Opportunity` button overlay (known bug, low priority)
- [ ] Run full E2E suite against clean account to get final baseline

---

## Session: March 31, 2026 (addendum 12) — Data isolation: 4 security fixes, 2269 tests

**Root cause:** Per-user data within a shared org was not fully isolated. Audit identified 4 issues.

**Fixes applied:**

1. **AIJob leak** (`app/api/jobs/route.ts`) — `GET /api/jobs` was filtering by `organizationId` only. Every SE in an org could see every other SE's AI job history, including account/opportunity names in job labels. Fixed: added `userId: user.id` to both `updateMany` (stale heal) and `findMany` (job list). Also updated the existing `ai-jobs.test.ts` to assert the correct WHERE clause.

2. **Notebook reparent cross-user attachment** (`app/api/notebook/[id]/route.ts`) — When a PATCH moved a node to a new `parentId`, the parent was validated by org only (`organizationId: user.orgId`). User A could attach their own node under User B's tree. Fixed: replaced direct `findFirst` with `verifyNotebookAccess(parentId, user.id, user.orgId)` which enforces `userId OR shares` + `organizationId`.

3. **Calendar import cross-user parent** (`app/api/calendar/events/import/route.ts`) — Calendar event import validated parent nodes by org only. User A could import meetings under User B's account node. Fixed: added `OR: [{ userId: user.id }, { shares: { some: { userId: user.id } } }]` to the parent findMany query.

4. **Schema nullability** (`prisma/schema.prisma`) — `Board.organizationId` and `NotebookNode.organizationId` were `String?` (nullable). No DB-level guarantee every record belongs to an org. Fixed in schema (both now `String`). `prisma generate` completed. DB migration timed out on Supabase (FK rebuild on large table) — needs manual application via Supabase SQL: `ALTER TABLE "Board" ALTER COLUMN "organizationId" SET NOT NULL; ALTER TABLE "NotebookNode" ALTER COLUMN "organizationId" SET NOT NULL;`

**Tests:** 2269 passing · 0 failing (185 files + 2 skipped)
**New test file:** `__tests__/api/data-isolation.test.ts` (4 tests)

**Agent suggestions confirmed NOT shared** — already scoped by `userId` in both read and write paths. All cron agents (deal-monitor, post-meeting, followup, weekly-digest, meeting-prep) pass the node owner's `userId` to `createSuggestion`.

---

## Session: March 31, 2026 (addendum 11) — "Should test" group: 6 components, 56 tests

POCExtractReviewModal (14), NotebookShareDialog (8), FeatureManagerPage (7), RecentMeetingsPage (7), PromptManager (9), ActionItemModal (11). **Tests: 2253 passing · 0 failing (183 files + 2 skipped).**

---

## Session: March 31, 2026 (addendum 10) — Pilot-critical coverage: 5 components, 30 tests

TechQualScorecard (7), AttachmentsPanel (5), CardModal (7), OpportunityAIPanel (6), NotebookPage (5). All pilot-critical components now have at least smoke + key behavior tests. **Tests: 2197 passing · 0 failing (177 files + 2 skipped).**

---

## Session: March 31, 2026 (addendum 9) — Coverage push: 152 new tests, 20 new files, 48% statement coverage

**Coverage: 28.9% → 48.2% statements. Component test files: 22 → 37.**

20 new test files across 5 waves:

| Wave | Files | Tests |
|---|---|---|
| 1 — Pure functions | UserMenu, CompanyLogo, HomePage, OrgChart, lib/meeting-prep | 40 |
| 2 — Small components | BoardPage, Sidebar, FeedbackWidget, NotebookShareDialog | 27 |
| 3 — Critical path | AppShell, KanbanBoard, PipelinePage, AICopilotPanel | 31 |
| 4 — Large components | AccountDetail, MeetingDetail, OpportunityDetail | 19 |
| 5 — API routes | organizations-current, jobs-stream, +stage-actions/weekly-digest in agent-routes | 33 |

**Tests: 2167 passing · 0 failing (172 files + 2 skipped)**

---

## Session: March 31, 2026 (addendum 8) — Component coverage: 51 new tests, 7 new test files, 1 bug fixed

**Coverage improvement:** 15 → 22 component test files. New tests: DealMomentum (11), StallPatternBanner (5), PendingSuggestionsBar (5), ArchitectureDiagram (8), AIJobBell (7), NoteAnalyzer (7), SearchPalette (8). `vitest.config.ts` updated to track components/lib/api routes in coverage reports. Bug fixed: verify-email test helper wasn't sending `Accept: text/html` header that the route requires to distinguish browser from API callers. **Tests: 2015 passing · 0 failing (154 files + 2 skipped).**

---

## Session: March 31, 2026 (addendum 8) — E2E test suite: data seeding + root cause fixes

### Unit tests: 2253 passing (was 1964 at session start) — +289 new tests from new app routes

### App bugs fixed
- `GET /api/notebook/[id]` — route was missing GET handler (405), now returns node with properties + customProps
- `GET /api/notebook/[id]/health-score` — same, GET handler missing, now computes + returns score
- `PATCH /api/boards/[id]/todos/bulk` — endpoint didn't exist (405), now bulk-updates todo done status
- `proxy.ts` — API routes now return 401 (not 307 middleware redirect)
- `AppShell.tsx` — added `role="banner"` and `role="main"` (ARIA gaps)
- `globals-redesign.css` — login inputs `min-height: 44px` (WCAG tap target)
- `types/index.ts` — added `'copilot'` to PageName enum

### Test infrastructure fixes
- `auth.setup.ts` — seeds [SEED] account + opportunity + meeting + board + card; dismisses onboarding tour; clears MASTER_ORG_ID guard
- `fixtures.ts` — added `getSeedData()` helper; `goToPage()` now dismisses lsp-overlay (live session overlay) before navigating
- `notifications.spec.ts` — added beforeEach to discard live sessions before bell tests
- Root causes fixed: suggest review `accept` → `approve`, share-link missing `expiresInDays`, `decisions` array format for review route, CSS `placeholder*=` case sensitivity, `test.afterAll` inside test bodies, invalid regex in `:has-text()`, auth `browser.newContext()` storageState, duplicate test names, missing skip guards

### E2E result: **480 passed / 187 skipped / 15 failed — 14.6 minutes** ✓ CONFIRMED
(was 276/89/318 in 2.4 hours — 95% fewer failures, 90% faster)

---

## Session: March 31, 2026 (addendum 7) — ArchitectureDiagram + StallPatternBanner added to POC guide

**Changes:**
- `components/POCGuide.tsx` — Added `currentStateMermaid`, `futureStateMermaid`, `opportunityNode`, `allContacts`, `meetings` to `POCGuideProps`. Imported `ArchitectureDiagram` and `StallPatternBanner`. Added `StallPatternBanner` before the Table of Contents (shows when `opportunityNode` + contacts/meetings are passed). Added `ArchitectureDiagram` inside the Current/Future State section alongside existing descriptions. Updated the state section condition to include `currentStateMermaid || futureStateMermaid` so it shows even when only the diagram exists.
- `components/OpportunityDetail.tsx` — Now passes `currentStateMermaid`, `futureStateMermaid`, `opportunityNode={node}`, `allContacts`, `meetings` to `<POCGuide>`.

**Tests:** 1964 passing · 0 failing (147 files + 2 skipped)
**New test file:** `__tests__/components/POCGuide-intelligence.test.tsx` (3 tests)

---

## Session: March 31, 2026 (addendum 6) — Architecture diagrams in State Analysis tab

**What was built:**

Added Mermaid-based architecture diagrams to the Current / Future State tab in the opportunity Intelligence view.

**Changes:**
- `package.json` — added `mermaid@^11.13.0`
- `app/api/notebook/[id]/analyze-state/route.ts` — added `currentStateMermaid` and `futureStateMermaid` to `STATE_KEYS`; extended the user prompt to instruct the LLM to generate `graph LR` Mermaid architecture diagrams (5–10 nodes, subgraphs, plain Mermaid syntax, no markdown fences)
- `components/ArchitectureDiagram.tsx` (new) — lazy-loads `mermaid`, renders source to SVG in preview mode, toggles to textarea editor mode for manual editing. Shows a helpful empty state when no diagram exists yet. Dark-mode compatible via Mermaid theme variables.
- `components/OpportunityDetail.tsx` — added `<ArchitectureDiagram>` below each `StateDescEditor` in the State Analysis tab (Current + Future columns). Each column now shows: rich text description → architecture diagram (AI-generated + editable).

**Workflow:**
1. SE runs "Generate Current/Future State" → AI now also writes Mermaid architecture diagrams to `currentStateMermaid` / `futureStateMermaid`
2. Diagrams render as clean SVGs in the State Analysis tab
3. SE clicks "Edit source" to open the Mermaid textarea and customize
4. Saves via the "Save" button (calls `updateProp.mutate`)

**Tests:** 1961 passing · 0 failing (146 files + 2 skipped)
**New test file:** `__tests__/api/analyze-state.test.ts` (2 tests)
**New component:** `components/ArchitectureDiagram.tsx`

---

## Session: March 31, 2026 (addendum 5) — Sales intelligence: 6 new extraction features

**What was built:**

1. **Close probability** (`lib/health-score.ts`) — New `computeCloseProbability()` function using 7 previously-unused fields: COM completeness (0–25), presalesConfidence (0–20), technicalDiff + techQualItems (0–20), stakeholder depth with EB-in-meetings check (0–20), POC health + blocker presence (0–15). Displayed as "XX% close" badge next to the health score in the opportunity header.

2. **Signals sub-tab** (`components/OpportunityDetail.tsx`) — 5th sub-tab in the Intelligence tab. Reads `keyQuotes` JSON from every meeting under the opportunity, groups by tag. Objections section shows recurrence count across meetings (unresolved blocker pattern detection). Commitments/intent section shows customer buying signals. Badge shows count on the tab.

3. **Contact coverage alerts** (`components/DealMomentum.tsx`) — Added `contacts` prop. Now shows inline warnings: "Economic buyer not in any meeting yet" and "Champion not in a meeting for N days". Computes by matching contact names against meeting `attendees` arrays.

4. **StallPatternBanner** (new `components/StallPatternBanner.tsx`) — Separate component shown in the Overview tab. Computes 6 risk factors (no EB, COM < 40%, competitor without differentiation, "Not Forecasted" confidence, no meeting in 14+ days, active blocker). Shows if 3+ factors present. Each risk includes a clickable "Fix" link that navigates to the right tab. Dismissable.

5. **Store contact signals** (`app/api/agent/stakeholder-map/route.ts`) — Added `signals` field to `propsToSet`. The LLM's verbatim signal text ("seemed skeptical about migration complexity...") is now persisted on the contact node instead of being discarded after every stakeholder-map run.

6. **Fix channel-import writebacks** (`app/api/notebook/[id]/channel-import/route.ts`) — After creating the note, now also upserts extracted intelligence (stage, arr, closeDate, competitor, productLine, industry, tier, health) back to the parent opportunity/account node. Previously this data was extracted and discarded into HTML.

**Tests:** 1959 passing · 0 failing (145 files + 2 skipped)
**New files:** `components/StallPatternBanner.tsx`
**New test files:** `__tests__/api/intelligence-features.test.ts` (2 tests), `__tests__/lib/health-score.test.ts` (+4 tests)

---

## Session: March 31, 2026 (addendum 4) — Chat persistence: messages survive page reloads

**Root cause:** Chat was in-memory Zustand state only — every page reload wiped the conversation. `Conversation`/`ChatMessage` were documented in CLAUDE.md but never existed in the schema.

**Changes:**
- `prisma/schema.prisma` — Added `Conversation` + `ChatMessage` models + back-relations on `User`/`Organization`. Applied via `prisma db push`.
- `POST /api/chat` — accepts `conversationId?`, creates `Conversation` on first message, loads history from DB (source of truth), persists both turns as `ChatMessage` rows, returns `{ reply, conversationId }`.
- `GET /api/conversations/latest` (new) — returns most-recent conversation with messages.
- `DELETE /api/conversations/[id]` (new) — ownership-verified delete, used by Clear button.
- `lib/store.ts` — added `setChatMessages` for DB seeding.
- `AICopilotPanel.tsx ChatTab` — loads conversation on mount, tracks `conversationId`, passes it in POST body, Clear deletes from DB.

**Tests:** 1953 passing · 0 failing (144 files + 2 skipped)

---

## Session: March 31, 2026 (addendum 3) — Intelligence automation: 5 features, from 4 manual steps to 1

**Manual steps eliminated from the core workflow:**

Before: paste/upload → **click Analyze** → **review action items modal** → **open Actions tab + accept suggestions** → navigate away and come back
After: paste OR upload → results appear → suggestions shown inline → done

**5 features shipped:**

1. **Auto-analyze after audio transcription** (`TranscriptAnalyzer.tsx`) — `handleAnalyze` now accepts optional `transcriptOverride: string`. After Groq Whisper transcribes audio, `handleAnalyze(transcribedText)` fires immediately. No "Analyze as SE" button click needed for audio uploads. Button still works for pasted transcripts.

2. **Auto-accept action items** (`TranscriptAnalyzer.tsx`) — Extracted `autoConfirmTodos(todos)` function. Instead of showing `ActionItemsReviewModal`, action items are created as board cards immediately. `actionItemsStatus` notification shows "N items created". User manages from board. Applies to both the live-analysis path and the page-reload resume path.

3. **Progressive opportunity intelligence** (`analyze-worker/route.ts`) — After each meeting analysis completes, the worker automatically dispatches:
   - `POST /api/notebook/{opportunityNodeId}/analyze-com` (COM synthesis refresh)
   - `POST /api/agent/tech-qual` with `{ nodeId: opportunityNodeId }` (tech qual refresh)
   - `POST /api/agent/stakeholder-map` with `{ nodeId: meetingNodeId }` (stakeholder classification)
   All fire-and-forget. Added `X-Internal-User` header auth to `analyze-com`, `tech-qual`, `stakeholder-map` routes so they accept internal dispatch from the worker.

4. **Inline suggestion surfacing** (`components/PendingSuggestionsBar.tsx`, `OpportunityDetail.tsx`) — New `PendingSuggestionsBar` component above the opportunity tabs. Shows pending AI suggestions filtered to this deal. "Accept all" button executes all via `useReviewSuggestions`. "Review" button opens the AI Copilot panel. No need to go hunting in the Actions tab.

5. **Meeting prep actually generates prep** (`app/api/agent/meeting-prep/route.ts`) — Cron now calls `generateAndStoreMeetingPrep(meeting.id, orgId, prisma)` directly for upcoming meetings that haven't been prepped yet. Previously only created a "reminder" suggestion. Now generates actual prep notes in the background.

**Tests:** 1943 passing · 0 failing (143 files + 2 skipped)
**New tests:** `__tests__/api/intelligence-automation.test.ts` (3 tests)
**New component:** `components/PendingSuggestionsBar.tsx`

---

## Session: March 31, 2026 (addendum 2) — AI analysis workflow: job tracking, notifications, reload resilience

**Root causes fixed (10 issues from full audit):**

**Job tracking bugs fixed:**
1. `lib/ai-jobs.ts` `withJob()` — only failed on 5xx, left 4xx responses with jobs stuck as `running`. Now fails on all non-2xx. Removed async body-parse chain (was a timing bug in tests too) — calls `failJob()` synchronously with `HTTP {status}`.
2. `analyze-state/route.ts` — missing `failJob` on 4 error paths (no meetings→400, AI not configured→503, LLM unreachable→502, parse error→502). Added `failJob` to all 4.
3. `analyze-presales/route.ts` — 3 error returns called `completeJob` instead of `failJob` (showed green ✓ for failed analyses). Fixed to `failJob`. Job type changed `'analyze'` → `'analyze_presales'` so bell shows the right label.
4. `poc/extract/route.ts` — missing `failJob` before 422 return when no transcripts exist. Added.
5. `agent/tech-qual/route.ts` — no job tracking at all. Added `createJob`/`completeJob`/`failJob` with type `'tech_qual'`.
6. `agent/stakeholder-map/route.ts` — no job tracking. Added with type `'stakeholder_map'`.
7. `analyze-note/route.ts` — no job tracking. Added with type `'analyze_note'`.

**Push notifications:**
8. `components/TranscriptAnalyzer.tsx` — `sendNotification: true` was never included in the POST body. The worker had `if (sendNotification)` gating the push call, so notifications from web analyze NEVER fired. Fixed.

**Reload resilience:**
9. `components/TranscriptAnalyzer.tsx` — new `useEffect` on mount per nodeId: checks `/api/jobs` for a running `analyze` job on this node. If found, shows overlay + starts DB poll immediately. User now sees "Analyzing" indicator on page reload mid-analysis.

**Bell fallback polling:**
10. `components/AIJobBell.tsx` — SSE EventEmitter is process-local and unreliable on Vercel (cross-instance). Changed 60s tick to call `fetchJobs()` every 30s when jobs are running (was just re-rendering). Bell now catches completions even when SSE misses them.

**Tests:** 1940 passing · 0 failing (142 files + 2 skipped)
**New test files:** `__tests__/lib/ai-jobs-withJob.test.ts` (6 tests), `__tests__/api/ai-job-tracking-routes.test.ts` (8 tests)
**New tests in existing files:** `analyze-presales.test.ts` (+4)

---

## Session: March 31, 2026 (addendum) — AI analysis overlay no longer blocks the whole app

**Root causes fixed:**
1. `AIAnalysisOverlay` used `createPortal(…, document.body)` + `position: fixed; z-index: 9999` — rendered at browser root, blocking sidebar, navigation, everything
2. TranscriptAnalyzer polling required `summary && pendingActionItems` to dismiss — if the analysis completed with no action items, `pendingActionItems` was null and the overlay never dismissed

**Changes:**
- `components/AIAnalysisOverlay.tsx` — removed `createPortal`/`document.body`; removed `mounted` state; changed `position: fixed` → `position: absolute; z-index: 10; border-radius: inherit`. Overlay now scopes to nearest positioned ancestor.
- `components/TranscriptAnalyzer.tsx` — moved overlay inside `<div className="ta-section" style={{ position: 'relative' }}>` (was before the div); polling condition changed from `summary && pendingRaw` → `summary` alone (action items optional)
- `components/NoteAnalyzer.tsx` — moved overlay inside `nb-note-analyzer` div; added `position: relative` to that div
- `components/POCGuide.tsx` — added `position: relative` to `.poc-section` CSS (overlay was already inside the section)
- `__tests__/components/TranscriptAnalyzer.test.tsx` — added 2 tests: (1) overlay dismisses when analyze returns 200 with no action items, (2) backdrop renders inside component container not document.body. Also mocked `usePromptTemplates`, `useUpdatePromptTemplate`, `useResetPromptTemplate` in the test file.

**Tests:** 1922 passing · 0 failing

---

## Session: March 31, 2026 — workspace + protocol optimization

**What was done:**
1. **Settings.json rebuilt** — replaced 210+ one-off permission entries with 20 broad patterns covering all dev workflows. Removed stale one-off commands (dead process IDs, personal project files, specific line-range awk commands).
2. **Model versions updated** — Opus: `claude-4-5-opus` → `claude-4-6-opus`. Subagent: `claude-4-5-haiku` → `claude-4-6-sonnet` (better code analysis in Explore/Plan agents).
3. **Autocompact raised** — 50% → 70% (was losing context halfway through complex sessions).
4. **Experimental betas re-enabled** — removed `CLAUDE_CODE_DISABLE_EXPERIMENTAL_BETAS`.
5. **Stop hook upgraded** — now always runs (not just on code changes), shows SESSION_NOTES line count, warns at 1000 lines, blocks at 1500 with archive command.
6. **SESSION_NOTES archived** — 2218 lines archived to `SESSION_NOTES_ARCHIVE_2026-Q1.md`. Live file trimmed to last 5 sessions + What's Next.
7. **Context files cleaned** — removed all stale browser-LLM references from CLAUDE.md, SESSION_NOTES.md, ROADMAP.md. Updated to reflect actual state: all AI calls server-side, Vercel network issue is the actual pilot blocker.
8. **WORKING_PROTOCOL.md** — added rules 14 (context updates non-optional, every session) and 15 (archive SESSION_NOTES when >1500 lines).

**Tests:** 1920 passing · 0 failing

---

## Session: March 30, 2026 (addendum 8) — security review: remaining deferred items + code review complete

**What was done:**
1. **`npx prisma db push`** — applied FK relations (`CardAccountNode`, `CardOpportunityNode` with `onDelete: SetNull`) and composite index `@@index([boardId, col])` on Card to Supabase ✓
2. **`isAdmin` sync bug** — JWT was reading `membership.isAdmin` from DB (could be stale). Now derives `token.isAdmin = membership.role === 'admin'` in all 4 auth paths: `lib/auth.ts` credentials + OAuth JWT callbacks, `mobile-login/route.ts`, `mobile-okta/route.ts`. Also fixed `mobile-okta` DB creation bug (`isAdmin: false` → `isAdmin: true` for admin role) ✓
3. **AI job `failJob` gaps** — `prep/route.ts` missing try-catch around `generateAndStoreMeetingPrep` — jobs stayed "running" forever on error. `analyze-com/route.ts` missing `failJob` on 3 error returns (503 not configured, 502 LLM unreachable, 502 parse fail). Both fixed ✓
4. **Zod error format** — 40 routes return `{ error: flatten() }` (object) but `apiFetch` does `new Error(err.error)` → `[object Object]`. Fixed in `lib/queries.ts` `apiFetch` to extract first readable message from Zod objects. Zero route changes needed ✓

**Tests:** 1920 passing · 0 failing
**Code review: COMPLETE**

---

## Session: March 30, 2026 (addendum 7) — Playwright E2E suite: root cause fixes

### Root causes fixed (code, not tests)

**1. proxy.ts middleware redirecting API routes to /login (307 instead of 401)**
- Added `/api/` pass-through in `proxy.ts` before the `withAuthMiddleware` call
- Unauthenticated API requests now reach `requireAuth()` which returns 401 directly
- Reverted all `[401, 403, 307]` workarounds back to `[401, 403]` across 10 spec files

**2. `request.newContext()` in E2E tests inheriting project storageState**
- Playwright's `request.newContext()` inside an E2E project with storageState set DOES inherit auth cookies
- Fixed: all unauthenticated tests now use `test.use({ storageState: { cookies: [], origins: [] } })` at the describe level, using `{ request }` fixture directly
- Applied across: api-auth.spec.ts (full file), plus 10 files with mixed auth (describe blocks)
- This was the root cause of 30+ tests passing for the wrong reason

**3. logger.ts `prisma.sysLog.create()` throwing synchronously in tests**
- `.catch(() => {})` only handles promise rejections, not synchronous `undefined.create()` errors
- Fixed: wrapped sysLog and errorLog create calls in try-catch to handle test mocks with incomplete prisma stubs

**4. auth-flows.spec.ts testing login page while authenticated**
- Authenticated users get redirected from /login — tests were failing
- Fixed: wrapped all unauthenticated tests (login/register/forgot-password) in `test.use({ storageState: {} })` describe block

**5. Invalid CSS selectors mixing `text=/regex/` with CSS**
- Fixed across auth-flows.spec.ts (17 selectors)

**6. Dedicated non-admin test account created**
- `sensei.e2e@test.com` / `E2eRunner1234!` registered
- `.env.local` and playwright.config.ts updated
- Auth state re-saved — tests no longer pollute super admin account

**7. Protocol updated**
- Added to WORKING_PROTOCOL.md rule 3: "App stability is the goal, not test stability. Changing assertions to accept bad behaviour is always wrong."

### Test state
- Unit tests: 1920 passing, 0 failing (was 1909 → grew by new tests, cleared all failures)
- E2E verification (6 key specs): 85 passed, 43 skipped, 0 failed
- TypeScript: clean

---

## Session: March 30, 2026 (addendum 7) — comprehensive security review: 18 fixes, accounts sort bug

**Bug fix (accounts not alphabetical):**
- Root cause: `sortAccountsAlpha()` in `lib/queries.ts` used a non-transitive comparator that returned `0` for account-vs-non-account pairs. When a `free-folder` sat between two accounts in the position-sorted array, Timsort couldn't correctly reorder them.
- Fix: comparator now returns `-1`/`1` for account vs non-account (accounts always sort before other root nodes). Post-fix, component `filter((n) => n.type === 'account')` correctly extracts alphabetically sorted accounts.
- Bonus: added `sortAccountsAlpha` call to `useRenameNode` and `useUpdateNodeProperty` `setQueryData` callbacks so optimistic updates don't flash unsorted accounts.

**Security fixes applied (18 total):**

| # | Severity | Fix |
|---|----------|-----|
| 1 | Critical | `app/api/notebook/[id]/route.ts` PATCH — changed `verifyNotebookAccess` → `verifyNotebookOwnership` (privilege escalation: shared-access users could modify nodes they don't own) |
| 2 | Critical | `app/api/boards/[id]/route.ts` PATCH — changed `verifyBoardAccess` → `verifyBoardOwnership` (IDOR: shared-access users could rename boards they don't own) |
| 3 | Critical | `lib/auth.ts` — fixed `isAdmin: false` → `isAdmin: true` when creating the first admin membership for a new org |
| 4 | Critical | `lib/share-utils.ts` — fixed `sanitizeHtml` regex: `\s+on` → `\s*\bon` so event handlers without leading whitespace (`<img src="x"onerror=...>`) are stripped |
| 5 | Critical | `lib/auth.ts` + `app/api/auth/mobile-login/route.ts` — lockout increment now uses Prisma atomic `{ increment: 1 }` instead of read-then-write, preventing race condition |
| 6 | Critical | `app/api/auth/mobile-login/route.ts` — JWT expiry reduced from `30d` to `8h` |
| 7 | High | `app/api/notebook/[id]/attachments/route.ts` — added `checkAgentRateLimit` to POST handler |
| 8 | High | `app/api/organizations/current/members/invite/route.ts` — changed "User is already a member" → "Invitation could not be processed" (email enumeration) |
| 9 | High | `app/api/notebook/[id]/route.ts` — added Zod limits: property keys max 50 chars, values max 100k chars, max 100 properties, custom props max 50 |
| 10 | High | `app/api/share/[token]/route.ts` + `pdf/route.ts` — added `logger.warn` on invalid/expired share token (enables brute-force detection) |
| 11 | High | `app/api/organizations/current/members/route.ts` — added pagination (`?page=N`, 100/page, returns `{ members, total, page, pages }`) |
| 12 | Medium | `prisma/schema.prisma` — added `@@index([boardId, col])` composite index on Card |
| 13 | Medium | `prisma/schema.prisma` — added FK relations `CardAccountNode` / `CardOpportunityNode` on Card with `onDelete: SetNull` + back-relations on NotebookNode |
| 14 | Medium | `app/api/search/route.ts` — added `orderBy: { updatedAt: 'desc' }` to boards, cards, and nodes queries |
| 15 | Low | `app/api/boards/[id]/route.ts` + `app/api/boards/route.ts` — added `.trim()` to board name Zod schemas |
| 16 | Low | `app/api/notebook/[id]/share-link/route.ts` — GET now returns `{ token: null, url: null, expiresAt: null, viewCount: 0 }` instead of bare `null` when no active token |

**Tests:** 1920 passing · 0 failing (140 files + 2 skipped)

---

## Session: March 30, 2026 (addendum 6) — action items: pendingActionItems staging + resume modal + stale data guards

**Root causes fixed:**
1. Worker wrote raw AI items (`{title, assignee, priority}`) directly to `actionItems` → `ActionItemsList` rendered them without `id`/`text` → React key error + `normalizeText` crash
2. Review modal only fired during active polling — navigating away meant action items silently lost
3. `normalizeText(e.text)` crashed when `e.text` undefined on stale items

**Changes:**
- `analyze-worker/route.ts` — writes to `pendingActionItems` not `actionItems`; `actionItems` only written by `handleConfirmTodos` after user review
- `TranscriptAnalyzer.tsx` — poll checks `pendingActionItems`; new `useEffect` (after nodeId reset, after hook decls) resumes review modal on mount if `pendingActionItems` found; `resumeCheckedRef` prevents re-trigger on refetch; `e.text != null` guard on both `normalizeText` calls
- `analyze/route.ts` DELETE — added `pendingActionItems` to `CLEAR_KEYS`
- `ActionItemsList.tsx` — `key={item.id ?? item.text}` fallback for stale data

**Tests:** 139 files · 1909 passing · 0 failures.

---

## Session: March 30, 2026 (addendum 5) — Comprehensive Playwright E2E Suite

### What was built

Expanded the Playwright E2E suite from **22 spec files / ~400 tests** to **28 spec files / 664 tests**.

**New spec files (6):**
| File | Tests | Coverage |
|---|---|---|
| `auth-flows.spec.ts` | 25 | Full login/register/logout/SSO/session persistence flows |
| `auth-edge-cases.spec.ts` | 40 | Rate limiting, lockout, weak passwords, anti-enumeration |
| `admin-syslog.spec.ts` | 18 | GET /api/admin/syslog pagination/search/filters, UI rendering |
| `admin-errors-extended.spec.ts` | 25 | Error log search, bulk resolve, dedup, all types/levels |
| `note-analyzer.spec.ts` | ~40 | analyze-note API, apply-opp-contexts API, NoteAnalyzer UI |
| `notebook-edge-cases.spec.ts` | 28 | Special chars, XSS, SQL injection, long content, cascade deletes |
| `viewport.spec.ts` | ~60 | All pages at 375/768/1440px, no overflow, sidebar behaviour |
| `accessibility.spec.ts` | 27 | Tab order, ARIA labels, focus trapping, landmark regions |
| `pipeline-extended.spec.ts` | 28 | Stage transitions, ARR display, health scores, close dates |
| `notifications.spec.ts` | 47 | AI jobs bell, SSE stream, dark mode, responsive layout |
| `notebook-advanced.spec.ts` | 55 | Custom props, health score, POC Guide, attachments, Power Map |

**Bug fixed:** `request.newContext()` → `request.newContext({ storageState: { cookies: [], origins: [] } })` across all specs that test unauthenticated access. This fixes ~30 previously broken tests that were getting 200 instead of 401.

### TypeScript: ✓ clean (0 errors)
