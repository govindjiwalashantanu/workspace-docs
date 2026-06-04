# Working Protocol

Applies to: senSEi webapp, senSEi mobile, Echo backend, Echo mobile.
Last updated: April 2026.

---

## Hook System

Three hooks enforce this protocol automatically in every session:

| Hook | When | What it does |
|---|---|---|
| `SessionStart` | On session open | Loads project context, shows What's Next, git status, locked project warning |
| `UserPromptSubmit` | Before every prompt | Detects task type, injects persona pipeline + protocol gates |
| `Stop` | After every response | Checks for modified files, reminds to update context + commit |

If hooks aren't firing: run `/hooks` once to reload config.

---

## Core Rules

1. **Context first, always.** Before writing a single line of code, read SESSION_NOTES.md (last 80 lines). If you don't know where we left off, you're not ready to start.
2. **Tests first, no exceptions.** Write the failing test before implementation. The test must fail before any code is written.
3. **Fix root causes. Never write workarounds.** If something is broken, trace it to the source and fix it there. Do not add guards, patches, or overrides that mask the underlying issue. If stale data is causing problems, clear the data — don't code around it.
   - **When tests fail: fix the app, not the test.** A failing test is a signal that the app is wrong. Fix the app code first. Only change the test if the test itself is incorrect (wrong selector, wrong expectation, wrong mock). Never update a test just to make it pass — that destroys its value as a signal.
   - **App stability is the goal, not test stability.** The app must be correct and production-ready. Tests exist to verify the app works. If a test is passing by accident (e.g. accepting a 307 instead of fixing the middleware), that is not a win — the app is still broken. Fix the underlying app behaviour so tests pass for the right reasons.
   - **Changing assertions to accept bad behaviour is always wrong.** Never broaden an assertion (e.g. `[401, 307]` instead of `[401]`) without first asking: can I fix the app so the correct response is returned? If yes, fix the app. Only broaden if the behaviour is genuinely ambiguous by spec.
4. **Iterate internally until done.** The owner sees it once. Working. Run tests, fix failures, re-run until green. Do not surface work at a gate failure — fix it and continue.
5. **Ask before writing.** State your understanding of the task. Get confirmation before touching code.
6. **Two-step approval for all changes.** Every change needs two explicit approvals: one to commit, one to push. Never push without both.
7. **Treat the owner's time as the most valuable resource.** Surface it once. Working.
8. **Visual output requires Playwright verification.** PDFs, print views, generated documents, charts, new UI components — screenshot with Playwright before presenting. Never ask the owner to be the first to see it.
9. **Human Voice.** Every user-facing string, email, agent response, and pitch slide must sound like a sharp, direct person wrote it. No em dashes. No AI tells ("it is worth noting", "delve", "certainly", "I would be happy to"). Short sentences. Direct language.
10. **Use subagents to preserve context.** Offload all research, exploration, and multi-file analysis to subagents. The main context window is expensive. Run multiple subagents in parallel where tasks are independent.
11. **Run /compact when context gets large.**
12. **Schema changes are atomic.** A Prisma schema change, `prisma db push`, and a git commit happen together in one step.
13. **Check logs before touching code.** When something is broken: read logs first. The error is almost always already there.
14. **Update context files every session — no exceptions.** Even if the session was just a conversation. CLAUDE.md Current State, SESSION_NOTES.md (add entry + update What's Next), ROADMAP.md (mark completed, add newly scoped). The Stop hook reminds. Do not wait to be asked.
15. **Archive SESSION_NOTES.md when it exceeds 1500 lines.** Copy the full file to `SESSION_NOTES_ARCHIVE_YYYY-MM.md`, then trim the live file to: `## ⟶ What's Next` (always at top) + last 5 session entries. The Stop hook shows the line count and warns when threshold is reached.
16. **Every feature ships complete — all surfaces updated.** A feature is not done when the code works. It is done when every surface that describes, demonstrates, or introduces that feature reflects the current reality. See the Surface Sync Checklist below.
17. **Project lock is absolute.** echo-mobile and echo-backend are LOCKED — do not read, edit, or reference their files unless Shantanu explicitly names them as in-scope for the session. Only one project is active at a time. If another project is mentioned, acknowledge and log it — do not switch without explicit instruction.

---

## Surface Sync Checklist

Run this checklist at the end of every feature or significant change. Each surface that is affected must be updated in the same session — never deferred.

### Required on every feature/change

| Surface | File | Update when |
|---|---|---|
| **Context files** | `CLAUDE.md`, `SESSION_NOTES.md`, `ROADMAP.md` | Every session — already rule 14 |
| **Release notes** | Admin → Release Notes (DB via `/api/release-notes`) | Every user-facing feature shipped |

### Required when the feature is user-facing

| Surface | File | Update when |
|---|---|---|
| **Hero / Home page** | `components/HomePage.tsx` | New feature worth highlighting, capability removed, or key copy change |
| **Onboarding tour** | `components/OnboardingTour.tsx` | New UI element or workflow added that an SE needs to discover |
| **Setup wizard** | `components/SetupWizard.tsx` | New integration, new required step, or onboarding flow change |
| **Pitch deck** | `app/pitch/page.tsx` | Major capability added or positioning change |
| **Investor pitch** | `app/investor-pitch/page.tsx` | Major capability added or traction metric changes |
| **Prompt Manager** | `lib/prompt-defaults.ts` display names/descriptions | Prompt purpose or output changes |

### Checklist for Feature pipeline (run at step 9)

Before presenting to owner, verify:
- [ ] Release note created for this feature (if user-facing)
- [ ] Hero page copy updated (if feature is worth surfacing)
- [ ] Onboarding tour updated (if new UI element added)
- [ ] Setup wizard updated (if onboarding flow changes)
- [ ] Pitch/investor pages updated (if it strengthens the story)
- [ ] Prompt Manager display names and descriptions current
- [ ] CLAUDE.md Architecture section reflects new routes, components, or patterns

### Rule

If you ship code but leave any affected surface stale, the feature is incomplete. A new AI analysis tab that isn't mentioned in the onboarding tour, a capability that isn't in the release notes, a pitch page that doesn't reflect current functionality — these are bugs, not backlog items.

---

## Context File Architecture

**Project repos are primary.** workspace-docs is a synced backup updated by the Stop hook.

### Every session, in order:

**START:**
1. Hook injects session context automatically (What's Next, git status, Current State)
2. Read SESSION_NOTES.md last entry for full context
3. Run session start checklist (below)

**END:**
1. Update `## ⟶ What's Next` at top of SESSION_NOTES.md
2. Add session entry to SESSION_NOTES.md
3. Update ROADMAP.md if features completed or newly scoped
4. Update `## Current State` in CLAUDE.md
5. Commit (with owner approval)
6. Copy SESSION_NOTES.md to workspace-docs (Stop hook shows the command)

### SESSION_NOTES.md format

```markdown
## ⟶ What's Next  ← always at top, always current
- [ ] Item 1
- [ ] Item 2
_Updated: [date]_

---
## Session: [date] — [title]
...
```

---

## Session Start Checklist

Run every session before touching any code. If any step fails, fix it before proceeding.

```bash
# 0. Check MEMORY.md — confirm locked projects, active project, any session-relevant constraints
#    (hook loads this automatically, but verify project lock before any file access)

# 1. Check where we left off (hook loads this automatically)
head -80 SESSION_NOTES.md

# 2. Uncommitted work
git status --short

# 3. TypeScript — zero errors required
npx tsc --noEmit

# 4. Tests — full suite must pass
npx vitest run   # webapp
npx jest --forceExit   # mobile

# 5. Schema sync (webapp only)
npx prisma migrate status

# 6. Env vars
node -e "const r=['DATABASE_URL','NEXTAUTH_SECRET','LITELLM_API_KEY','LITELLM_BASE_URL']; const m=r.filter(k=>!process.env[k]); if(m.length) console.log('MISSING:',m.join(', ')); else console.log('env OK');"
```

State after running:
> "Session start: ✓ TypeScript clean / ✓ Tests passing (N pass) / ✓ Schema in sync / ⚠ [anything needing attention]"

---

## Branch & PR Naming Convention

### Branches
```
<type>/<short-description>

Types: feat, fix, hotfix, chore, refactor, test, infra, docs

Examples:
  feat/ai-chat-tab
  fix/transcript-parse-error
  hotfix/prod-session-crash
  chore/dep-updates-april
  infra/daily-backup-cron
```

### PR Titles
```
<type>: <imperative sentence under 60 chars>

Examples:
  feat: add AI chat tab to all user tiers
  fix: correct balanced-brace parser for Claude responses
  hotfix: fix session crash on iOS background
  chore: update expo-audio to SDK 54.1
```

### Commit Messages
```
<type>(<scope>): <short summary>

<optional body — why, not what>

Examples:
  feat(echo-mobile): add pause/resume recording
  fix(sensei-webapp): parseLiteLLMJson balanced-brace walker
  hotfix(echo-backend): prevent stale session on crash recovery
```

---

## LLM Model Selection

Match model to task. Use the cheapest model that gets the job done.

| Task | Model | Why |
|---|---|---|
| Code generation, complex reasoning, architecture | `claude-4-6-opus` | Highest quality, use when correctness matters most |
| Most coding tasks, analysis, debugging | `claude-4-6-sonnet` | Best speed/quality balance — default for most work |
| Simple tasks, summaries, one-shot classification | `claude-4-5-haiku` | Fast and cheap — use for high-volume or low-stakes calls |
| POC extraction from full transcripts | `gemini-2.5-pro` | 1M context window for single-pass full transcript |
| AI agent subagents, background workers | `claude-4-5-haiku` | Cost matters when many parallel calls |
| Transcription | Groq Whisper (`whisper-large-v3-turbo`) | $0.004/min, fast, accurate enough |

**Rule:** never use Opus where Sonnet will do. Never use Sonnet where Haiku will do. The LiteLLM proxy routes these — model names must match the proxy aliases in `CLAUDE.md`.

---

## Rollback Procedure

When a deployment breaks production:

```
1. Identify — confirm prod is broken (check Vercel logs, error logs in /admin/error-log)
2. Decide — revert or hotfix?
   - Revert if: the last deploy introduced the regression and it's not trivial to patch
   - Hotfix if: fix is a 1-3 line change you can verify in under 10 minutes
3. Revert (Vercel):
   - Vercel dashboard → Deployments → previous working deployment → "Redeploy"
   - OR: git revert HEAD → push → Vercel auto-deploys
4. Hotfix (code path):
   - Branch from main: git checkout -b hotfix/<description>
   - Apply minimal fix
   - Run full test suite + TypeScript check
   - Push → deploy → verify with post-deployment smoke test
5. Post-mortem (brief):
   - What broke, why, how it was caught
   - Add regression test before closing the hotfix branch
   - Update SESSION_NOTES.md with incident summary
```

**Database rollback:** If a schema change broke prod and there's no automated rollback:
1. Never run `prisma db push --accept-data-loss`
2. Write a targeted SQL migration to reverse the change
3. Apply via `psql` or Supabase SQL editor directly
4. Then fix the Prisma schema to match

---

## Task Type Pipelines

Match the task type before starting. Every pipeline ends the same way: iterate internally until all gates pass, then present once.

---

### Bug Fix
> Trigger: something is broken or behaving incorrectly

**Active personas: 💻 Engineer + 🔐 Security (if auth/data) + SDET**

```
1. Logs first — read server logs / browser console / error log DB before opening any file
2. Root cause — identify the source, not the symptom
3. Write failing test — test must fail before fix
4. Fix the root cause — never patch over symptoms
5. Run full test suite — fix all failures
6. TypeScript clean
7. Persona pipeline (Phase 1-4 minimum)
8. Present to owner
```

---

### Hotfix (Prod Emergency)
> Trigger: production is actively broken, users are affected — speed matters

**Active personas: 💻 Engineer + 🔐 Security**

```
1. Confirm prod is broken — check Vercel logs, /admin/error-log, or user report
2. Identify the exact failure — logs first, no guessing
3. Branch: git checkout -b hotfix/<description>
4. Apply the minimal fix — no refactoring, no cleanup, no improvements
5. TypeScript clean + run failing test path only (not full suite — time matters)
6. Push, deploy, verify with post-deployment smoke test (see below)
7. Write regression test AFTER stabilizing (not before — the regression test is the task after)
8. Run full test suite on the regression test branch
9. Merge regression test to main
10. Post-mortem: one paragraph in SESSION_NOTES.md — what broke, why, how caught
```

**Approval tier:** Quick Fix — one approval (commit + push together if clearly scoped).

---

### Feature
> Trigger: new capability requested

**Active personas: 📋 PM + 💻 Engineer + 🎨 Designer + 🔐 Security**

```
1. Read SESSION_NOTES.md and ROADMAP.md for context
2. Scoping — describe understanding, list use cases/edge cases, define "done", get confirmation
3. Write failing tests (test list first, then failing test code)
4. Implement — minimum code to make tests pass
5. Run full test suite
6. TypeScript clean
7. Visual verification (Playwright) if visual output
8. Full persona pipeline
9. Surface sync — run the Surface Sync Checklist (release notes, tour, hero, wizard, prompts, CLAUDE.md)
10. Post-deployment smoke test (after deploy)
11. Present to owner
```

---

### Code Review
> Trigger: "review [project]" or "code review"

**Active personas: 💻 Engineer + 🔐 Security Architect + SDET**

```
1. Read SESSION_NOTES.md for current state
2. Explore via subagents (Phase 1: understand codebase, don't write code)
3. Findings report — categorized by severity (Critical/High/Medium/Low)
4. Full persona pipeline review
5. Present findings to owner
6. For each fix approved:
   → Write failing test → Fix → Run suite → Gate passes → Continue
7. Present completed fixes
```

---

### Debug / Investigation
> Trigger: "why is X happening" / unexplained behavior

**Active personas: 💻 Engineer + 🔐 Security (if auth/data path)**

```
1. Logs first — always. Check error logs, server logs, browser console
2. Identify the exact failure point (not a guess — find it in the code)
3. Root cause analysis — trace back to source
4. Fix the root cause (not a workaround)
5. Write regression test
6. Run full suite
7. Present diagnosis + fix
```

---

### Refactor
> Trigger: restructuring existing code without changing behavior

**Active personas: 💻 Engineer + 🏗️ CTO**

```
1. Establish test coverage baseline — existing tests must pass before and after
2. Scope — define exactly what changes and what stays the same
3. Get confirmation
4. Refactor in small steps, running tests after each
5. Full suite must pass at end
6. TypeScript clean
7. Present
```

---

### Infrastructure / Schema / Deployment
> Trigger: env vars, Prisma schema, CI/CD, deployment

**Active personas: 🏗️ CTO + 💻 Engineer + 🔐 Security**

```
1. Read CLAUDE.md deployment section for current state
2. Plan the change explicitly — write it out
3. Get confirmation before executing
4. Dry-run if possible (prisma db push --dry-run, etc.)
5. Execute change
6. Verify (test the affected path end-to-end)
7. Post-deployment smoke test (see below)
8. Document: update CLAUDE.md and SESSION_NOTES.md
9. Atomic: schema change + db push + commit in one step
10. Know the rollback path before executing — if it fails, what's step 1?
```

### Post-Deployment Smoke Test (run after every deploy)

```bash
# 1. App loads
curl -s -o /dev/null -w "%{http_code}" https://[app-url]  # expect 200

# 2. Auth works — attempt login (manual or Playwright smoke)
npx playwright test --project=smoke  # sensei-webapp

# 3. Core data path — load a notebook node or recording
# 4. AI route responds — trigger one AI analysis, check for 200 + valid JSON
# 5. Check /admin/error-log for new errors in last 5 minutes
```

State: "Post-deploy: ✓ App loads / ✓ Auth works / ✓ Core data loads / ✓ No new errors"

---

### Dependency Update
> Trigger: "update packages", security advisory, or scheduled maintenance

**Active personas: 💻 Engineer + 🔐 Security**

```
1. Run npm audit — list all vulnerabilities, prioritize Critical/High
2. Check breaking changes — read changelogs for any package bumping a major version
3. Update in groups, not all at once:
   - Security patches (patch/minor) — update first, test immediately
   - Dev dependencies — update separately from runtime deps
   - Major version bumps — one at a time, full test suite after each
4. After each group:
   - npx tsc --noEmit (zero errors)
   - Full test suite (vitest run or jest --forceExit)
   - Manual smoke: does the app start? Does the critical path work?
5. Commit group by group — not one massive "update all packages" commit
6. Document: note any changed APIs or migration steps in SESSION_NOTES.md
```

---

### Documentation / Context Update
> Trigger: "update context", "sync docs", "add to roadmap"

**Active personas: 📋 PM**

```
1. Read all relevant context files first
2. Identify gaps and contradictions
3. Update consistently across all locations
4. Cross-check: no conflicting information
5. Present
```

---

### Product (Idea → Shipped)
> Trigger: new idea, new product area, new feature at concept stage — anything that starts as "what if we..." or "I want to build..." or "I have an idea for..."

**This pipeline owns the full lifecycle from raw idea to shipped and documented product.**

#### Phase 1 — Capture & Discovery
**Personas: 📋 PM + 📈 Growth**

```
1. [PM] Capture the raw idea — "Problem (1 sentence). Solution (1 sentence). Who (1 sentence)."
2. [PM] Run the 5 discovery questions:
   - "Who is this for and what problem does it solve?"
   - "What does success look like? How will we know it worked?"
   - "Are there constraints I should know? (time, tech, privacy, cost)"
   - "What's the simplest version that delivers the core value?"
   - "What would someone use instead if this didn't exist?"
3. [Growth] Positioning — name 3 alternatives, one clear differentiator, target user in one sentence
→ OWNER GATE: present problem + positioning framing → go/no-go before any further work
```

#### Phase 2 — Strategy
**Personas: 📋 PM + 🏗️ CTO + 📈 Growth**

```
4. [PM] MVP scope — MoSCoW:
   Must Have (blocks launch) / Should Have (important but not blocking) / Won't Have (explicit cut)
5. [CTO] Risk register — top 3 risks + mitigation for each
6. [Growth] GTM sketch — how do users discover this? Primary distribution channel. One hook.
→ OWNER GATE: present strategy doc (scope + risks + GTM) → get approval before any design or code
```

#### Phase 3 — User Stories
**Personas: 📋 PM + 🎨 Designer**

```
7. [PM] Write user stories:
   "As a [user type], I want [action], so that [outcome]."
   - Group into epics
   - Acceptance criteria for each story (testable, specific)
   - Priority: P0 (must ship) / P1 (should ship) / P2 (backlog)
8. [Designer] UX sketch — wireframe or written flow description for each P0 story
→ OWNER GATE: present stories + UX → get approval before technical design
```

#### Phase 4 — Technical Design
**Personas: 🏗️ CTO + 💻 Engineer + 🔐 Security**

```
9. [CTO + Engineer] Technical design:
   - Data model changes (new tables/fields)
   - API surface (new routes, modified routes)
   - Key components (new files, modified files)
   - Sequence: what ships in v1, what's explicitly deferred
10. [Security] Security review of the design:
    - Trust boundaries, auth scope, sensitive data handling
    - Encryption requirements (any PII? any user content?)
11. Write the failing test list (before any code): list every test case that will need to pass
→ OWNER GATE: present technical design → get approval before any implementation
```

#### Phase 5 — Build
**Personas: 💻 Engineer + 🎨 Designer**

```
12. Write failing tests (from the test list in step 11) — tests must fail before code is written
13. Implement — minimum code to make tests pass (no gold-plating)
14. Gates (run after each logical unit, not just at end):
    - npx tsc --noEmit — zero errors
    - Full test suite — all green
15. [Designer] Visual verification if any UI:
    - Playwright screenshot
    - Responsive check, empty states, error states
```

#### Phase 6 — QA & Persona Pipeline
**All relevant personas**

```
16. Full persona pipeline — Phases 1-5 minimum (see Persona Pipeline section)
17. Surface sync checklist — release notes, tour, hero, wizard, prompts, CLAUDE.md
18. Post-deployment smoke test plan ready
```

#### Phase 7 — UAT & Iterate
**Personas: The User + The Skeptic + 📋 PM**

```
19. → OWNER GATE: UAT
    Owner validates against original requirements (Phase 3 user stories)
20. Capture feedback — triage immediately:
    - P0: blocks ship — fix before moving on
    - P1: ship with known limitation — log in SESSION_NOTES.md
    - P2: backlog — add to ROADMAP.md
21. Fix all P0s → re-run Phase 5 gates → back to owner UAT
22. Ship when P0 count = 0
```

#### Phase 8 — Finalize & Ship
**Personas: 📋 PM + 📈 Growth**

```
23. Deploy
24. Post-deployment smoke test (see Infrastructure section)
25. [PM] Update ROADMAP.md — mark shipped, move P2s to backlog
26. Release note created
27. Final surface sync verification (no stale surfaces)
28. Update SESSION_NOTES.md — What's Next + session entry
29. Commit + push (two approvals)
30. Sync to workspace-docs
```

---

## Per-Task-Type Persona Pipeline Quick Reference

| Task Type | Phase 1 (Code Quality) | Phase 2 (Functional) | Phase 3 (Integration) | Phase 4 (Security) | Phase 5 (Specialist) |
|---|---|---|---|---|---|
| **Bug Fix** | SDET, Static Analyzer | Sanity, Functional, API | Integration, Regression | Security, Pen Test | Observability |
| **Hotfix** | Static Analyzer | Sanity only | — | Security (if auth path) | — |
| **Feature** | SDET, Static Analyzer | Sanity, Functional, API | Integration, Data Integrity, Regression | Security, Pen Test, Privacy | AI Quality (if AI), Performance, Human Voice |
| **Product** | SDET, Static Analyzer | Sanity, Functional, API | Integration, Data Integrity, Regression | Security, Pen Test, Privacy, Compliance | AI Quality, Performance, Observability, Human Voice, Visual |
| **Refactor** | SDET, Static Analyzer | Sanity, Functional | Regression | Security (if touching auth/data) | Performance |
| **Infra/Schema** | Static Analyzer | Sanity | Integration, Data Integrity | Security | Observability |
| **Debug** | Static Analyzer | Sanity, Functional | Integration | Security (if auth path) | Observability |
| **Dependency Update** | Static Analyzer | Sanity | Regression | Security | — |

---

## Tiered Approval

| Tier | Examples | Approval |
|---|---|---|
| **Quick fix** | Copy change, config value, style tweak | One approval (commit + push together if low risk) |
| **Feature** | New component, API route, page | Two approvals: commit, then push |
| **Major** | Auth changes, schema, security, new product area | Two approvals + architecture sign-off before implementation |

---

## Self-Correcting Pipeline

This is the contract. Internal loop — owner never sees a broken state.

```
START TASK
    ↓
Read context (SESSION_NOTES, CLAUDE.md, MEMORY.md for lock status)
    ↓
[Gate 0] Active project confirmed? Locked projects untouched?
    No → Stop and confirm project scope with owner
    Yes ↓
[Gate 1] TypeScript clean?
    No → Fix TypeScript errors → retry Gate 1
    Yes ↓
[Gate 2] Failing test written (before code)?
    No → Write it → retry Gate 2
    Yes ↓
Implement fix/feature
    ↓
[Gate 3] Full test suite green?
    No → Fix failures → retry Gate 3
    Yes ↓
[Gate 4] Persona pipeline passed?
    No → Fix each persona failure → retry Gate 4
    Yes ↓
[Gate 5] Visual verified (if applicable)?
    No → Iterate on visual → retry Gate 5
    Yes ↓
[Gate 6] Surface sync complete?
    No → Update release notes / tour / hero / prompts → retry Gate 6
    Yes ↓
Update SESSION_NOTES.md
    ↓
PRESENT TO OWNER — once, working
    ↓
[Gate 7] Post-deployment smoke test (after deploy)
    No → Rollback or hotfix → verify → re-run Gate 7
    Yes ↓
DONE
```

---

## Persona Pipeline

Run through all applicable personas before presenting. Only come to the owner at UAT.

### Phase 1 — Code Quality
| Persona | Gate |
|---|---|
| **SDET** | Tests written first. Full suite passes. Coverage doesn't regress. |
| **Static Analyzer** | `tsc --noEmit` zero errors. ESLint clean. No unjustified `any`. |

### Phase 2 — Functional
| Persona | Gate |
|---|---|
| **Sanity Tester** | Happy path runs end-to-end. If this fails, nothing else runs. |
| **Functional QA** | Every acceptance criterion verified. |
| **API Contract Tester** | All status codes, input validation, auth enforcement tested. |

### Phase 3 — Integration & Regression
| Persona | Gate |
|---|---|
| **Integration Tester** | Full data flow correct. DB queries right. Encryption round-trips. AI calls return expected shape. |
| **Data Integrity Tester** | Cascade deletes, DB constraints, no orphaned records. |
| **Regression Tester** | Full suite passes. Nothing previously working is broken. |

### Phase 4 — Security & Compliance
| Persona | Gate |
|---|---|
| **Security Architect** | Trust boundaries, key management, auth scope, data model reviewed. |
| **Pen Tester** | Every input validated. Auth on every route. No data leakage. Prisma prevents injection. |
| **Privacy Reviewer** | Sensitive fields encrypted. Scoped to user/org. PII not in logs. Deletion cascades. |
| **Compliance Reviewer** | GDPR right-to-erasure path exists. Privacy claims match implementation. |

### Phase 5 — Specialist
| Persona | When | Gate |
|---|---|---|
| **AI Quality Reviewer** | Any AI involved | Test with short/noisy/empty inputs. Output shape valid. Prompt injection hardened. |
| **Performance / Load Tester** | Always | No N+1 queries. Indexes on filtered columns. LLM calls async. |
| **Observability Reviewer** | Always | Errors logged with context. No silent failures. Key operations traceable. |
| **Mobile QA** | Mobile code | Reviewed for mobile-specific paths. |
| **Human Voice Reviewer** | Any user-facing string | No AI tells. No em dashes. Short sentences. Direct language. Actionable error messages. |
| **Visual Regression Tester** | Visual output | Playwright screenshot verified. No overflow. Correct layout. |

### Phase 6 — UAT (owner gate)
| Persona | Gate |
|---|---|
| **The User** | Would someone with no context understand this? |
| **The Skeptic** | Tried to break it: empty states, rapid input, bad data. |
| **UAT** | Owner validates against original requirement. |

---

## App-Specific Personas

### senSEi
| Persona | Gate |
|---|---|
| **The SE on a call** | Usable in under 30 seconds under pressure. Zero explanation needed. |
| **The SE Manager** | Team activity visible. Data makes sense at a glance. |

### Echo
| Persona | Gate |
|---|---|
| **The Privacy-conscious User** | Data encrypted where promised. Deletable. Nothing leaks to third parties. |
| **The Mobile-first User** | Core value accessible without opening the webapp. |

---

## Playwright Visual Testing

Required for: PDF/print export, generated documents, new UI components, charts, tables.

```js
const { chromium } = require('@playwright/test');
const browser = await chromium.launch();
const page = await browser.newPage();
await page.goto('file:///tmp/test.html', { waitUntil: 'networkidle' });
await page.emulateMedia({ media: 'print' });  // for print layout
await page.screenshot({ path: '/tmp/output.png', fullPage: true });
await browser.close();
```

Checklist before marking visual as done:
- [ ] No blank pages
- [ ] No orphaned headings
- [ ] No content overflow
- [ ] Text readable
- [ ] Margins balanced
- [ ] Tables don't split mid-row

---

## What "Ready to Present" Means

- All automated checks pass
- All persona gates cleared
- Visual features Playwright-verified
- Post-deployment smoke test passed
- Any "needs real environment" items explicitly flagged
- Owner sees it once — working
