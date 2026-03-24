# senSEi — Roadmap

---

## ✅ Completed

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

**Still to build:** search by message/user email, assignee per error, critical alerts (email/Slack), error frequency sparkline

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

### Deployment Guide AI (Browser-side LLM) — March 20, 2026
Fixed AI generation for the Okta deployment guide. Root cause: Okta LiteLLM proxy is IP-restricted; Vercel servers get 403.

**Delivered:**
- `lib/litellm-browser.ts` — client-side LiteLLM caller using `NEXT_PUBLIC_LITELLM_*` vars
- `DeploymentGuide.tsx` — browser-first LLM call; actual error message in toast
- `NEXT_PUBLIC_LITELLM_*` env vars added to `.env.local` and Vercel

**Note:** Pattern needs to be applied to all other user-triggered AI routes (next-action, research, analyze, etc.).

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

3. **Browser-side LLM for all AI routes** — Apply `callLiteLLMBrowser()` pattern to: `next-action`, `research`, `analyze`, `analyze-state`, `analyze-com`, `okta-advisor`, `stakeholder-map`, `meeting-prep`. Currently only `deployment-guide` is fixed.

### Nice-to-have before pilot

4. **Domain cut-over** — Update `okta.se-n-sei.com` DNS A record to Vercel (`76.76.21.21`), attach domain to Okta SE Vercel project, update `NEXTAUTH_URL` to `https://okta.se-n-sei.com`.

5. **Vercel cron** — Add `vercel.json` with cron for post-meeting agent if background AI processing is needed.

---

### Scaling — 100 Users (post-pilot)

Before expanding beyond the 30-SE pilot:

1. **Browser-side LLM for all AI routes** — apply `callLiteLLMBrowser()` to next-action, research, analyze, analyze-state, analyze-com, okta-advisor, stakeholder-map, meeting-prep. Alternatively: get Vercel IPs whitelisted in `llm.atko.ai`.

2. **Replace SSE EventEmitter** — `ai-jobs.ts` uses in-process EventEmitter; doesn't work across Vercel function invocations. Replace with Supabase Realtime or simple DB polling. ~1 day of work.

3. **Wire Upstash Redis** — Distributed rate limiting. Config already in `.env.local` (commented out). Just needs credentials added to Vercel env vars.

4. **Rotate `NEXT_PUBLIC_LITELLM_API_KEY`** — Exposed in browser bundle for the pilot. Rotate after pilot ends, implement proper server-side fix.

### Scaling — 300 Users (Series A territory)

1. All items from 100-user list
2. Post-meeting agent job queue (Upstash QStash or similar) — cron won't keep up at volume
3. Supabase Pro confirmed and monitored
4. Load test with k6 before rollout (script ready to write)

---

### Share Link + Think Tank

Two external collaboration features. Building Share Link first (no new deps), Think Tank second.

#### Share Link ✅ Shipped March 18, 2026
A read-only public link to any notebook node. Anyone can view it. Registered sensei users can import it as a new standalone node.

**Delivered:**
- `ShareToken` Prisma model — 256-bit entropy token, configurable expiry, view count, cascade delete
- `GET/POST/DELETE /api/notebook/[id]/share-link` — check/generate/revoke (owner only)
- `GET /api/share/[token]` — public node data API (no auth required)
- `POST /api/notebook/import` — authenticated import as new standalone node
- `app/share/[token]/page.tsx` — clean public page with expired state
- `useGetShareLink`, `useCreateShareLink`, `useRevokeShareLink` hooks
- NotebookShareDialog extended with share link section (expiry picker, copy, revoke)
- 26 new tests passing, Playwright visual verification done

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

## Positioning Notes

The strongest defensible features to double down on (not gaps — keep investing):
- **Meeting → Action Item → Board Card pipeline**: end-to-end workflow no competitor owns
- **Account 360 auto-summary**: computed health, open opps, last meeting, overdue tasks
- **Opinionated sales hierarchy**: Account → Opportunity → Meeting is structured in a way Notion requires weeks to replicate
- **Sales-specific meeting templates**: zero-friction prep for Discovery, QBR, EBR, Demo
- **Think Tank / external collaboration**: SE-to-customer living workspace — no competitor does this for the SE motion
