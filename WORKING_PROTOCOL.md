# Working Protocol

Applies to: senSEi webapp, senSEi mobile, Echo backend, Echo mobile.
Last updated: March 2026 (added Playwright visual testing requirement + subagent usage rule).

---

## Core Rules

1. **Ask before writing anything.** State what you understand the task to be. Get confirmation before touching code.
2. **Test-first development.** Write failing tests before implementation. No exceptions.
3. **Never push without explicit approval.** Every GitHub push triggers CI/CD and costs money. Stage locally, present the diff, wait for go-ahead. Two separate approvals: commit, then push.
4. **Never present broken work.** Run the full pipeline internally before surfacing anything. If it fails any gate, fix it first.
5. **Treat the owner's time as the most valuable resource.** Iterate until it works. Surface it once.
6. **Visual components require Playwright verification.** Any feature that produces a visual output — PDF export, print view, generated document, chart, modal, UI component — must be screenshotted with Playwright and visually verified before being presented. Iterate on the screenshot until the layout is correct. Never ask the owner to be the first person to see it.
7. **Use subagents aggressively to preserve context.** Offload all research, exploration, and multi-file analysis to subagents (Explore, Plan, general-purpose). The main context window is expensive — subagents protect it. Run multiple subagents in parallel where tasks are independent. Never search, read, or grep large codebases in the main thread when a subagent can do it.
8. **Run /compact when context gets large.** If the conversation is getting long and responses are slowing down, run `/compact` to summarise context before continuing. Do not wait to be asked.
9. **Update context files at the end of every session, without being asked.** Before closing any session, update the relevant files:
   - `CLAUDE.md` — if architecture, deployment, env vars, or key patterns changed
   - `SESSION_NOTES.md` — what was built, decisions made, commit hashes, what to pick up next
   - `ROADMAP.md` — mark completed features, add newly scoped work
   This applies to all repos: sensei-webapp, echo-backend, echo-mobile.

---

## Feature Workflow

```
1. Scoping
2. Tests first (failing)
3. Implementation (make tests pass)
4. Visual verification with Playwright (if visual output)
5. Full persona pipeline
6. Present to owner
7. Owner approves → commit
8. Owner approves → push
```

### 1. Scoping
Before any code:
- Describe what you understand the feature to be
- List all use cases and edge cases
- State what "done" looks like
- Get confirmation

### 2. Tests First
- Write the test list (not code yet) — happy path, error states, edge cases
- Present the test list for review if the feature is significant
- Write the failing tests
- Run them — they must fail before implementation starts

### 3. Implementation
- Write minimum code to make tests pass
- No abstractions built for hypothetical future needs
- No scope creep beyond what was agreed

---

## Persona Pipeline

Every feature runs through all applicable personas before being presented.
Only come to the owner at the **UAT** gate.

### Phase 1 — Code Quality
| Persona | How applied |
|---|---|
| **SDET** | Tests written first, full suite passes, coverage doesn't regress |
| **Static Analyzer** | `tsc --noEmit` zero errors, ESLint clean, no unjustified `any` |

### Phase 2 — Functional
| Persona | How applied |
|---|---|
| **Sanity Tester** | Happy path runs end-to-end. If this fails, nothing else runs. |
| **Functional QA** | Every acceptance criterion in the feature brief verified |
| **API Contract Tester** | All status codes, input validation, auth enforcement tested |

### Phase 3 — Integration & Regression
| Persona | How applied |
|---|---|
| **Integration Tester** | Full data flow traced: DB queries correct, encryption round-trips, AI calls return expected shape |
| **Data Integrity Tester** | Cascade deletes, DB constraints, migration safety, no orphaned records |
| **Regression Tester** | Full test suite passes. Nothing previously working is broken. |

### Phase 4 — Security & Compliance
| Persona | How applied |
|---|---|
| **Security Architect** | Trust boundaries, key management, auth scope, data model reviewed for structural flaws |
| **Security / Pen Tester** | Every input validated, auth on every route, no data leakage, Prisma prevents injection |
| **Privacy Reviewer** | Sensitive fields encrypted, scoped to user/org, PII not in logs, deletion cascades |
| **Compliance Reviewer** | GDPR right-to-erasure path exists, privacy claims match implementation |

### Phase 5 — Specialist
| Persona | How applied |
|---|---|
| **AI Quality Reviewer** | Test with short/noisy/empty inputs, output shape always valid, prompt injection hardened *(only if AI involved)* |
| **Performance / Load Tester** | No N+1 queries, indexes on filtered columns, LLM calls async, P95 acceptable |
| **Observability Reviewer** | Errors logged with context, no silent failures possible, key operations traceable |
| **Mobile QA** | Code reviewed for mobile-specific paths. Flag if device test needed before ship. |
| **Cross-browser / Platform** | Reviewed for known Safari/Firefox issues. Flag if browser test needed before ship. |
| **Accessibility Tester** | Color contrast, ARIA labels, keyboard nav checked. Flag if screen reader test needed. |
| **Copy Reviewer** | All user-facing strings clear, no typos, error messages actionable, consistent tone |
| **Visual Regression Tester** | *(applies whenever the feature produces visual output)* Use Playwright to screenshot the rendered output — PDF, print view, generated document, UI component, modal. Verify: no overflow, no blank pages, correct margins, correct layout, readable text. Iterate on the screenshot until it looks correct before proceeding to UAT. Do not ask the owner to be the first person to see the visual output. |

### Phase 6 — User Acceptance ← only gate where owner is involved
| Persona | How applied |
|---|---|
| **The User** | Would someone with no context understand this without explanation? |
| **The Skeptic** | Deliberately tried to break it: empty states, rapid input, network drops, bad data |
| **UAT** | Feature presented to owner. Owner validates against original requirement. |

### Conditional
| Persona | When |
|---|---|
| **SEO Auditor** | Only when touching public marketing pages (`/`, `/pricing`, `/docs`, `/blog`). Check meta, canonical, structured data, Core Web Vitals. |
| **Dependency Reviewer** | Quarterly. Flag any dependency with known CVE. |

---

## App-Specific Personas

### senSEi
| Persona | Checks |
|---|---|
| **The SE on a call** | Usable in under 30 seconds under pressure, zero explanation needed |
| **The SE Manager** | Team activity visible, data makes sense at a glance |

### Echo
| Persona | Checks |
|---|---|
| **The Privacy-conscious User** | Data encrypted where promised, deletable, nothing leaks to third parties |
| **The Mobile-first User** | Core value accessible without ever opening the webapp |

---

---

## Playwright Visual Testing

Triggered whenever a feature produces visual output. `@playwright/test` is already installed.

### When to use
- PDF / print export (e.g. POC Guide, digest)
- Generated documents (.docx, .pdf)
- New UI components or major UI changes
- Print views or dedicated print pages
- Charts, tables, or data visualisations

### How to use

**For pages that don't require auth (static HTML or public routes):**
```js
const { chromium } = require('@playwright/test');
const browser = await chromium.launch();
const page = await browser.newPage();
await page.goto('file:///tmp/test.html', { waitUntil: 'networkidle' });
await page.emulateMedia({ media: 'print' });  // for print layout
await page.screenshot({ path: '/tmp/output.png', fullPage: true });
await page.pdf({ path: '/tmp/output.pdf', format: 'Letter', printBackground: true });
await browser.close();
```

**For pages that require auth:** Create a minimal standalone HTML file that replicates the CSS and sample content of the component, then render that.

### Checklist before marking visual as done
- [ ] No blank pages (every page has meaningful content)
- [ ] No orphaned headings (section title at bottom of page with content on next)
- [ ] No content overflow past page margins
- [ ] Text is readable (not too small after scaling)
- [ ] Margins are balanced — not so wide they squeeze content
- [ ] Tables don't split mid-row across pages
- [ ] Layout matches the design intent

---

## What "Ready to Present" Means

All automated checks pass.
All code-review personas cleared.
For visual features: Playwright screenshot verified and layout approved internally.
Any "needs real environment" items explicitly flagged.
Owner sees it once — working.
