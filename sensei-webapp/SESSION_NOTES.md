# senSEi — Session Notes
_Last updated: March 18, 2026_

> **When starting a new session:** Read this file top to bottom to understand what was implemented and what still needs work.

---

## Session: March 18, 2026 — AI Job Tracker, EC2 Documentation, Protocol Updates

### Summary
Built real-time AI job tracking replacing NotificationBell. Documented EC2 architecture in CLAUDE.md. Updated WORKING_PROTOCOL.md with context file update rule and /compact rule. Fixed hardcoded API keys in ecosystem.config.js.

### What Changed

#### 1. AI Job Tracker (real-time notification bell)

**New `AIJob` Prisma model** — tracks every user-triggered AI analysis:
- Fields: id, organizationId, userId, type, label, status (running/completed/failed), nodeId, targetPage, errorMessage, createdAt, completedAt
- `prisma db push` applied — table live in production

**New files:**
- `lib/ai-jobs.ts` — `createJob`, `completeJob`, `failJob` helpers + `jobEmitter` (EventEmitter for SSE) + `withJob` wrapper
- `app/api/jobs/route.ts` — `GET /api/jobs` returns last 20 jobs, org-scoped
- `app/api/jobs/stream/route.ts` — SSE stream; broadcasts job updates to connected clients in real-time
- `components/AIJobBell.tsx` — replaces NotificationBell; SSE-connected; shows running spinner, status dots (green ✓ / red ✕), click-to-navigate

**AppShell:** `NotificationBell` → `AIJobBell`

**Routes wired (job tracking added):**
- `app/api/notebook/[id]/poc/extract/route.ts` — type: `poc_extract`
- `app/api/notebook/[id]/analyze/route.ts` — type: `analyze`
- `app/api/notebook/[id]/analyze-state/route.ts` — type: `analyze_state`
- `app/api/notebook/[id]/analyze-com/route.ts` — type: `analyze_com`
- `app/api/agent/next-action/route.ts` — type: `next_action`
- `app/api/agent/research/route.ts` — type: `research`
- `app/api/notebook/[id]/prep/route.ts` — type: `prep`

**Tests:** `__tests__/api/ai-jobs.test.ts` — 5 tests, all pass

**CSS animations added** to `globals-redesign.css`: `@keyframes spin`, `@keyframes pulse-dot`

#### 2. Security Fix
`ecosystem.config.js` — removed hardcoded API keys (LITELLM_API_KEY, GROQ_API_KEY, etc.). Secrets now loaded from `.env.local` via OS environment inherited by PM2. **Rotate the keys that were hardcoded.**

#### 3. Infrastructure Documentation
- `CLAUDE.md` — added full EC2 architecture section: instance type, Nginx/PM2/Certbot setup, deploy flow, Docker note, cron job instructions
- `WORKING_PROTOCOL.md` — added rule 8 (update context files every session) and rule 9 (/compact when context fills)

### Commits
| Description | Hash |
|---|---|
| feat: real-time AI job tracker + EC2 docs + protocol updates | `271289a` |
| harden: rate limiting, health check, silent failure fixes, email safety | `25bb856` |
| feat: live session inject + save-as-note endpoints | `efd0bf3` |

### Next Session — Where to Pick Up

**sensei-webapp MVP hardening — all 4 items shipped ✅ (`25bb856`)**
- Rate limiting: 200/hr per user on all 9 user-triggered AI routes
- Email safety: all Resend calls try/caught, auth routes never crash on email failure
- Health check: `GET /api/health` live
- Silent failures: next-action/research now return 503/502, failJob() called on errors

**Still needed on EC2 (config, not code):**
- Set `RESEND_API_KEY` + `EMAIL_FROM` for emails to work
- Set `AGENT_CRON_SECRET` + systemd timer for post-meeting agent

**Other Claude session:** Working on Think Tank + Share Link (Phase 1)

**EC2:** Docker available — could be used for containerised deployment if needed

**Key decisions made this session:**
- Internal Okta app → no GDPR data export/account deletion needed
- AI rate limits: generous (200/hr) since going through Okta LiteLLM
- SSE over WebSocket for job updates (simpler, works on single EC2 instance)
- Notification bell replaced entirely with AI job history

---

## Session: March 16, 2026 — Design Overhaul, AI Fixes, Mobile Sprint, Protocol

### Summary
Full design overhaul of the webapp (CSS consolidation, typography, dark mode surface depth), fixed critical AI suggestion bugs (race condition, SSE reconnect, silent failures), merged Okta chatbot into main chat, wired post-meeting agent to mobile recordings, ran sensei-mobile design sprint, set up Playwright for echo-backend, and locked in the shared working protocol.

---

### 1. CSS System Consolidation

**Problem:** `globals.css` and `globals-redesign.css` both defined `:root` with conflicting values (52px vs 56px header, cyan vs indigo accent, 4 font families loading). 378KB of CSS with two competing systems.

**Fix:**
- `globals.css` — removed duplicate `@import` and entire `:root` block. Now only base reset + layout classes.
- `globals-redesign.css` — single canonical token source. Removed Inter/Sora imports → Outfit + Plus Jakarta Sans only (~140KB font savings)
- Dark mode: full surface stack updated (`#08090E` → `#282C3F`, 10-12 luminance steps apart)
- Font-size sweep: `10px` → `11px`, `9px` → `10px` across both files
- Added `.icon-btn`, sidebar CSS classes, pipeline-glance classes

**Commits:** `efd58d8`

---

### 2. Typography

- Body: `13px` → `14px`
- Sidebar, header, nav labels: system sans-serif via `fonts.body`
- `live-session-btn` hardcoded colors → CSS variables

---

### 3. Chat Reliability

**Problem:** `/api/chat` route's Prisma context queries (notebook nodes + boards) had no try-catch. DB errors crashed the route, Next.js returned HTML 500, `res.json()` threw, user saw "Could not reach server."

**Fix:** Wrapped both queries in try-catch. Chat continues with degraded context (empty arrays) rather than crashing.

Also: frontend now parses JSON safely and shows HTTP status code on error instead of swallowing it.

---

### 4. Merged Okta Chatbot

Removed the separate "Okta" tab from AICopilotPanel. The main `/api/chat` route now injects `FULL_OKTA_KNOWLEDGE` when Okta keywords detected (`okta`, `oidc`, `saml`, `sso`, `scim`, etc.) — lean for non-Okta queries, expert for identity questions.

---

### 5. AI Suggestions Fix (Live Session Panel)

Three bugs fixed:
1. **Race condition** — `selectedIdRef.current` was null when `LiveTranscript`'s SSE fired first utterances (child effects run before parent effects). Fixed by passing `selectedId` directly as parameter.
2. **SSE reconnect** — on server restart, EventSource error handler closed and never reconnected. Fixed with exponential backoff (2s→15s).
3. **Silent failure** — route returned `{ suggestedResponses: [], ... }` with 200 on LLM failure, hiding "Click ↻" prompt. Fixed to return 422 so frontend knows to show the prompt.

---

### 6. Post-Meeting Agent → Mobile Task Creation

`app/api/live-sessions/[id]/finalize/route.ts` now fires `POST /api/notebook/{id}/analyze` in background after creating the meeting node. Chains: finalize → analyze (extracts summary/actionItems/nextSteps) → PATCH triggers post-meeting agent → AgentSuggestions appear in webapp Actions inbox.

---

### 7. sensei-mobile Design Sprint

- `constants/theme.ts` rebuilt: `fonts.body` (system sans), `fonts.mono` (timestamps only), desaturated status colors
- Tab bar: 80px → 60px
- Progress bars: 3px → 6px
- Speaker colors: hardcoded hex → `getSpeakerColor()` from theme (single source of truth)
- Hint tooltip: `#FFFFFF` → `colors.surfaceAlt`
- Glow shadows normalized: `shadowOpacity` capped at 0.25, `shadowRadius` capped at 10
- `SenseiLogo` component added to every header
- Auth fix: `inset: 0` invalid in React Native → `top/left/right/bottom: 0`
- Navigation: "Note" landing page fixed with `hasRedirected` ref + `unstable_settings`
- Board: `KanbanColumn` height fixed (no `flex: 1` scroll container issue), `Board` type updated with `todos`

---

### 8. Database Sync

`prisma db push` run — `Todo` table (and other schema additions) didn't exist in Supabase. Fixed `/api/boards` 500 errors. Both sensei-webapp and sensei-mobile now work correctly.

---

### 9. SetupWizard

`components/SetupWizard.tsx` — 3-step new user onboarding (role → first account → tour/skip). `setupWizardDone` in Zustand store. AppShell auto-sets `setupWizardDone = true` for existing users (boards.length > 0) to prevent wizard from blocking the board.

---

### 10. Working Protocol

`/Users/shantanu.govindjiwala/Documents/WORKING_PROTOCOL.md` — shared across all repos. Full persona pipeline defined (20 personas from SDET to UAT). Referenced in all four CLAUDE.md files. TDD protocol: write failing tests first, implementation second.

---

### Commits

| Description | Hash |
|---|---|
| Fix AI suggestions, design overhaul, chat reliability, merged chatbots | `efd58d8` |
| UX overhaul, AI quality improvements, prompt system | `55e3b69` (by owner) |

---

### Next Session — Where to Pick Up

**sensei-webapp:**
- Local changes committed (`55e3b69`) but Vercel deployment not tested
- `RESEND_API_KEY` empty — password reset emails don't send
- `AGENT_CRON_SECRET` needs to be set on Vercel for post-meeting agent

**sensei-mobile:**
- All changes committed (`ff37bec`, `e9b58dd`) — no remote configured yet
- Note landing page fix deployed, board fix deployed

**General:**
- Working protocol locked — all new work follows TDD + persona pipeline

---

## Session: March 10, 2026 — POC Guide Improvements

### What Changed

#### 1. POC Clear All Button
**Added comprehensive POC data deletion:**
- `DELETE` handler in `/app/api/notebook/[id]/poc/route.ts`
  - Atomically deletes all 17 POC property keys
  - Also deletes contact nodes that have `poc_role` (stakeholders added via Extract)
  - Regular contacts without `poc_role` are preserved
- `useClearPOC()` hook in `/lib/queries.ts`
- "🗑 Clear All" button in POCGuide header with detailed confirmation dialog
- Appears in both collapsible header and always-expanded controls strip
- State analysis (currentState/futureState) is NOT deleted (separate from POC data)

#### 2. Rich Current/Future State Diagrams
**Upgraded from 300px simple diagrams to 450-550px Okta-specific architecture diagrams:**

**Backend (`/app/api/notebook/[id]/analyze-state/route.ts`):**
- Extracts CoM fields from opportunity (identifiedPain, compellingEvent, pbo, metrics, etc.)
- Injects as `=== OPPORTUNITY CONTEXT ===` in LLM prompt
- Increased timeout from 90s → 240s
- Added `maxDuration: 300` for Next.js serverless
- Added error logging for parse failures

**Prompt (`/lib/prompt-defaults.ts`):**
- Rewrote `meeting_analyze_state` system prompt with Okta IAM architecture requirements
- Taller diagrams (450-550px vs 300px)
- Okta product names: OIE, Universal Directory, LCM, OIG, ODA, Workflows, Okta Verify, etc.
- Protocol labels: SAML 2.0, OIDC, OAuth 2.0, SCIM 2.0, RADIUS
- Pain points with ❌ icons, resolutions with ✅ icons
- Zero-trust principles referenced
- 4-6 sentence descriptions with specific details
- Simplified from initial complex 5-layer CSS Grid spec to achievable flexbox/grid with simple arrows

**UI (`/components/POCGuide.tsx` + `/components/OpportunityDetail.tsx`):**
- Changed layout from horizontal 2-column grid to vertical stack (`flex-direction: column`)
- Current State on top, Future State below with 24px gap
- Applied to both main POC section and OpportunityDetail overview

#### 3. Use Case Modal
**Moved use case editing to full-screen modal to reduce clutter:**

**New file: `/components/UseCaseModal.tsx`**
- Full modal following existing ActionItemsReviewModal pattern
- Modal backdrop with click-outside and ESC to close
- Header: title input, priority dropdown, close button
- Scrollable body with:
  - Description textarea
  - Success criteria list (inline add/delete)
  - Architecture notes textarea
  - Numbered build steps with:
    - Step title + content textarea
    - Callouts (info/warning/tip/ea) with colored left borders
    - Add/delete controls
- Footer: "Delete Use Case" danger button + Close button
- Deduplicated contacts on Extract by name (case-insensitive)

**Updated: `/components/POCGuide.tsx`**
- Replaced 200-line inline use case expansion with compact clickable card list
- Cards show: priority dot, title, description preview, step count, chevron
- Click card → opens modal
- Removed unused imports: `useUpdatePOCUseCase`, `useDeletePOCUseCase`, `POCBuildStep`, `POCCallout`, callout constants
- Added `editingUseCase` state and modal portal
- Contact deduplication in `handleAcceptExtract`: checks if contact with same name exists, updates instead of creating duplicate

**New CSS: `/app/globals-redesign.css`**
- Added `.uc-modal-*` styles for modal layout
- Added `.uc-sc-*`, `.uc-build-step`, `.uc-callout*`, `.uc-add-btn`, `.uc-delete-btn`, `.uc-step-*` styles
- Added `.poc-clear-btn:hover` and `:disabled` styles

#### 4. PDF/DOCX Export Updates
**Made checkboxes and status fields fillable in PDF export:**

**`/app/poc-guide/[opportunityId]/page.tsx`:**
- Changed state descriptions from plain text to `dangerouslySetInnerHTML` to render LLM-generated HTML (fixes raw `<p>` tags showing)
- Print CSS: checkboxes always render as empty `[ ]` boxes (hides checkmarks)
- Print CSS: hides progress bar ("X of Y criteria met")
- Print CSS: status field shows empty brackets `[ &nbsp;&nbsp;&nbsp; ]` instead of current value
- Screen CSS: status box hidden (only shows in print)
- Diagram overflow fixes:
  - Scale diagrams to 85% in screen view, 80% in print
  - Compensate width to fit container after scaling
  - Force `overflow: hidden`, `max-width: 100%` on all diagram elements
  - Override inline width styles from LLM-generated HTML
  - Force flex wrapping on flex containers

**DOCX export (`/app/api/notebook/[id]/poc/export-docx/route.ts`):**
- No changes needed — already vertical layout with sequential sections

#### 5. POC Extract Improvements
**Increased timeouts and added overlay:**

**`/app/api/notebook/[id]/poc/extract/route.ts`:**
- Increased abort timeout from 90s → 240s
- Added `maxDuration: 300` for Next.js serverless
- Extract prompt includes: transcripts, channel signals, attachments, opportunity context, account context, stakeholder data

**`/components/POCGuide.tsx`:**
- Imported `AIAnalysisOverlay`
- Added `{extractPOC.isPending && <AIAnalysisOverlay action="poc" />}`
- Shows full-screen overlay with animated orb and cycling phrases during extraction

#### 6. Bug Fixes

**State analysis deleted by Clear All:**
- Removed `currentStateDesc`, `futureStateDesc`, `currentStateHtml`, `futureStateHtml` from DELETE handler
- State analysis is independent from POC data

**Wrong query key causing stale UI:**
- Fixed `useClearPOC` to invalidate `['notebook']` instead of `['nodes']`
- This was causing vendor team and other data to show stale after operations

**Uncontrolled textarea stale values:**
- Added `key` prop to exec summary, background, and blockers textareas
- Forces React to re-mount when server value changes (e.g., after Extract or Clear All)

**Contact duplicates on every Extract:**
- Added deduplication logic in `handleAcceptExtract`
- Checks if contact with same name (case-insensitive) already exists
- If exists: updates `poc_role` and `title` instead of creating duplicate
- If not exists: creates new contact node

**Okta chat failing:**
- Fixed unescaped backticks in `/lib/okta-knowledge.ts` template literal
- Escaped 68 unescaped backticks that were breaking JSON parsing

---

## Architecture Notes

### POC Data Model
**Properties stored in `NodeProperty` table:**
- POC metadata: `poc_status`, `poc_health`, `poc_start_date`, `poc_end_date`, `poc_owner`
- POC content: `poc_exec_summary`, `poc_background`, `poc_blockers`, `poc_arch_notes`
- POC structured data (JSON): `poc_use_cases`, `poc_success_criteria`, `poc_required_features`, `poc_documentation`, `poc_vendor_team`, `poc_environment`, `poc_tech_notes`, `poc_doc_references`
- State analysis: `currentStateDesc`, `futureStateDesc`, `currentStateHtml`, `futureStateHtml` (separate from POC, survives Clear All)

**Child nodes:**
- Contacts with `poc_role` property are POC stakeholders (deleted by Clear All)
- Contacts without `poc_role` are regular contacts (preserved)

### Query Keys
- `['poc', nodeId]` — POC data for specific opportunity
- `['notebook']` — Full notebook tree (accounts, opportunities, meetings, contacts, notes)
- `['poc-versions', nodeId]` — POC version history
- `['suggestions', 'pending']` — Pending AI suggestions

### AI Synthesis Pipeline
1. **Synthesize COM** → `/api/notebook/:id/analyze-com` → extracts 9 CoM fields
2. **Generate State Analysis** → `/api/notebook/:id/analyze-state` → creates currentState/futureState diagrams
3. **Extract POC** → `/api/notebook/:id/poc/extract` → extracts use cases, criteria, features, stakeholders
4. **Suggest Next Actions** → `/api/agent/next-action` → recommends 3 deal advancement steps

All available via OpportunityAIPanel "✦ Synthesize All" button (runs sequentially).

### Modal Pattern
- Backdrop: fixed overlay with `z-index: 10000`, backdrop-filter blur
- Card: centered with `max-height: 90vh`, flex column
- Header: title + close button, `border-bottom`
- Scrollable body: `flex: 1`, `overflow-y: auto`
- Footer: action buttons, `border-top`
- ESC handler + click-outside-to-close
- Portal to `document.body` using existing DOM pattern (no createPortal needed)

---

## Known Issues / Tech Debt

1. **Diagram scaling in PDF** — Current scale(0.80) works for most diagrams but very wide diagrams (8+ horizontal steps) may still overflow slightly. Could reduce to scale(0.70) if needed.

2. **Extract timeout** — 240s should handle most cases, but extremely large opportunities (10+ meetings with long transcripts) might still timeout. Could increase to 300s or add prompt optimization.

3. **Use Case Modal mobile** — Not optimized for mobile (assumes desktop/tablet). Would need responsive max-width and adjusted padding for phone screens.

4. **State analysis prompt complexity** — LLM sometimes generates diagrams with inline widths that exceed page width. Current CSS overrides handle most cases but could benefit from explicit width constraints in the prompt itself.

---

## Key Files Modified (This Session)

### Backend
- `/app/api/notebook/[id]/poc/route.ts` — DELETE handler
- `/app/api/notebook/[id]/poc/extract/route.ts` — timeout + maxDuration
- `/app/api/notebook/[id]/analyze-state/route.ts` — CoM context extraction, timeout, error logging
- `/lib/prompt-defaults.ts` — rewrote `meeting_analyze_state` prompt
- `/lib/queries.ts` — added `useClearPOC` hook
- `/lib/okta-knowledge.ts` — escaped 68 backticks

### Frontend Components
- `/components/POCGuide.tsx` — compact card list, modal state, Clear All button, AIAnalysisOverlay, textarea key props, contact deduplication
- `/components/UseCaseModal.tsx` — NEW full modal for use case editing
- `/components/OpportunityDetail.tsx` — vertical state layout in overview tab

### Styling
- `/app/globals-redesign.css` — modal styles (`.uc-modal-*`, `.uc-sc-*`, etc.), clear button hover/disabled

### Exports
- `/app/poc-guide/[opportunityId]/page.tsx` — PDF export with fillable checkboxes/status, diagram scaling, HTML descriptions
- `/app/api/notebook/[id]/poc/export-docx/route.ts` — no changes (already correct)

---

## Testing Checklist

- [x] Clear All removes POC data but preserves state analysis
- [x] Clear All deletes contacts with poc_role but preserves regular contacts
- [x] Extract doesn't create duplicate contacts (deduplicates by name)
- [x] State analysis generates taller diagrams with Okta products
- [x] State analysis includes CoM context in prompt
- [x] Use case modal opens/closes on card click/ESC/backdrop
- [x] Use case modal saves changes on blur
- [x] PDF checkboxes render as empty boxes (fillable)
- [x] PDF status fields render as empty brackets (fillable)
- [x] PDF diagrams fit page width without horizontal scroll
- [x] Exec summary/background/blockers update after Extract
- [x] AIAnalysisOverlay shows during POC extraction
- [x] Okta chat works (backticks escaped)

---

## Next Steps / Future Improvements

### High Priority
- Test state analysis with real opportunity data to validate diagram quality
- Gather user feedback on use case modal UX
- Monitor Extract timeout success rate (may need further optimization)

### Medium Priority
- Add ability to edit state descriptions directly in UI (currently read-only)
- Add "Regenerate" button for state analysis to re-run without re-doing Extract
- Mobile responsive improvements for use case modal
- Add confirmation when closing use case modal with unsaved changes

### Low Priority
- Add drag-to-reorder for use cases in compact card list
- Add use case templates (common patterns for different Okta products)
- Export state diagrams as PNG for presentations
- Add diagram zoom/pan controls for complex architectures

---

## Environment

- Node: v18+
- Next.js: 15.1.4
- React: 19.0.0
- Database: PostgreSQL via Prisma
- Auth: NextAuth v4
- AI: LiteLLM proxy → Claude 4.6 Sonnet
- Deployment: Vercel

## Related Documentation

- `/CLAUDE.md` — Architecture overview for Claude Code
- `/README.md` — Project setup and dev commands
- `/ROADMAP.md` — Feature roadmap and planned improvements
- `/prisma/schema.prisma` — Database schema (14 models)
