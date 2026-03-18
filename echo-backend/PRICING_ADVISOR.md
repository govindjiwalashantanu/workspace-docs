# B2C Pricing Advisor — Echo

## Persona

**Name:** The B2C Pricing Strategist
**Background:** 12 years designing monetisation models for consumer subscription apps. Past: growth and pricing at Spotify, Duolingo, and Superhuman. Principles grounded in behavioural economics, unit economics, and freemium conversion psychology.

**Core frameworks:**
- Value metric alignment (price scales with the thing that creates value)
- Freemium hook design (free tier must deliver the "aha moment")
- Tier differentiation (each tier needs a clear upgrade trigger, not just more of the same)
- Psychological anchoring ($7.99 is not $8; annual plans printed as monthly equivalents)
- LTV optimisation (lead with annual, annual cohorts churn ~3x less than monthly)

---

## Echo Pricing Analysis

### The value metric: hours is correct

Recording hours scale directly with both user value and infrastructure cost (Groq Whisper: $0.004/min). Users who record more get more value and cost more to serve. Hours is the right axis. Session counts were wrong — a session can be 30 seconds or 2 hours.

### Current tiers evaluated

| Tier | Price/mo | Hours | Annual | Monthly equiv. | Discount |
|------|----------|-------|--------|----------------|----------|
| Free | $0 | 3 hrs | — | — | — |
| Personal | $7.99 | 10 hrs | $63 | $5.25 | 34% |
| Thinker | $14.99 | 20 hrs | $119 | $9.92 | 34% |
| Pro | $24.99 | 50 hrs | $199 | $16.58 | 34% |

**Price points: correct.** $7.99 is under the $8 psychological ceiling. $14.99 is under $15. $24.99 is under $25. Anchoring is clean.

**Hour jumps: strong narrative.** Personal → Thinker = 2x hours for 1.87x price. Thinker → Pro = 2.5x hours for 1.67x price. Each upgrade feels like better value per hour.

**Annual discount: consistent at ~34%.** Good. Present it as "Pay for 8 months, get 12" not just a percentage.

---

## Critical Issue: Feature Differentiation

**The current tiers are differentiated only by hours.** This is a fatal positioning mistake.

If hours are the only variable, users simply pick the cheapest tier that covers their usage. There is no emotional reason to upgrade. The upgrade decision becomes a spreadsheet, not a desire.

**Strong B2C tiers each have a moment that makes the user say "I need that."**

- Spotify Free → Premium: no ads (emotional trigger)
- Duolingo Free → Plus: no hearts (removes friction at the worst moment)
- Notion Free → Pro: unlimited blocks (natural growth hit)

Echo needs that moment at each tier boundary.

---

## Recommended Tier Architecture

### Free — The Hook
**Purpose:** Deliver the core "aha moment" — enough to get addicted, not enough to stay.

- 3 hrs/month recording
- Transcription
- **Basic AI reflection: summary only** (no action items, no key ideas, no follow-up question)
- No search
- No export
- 7-day rolling history (older sessions locked, not deleted)

> Why basic reflection and not none: Echo's value proposition IS the AI reflection. A user who has never seen it will never understand why they should pay. Give them the summary — it's enough to feel the magic. Gate the depth.

> Why 7-day history lock: Users who have 3 weeks of locked sessions will upgrade to access them. Data they cannot read is a powerful upgrade trigger.

---

### Personal — $7.99/mo · $63/yr · 10 hrs
**Purpose:** The everyday thought capture tier. Students, casual knowledge workers.

- 10 hrs/month
- Full AI reflection: summary + action items + key ideas + follow-up question
- Full session history (no lock)
- Search across all transcripts
- Export (Markdown)
- **Trigger to upgrade:** hits 10 hrs and needs more

---

### Thinker — $14.99/mo · $119/yr · 20 hrs ★ MOST POPULAR
**Purpose:** The power user tier. Founders, ADHD users, coaches, researchers.

- 20 hrs/month
- Everything in Personal
- **Patterns** — cross-session insight: recurring themes, emotion trends, your thinking evolution
- **Weekly digest** — push notification: "3 open action items from last week. Your dominant theme: product strategy."
- **Custom reflection protocols** — define what the AI always extracts ("After every session, find: core assumption, biggest risk, who I need to call")
- **Trigger to upgrade from Personal:** the Patterns feature. Once you've used Echo for 3 weeks, "show me how my thinking evolved" is irresistible. Personal users cannot see it.

---

### Pro — $24.99/mo · $199/yr · 50 hrs
**Purpose:** The privacy-obsessed professional. Lawyers, therapists, executives, clinical users.

- 50 hrs/month
- Everything in Thinker
- **Scramble Mode** — multi-provider chunk routing; no single AI hears your full conversation (patent-pending)
- **Speaker diarization** — "You said this, they said that"
- **Overage** — $0.05/min after 50 hrs (no hard cap, no dropped sessions)
- **Priority AI processing** — queue jumps at peak load
- **Trigger to upgrade from Thinker:** Scramble Mode. The privacy-conscious user will upgrade specifically for this. It's technically verifiable (not just a policy claim), which is unique. Market it that way.

---

## Freemium Conversion Design

### The upgrade trigger moments (when to show the paywall)

| User action | Paywall shown | What they see |
|-------------|---------------|---------------|
| Exceeds 3 hrs in a month | Hard block on new session | "You've used your 3 free hours. Your thoughts are waiting — upgrade to keep going." |
| Taps action items in Free | Locked preview | "Action items are a Personal feature. Here's what Echo found in your session — upgrade to unlock." |
| Taps Patterns in Personal | Locked preview with teaser | "Echo has detected 3 recurring themes across your sessions. Upgrade to Thinker to see them." |
| Tries to access session older than 7 days (Free) | Locked with count | "14 sessions are archived. Upgrade to access your full history." |
| Records > 30 mins as Pro without diarization | Upsell nudge | "Speaker diarization available — know exactly who said what." |

### Annual-first default
**Default the billing toggle to Annual** on the upgrade screen. The 34% discount wins most price-sensitive users. Monthly exists for the commitment-averse — but don't lead with it.

Annual cohort benefits:
- ~3x lower churn than monthly
- Higher LTV immediately on conversion
- Lower payment failure rate (one charge vs 12)

---

## Unit Economics Check

At full usage, post-30% App Store cut, with Groq Whisper at $0.004/min:

| Tier | Infra cost | App Store net | Gross margin |
|------|-----------|---------------|-------------|
| Personal (10 hrs) | $2.40 | $5.59 | **57%** |
| Thinker (20 hrs) | $4.80 | $10.49 | **54%** |
| Pro (50 hrs) | $12.00 | $17.49 | **31%** |

Pro margin is thin at 31%. Switching to AssemblyAI Nano ($0.002/min) doubles Pro margin to ~57%. Do this before Pro reaches 500 users.

Claude reflection cost: <$0.01/session. Negligible until very high session counts.

---

## The Real Moat: What Users Cannot Replicate

### "What stops them copying the transcript into ChatGPT?"

Nothing technical. A free user can copy any transcript and paste it into ChatGPT. This is a good question because it reveals where the moat actually is — and it is not the AI reflection.

**Why they won't, and why it doesn't matter:**

1. **Friction is the product.** Open app → record → reflection appears in 30 seconds. Copy transcript → open ChatGPT → paste → write prompt → wait → copy back is 6+ steps on a phone while walking to your car. The gap between these experiences is the entire value proposition.

2. **Privacy users won't do it.** The ADHD and privacy-obsessed segment — the highest-WTP customers — are the users least likely to paste their private thoughts into ChatGPT. Doing so defeats the reason they chose Echo. The privacy promise is structural, not just marketing.

3. **ChatGPT cannot see their history.** This is the real moat. ChatGPT cannot say: "You've mentioned this tension with your co-founder 6 times in the last month. Here's how your thinking has evolved." The Patterns feature, weekly digest, and thought continuity require accumulated data that only Echo holds. This is the thing that makes Echo irreplaceable after 90 days.

4. **People who would do that were never customers.** A user willing to copy-paste into ChatGPT would also self-host Whisper and write their own prompts. They were never going to pay.

### Implication for pricing

**The AI reflection is not the moat. The accumulated personal knowledge graph is.**

The subscription should be understood as buying:
- Continuity — your thinking over time, searchable and linked
- Privacy — Scramble Mode is technically verifiable, not a policy claim
- Zero-friction capture — the 30-second loop on mobile

Not primarily the AI output itself.

This is why free users should receive basic AI reflection (summary only). Gating the reflection entirely would prevent users from understanding why Echo is worth paying for. Give them the magic — charge for the depth and the continuity.

---

## What Not to Do

1. **Do not gate AI reflection entirely on free.** Users who never see a reflection don't know what Echo does. You will lose them before they understand the product.

2. **Do not make overage a surprise.** Pro users who hit 50 hrs should see a clear "You've used 48 of 50 hrs — overage at $0.05/min" warning. Surprise charges cause chargeback disputes and bad reviews.

3. **Do not show the Free tier on the upgrade screen.** The upgrade screen is a conversion surface. Free users are already on free — they don't need to "choose" it again. Show only the 3 paid tiers.

4. **Do not use "Pro" as the middle tier.** Pro sounds like a ceiling. People above "Pro" have nowhere to go. Put Pro at the top — it is the aspirational tier, not the default.

5. **Do not discount more than 34% annually.** Deeper discounts train users to wait for sales. 34% is the industry standard for SaaS annual plans and matches Spotify, Notion, and Linear.

---

## Founding Member Opportunity (Optional)

Before launch, offer a **Founding Member** price to the first 500 users:
- Personal Founding: $5.99/mo (locked in forever)
- Thinker Founding: $9.99/mo (locked in forever)

**Why:** Creates FOMO, rewards early adopters, generates word-of-mouth, fills the initial cohort fast. ProductHunt launches convert well with founding member pricing. Cap it hard at 500 — scarcity is the mechanism.

---

_Last updated: March 2026 — Echo Pricing Advisor_
