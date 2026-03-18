# Echo — Patent Strategy & Filing Guide
_Last updated: March 2026_

---

## Overview

Echo has several technically novel features that are worth protecting via patent. This document covers what's patentable, what isn't, how to file, and the priority order.

**Key principle:** Software patents are defensible only when they describe a *specific technical method*, not a general idea. Everything in this guide passes that test.

---

## Priority Matrix

| Idea | Novelty | Prior Art Risk | Recommendation |
|------|---------|---------------|----------------|
| #1 — Scramble Mode (multi-provider chunk routing) | HIGH | LOW | **File provisional immediately** |
| #2 — Per-user zero-knowledge encryption for voice AI | HIGH | LOW-MEDIUM | **File provisional** |
| #3 — Thought continuity / session linking algorithm | HIGH | LOW | **File provisional** |
| #4 — Custom reflection protocols (user-defined extraction) | HIGH | LOW | File after launch |
| #5 — Forgetting mode with cryptographic key deletion | MEDIUM | LOW | File after launch |
| #6 — Cross-session pattern detection algorithm | MEDIUM | MEDIUM | Trade secret first |

---

## Patent #1 — Scramble Mode
**Priority: File immediately. This is the crown jewel.**

### What it is
A method for transcribing private audio by distributing successive audio chunks across multiple third-party speech-to-text providers in a round-robin or pseudo-random sequence, such that no single provider receives a sufficient portion of the audio to reconstruct the full conversation.

### Why it's patentable
- Specific technical method (not just "use multiple providers")
- The distribution scheme (chunk size, ordering, provider selection algorithm) is novel
- The claim that no single provider receives >1/N of the conversation is mathematically verifiable
- No competitor has built this. Otter, Fireflies, AssemblyAI, Deepgram are all single-provider by default.

### Claims to protect
1. A method of transcribing audio comprising: dividing audio into sequential time-bounded chunks; routing each chunk to a provider selected from a set of providers according to a predetermined rotation scheme; assembling the full transcript only on the operator's server; wherein no provider in the set receives more than 1/N of the total audio.
2. The method of claim 1, wherein the rotation scheme is round-robin (Groq → AssemblyAI → Deepgram → repeat).
3. The method of claim 1, wherein the chunk index is transmitted with each audio segment to allow server-side assembly with positional ordering.
4. The method of claim 1, further comprising displaying to the user a visual indicator confirming that Scramble Mode is active.
5. A system implementing the method of claims 1-4 wherein the assembled transcript is encrypted with a per-user key before storage.

### Prior art to search
- Google Patents: `audio transcription privacy multiple providers`
- Assignee searches: `Otter.ai`, `AssemblyAI`, `Deepgram`, `Rev.com`, `Nuance`
- Focus on Claim 1 — if no one claims the round-robin distribution method, you're clean.

---

## Patent #2 — Per-User Zero-Knowledge Encryption for Voice AI
**Priority: High. Pairs with Scramble Mode for a combined privacy story.**

### What it is
A system for storing AI-processed voice data (transcripts, summaries, action items) wherein each user's data is encrypted with a unique encryption key derived from a secret known only to the user's account, stored encrypted server-side, such that the operator cannot access plaintext content without the user's key.

### Why it's patentable
- The combination of: (a) voice input, (b) AI extraction, (c) per-user key storage, (d) encrypted AI output storage is novel as a system
- Zero-knowledge encryption exists, but applying it specifically to AI-extracted metadata (summaries, patterns, action items) from voice recordings is novel
- The key management scheme (where is the key stored, how is it derived, what happens on account deletion) is specific and protectable

### Claims to protect
1. A system for storing AI-processed voice data comprising: a per-user encryption key generated and associated with each user account; a voice transcription pipeline; an AI extraction pipeline that produces structured outputs; and a storage layer wherein all structured outputs are encrypted with the per-user key before persistence.
2. The system of claim 1, wherein account deletion comprises cryptographic key destruction, rendering all associated data permanently inaccessible.
3. The system of claim 1, wherein the per-user key is itself stored encrypted using a master key, and decrypted only during active API requests.

### Prior art to search
- Google Patents: `per-user encryption key voice transcription`
- Assignee: `Zoom`, `Otter.ai`, `Microsoft Teams`

---

## Patent #3 — Thought Continuity / Session Linking Algorithm
**Priority: High. This is the long-term retention moat.**

### What it is
A method for automatically detecting semantic relationships between voice recording sessions and surfacing those connections to the user — specifically: identifying when a new recording discusses concepts, people, or decisions mentioned in prior recordings, and presenting a "your thinking has evolved" view.

### Why it's patentable
- The specific algorithm for semantic similarity scoring between voice-derived AI summaries is novel
- The user-facing interface for showing "you mentioned this tension on [date] — here's how your thinking changed" is novel
- No competitor (Otter, Limitless, AudioPen) does this at all

### Claims to protect
1. A method for linking voice recording sessions comprising: extracting semantic embeddings from AI-generated summaries of each session; computing similarity scores between a new session and historical sessions; identifying historical sessions exceeding a similarity threshold; and presenting the user with a temporal view of how their thinking on a topic has evolved.
2. The method of claim 1, wherein the temporal view includes the original statement from a prior session and the current session's treatment of the same topic.
3. The method of claim 1, applied to a voice journaling application wherein sessions represent personal thought recordings.

### Prior art to search
- Google Patents: `voice session semantic linking`, `personal knowledge graph voice notes`
- Assignee: `Notion`, `Roam Research`, `Obsidian`

---

## Patent #4 — Custom Reflection Protocols
**Priority: File after launch.**

### What it is
A system allowing users to define custom AI extraction templates — specifying what structured information the AI should always extract after a voice recording. Example: a therapist defines "identify core assumption, biggest fear, and one concrete action." Different from generic AI prompting because the configuration persists across sessions and is user-owned.

### Claims to protect
1. A method for personalised AI extraction from voice recordings comprising: a user-defined extraction template specifying named fields and extraction instructions; application of the template as a persistent system prompt modifier on each recording session; storage of extracted fields as structured data associated with the session.

---

## Patent #5 — Forgetting Mode with Cryptographic Deletion
**Priority: File after launch.**

### What it is
A privacy feature wherein a user schedules automatic deletion of recordings after a defined period (7, 30, 90 days), implemented via cryptographic key rotation or destruction rather than simple file deletion — making recovery computationally infeasible.

### Claims to protect
1. A method for scheduling irrecoverable deletion of voice data comprising: associating a time-bounded encryption key with each recording; upon expiry of the time bound, deleting the encryption key without deleting the ciphertext; whereby the plaintext becomes permanently inaccessible even if the ciphertext is retained by backup systems.

---

## How to File a Provisional Patent

### Step 1: Do the prior art search (1-2 hours)
Use these tools:
- **Google Patents:** https://patents.google.com
- **USPTO Full-Text Search:** https://ppubs.uspto.gov/pubwebapp/
- Search terms for Scramble Mode: `audio transcription multiple providers chunk rotation privacy`
- Search terms for encryption: `per-user encryption voice AI zero knowledge`
- If no claim 1 match found → you're clear to file

### Step 2: File the provisional ($320 DIY, no attorney required)
- Go to: https://www.uspto.gov/patents/basics/types-patent-applications/provisional-application-patent
- Use EFS-Web (USPTO electronic filing)
- A provisional doesn't need formal claims — it just needs a clear description of the method
- **Locks your priority date for 12 months**
- You have 12 months to file the full utility patent or abandon

### Step 3: File utility patent within 12 months ($800-$3,000 with attorney)
- At this point, get a patent attorney (Fenwick & West, Wilson Sonsini, or a startup-focused IP firm)
- Bundle patents #1 + #2 into one application if they're part of the same system
- Estimated total cost with attorney: $5,000-$10,000 per patent

---

## What Is NOT Patentable (Don't Waste Money)

| Idea | Why Not Patentable |
|------|-------------------|
| "AI voice journal app" | Abstract idea, not a specific method |
| "Summarise voice recordings with AI" | Too generic, prior art everywhere |
| "Privacy-first voice notes" | Business method, not technical |
| The pricing model or freemium tiers | Business method |
| The UI design | Protectable by trademark/copyright, not patent |

---

## Trademark (Separate from Patents)

File these now before launch:
- **"Echo"** — word mark, Class 9 (software) and Class 42 (SaaS)
- **Scramble Mode™** — the specific mode name, Class 42
- The Echo logo/icon once finalized

**File at:** https://www.uspto.gov/trademarks
**Cost:** ~$250 per class per mark
**Timeline:** 8-12 months to registration, protection starts from filing date

---

## Trade Secrets (What to Keep Private)

Don't publish these until patented or until you decide to keep them secret:
- The exact chunk routing algorithm implementation
- The session linking similarity threshold and scoring weights
- The per-user key management architecture details

Keep them secret by:
- NDAs with all contractors/employees
- Don't describe implementation specifics in blog posts or talks
- Keep detailed technical specs out of GitHub public repos

---

## Recommended Action Plan

**This week:**
1. Run Google Patents search for Scramble Mode claims (1 hour)
2. File provisional patent for Scramble Mode ($320)

**Before launch:**
3. File provisional for per-user encryption (#2)
4. File provisional for session linking (#3)
5. File trademark for "Echo" and "Scramble Mode"

**After 500 paid users (proof of traction):**
6. Hire patent attorney for full utility patents on #1 and #2
7. Cost: ~$8,000-12,000 total

**Total investment before full utility filing: ~$1,280 (3 provisionals + 2 trademarks)**

---

## Questions to Ask a Patent Attorney

1. "Can we bundle Scramble Mode and per-user encryption into one patent application as a combined privacy system?"
2. "What claims language would you recommend for the round-robin audio distribution method?"
3. "Is the session linking algorithm better protected as a patent or trade secret given how quickly AI evolves?"
4. "Should we file in the US only, or also PCT (international) for EU and Canada markets?"

---

## Resources

- **USPTO Provisional Application Guide:** https://www.uspto.gov/patents/basics/types-patent-applications/provisional-application-patent
- **Google Patents:** https://patents.google.com
- **USPTO Patent Search:** https://ppubs.uspto.gov/pubwebapp/
- **Alice Corp. v. CLS Bank (2014):** The landmark case on software patent eligibility — your patents need to describe a specific technical improvement, not just "do X on a computer"
- **Recommended firms:** Fenwick & West, Wilson Sonsini, Cooley LLP (all startup-friendly)

---

_Echo — protecting ideas worth keeping private_
