# senSEi — Roadmap

---

## ✅ Completed

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

## 🚧 In Progress — Pre-Launch (last features before Okta go-live)

After these two features ship: **no new features until post-launch**. Focus shifts to stability, performance, and optimizations only.

### Share Link + Think Tank

Two external collaboration features. Building Share Link first (no new deps), Think Tank second.

#### Share Link
A read-only public link to any notebook node. Anyone can view it. Registered sensei users can import it as a new standalone node.

**User stories:** SL-1 through SL-7
**Decisions:**
- Any notebook node can be shared (accounts, opportunities, meetings, contacts, notes, POC guides)
- Read-only for anyone without sensei account
- Configurable expiry (1–365 days)
- Import creates a brand-new standalone node — zero interference with existing workspace
- View count tracked on the token

**Scope:**
- `ShareToken` Prisma model: `id`, `organizationId`, `nodeId`, `token` (64-char hex), `expiresAt`, `viewCount`, `createdAt`
- `GET /share/[token]` — public page (no auth, clean UI)
- `POST /api/notebook/[id]/share-link` — generate with expiry (owner only)
- `DELETE /api/notebook/[id]/share-link` — revoke (owner only)
- `GET /api/share/[token]` — returns node data for public page
- `POST /api/notebook/import` — authenticated, imports shared node as new standalone
- Share UI in notebook: copy link, view count, expiry, revoke button

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
