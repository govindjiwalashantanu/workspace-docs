# Working Protocol

Applies to: senSEi webapp, senSEi mobile, Echo backend, Echo mobile.
Last updated: March 2026.

---

## Hook System

Three hooks enforce this protocol automatically in every session:

| Hook | When | What it does |
|---|---|---|
| `SessionStart` | On session open | Loads project context, shows What's Next, git status |
| `UserPromptSubmit` | Before every prompt | Injects protocol gates as a reminder |
| `Stop` | After every response | Checks for modified files, reminds to update context + commit |

If hooks aren't firing: run `/hooks` once to reload config.

---

## Core Rules

1. **Context first, always.** Before writing a single line of code, read SESSION_NOTES.md (last 80 lines). If you don't know where we left off, you're not ready to start.
2. **Tests first, no exceptions.** Write the failing test before implementation. The test must fail before any code is written.
3. **Fix root causes. Never write workarounds.** If something is broken, trace it to the source and fix it there. Do not add guards, patches, or overrides that mask the underlying issue. If stale data is causing problems, clear the data — don't code around it.
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
14. **Update context files every session.** CLAUDE.md (if architecture changed), SESSION_NOTES.md (what was built, what's next), ROADMAP.md (mark completed, add newly scoped). This is not optional.

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

## Task Type Pipelines

Match the task type before starting. Every pipeline ends the same way: iterate internally until all gates pass, then present once.

### Bug Fix
> Trigger: something is broken or behaving incorrectly

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

### Feature
> Trigger: new capability requested

```
1. Read SESSION_NOTES.md and ROADMAP.md for context
2. Scoping — describe understanding, list use cases/edge cases, define "done", get confirmation
3. Write failing tests (test list first, then failing test code)
4. Implement — minimum code to make tests pass
5. Run full test suite
6. TypeScript clean
7. Visual verification (Playwright) if visual output
8. Full persona pipeline
9. Present to owner
```

### Code Review
> Trigger: "review [project]" or "code review"

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

### Debug / Investigation
> Trigger: "why is X happening" / unexplained behavior

```
1. Logs first — always. Check error logs, server logs, browser console
2. Identify the exact failure point (not a guess — find it in the code)
3. Root cause analysis — trace back to source
4. Fix the root cause (not a workaround)
5. Write regression test
6. Run full suite
7. Present diagnosis + fix
```

### Refactor
> Trigger: restructuring existing code without changing behavior

```
1. Establish test coverage baseline — existing tests must pass before and after
2. Scope — define exactly what changes and what stays the same
3. Get confirmation
4. Refactor in small steps, running tests after each
5. Full suite must pass at end
6. TypeScript clean
7. Present
```

### Infrastructure / Schema / Deployment
> Trigger: env vars, Prisma schema, CI/CD, deployment

```
1. Read CLAUDE.md deployment section for current state
2. Plan the change explicitly — write it out
3. Get confirmation before executing
4. Dry-run if possible (prisma db push --dry-run, etc.)
5. Execute change
6. Verify (test the affected path end-to-end)
7. Document: update CLAUDE.md and SESSION_NOTES.md
8. Atomic: schema change + db push + commit in one step
```

### Documentation / Context Update
> Trigger: "update context", "sync docs", "add to roadmap"

```
1. Read all relevant context files first
2. Identify gaps and contradictions
3. Update consistently across all locations
4. Cross-check: no conflicting information
5. Present
```

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
Read context (SESSION_NOTES, CLAUDE.md)
    ↓
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
Update SESSION_NOTES.md
    ↓
PRESENT TO OWNER — once, working
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
- Any "needs real environment" items explicitly flagged
- Owner sees it once — working
