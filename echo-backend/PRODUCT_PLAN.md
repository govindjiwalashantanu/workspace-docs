# Echo — Product Plan
_Last updated: March 17, 2026_

---

## Philosophy

Ship the minimum that lets someone record a thought, get AI structure back, and pay for it. Validate the idea. Grow the story when it works.

---

## Positioning

**Don't pigeonhole it.** Echo is not "Otter for private thoughts" — that's a competitive positioning move, not a product definition. It works for any conversation worth remembering: your own thoughts, a meeting, a coaching session, a 1:1, a lecture.

**The real differentiator:** Privacy + structure. No competitor owns the high-structure + high-privacy quadrant.

**Working one-liner:** *Private AI for any conversation worth remembering.*

---

## Tiers

| Tier | Price | Annual | Hours/mo | Purpose |
|---|---|---|---|---|
| Free | $0 | — | 3 hrs | Hook — deliver the aha moment, not enough to stay |
| Personal | $7.99/mo | $59.99/yr | 10 hrs | Everyday thought capture |
| Thinker | $14.99/mo | $119/yr | 20 hrs | Power user — founders, ADHD, coaches, researchers |
| Pro | $24.99/mo | $199/yr | 50 hrs | Privacy-obsessed professional |

**At launch: Free + Personal only.** Thinker and Pro go live in Phase 2 when the features that justify them are built.

**Annual-first default** on upgrade screen. Present as "2 months free" or "$5/mo billed annually". The pay-for-8 framing no longer holds exactly at $59.99/yr.

**Founding Member (optional, first 500 users):**
- Personal Founding: $5.99/mo locked forever
- Thinker Founding: $9.99/mo locked forever
Hard cap at 500. Scarcity is the mechanism.

---

## The USP

**Encryption throughout — not as a feature, as a promise.**

Per-user AES-256-GCM encryption on every piece of content a user creates. Not just utterances. Everything. Even Echo cannot read your data without your key. This is the thing no competitor can credibly claim. It's technically verifiable, not just a policy statement. It must be true at launch — a partial implementation breaks the promise entirely.

---

## MVP — Minimum to Ship and Take Real Money

These things. Nothing else.

| # | What | Why it blocks launch |
|---|---|---|
| 1 | Unhardcode `isPro = true` in `lib/store/billing-store.ts:23` (echo-mobile) | Every user is on Pro for free |
| 2 | Enforce free tier at session creation — 3 hrs, summary only, 7-day history lock | Can't charge without a real free tier |
| 3 | Fix forgot password — `app/(auth)/forgot-password.tsx:32` (echo-mobile) | App Store rejects broken auth flows |
| 4 | Mic permission denial — handle gracefully, no crash | App Store rejects on crash |
| 5 | Transcription failure mid-recording — surface error to user | Silent failure = 1-star reviews on day one |
| 6 | App icon PNGs — export from updated SVG (1024×1024, splash, Android variants) | Currently shows old SE mark |
| 7 | Privacy settings — account deletion + raw data export | App Store requirement. GDPR. Non-negotiable. |
| 8 | Rate limiting on `/api/transcribe` and `/api/agent/reflect` | Open to abuse on launch day |
| 9 | Encrypt `NotebookNode.content` on write + decrypt on all read paths | Plaintext transcript in DB breaks the zero-knowledge promise |
| 10 | Encrypt `SpeakerProfile.name` and `email` on write + decrypt on read | Same — partial encryption is a broken promise |
| 11 | Migration script for existing unencrypted rows | Any existing data must be encrypted before launch |
| 12 | Webapp → hero page only. `/` shows name + iOS/Android CTAs. All other page routes redirect to `/`. Turn off stream/assist/agent routes. | Reduces surface area, domain stays useful |
| 13 | Free tier UI — lock action items, key ideas, follow-up question with upgrade prompt | Free users must see what they're missing, not just missing it silently |
| 14 | Free tier UI — session history shows 7 days, older sessions appear locked (not deleted) | The locked history is an upgrade trigger |
| 15 | Free tier UI — search locked | Consistent with tier rules |
| 16 | Hour limit UI — block new recording at 3 hrs, show upgrade screen | Without this the hard cap has no UX |
| 17 | Hour warning — notify at 80% and 95% of monthly hours | Gives users time to upgrade before hitting the wall |
| 18 | Upgrade screen — Personal tier, annual-first, "Pay for 8 months, get 12" | The conversion surface |

**Onboarding:** Include a minimal single-screen explainer of the core loop. Not a full tour — just enough that a new user understands what to do.

---

## Feature Matrix — Launch (Phase 1)

Two tiers only. Nothing promised that isn't built.
Thinker and Pro do not appear in the UI at launch.

| Feature | Free | Personal |
|---|:---:|:---:|
| | $0 | $7.99/mo · $59.99/yr |
| **RECORDING** | | |
| Hours/month | 3 hrs | 10 hrs |
| Overage | Hard stop | Hard stop |
| **AI REFLECTION** | | |
| Transcription | ✓ | ✓ |
| Summary | ✓ | ✓ |
| Action items | ✗ | ✓ |
| Key ideas | ✗ | ✓ |
| Follow-up question | ✗ | ✓ |
| **HISTORY & SEARCH** | | |
| Session history | 7 days | Full |
| Search across sessions | ✗ | ✓ |
| **PRIVACY** | | |
| AES-256 encryption | ✓ | ✓ |
| No AI training on your data | ✓ | ✓ |
| Account deletion | ✓ | ✓ |
| Raw data export | ✗ | ✓ |
| **PLATFORM** | | |
| iOS + Android | ✓ | ✓ |
| Web dashboard | — | — |

---

## Feature Matrix — Full (Phase 2+)

Thinker and Pro go live when the features that justify them are built.

| Feature | Free | Personal | Thinker | Pro |
|---|:---:|:---:|:---:|:---:|
| | $0 | $7.99/mo | $14.99/mo | $24.99/mo |
| **RECORDING** | | | | |
| Hours/month | 3 hrs | 10 hrs | 20 hrs | 50 hrs |
| Overage | Hard stop | Hard stop | Hard stop | $0.05/min |
| **AI REFLECTION** | | | | |
| Transcription | ✓ | ✓ | ✓ | ✓ |
| Summary | ✓ | ✓ | ✓ | ✓ |
| Action items | ✗ | ✓ | ✓ | ✓ |
| Key ideas | ✗ | ✓ | ✓ | ✓ |
| Follow-up question | ✗ | ✓ | ✓ | ✓ |
| Patterns (cross-session themes) | ✗ | ✗ | ✓ | ✓ |
| Custom reflection protocols | ✗ | ✗ | ✓ | ✓ |
| Weekly insight digest | ✗ | ✗ | ✓ | ✓ |
| Speaker diarization | ✗ | ✗ | ✗ | ✓ |
| Priority AI processing | ✗ | ✗ | ✗ | ✓ |
| **HISTORY & SEARCH** | | | | |
| Session history | 7 days | Full | Full | Full |
| Search across sessions | ✗ | ✓ | ✓ | ✓ |
| Thought continuity (session linking) | ✗ | ✗ | ✓ | ✓ |
| **PRIVACY** | | | | |
| AES-256 encryption at rest | ✓ | ✓ | ✓ | ✓ |
| No AI training on your data | ✓ | ✓ | ✓ | ✓ |
| Account deletion | ✓ | ✓ | ✓ | ✓ |
| Raw data export | ✗ | ✓ | ✓ | ✓ |
| Forgetting mode (auto-delete after N days) | ✗ | ✓ | ✓ | ✓ |
| Scramble Mode | ✗ | ✗ | ✗ | ✓ |
| **IMPORT** | | | | |
| Import from Notion / Obsidian / Markdown | ✗ | ✗ | ✓ | ✓ |
| Export to third-party apps | ✗ | ✗ | ✗ | ✗ |
| **PLATFORM** | | | | |
| iOS + Android | ✓ | ✓ | ✓ | ✓ |
| Web — live companion (Phase 2) | ✗ | ✗ | ✗ | ✓ |
| Offline recording | ✗ | ✓ | ✓ | ✓ |

**Export to third-party apps is deliberately off.** Data gravity is the moat. Review when the product is proven.

---

## Hour Limit Behaviour

- **Never cut off a live session mid-recording.** Once started, let it finish. Block starting new sessions, not completing current ones.
- **Warn at 80% and 95% of monthly hours** via push notification / in-app banner.
- **At the limit:** show a full upgrade screen on next recording attempt — not a toast.
- **Monthly hours reset** on the same calendar day each month. Notify user when hours refresh.
- **Overage for Personal:** $0.99/hr after 10 hrs. Shown on the landing page. Removes anxiety about hitting the cap without requiring a full tier upgrade. At Groq pricing ($0.004/min = $0.24/hr), margin is ~76%. Clean and simple.

---

## Upgrade Triggers

| User action | What they see |
|---|---|
| Hits 3 hrs in month | "You've used your 3 free hours. Your thoughts are waiting — upgrade to keep going." |
| Taps action items (Free) | Locked preview: "Here's what Echo found — upgrade to unlock." |
| Tries to open session >7 days old (Free) | "14 sessions are archived. Upgrade to access your full history." |
| Taps Patterns (Personal) | "Echo detected 3 recurring themes. Upgrade to Thinker to see them." |

---

## Phases

### Phase 1 — MVP (now)
Everything in the MVP table above. Free + Personal tiers. Validate the idea.

**Webapp at launch:** Hero page only.
- `/` — product name, one-liner, iOS + Android download CTAs
- All other page routes redirect to `/`
- All `/api/*` routes stay live for the mobile app
- No Library, no Insights, no Settings, no auth pages
One surface. The domain does useful work. The webapp earns its full role in Phase 2.

### Phase 2 — ~6 weeks post-launch
*Turn the upgrade story on. Unlock Thinker and Pro.*

- Patterns (cross-session themes) → Thinker unlocked
- Speaker diarization → Pro unlocked
- Weekly insight digest → Thinker
- Custom reflection protocols → Thinker
- Export (PDF + Markdown) → Personal+
- Scramble Mode → Pro unlocked
- Offline recording (queue + sync on reconnect)
- Full 4-tier billing live
- **Webapp returns** as live companion screen (Pro only) — real-time transcript + suggestions on desktop while recording on phone
- Encryption Phase 2: `NotebookNode.content` for non-meeting types (if applicable)

### Phase 3 — 3–6 months post-launch
*The moat features. Make Echo irreplaceable.*

- Thought continuity — session linking, personal knowledge graph
- Import from Notion / Obsidian / Markdown → Thinker+
- Session templates (1:1, therapy prep, decision dump, idea dump)
- Energy / tone pattern tracking across sessions
- Forgetting mode (auto-delete after N days)
- Smart action item reminders from your own voice

### Deliberately deferred
- Export to third-party apps — review when product is proven
- Team plans — Phase 4 at earliest
- B2B / enterprise — long-term expansion

---

## The Real Moat

The AI reflection is not the moat. The accumulated personal knowledge graph is.

After 90 days of daily use, Echo knows how your thinking evolves, which themes recur, what you said you'd do and never did. No competitor can replicate that. ChatGPT can't. Notion can't. That's the thing that makes churn structurally impossible — and it only exists because the data stays in Echo.

This is why export to third-party apps stays off. This is why import is a Thinker feature. And this is the feature roadmap's north star: every Phase 3 feature deepens the knowledge graph.

---

## Infrastructure Note

Switch Pro tier to AssemblyAI Nano ($0.002/min) before Pro reaches 500 users. At current Groq pricing, Pro gross margin is 31%. AssemblyAI Nano doubles it to ~57%.

---

_Echo is built by [founder] · echo.app_
