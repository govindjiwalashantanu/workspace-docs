# senSEi — Roadmap

---

## ✅ Completed

### Code Review Cleanup (27 findings) — May 4, 2026
All 27 code review findings resolved across 5 batches. Magic number constants extracted to `lib/transcript-utils.ts` and `lib/health-score.ts`. Silent catch blocks now log via `logger.error`. `SpeechRecognition` type corrected + global declaration added. `Promise.allSettled().catch()` unreachable branch fixed; save status now set only on mutation success. `toMap()` extracted to `lib/agent-helpers.ts` (shared across 8 routes); `research` + `followup` + `next-action` + `stage-actions` migrated off manual fetch loops to shared `callLiteLLM()`. 7 test files updated for new mock exports; 4 assertions updated for changed 503→202/200 behavior. 2979 passing, TypeScript clean.

### Product Knowledge Bases — All 7 Okta Products — April 12, 2026
`lib/poc-product-guides.ts` — comprehensive reference per product: component tables, use case patterns, discovery questions, Mermaid class mappings, honest capability assessment. Products: WIC, LCM, OCI, CIC/Auth0, PAM, ODA, OIG. Injected into every `poc/generate` call as PRODUCT KNOWLEDGE REFERENCE — AI uses accurate component names and realistic flows. `architectureDiagram` field added to PocDraft — AI generates Mermaid flowchart of customer identity architecture, rendered in the export page. Full Mermaid standards (7 color classes, anti-overlap rules, max 14 nodes) embedded in poc_generate system prompt. Skill reference files saved to `.claude/skills/okta/references/`. 2572 passing.

### POC Guide Rework — AI Generation + Product Line Templates — April 12, 2026
Complete rework of the POC section. **7 product line templates** (WIC, CIC, OCI, OIG, PAM, ODA, LCM) each with use cases, success criteria, features, environment checklist. `POST /api/notebook/[id]/poc/generate` generates a complete customer-tailored guide from template + CoM fields + transcripts — Gemini 2.5-pro first, Claude fallback. All 11 POC sections filled (exec summary, background, arch notes, blockers, environment, use cases, criteria, milestones, features, docs, vendor team). Empty state shows product line cards when no POC started. ✦ Regenerate Guide button in active state. Locked fields respected. poc_generate system prompt requires AI to replace all generic text with customer-specific content. 2572 passing.

### Schema Hardening (15-issue review) — April 15, 2026
Full schema audit of all 29 models. Applied: dropped `OrganizationMember.isAdmin` (redundant column), `NodeCustomProp @@unique`, `Board→Organization CASCADE`, `Contact name index`, `Feature userId index`. Code: Conversation find-or-create-latest (stops unbounded duplication), OIDC initial-setup secret validation, SysLog 30-day TTL cleanup cron. Deferred (data migrations): `dueDate` String→DateTime, `ReleaseNote` features/fixes String→Json, `Contact` NULL email partial index, `AgentSuggestion`/`MeetingActionItem` FK relations. 2572 passing, TypeScript clean.

### Database Schema Viewer (ER Diagram) — April 15, 2026
New `Admin → Database` page rendering a live Mermaid `erDiagram` from `information_schema` at runtime. Shows all tables, columns with PG→Mermaid type mapping, PK/FK labels, and FK relationship arrows. Search filter narrows to matching tables + 1-hop FK neighbors with client-side re-render. Superadmin-only. `lib/database-schema.ts` is a shared server+client utility. 17 new tests. 2579 passing.

### AI Analysis Overlay Rework Parts E + F — April 11, 2026
**Part E:** Results diff panel fades in after analysis — green card showing elapsed time, summary updated, action item count, opportunity field changes. Built in `processResult` before status=success. **Part F:** Error categorization in `TranscriptAnalyzer`: 429 → rate limit message with wait hint; 502/503/network → VPN check message; parse/JSON error → silent auto-retry once then human-readable message (uses `autoRetriedRef` to avoid stale closure loop). 2385 passing.

### AI Analysis Overlay Rework Parts B + C + D — April 11, 2026
**Part B:** `AIAnalysisOverlay` non-blocking — full-screen for 3s then collapses to fixed bottom-right mini-panel with title + stage + live elapsed timer. Users can keep working while analysis runs. **Part C:** Pre-analysis warning banners: amber notice for long transcripts (>15k chars) with estimated time, blue notice if already analyzed. **Part D:** Action items now always show `ActionItemsReviewModal` with 10-second auto-accept countdown instead of silent auto-confirm. User can click "Review" to stop countdown and make individual decisions. 2369 passing.

### AI Analysis Overlay Rework Part A — Full Transcript, No Timeouts — April 11, 2026
Removed all transcript truncation (`smartTruncate`, `capTranscripts`) and `AbortController` timeouts from all AI routes. Full transcript context sent to LLM every time — Claude's 200K window makes truncation unnecessary and it was silently degrading analysis quality. Timeouts removed because ECS Fargate has no function timeout limit (unlike Vercel's 60s). `analyze-worker` `maxDuration` raised 240→600. Affects: `analyze-worker`, `analyze-bv`, `analyze-com`, `analyze-state`, `analyze-presales`, `litellm-client`. 2365 passing.

### Patent IDF — April 8–11, 2026
23-innovation Invention Disclosure Form complete (`PATENT_IDF.md`). All sections filled including dates, public disclosure, prior art (verified — all Gemini patent numbers confirmed hallucinated, competitive product analysis accurate), defensibility tier analysis, draft claim language for Innovations 3 + 8, two-application filing strategy. Supporting files: `PATENT_PRIOR_ART_SEARCH.md`, `PATENT_PRIOR_ART_SEARCH_V2.md`, `PATENT_DISCLOSURE.md`. Ready to route to Okta IP counsel for provisional filing.

### Patent Demo Video — April 8, 2026
2:31 Remotion video (`public/demos/patent-demo.mp4`, 18MB) covering Innovations 1–4. TTS audio via LiteLLM, paragraph timings silence-detected with ffprobe. Accessible at `/admin/patent-demo` (superadmin only) with video player, download link, and 4 innovation summary cards.

### GitHub Migration to atko-presales/sensei — April 10, 2026
Repo created at `atko-presales/sensei` via Terminus. Remote added as `okta`. `public/demos/*.mp4` (206MB) removed from git + gitignored. Push pending OCM setup.

### Sidebar Z-Index Fix — April 10, 2026
`position: relative; z-index: 10` added to `.sidebar-wrap` in `globals-redesign.css`. Content area stacking context was capturing pointer events over sidebar nav items on complex admin pages.

### Seed Demo Data (Waters + BMC) — April 7, 2026
`POST /api/seed-demo-data` seeds two real deal examples (Waters OCI + Boston Medical Center - BMC) with every AI-generated field pre-populated — transcripts, CoM + evidence, BV Slides, State Analysis (HTML + Mermaid), TechQual, presales, health score, meeting summaries/problems/nextSteps/follow-ups/prep/SFDC drafts, accountSnapshot, trackingItems. 14 board cards across all 5 columns. After seed: navigates to first opportunity + fires onboarding tour. `DELETE` removes all tagged nodes. 2319 passing.

### Demo Library + Liquid Glass UI — April 5, 2026
`app/demos/page.tsx` rewritten with liquid glass design (animated orbs, backdrop-filter blur, dark theme). 9 acts rendered and stitched in correct narrative order as `01-the-pitch.mp4` → `09-the-business-case.mp4` + `sensei-full-demo.mp4` (13:57). Audio/script files renamed to match narrative order throughout Remotion codebase.

### Executive Meeting Deck — April 7, 2026
`app/execmeeting2/page.tsx` — public 9-slide executive deck with embedded full demo video, integrations roadmap (Salesforce/Gong/Zoom/Calendar/Slack/MCP), FY27 alignment, $3.3M business case, 30-SE pilot ask. Deployed at `/execmeeting2`. Used for Joel Hanson (VP Presales) briefing.

### AI JSON Parsing Fix — April 6, 2026
`parseLiteLLMJson` in `lib/litellm-client.ts` replaced greedy regex with balanced-brace walker — fixes "Failed to parse AI response" errors when Claude appends trailing text after closing JSON brace. `analyze-note` now falls back to `properties.transcript` when `node.content` is sparse.

### Live Okta OIDC Config (No Restart) — April 4, 2026
`lib/auth-config-cache.ts` — 60s TTL cache for Okta OIDC settings. `buildAuthOptions()` factory in `lib/auth.ts` fetches live config from DB on each auth event. Admin → Identity Provider page allows switching Okta tenant without code change or server restart. Source badge shows DB vs env var fallback.

### Architecture Diagrams in State Analysis — March 31, 2026
Added Mermaid-based architecture diagram section to the Current / Future State tab. AI now generates `graph LR` Mermaid diagrams alongside the existing text descriptions. Rendered as clean SVG with a toggle to "Edit source" for manual customization. `ArchitectureDiagram` component lazy-loads mermaid, handles syntax errors gracefully, and saves via `updateProp`. 1961 passing.

### Sales Intelligence Layer — March 31, 2026
6 new intelligence extractions from existing data. `computeCloseProbability()` combines 7 previously-unused fields into a 0–100% probability score shown in the opportunity header. Signals sub-tab in Intelligence shows objections (with recurrence count) and buying intent from `keyQuotes`. DealMomentum now shows EB/champion coverage gaps. StallPatternBanner detects 6-factor stall patterns. Contact `signals` field now persisted from stakeholder-map (was being discarded). Channel-import now writes extracted intelligence back to parent node properties (was discarding structured data into HTML). 1959 passing.

### Chat History Persistence — March 31, 2026
Chat messages now survive page reloads and work across devices. Added `Conversation` + `ChatMessage` models to schema (applied via prisma db push). `POST /api/chat` now creates/continues conversations and persists both turns to DB. New `GET /api/conversations/latest` seeds the chat UI on mount. New `DELETE /api/conversations/[id]` backs the Clear button. History is now loaded from DB server-side, not trusted from the client. 1953 passing.

### Intelligence Automation — March 31, 2026
Reduced the meeting-to-insights workflow from 4 manual steps to 1. Auto-analyze fires after audio transcription (no button click). Action items auto-create board cards (no blocking modal). Every meeting analysis now automatically re-synthesizes COM, tech-qual, and stakeholder map for the parent opportunity (progressive intelligence). Inline `PendingSuggestionsBar` shows pending AI suggestions on the opportunity page with one-click "Accept all". Meeting prep cron now actually generates prep notes (was previously just a nudge reminder). X-Internal-User header auth added to analyze-com, tech-qual, stakeholder-map for secure internal dispatch. 1943 passing.

### AI Analysis Workflow Overhaul — March 31, 2026
Full audit and fix of the AI analysis pipeline. 10 root causes fixed: `withJob()` now fails on all non-2xx; `analyze-state` missing 4 failJob calls; `analyze-presales` called completeJob on errors + wrong job type (`analyze_presales` now); `poc/extract` missing failJob on 422; `tech-qual`, `stakeholder-map`, `analyze-note` had zero job tracking (all added). Push notifications from web analyze silently suppressed — `sendNotification: true` now sent. Reload resilience added to TranscriptAnalyzer. AIJobBell polls every 30s when jobs are running. Overlay scoped to section. 18 new tests. 1940 passing.

### Comprehensive Security + Code Quality Review — March 30, 2026
Full security audit of the webapp. 18 issues fixed across critical/high/medium/low severity. Key fixes: notebook PATCH privilege escalation (shared-access users could modify nodes they don't own), board rename IDOR, `isAdmin` JWT always derived from `role` (not DB field), `sanitizeHtml` regex bypass for event handlers without leading whitespace, account lockout now uses atomic Prisma `increment` (race condition), mobile JWT reduced from 30d to 8h, attachment upload rate limit added, email enumeration in invite endpoint closed, property size/count limits added (keys 50 chars, values 100k chars, max 100 properties), share token invalid attempts now logged, members list paginated (100/page), composite index `(boardId, col)` on Card, FK relations for `Card.accountNodeId`/`opportunityNodeId` with `onDelete: SetNull`, search results ordered by `updatedAt`, AI job `failJob` gaps fixed in `prep` and `analyze-com` routes, Zod error format normalized in `apiFetch`. Schema changes applied to Supabase via `prisma db push`. 1920 tests passing.

### Proper SysLog — March 30, 2026
New `SysLog` DB table (append-only, 30-day TTL). `logger.info/warn/error` all persist to SysLog; `warn/error` also persist to ErrorLog. `source` field on every entry. `GET /api/admin/syslog` route with search, level, source, time-range filters. Syslog admin page rebuilt: level/source/time-range chips, read-only expand panel. Instrumentation added to auth routes (register, reset-password success), 9 agent/cron routes (start + completion). 1904 tests passing.

### Multi-Opp Context Extraction + Comprehensive Logging + Syslog — March 30, 2026
`analyze-note` now runs a second LLM pass to detect which opportunities are discussed in sync call notes and extract relevant context per opp. New `apply-opp-contexts` route creates "additional context" note children under each accepted opp. `NoteAnalyzer` renders the review/accept UI. Auth routes now log warn/error on bad credentials and unexpected failures. All 15 admin routes wrapped with try/catch + `logger.error`. `audit()` added to member deletion. `GET /api/errors` gains `search` param. 1890 tests passing.

### AI Context Improvements — March 26, 2026
Audited all 13 AI-calling routes and fixed the highest-impact context gaps. `analyze` now fetches attachments from both the meeting node and its linked opportunity. `analyze-note` was operating with zero context — now includes deal context so action items are assigned to the right people. `followups` upgraded from basic `getDealContext` to `getRichDealContext` (full COM fields + contacts + channel signals) plus attachment context. `next-action` adds uploaded document context and a health score signal that flags RED/YELLOW deals for re-engagement. `meeting-prep` now includes `actionItems` from sibling meetings (not just summary + problems) and attachment context from the opportunity. New `getRichDealContext()` in `lib/deal-context.ts` as the single rich context builder for action-oriented routes.

### CSV + Excel File Upload — March 26, 2026
Document uploads now accept `.csv`, `.xlsx`, `.xls` in addition to PDF, Word, HTML. Text extraction produces an LLM-readable pipe-separated table (500-row cap, 60 KB limit). Excel workbooks render as multiple named sheets. Supabase MIME allowlist bypassed gracefully via `application/octet-stream` fallback on upload; real MIME type preserved in DB for viewer/download.

### Comprehensive Test Coverage — March 26, 2026
Grew from 89 test files / 982 tests to **137 test files / 1876 tests** across sensei-webapp. Added 65+ MSW handlers covering all API domains. New test files: 11 lib utilities, 25+ React Query hooks (mutations and queries), 21 API route files. API route coverage: ~55/163 routes (34% → ~78% for implemented routes). Hook coverage: 14/95 → 92/95 (97%).

### Error Log Cleanup — March 26, 2026
All 70 unresolved errors cleared. `[stage-actions] LLM JSON parse error` fixed: bare `JSON.parse` wrapped in inner try/catch, falls back to `STAGE_FALLBACK` silently. 63 infrastructure errors (LLM 403/timeout on Vercel) marked as known Vercel→Okta network limitation. 5 stale client bugs already fixed in code marked resolved.

### Recording Auto-Pause Bug Fixed (sensei-mobile) — March 26, 2026
Recording was pausing after every utterance (every 12-second chunk rotation). Root cause: `rotateChunk()` in `openai-realtime.ts` did not remove the old recorder's `statusListener` before `rec.stop()`. After the `finally` block reset `intentionallyStopping = false`, the old recorder's final `recordingStatusUpdate(isFinished: true)` event arrived, passing all guards and triggering `handleInterruption()` → `onInterrupted()` → UI showed "paused" after every utterance. Fix: 2-line addition at top of `rotateChunk()` to remove and null the listener before stopping. Full test suite: 524 passing.

### Playwright E2E — Full Flow Coverage — March 26, 2026
Added 6 new spec files covering all previously untested flows: post-meeting agent pipeline (all 12 agents), todos + checklists, admin management (members/releases/audit/errors/feedback), auth edge cases (mobile JWT, lockout, reset, rate limiting), notebook advanced features (power map, custom props, POC snapshots, attachments, meeting templates), notifications + dark mode + responsive layout. 69/149 new tests passing. Known failure pattern: `request.newContext()` inherits Playwright auth in e2e project — fix is to use `browser().newContext()` for unauthenticated test contexts.

### Kanban Drag-Drop Fix (Today/Blocked columns) — March 25, 2026
Dragging cards to Today and Blocked columns was silently failing. Root cause: `closestCorners` returned adjacent card IDs as geometrically closer than the empty column's small droppable zone. Fix: (1) moved `setNodeRef` to the full `kanban-col` div, (2) replaced `closestCorners` with `kanbanCollisionDetection` — a custom `pointerWithin`-first algorithm that snaps to the column when the pointer is inside it but over no card. Regression tests added for all 5 column drag targets.

### Comprehensive Playwright E2E Suite — March 25, 2026
Grew the E2E suite from 94 tests to ~290 tests. New files: `notebook.spec.ts` (40 tests, full CRUD + API), `ai-features.spec.ts` (20 tests), `live-session.spec.ts` (18 tests, full session API coverage), `share.spec.ts` (15 tests). Expanded board/pipeline/critical-path specs with API tests, drag-drop for all 5 columns, card field validation, board management, and session persistence critical path. Fixed auth setup timeout (replaced `networkidle` with element selector wait). Auth works: 14/17 auth tests pass; API tests: 51/72 pass (18 failures are "unauthenticated" tests that need `request.newContext()` fix).

### Notebook Tree Sync Fix — March 25, 2026
Fixed "tree doesn't update without reload" across all create/delete/reparent mutations. Root cause: `invalidateQueries` only marks queries stale — with `staleTime: 30min` and no active observer (after navigation), the cache was considered fresh and never refetched. Fix: switched to `refetchQueries({ type: 'all' })` in `useAddTopLevelAccount`, `useAddFreeFolder`, `useAddChildNode`, `useDeleteNode`, `useReparentNode`. Also added missing `onSettled` to `useUpdateNodeContent` so content saves sync the tree. 6 new tests.

### Account Display Name Bug Fixed — March 25, 2026
Renaming an account in the tree didn't update the main view header. Root cause: header rendered `p.company || node.title` — `p.company` silently overrode `node.title`. Fix: `node.title` is now the single source of truth via `getAccountDisplayName()` in `lib/account-utils.ts`. Removed the `p.company` priority in `AccountDetail.tsx` and `NotebookPage.tsx`. 3 new tests.

### Schema Index Optimisations — March 25, 2026
Added missing indexes: `NotebookNode.parentId` (every tree load), `Card.col`, `Card.accountNodeId`, `Card.opportunityNodeId`, `AIJob.userId`, `LiveSession.userId`. Removed redundant indexes: `ShareToken.token` (covered by `@unique`), duplicate `ErrorLog.createdAt`, duplicate `Feedback.createdAt`. DB backup taken before changes.

### Transcription Fixed (Groq) — March 23, 2026
Switched `/api/transcribe` and `/api/notebook/transcribe` from LiteLLM to Groq (`whisper-large-v3-turbo`). Root cause: LiteLLM blocks non-Okta-network IPs server-side — Vercel gets 403. Groq is publicly accessible. Also restored schema fields accidentally removed in chat rollback: `AuditLog`, `User.notificationPrefs`, `Feature.userId`.

### ~~Chat History + AI Artifact Generation~~ — Rolled back March 23, 2026
Built and then reverted: persistent Conversation/ChatMessage DB models, conversation sidebar, markdown rendering, copy button, raised token limits. Reverted due to product direction change. May be re-scoped later.

---

### Mobile Dark Mode — March 22, 2026
Full dark mode for sensei-mobile. Follows system preference by default; Light/System/Dark toggle in Profile → Display. `darkColors` palette added to `constants/theme.ts`. `lib/store/ui-store.ts` persists preference. `lib/hooks/useColors.ts` + `useIsDark()` hooks. 41 files migrated from static import to `useColors()`. Recording screens stay intentionally dark. 13 new tests.

---

### Full Security + Code Quality Audit — March 22, 2026
**SEC-1:** Cross-org admin modification closed — `organizationMember.updateMany` now scoped to `admin.orgId`. **SEC-2:** CORS wildcard removed from `proxy.ts` — unknown origins get no ACAO header. **QUAL-1:** `lib/com-constants.ts` created as single source for COM_KEYS/COM_LABELS — 10 duplicate definitions removed. **QUAL-2:** 4 unnecessary `!important` rules removed from CSS. Dead code deleted (`api-wrapper.ts`, `api-errors.ts`). 14 new security/regression tests.

---

### AI Query Audit Logging + Prompt Guard — March 22, 2026
Every user-triggered AI call (12 routes) logs to the audit trail as `ai.{agent}`. Suspicious user input blocked by `lib/ai-guard.ts` before reaching LLM — detects jailbreak/injection, off-topic content, excessive length. Flagged requests logged as `ai.flagged` with reason + snippet. Admin audit log adds AI Only / Flagged quick filters. 20 new tests.

---

### CoM + Presales List Rendering — March 21, 2026
COM fields render as proper numbered/bulleted lists (fully expanded, no truncation). Evidence quotes show as styled blockquotes. Presales text fields (Risks, Next Steps, Notes, SE Manager Notes) use click-to-edit pattern — formatted list view by default. `lib/com-parser.ts` with `parseListItems()`. 11 tests.

---

### CoM VBC Restructure — March 21, 2026
Replaced 9-field generic CoM with Okta's 7-element Force Management Mantra: Challenges → PBO → Required Capabilities → Metrics → How We Do It → How We Do It Better → Proof Points. Backward-compat legacyKey fallback preserves existing data. Mantra view assembles fields into the conversational script. Live coaching now detects VBC elements (challenge, pbo, capability, metric, competitor) to feed the CoM directly from calls. 14 webapp files + 3 mobile files changed.

---

### Intelligence Tab Sub-tabs — March 21, 2026
Replaced 4 vertically-stacked sections with inner sub-tabs (COM | Presales | State Analysis | TechQual). Eliminates inter-section scroll. Deal fields row always visible. No logic changed.

---

### Real-time Coaching During Live Sessions — March 21, 2026
30s polling interval in `LiveSessionContext` analyses last 15 utterances via `/api/live-sessions/[id]/coaching`. Detects compelling events, competitors (Ping, ForgeRock, SailPoint, CyberArk, MS Entra), and action item commitments. Surfaces as dismissible `CoachingBanner` in recording screen (auto-expires 60s). Okta-specific, SE-specific. 16 tests.

---

### Admin Console Redesign — March 21, 2026
Admin console now matches the user dashboard: left collapsible sidebar, same header branding, same CSS classes. Horizontal tab bar removed. "Back to App" at sidebar bottom. Collapses to icon-only at `< 1100px`.

---

### Audit Logging — March 21, 2026
Fire-and-forget audit trail for all key user mutations. `AuditLog` DB model, `lib/audit.ts` helper, `GET /api/admin/audit-log`, admin UI at `/admin/audit-log`. Tracks 8 event types: board/card create+delete, notebook node create+delete, member invite, role change. 12 tests.

---

### CSS Consolidation — March 21, 2026
Deleted `globals.css` (5,037 lines). Migrated 5 unique rule groups to `globals-redesign.css`. Single CSS source of truth. Eliminates cascade confusion — all future style changes go to one file.

---

### SE Feature Proposal Flow — March 21, 2026
Non-admins can suggest features from the Roadmap page. Proposals land as `status: 'proposed'`. Admins see all; non-admins see published + own. 9 tests.

---

### Privacy Policy — March 21, 2026
`/privacy` page live (public, no auth). Covers data collection, GDPR rights, third parties, mobile permissions. Referenced in mobile `app.json` for App Store/Play Store. Privacy Policy link on login page.

---

### Release Notes System — March 21, 2026
Full release notes system: `ReleaseNote` Prisma model, admin CRUD at `/admin/releases`, settings "What's New" tab, email broadcast via Resend. 31 tests.

---

### Presales AI Improvements — March 21, 2026
- Per-meeting blurbs: `03/21 SG - [insight]` prepended to presalesNotes log per run, deduped on re-run
- Deal context injection: account, stage, champion, pain, CoM fields passed to prompt
- Output validation: invalid enum values dropped before DB write
- Stage definitions added to prompt
- Summary fallback: uses meeting summary when no transcript

---

### CoM Gap Analysis in Analyze Route — March 21, 2026
Analyze prompt now shows AI which CoM fields are already filled vs missing. AI prioritises GAPS, not re-extraction. Makes each run additive.

---

### Feedback Reply Email — March 21, 2026
Admins can send reply emails to users directly from the feedback admin panel (`replyNote` + `sendReply` on PATCH).

---

### Error Log Deduplication Schema — March 21, 2026
`ErrorLog` gets `fingerprint`, `occurrences`, `lastSeenAt` for grouping identical errors. Schema pushed to DB.

---

### AI Job Bell Fix — March 21, 2026
Stale running jobs (Vercel timeout orphans) now auto-healed: `GET /api/jobs` marks any running job >10 min as failed. Badge counts only genuinely running jobs. `isStaleRunning()` extracted to `lib/job-utils.ts` (shared by client + server).

---

### Auto Contact Sync + Action Item Assignment — March 21, 2026
- Contacts auto-synced on meetings page load and whenever meetings/attendees change
- Post-meeting agent now forwards AI-extracted `assignee` to card suggestions (was silently dropped)
- Meeting attendees passed to analyze prompt so AI can attribute action items by name

---

### Timeline Tab Fixes — March 21, 2026
- Action items now parse JSON correctly (was rendering raw JSON blob as text)
- Clicking a timeline item expands collapsed ancestors in the tree and scrolls to the selected node

---

### Email (Resend) — March 21, 2026
Transactional emails live. Key in `.env.local` + Vercel. Sender: `noreply@se-n-sei.com` (verified).
Password reset, verification, invites, password changed — all working.

---

### Error Log — Full Context — March 21, 2026
Admin error log rebuilt with complete investigative context per error entry.

**Delivered:**
- Schema: `adminNote`, `resolvedByUserId`, `updatedAt` on `ErrorLog` — pushed to DB
- API GET: batch User + Org JOIN — returns name, email, org name per error
- API PATCH: accepts `adminNote`, records `resolvedByUserId` from admin session
- Admin page: user name/email/org, browser/OS parsed from platform, method+path+status in row, expandable detail with stack trace, resolution note + "Mark resolved" workflow, shows resolver + timestamp + note after resolve

**Still to build:** assignee per error, critical alerts (email/Slack), error frequency sparkline. Search by message/path: done (March 30, 2026) — see Syslog page.

---

### Joel Hanson Pitch Deck + Exec Brief — March 21, 2026
9-slide pitch deck at `/pitch` and 1-page exec brief at `/brief`, both tailored for Joel Hanson (VP of Presales).
- Pitch: 9 main slides, 5 appendix slides, fully fact-checked, 12 agents, Joel's name on cover
- Brief: A4 portrait PDF, user-written opening, aligned asks, Playwright-verified layout
- Old pitch preserved at `/old-pitch` for reference

---

### Investor Pitch — March 21, 2026
11-slide seed deck at `/investor-pitch` (public route, no auth).

**Delivered:**
- Cover, Problem, Market (TAM/SAM/SOM), Solution, 12 Agents, Traction, Why Now, Business Model, GTM, Team, Ask
- $1.5M seed ask — 45% product / 30% sales / 25% infra
- Same design system as `/pitch` — Sora + Inter, dark hero, print-to-PDF
- All facts verified, Playwright-screenshotted before commit
- `proxy.ts` updated to allow public access

---

### Vercel Migration — March 20, 2026
Moved production from EC2 to Vercel (Okta Solutions Engineering team).

**Delivered:**
- `sensei-webapp` project under `okta-solutions-engineering` Vercel team
- Production URL: `https://sensei-webapp-eta.vercel.app`
- All secrets migrated to Vercel env vars
- Personal Vercel project deleted
- EC2 deploy scripts, PM2 config, and all EC2 references removed from repo
- CI workflow updated: test job + smoke test against Vercel URL
- `NEXTAUTH_URL` corrected

---

### E2E Test Suite — March 20, 2026
Full Playwright E2E suite covering every major flow: auth, home, notebook CRUD, board, pipeline, meetings, settings, search, admin enforcement, API auth.

**Delivered:**
- 106 tests across 10 spec files in `__tests__/e2e/`
- `auth.setup.ts` — single login, saves `storageState` for all tests
- `fixtures.ts` — `goToPage()`, `deleteNodes()`, `deleteBoards()` helpers
- `playwright.config.ts` — 3 projects: setup → e2e → smoke
- `.github/workflows/e2e-daily.yml` — daily 06:00 UTC + workflow_dispatch
- `data-testid` attributes added to Sidebar nav and UserMenu trigger
- Dedicated test user: `e2e.runner@se-n-sei.com` (GitHub secrets needed)
- Covers happy paths, unhappy paths (wrong creds, empty fields, wrong password), error paths (401/403 enforcement), edge cases (empty states)

---

### Feedback Widget + Error Tracking — March 20, 2026
User feedback submission and error tracking infrastructure.

**Delivered:**
- `Feedback` and `ErrorLog` Prisma models — committed and migrated to production DB
- `POST/GET /api/feedback` — submit feedback (authenticated), list feedback (super admin)
- `FeedbackWidget` component — modal with bug/feature/general tabs, screenshot attachment
- `lib/error-reporter.ts`, `components/ErrorReporterInit.tsx` — client-side error reporting
- `GET/PATCH /api/errors` — error log endpoints
- Admin pages: `/admin/feedback`, `/admin/errors`

---

### ~~Deployment Guide AI (Browser-side LLM)~~ — Built March 20, 2026, reverted March 2026
Built browser-side LiteLLM calling (`lib/litellm-browser.ts`, `NEXT_PUBLIC_LITELLM_*` vars) to work around the Vercel IP restriction. Later reverted: key exposed in browser bundle, approach inconsistent with security requirements. All LiteLLM calls are now server-side only. The Vercel network issue remains unsolved — see CLAUDE.md deployment section for options.

---

### Super Admin Console
Platform-wide control room gated by `SUPER_ADMIN_EMAIL`. Replaces org-scoped Overview tab.

**Delivered:**
- `requireSuperAdmin()` server helper + `SuperAdminGuard` client component
- 7 new platform API routes (stats, orgs, data, SSO, integrations, agents/run, me)
- 7 new admin pages: Platform Health, Organisations, Agents, Data, SSO Management, Integrations (Users page retained)
- Updated nav with 7 tabs replacing original 5
- 26 new tests covering auth enforcement + response shapes

---

### UI Redesign — Light Theme
Full visual overhaul from dark neo-brutalist to a professional light theme.

**Delivered:**
- New `globals-redesign.css` with complete design token system (colors, spacing, typography, shadows)
- Clean light theme inspired by Linear/Notion/Airtable aesthetics
- Consistent component styling across all pages

---

### Settings Page Overhaul
Full rewrite of the Settings page to be functional and polished.

**Delivered:**
- Sidebar nav with active-state highlighting and nested Organization sub-items (General / SSO)
- **Profile** — read-only display from session (name, email, role)
- **Organization > General** — prefills from `GET /api/organizations/current`, PATCH on save with "Saved." feedback, plan badge
- **Organization > SSO / IDP** — `<IDPWizard />` rendered full-width with descriptive copy
- **Team Members** — table of current members from API, role dropdown (calls PATCH), remove button (disabled for self), invite form (email + role)
- **Integrations** — Salesforce, Zoom, Gong, Google Calendar, Microsoft 365 — moved here from Notebook page
- Removed gear icon + integrations panel from Notebook page
- Removed `showIntegrations` from Zustand store
- Added full `settings-*` CSS class set to `globals.css`
- 6 new React Query hooks in `lib/queries.ts`: `useCurrentOrg`, `useUpdateOrg`, `useOrgMembers`, `useInviteMember`, `useRemoveMember`, `useUpdateMemberRole`

---

### Security: Encrypt OIDC Client Secret
OIDC client secret was stored in plaintext in the `Organization` table.

**Delivered:**
- Modified `/app/api/setup/idp/configure/route.ts` to encrypt `oidcClientSecret` using AES-256-GCM before persisting
- Uses existing `encrypt()` utility from `lib/encrypt.ts`

---

### Global Search (Cmd+K)
Command palette for searching across all content in the organization.

**Delivered:**
- `GET /api/search?q=` — searches boards by name, cards by title/tag/account/opportunity, notebook nodes by title/property values; scoped to org; max 8 results per category
- `useSearch` hook in `lib/queries.ts` with React Query; enabled only when query ≥ 2 chars
- `SearchPalette` component — floating modal with grouped results, keyboard navigation (↑↓ arrows, Enter, Escape), per-type icons
- Cmd+K / Ctrl+K global shortcut in `AppShell`; navigates to correct page and node on selection

---

### AI Job Tracker — Real-time Notification Bell ✅ March 18, 2026
Every AI analysis is tracked from trigger to completion. User can navigate away; the bell updates in real-time when the job finishes and takes them back.

**Delivered:**
- `AIJob` Prisma model — persists every user-triggered AI call (status, label, nodeId, targetPage)
- `GET /api/jobs` — last 20 jobs, org-scoped
- `GET /api/jobs/stream` — SSE endpoint for real-time push updates
- `lib/ai-jobs.ts` — `createJob`, `completeJob`, `failJob`, `withJob` wrapper, `jobEmitter`
- `components/AIJobBell.tsx` — replaces NotificationBell entirely; live spinner, click-to-navigate
- Wired into 7 routes: poc/extract, analyze, analyze-state, analyze-com, next-action, research, prep

---

## 🚧 In Progress — 30-SE Pilot Readiness

App is deployed on Vercel. The following items must be completed before the pilot starts.

### Blocking — Must complete before SEs log in

1. **OAuth callback URLs** — Add `https://sensei-webapp-eta.vercel.app/api/auth/callback/okta` and `/google` and `/credentials` to the Okta OIDC app and Google OAuth app. Without this, login will fail.

2. **GitHub secrets for E2E** — Add `TEST_USER_EMAIL=e2e.runner@se-n-sei.com` and `TEST_USER_PASSWORD=SenseiE2eBot#2026!` in repo Settings → Secrets → Actions.

3. **Vercel → LiteLLM network fix** — All AI routes are server-side. Vercel servers get 403 from `llm.atko.ai` (not on Okta network). Options: Okta IT whitelist Vercel CIDR, Vercel Static Outbound IPs (~$50/mo), or relay proxy on Okta network.

### Nice-to-have before pilot

4. **Domain cut-over** — Update `okta.se-n-sei.com` DNS A record to Vercel (`76.76.21.21`), attach domain to Okta SE Vercel project, update `NEXTAUTH_URL` to `https://okta.se-n-sei.com`.

5. **Vercel cron** — Add `vercel.json` with cron for post-meeting agent if background AI processing is needed.

---

### Scaling — 100 Users (post-pilot)

Before expanding beyond the 30-SE pilot:

1. **Vercel → LiteLLM network fix** — if not already done for pilot, this is mandatory at scale. All AI routes are server-side; pick one of the three options documented in CLAUDE.md.

2. **Replace SSE EventEmitter** — `ai-jobs.ts` uses in-process EventEmitter; doesn't work across Vercel function invocations. Replace with Supabase Realtime or simple DB polling. ~1 day of work.

3. **Wire Upstash Redis** — Distributed rate limiting. Config already in `.env.local` (commented out). Just needs credentials added to Vercel env vars.

### Scaling — 300 Users (Series A territory)

1. All items from 100-user list
2. Post-meeting agent job queue (Upstash QStash or similar) — cron won't keep up at volume
3. Supabase Pro confirmed and monitored
4. Load test with k6 before rollout (script ready to write)

---

### Share Link + Think Tank

Two external collaboration features. Building Share Link first (no new deps), Think Tank second.

#### Share Link ✅ Shipped March 18, 2026 · Extended April 19, 2026
A read-only public link to any notebook node. Anyone can view it. Registered sensei users can import it as a new standalone node. Opportunity shares include a "Download for AI" button to export a clean `.md` file for ChatGPT/Gemini analysis.

**Delivered (March 18):**
- `ShareToken` Prisma model — 256-bit entropy token, configurable expiry, view count, cascade delete
- `GET/POST/DELETE /api/notebook/[id]/share-link` — check/generate/revoke (owner only)
- `GET /api/share/[token]` — public node data API (no auth required)
- `POST /api/notebook/import` — authenticated import as new standalone node
- `app/share/[token]/page.tsx` — clean public page with expired state
- `useGetShareLink`, `useCreateShareLink`, `useRevokeShareLink` hooks
- NotebookShareDialog extended with share link section (expiry picker, copy, revoke)
- 26 new tests passing, Playwright visual verification done

**Extended (April 19) — Opportunity Markdown Export:**
- `GET /api/share/[token]/md` — public markdown export (title, metadata, CoM, Mantra, Presales, State Analysis, Meeting Summaries newest-first). HTML/attribution stripped. Filename sanitized.
- `GET /api/share/[token]` — now returns child meeting summaries for opportunity nodes
- Share page now renders Meeting Summaries collapsible section
- "⬇ Download for AI" button (opportunity-only) on the share page header
- 16 new tests; 2869 total passing, TypeScript clean

---

#### Think Tank
A living collaborative document an SE creates and shares with a customer. Both sides edit in real-time. Purpose-built for POC tracking, deployment guides, and deal collaboration.

**User stories:** TT-1 through TT-15
**Decisions:**
- Any notebook content can be included (POC guide, meetings, deployment docs, action items)
- Customer types their name on first open → shown in presence + comments
- Live editing (Liveblocks) + inline comments per block
- Configurable expiry
- Tied to an account node in sensei

**Scope:**
- `ThinkTank`, `ThinkTankComment`, `ThinkTankGuest` Prisma models
- `GET /share/think-tank/[token]` — public guest landing page
- `POST /api/think-tanks` — create (owner, org-scoped)
- `GET/PATCH/DELETE /api/think-tanks/[id]` — manage
- `POST /api/think-tanks/[id]/comments` — comment (auth or guest token)
- `POST /api/think-tanks/guest-auth` — guest enters name, gets session token
- Real-time: Liveblocks
- Editor: Tiptap (already installed) + Liveblocks collaboration extension

**New dependencies:** `@liveblocks/client`, `@liveblocks/react`, `@liveblocks/node`

---

## 🐛 Known Issues

### Notebook Action Button Overlay
`+Contact` and `+Opportunity` quick-add buttons visually overlap on the Notebook page. Low priority UX fix.

---

## 🚀 SE Differentiation — Competitive Moat

Features that deepen senSEi's lead. None of the market has all of these. Each one makes switching harder.

---

### Technical Win Memo Generator

After a POC closes, one AI agent generates a structured technical win/loss memo from CoM fields + meeting history + POC criteria. Output: customer environment, what was proved, why won/lost technically, lessons learned.

**Why:** Most hated SE deliverable — written from scratch every time. Nobody automates it.

**How:** New agent `/api/agent/tech-win-memo`. Triggered from opportunity detail after Closed stage. Takes all meeting transcripts + CoM + POC criteria → returns structured markdown memo. One-click copy/DOCX export.

---

### Competitive Battle Card on Detect

When `intelligence.opportunity.competitor` is extracted from a transcript, automatically surface a battle card: common technical objections, Okta advantages, relevant proof points, integration comparison. Inline in the opportunity detail when a competitor is logged.

**Why:** `competitor` field is already extracted. This is a 1-session feature. Vivun/Gong have generic battle cards — senSEi's are Okta-specific and context-triggered.

**How:** `BattleCard` component in OpportunityDetail. When `p.competitor` is set, call `/api/agent/battle-card` with competitor + deal context. LLM pulls from Okta knowledge base. Cache by competitor name.

---

### Customer Proof Point Matching

When a use case is mentioned in a meeting transcript ("we need SCIM for 50k contractors"), surface the most relevant Okta customer case studies and reference customers. Displayed in meeting detail or opportunity intelligence panel.

**Why:** Seismic/Highspot charge enterprise prices for generic content matching. senSEi can do it better — transcript context is already there and Okta knowledge base covers the products.

**How:** Secondary LLM call after transcript analysis matches extracted use cases to a curated proof point library (DB table). Proof point management in admin panel.

---

### Technical Qualification Scorecard

Objective technical readiness score alongside CoM. Structured checklist: integration stack confirmed, required Okta features available, customer infrastructure compatible, security/compliance identified, champion technical sign-off obtained.

**Why:** CoM = *why* they should buy. Technical scorecard = *if* the deal is technically qualified. Prevents deals moving to POC before feasibility is confirmed. Nothing in market has this.

**How:** New section in OpportunityDetail Intelligence tab. Store as `NodeProperty` keys (`techQual_*`). Add `techQualScore` to `lib/health-score.ts` computation. AI pre-populates from transcripts; SE refines manually.

---

### Win/Loss Analysis at SE Level

After every closed deal: snapshot CoM completeness, agent usage, POC outcome, stage velocity. Over time surface patterns — which CoM gaps correlate with losses? Which agents predicted wins? Evidence-based SE playbook.

**Why:** Win/loss for AEs exists everywhere. Nobody does it for SEs at the technical level. Over 50-100 deals this builds a proprietary dataset about SE effectiveness at Okta.

**How:** Add `outcome` field to opportunity nodes. On Closed Won/Lost, snapshot CoM score, agent usage count, POC status, days-in-stage. New admin page `/admin/win-loss`. Prioritise after pilot generates enough deal data.

---

### ✅ AI Chat Artifact Generation — SHIPPED March 22, 2026

Ask the AI assistant to produce structured documents — deal briefs, blog posts, POC summaries, slide outlines, battle card drafts, email templates — directly in the chat tab (web + mobile).

**Why:** The LLM can already write these, but three technical gaps make it unusable today: `max_tokens: 1000` truncates anything longer than ~750 words; the mobile chat renders plain text (no markdown); there is no way to copy or export the output. Adding proper rendering and copy-out turns the AI tab from a Q&A widget into a lightweight document generator.

**How:**
- Raise `max_tokens` to 3000–4000 for document-type prompts (detect intent from keywords: "write", "draft", "create", "generate", "summarise as")
- Add `react-native-markdown-display` to mobile chat — render assistant messages as formatted markdown (headings, bullets, bold, code blocks, tables)
- Add "Copy" button to every assistant message bubble
- Match treatment in webapp chat (already renders markdown via `marked`)
- No new routes or DB changes needed

**Scope:** 1 session. Mobile + webapp chat. No export to DOCX/PPT in this phase — that's a separate feature.

---

### ✅ Real-time Coaching During Live Sessions — SHIPPED March 21, 2026

During a live mobile recording, AI detects signals in real time: compelling event mentioned → nudge to capture; competitor detected → battle card surfaces; commitment made → action item pre-staged. Non-intrusive overlay the SE can dismiss.

**Why:** Gong's Call AI does this for AEs generically. senSEi's would be Okta-specific and SE-specific. Transforms the mobile app from passive recorder to active SE assistant.

**How:** In `LiveSessionContext`, run a lightweight LLM call every 30s on the last 2 minutes of utterances. Returns signal flags `{ type: 'compelling_event', content: '...' }`. Displayed as dismissible banner. Requires low-latency LLM endpoint.

---

## 📋 Phase 2 — Post-Launch

Everything below is deferred until after the Okta go-live. No new features until Share Link + Think Tank have shipped and the app is stable in production.

---

### Contact Management Phase 2 — UI Enhancements

**Status:** ✅ Shipped — April 11, 2026

1. `OrgChart.tsx` — `poc_role` badge inline with role pill. `hasAnyRole` and flat-grid filter now also check `poc_role`.
2. `NotebookPage.tsx` (ContactPanel) — AI intel block: role badge, poc_role badge, sentiment indicator, signals quote. All conditional.
3. Email extraction live in stakeholder-map prompt — real-world extraction rate TBD from pilot data.

---

### Contact Management Phase 3 — Contacts Tab

**Status:** ✅ Shipped — April 11, 2026

`AccountDetail.tsx` — new "Contacts (N)" tab. Enriched cards: name, title, email, sentiment indicator, role badge, poc_role badge. Clickable rows navigate to contact detail. "In Deals" column deferred (needs a dedicated query — not in scope for Phase 3).

---

### Zoom App Plugin — Live Captions → sensei-webapp

**Status:** Planned — implementation deferred, full plan ready (March 2026)

**What it does:** A Zoom App panel (iframe inside Zoom desktop client) that streams live meeting captions directly into the sensei live session pipeline — no mobile device needed.

**Architecture (plan complete):**
- Zoom App hosted at `/zoom-app` — iframe runs inside Zoom sidebar, auth via existing NextAuth session cookie (same domain, no extra auth)
- `zoomSdk.addEventListener('onLiveTranscriptionMessage', ...)` → batch POST to existing `/api/live-sessions/{id}/utterances`
- `zoomSdk.addEventListener('onMeetingEnded', ...)` → POST to existing `/api/live-sessions/{id}/finalize` → triggers post-meeting agent (unchanged)
- SE picks account/opportunity from the sidebar before recording starts

**What needs to be built:**
1. `app/zoom-app/layout.tsx` + `app/zoom-app/page.tsx` — Zoom App UI (4 states: unauthenticated, idle, recording, done)
2. `prisma/schema.prisma` — add `source String @default("mobile")` and `zoomMeetingId String?` to `LiveSession`
3. `app/api/live-sessions/route.ts` — accept `source` and `zoomMeetingId` in POST body
4. `proxy.ts` — add `/zoom-app/**` to public paths
5. Install `@zoom/appssdk`
6. Register Zoom App in Zoom Marketplace (external one-time setup)

**All existing API routes reused as-is** — no changes to finalize, utterances, or post-meeting agent.

**Tests to write first:** `__tests__/api/live-sessions/create-zoom-source.test.ts` (source=zoom + zoomMeetingId validation)

---

### IAM — OpenFGA + OIDC SP + SCIM + Multi-Tenancy

**Decisions already locked (design complete, implementation deferred):**
- Flat org + user attributes (segment, title, region) — no Team model in DB
- Groups: Okta Groups via SCIM when IDP connected; native CRUD in sensei when no IDP
- Permission levels: viewer / editor (two levels)
- Inheritance: sharing a parent node grants access to all children automatically
- Sharing: within-org only (persistent). External expiring links shipped in pre-launch above
- OIDC: fully compliant SP, dynamic per-org config, domain-based routing, RP-initiated logout, JIT user creation (domain-guarded, always member role)
- Tenant onboarding: manual for now, self-service when scale demands it

**FGA Authorization Model (ready to implement):**
```
model
  schema 1.1

type user

type organization
  relations
    define member: [user, group#member]
    define admin: [user, group#member]

type board
  relations
    define org: [organization]
    define owner: [user]
    define viewer: [user, group#member]
    define editor: [user, group#member]
    define can_view: viewer or editor or owner
    define can_edit: editor or owner

type notebook_node
  relations
    define org: [organization]
    define owner: [user]
    define parent: [notebook_node]
    define viewer: [user, group#member]
    define editor: [user, group#member]
    define can_view: viewer or editor or owner or (parent->can_view)
    define can_edit: editor or owner or (parent->can_edit)

type group
  relations
    define org: [organization]
    define member: [user]
```

**Work breakdown:**
1. FGA Foundation — `@openfga/sdk`, `lib/fga.ts`, Prisma models (`Group`, `GroupMember`, `OrgDomain`, `UserProfile`, `FGAToken`), data migration, replace access helpers
2. Dynamic OIDC SP + JIT — dynamic per-org provider, login domain routing, claims mapping, RP logout
3. SCIM 2.0 — RFC 7643/7644 endpoints for Users + Groups, per-org bearer tokens
4. Native Group Management — admin UI when SCIM is off
5. Tenant Onboarding Tool — internal admin page, create org + SCIM token
6. Permission Enforcement Sweep — audit all routes, enforce `canEdit`

---

### Competitive Gap — Real-time Collaboration (within-org)
**Gap vs.** Notion, Linear
Per-node comment threads, @mentions, live presence on shared views.
*(Think Tank handles external collaboration — this is within-org collaboration.)*

---

### Competitive Gap — Bi-directional Salesforce Sync
**Gap vs.** Salesforce, Attio
Pull account/contact/opportunity data from Salesforce. Push meeting notes and action items back as activities.

---

### Competitive Gap — Block-based Editor
**Gap vs.** Notion
Replace plain contenteditable with Tiptap slash-command blocks, inline formatting, image embeds, markdown shortcuts.
*(Tiptap already installed for Think Tank — reuse for notebook.)*

---

### Competitive Gap — Reporting & Analytics
**Gap vs.** Salesforce, Gong
Pipeline summary (ARR by stage, win rate), activity feed, account health distribution, overdue action items report.

---

### Competitive Gap — Customizable Pipeline Stages
**Gap vs.** Salesforce, Attio
Stages hardcoded in `constants.ts`. Admin UI to add/rename/reorder/delete. Stage-level conversion tracking.

---

### Competitive Gap — Google Calendar / Microsoft 365 Real Sync
**Gap vs.** Attio, Notion Calendar
GCal and M365 are toggle-only today. Pull upcoming meetings into Notebook, push action items to calendar.

---

### Competitive Gap — Email Activity Capture
**Gap vs.** Attio, Affinity
BCC address to auto-create meeting notes, Gmail/Outlook sidebar extension, timeline view of all touchpoints.

---

### Competitive Gap — Custom Account Properties
**Gap vs.** Notion, Attio
Admin-defined field schema per node type (text, number, date, select, multi-select, user). Required fields.

---

### Competitive Gap — Customizable Meeting Templates
**Gap vs.** Notion
Admin UI to create/edit/delete templates. Template variables (account name, AE, date). Sharing across org.

---

### Competitive Gap — Gong Call Intelligence
**Gap vs.** Gong
Pull call transcript + AI summary into meeting notes. Surface deal signals. Coaching flags for managers.

---

## Planned

### Autonomous Bug-Fix Agent (EC2 + Claude Code)
A always-on daemon running on EC2 that watches the sensei error log and auto-fixes production bugs using Claude Code.

**Architecture:**
- EC2 polls `/api/admin/errors` every 5 minutes for unresolved errors
- On new error: `git pull` → invokes `claude --dangerously-skip-permissions` with error context
- Claude Code reads codebase, edits files, runs `vitest`
- If tests pass → commits to `hotfix/auto-[error-id]` branch → pushes → opens GitHub PR
- Admin panel shows "Fix Pending" status + link to PR per error
- Human reviews PR → approves → Vercel auto-deploys

**What's needed:**
- Claude Code installed + authenticated on EC2
- `GITHUB_TOKEN` with repo write access on EC2
- `scripts/bug-agent.ts` daemon (designed, ready to build)
- `fix_pending` status on `ErrorLog` model
- "View PR" link in admin error log UI
- PM2 to keep daemon alive across reboots

**Safety:** Commits to hotfix branch only — never directly to main. Tests must pass before push.

---

### sensei MCP Server
A Model Context Protocol server that connects Claude (claude.ai or Claude Code) to live sensei deal data, enabling AI-generated content grounded in real opportunity context.

**What it enables:**
- *"Draft the follow-up email for the Waters POC checkpoint"* → Claude knows Waters context
- *"What are my top 3 at-risk deals?"* → Claude queries live pipeline
- *"Summarize everything before my Acme call tomorrow"* → full account + meeting history
- *"Generate BV slides for Waters"* → uses real CoM data

**Architecture:**
- TypeScript MCP server (`@modelcontextprotocol/sdk`) that authenticates to sensei API
- Tools exposed: `get_opportunity`, `get_pipeline`, `get_meetings`, `get_action_items`, `get_com`, `search_deals`, `get_account`
- Claude Code skill (Bash-based) for CLI/IDE use — no MCP server needed for that path
- MCP server needed only for claude.ai web app integration

**What needs building:**
1. API key authentication endpoint (`POST /api/auth/api-key`) — long-lived key for non-browser auth
2. MCP server (~300 lines TypeScript) connecting those tools to existing API endpoints
3. User setup: add server to Claude Desktop / claude.ai MCP config

**Note:** For Claude Code CLI, a simpler Bash-based skill can achieve the same without a server.

---

### Cross-Deal MCP Analysis Layer

Extends the sensei MCP server with cost-efficient cross-deal analysis — so Claude can answer questions across the full book of business without loading every full deal brief into context.

**The problem:** A full deal brief is ~3–5k tokens. Querying 50 deals naively costs $0.60–$3.00 per question. Tiered tools + pre-computed summaries bring that down to ~$0.05.

**Pre-computed deal summaries:**
- Add `aiSummary String?` and `aiSummaryAt DateTime?` to `NotebookNode` (opportunities)
- ~150 tokens per deal — generated automatically on CoM synthesis, regenerated when deal updates
- Staleness flag: `aiSummaryAt` compared to last `updatedAt` — re-generate if stale

**Tiered MCP tools:**
| Tool | Tokens per call | Use |
|---|---|---|
| `list_deals_summary` | ~200/deal × N deals | Cross-analysis entry point |
| `get_deal_full` | ~4k | Drill into a specific deal |
| `search_deals` | ~4k × top-5 | Semantic search, returns most relevant |
| `query_by_field` | ~200/deal × filtered set | Filter by stage, competitor, ARR, tags |

**Saved analyses (explicit only — not auto-saved):**
- New `DealAnalysis` model: `{ id, userId, orgId, query, response, dealIds[], model, createdAt, stale Boolean }`
- `stale` auto-sets when any referenced deal updates after `createdAt`
- Only saved when user explicitly pins — no automatic accumulation of noise
- Admin can browse pinned analyses at `/admin/deal-analyses`

**What needs building:**
1. `aiSummary` + `aiSummaryAt` on `NotebookNode` schema (prisma migrate)
2. Summary generation hook in `analyze-com` route (post-synthesis, ~1 extra LLM call)
3. `GET /api/deals/summary` — all opportunities with compact summaries, org-scoped
4. `DealAnalysis` model + `POST /api/deal-analyses` (save pinned) + `GET /api/deal-analyses`
5. MCP server tools wired to the above endpoints
6. Staleness sweep: mark `DealAnalysis.stale = true` when any referenced deal node updates

---

### B2B Tenant Provisioning
`POST /api/admin/organizations` + `/admin/tenants` page. Fill org name/slug/admin email → click Provision → creates org, sets billing active, sends invite email. Needed before external rollout.

---

### Switch Okta Issuer to Production
Change from `goals.oktapreview.com` → `sen-sei.okta.com` via Admin → Identity Provider. No code change needed — takes effect immediately via 60s cache.

---

### Cross-Deal Intelligence Layer (LLM Wiki Pattern)
senSEi already accumulates knowledge within a deal (meeting → CoM → account synthesis). This feature extends that to *across* deals — building a persistent, compounding wiki that spans the full book of business and gets richer with every deal, meeting, and question asked.

**The gap it closes:**
Today, each deal's CoM, state analysis, and presales data is isolated in its own node tree. If a manager asks "what are the common technical objections across all our Entra competitive deals?" or "which POC patterns tend to close fastest?", there's no answer — the cross-deal signal is buried in individual opportunity fields. This layer surfaces it.

**What it enables:**
- *"Show me every deal where SailPoint was the incumbent — what were the migration blockers?"* → synthesized from all opportunities with competitor=SailPoint
- *"What's our win pattern for deals in the POC stage longer than 60 days?"* → computed from pipeline history
- *"Which technical objections come up most often in healthcare deals?"* → cross-referenced from meeting transcripts
- *"What does Waters tell us about how BMC might progress?"* → pattern match across accounts in the same industry + stage
- Persistent synthesis pages per competitor, per industry, per product line — updated automatically when new deals move through stages

**Architecture:**
- **Wiki layer**: A new `Wiki` model (or a `wiki/` directory of markdown stored in S3) — one page per competitor, per industry vertical, per product line, per common objection pattern. Structured markdown, LLM-maintained.
- **Index**: `wiki/index.md` — catalog of all wiki pages with one-line summaries
- **Synthesis triggers**: When a deal closes (won or lost), when a CoM is generated, or on-demand — the LLM reads the relevant wiki page and updates it with new signal from the deal
- **Query interface**: New AI Copilot tool — `search_wiki(query)` — reads the index, drills into relevant pages, answers with citations to specific deals
- **Contradiction detection**: During wiki update, LLM flags if new deal data contradicts existing synthesis (e.g., "Entra usually loses on session termination — but this deal shows a different objection pattern")
- **Cross-reference maintenance**: When a new competitor page is updated, the LLM also updates any product line or industry pages that reference it

**What needs building:**
1. `Wiki` Prisma model (or S3-backed markdown store) — `{ id, slug, title, content, updatedAt, sourceDeals: string[] }`
2. `POST /api/wiki/ingest` — takes an opportunity ID, reads its CoM + presales data, updates relevant wiki pages
3. `GET /api/wiki/search` — index-first search returning ranked wiki pages
4. Wiki admin UI (`app/admin/wiki/`) — browse, view, manually trigger rebuilds
5. Copilot tool: `search_wiki` — surfaces wiki pages in AI Copilot answers
6. Synthesis trigger: hook into `analyze-com` route to auto-ingest on CoM generation
7. Cron: weekly full-wiki lint (find contradictions, orphan pages, stale claims)

**Relationship to current architecture:**
- Adds to existing progressive synthesis cascade (meeting → opportunity → account → **portfolio**)
- The wiki is the 4th synthesis level — above account, across the entire book of business
- SE-owned data stays private; wiki is org-wide and manager-visible by default

---

## Positioning Notes

The strongest defensible features to double down on (not gaps — keep investing):
- **Meeting → Action Item → Board Card pipeline**: end-to-end workflow no competitor owns
- **Account 360 auto-summary**: computed health, open opps, last meeting, overdue tasks
- **Opinionated sales hierarchy**: Account → Opportunity → Meeting is structured in a way Notion requires weeks to replicate
- **Sales-specific meeting templates**: zero-friction prep for Discovery, QBR, EBR, Demo
- **Think Tank / external collaboration**: SE-to-customer living workspace — no competitor does this for the SE motion
