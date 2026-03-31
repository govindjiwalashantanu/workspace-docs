# senSEi Mobile — Session Notes
_Last updated: March 30, 2026_

---

## ⟶ What's Next
_Updated: March 30, 2026 — update this section at the start of every session_

**Physical testing (requires device):**
- [ ] Test on iOS simulator — verify phone call interruption flow (banner appears, auto-resumes)
- [ ] Test on physical iPhone 16 — same as above

**Remaining code review items (medium priority):**
- [ ] M2 — Coaching interval should pause/restart on state change (not guard on each tick)
- [ ] M3 — Cap utterances array for very long sessions (1,000 entry ceiling)
- [ ] M5 — Document the 500ms `setTimeout` constant with a named variable + comment
- [ ] L3 — Mic permission denied: guide user to Settings instead of generic error

**Testing gaps:**
- [ ] Integration tests for full record → transcribe → finalize path (no cross-service tests yet)

---

## Session: March 30, 2026 — Code Review Fixes (6 items)

Comprehensive professional code review conducted, then all actionable findings resolved. TypeScript clean, 50 suites / 0 failures after changes.

### Changes made

**H1 — `autoSave` silent failure fixed (`lib/contexts/LiveSessionContext.tsx`)**
- Removed `.catch(() => {})` from `finalizeSession` in `autoSave`
- Added `catch (err)` block that calls `setError(...)` — user now sees an actionable error if the session fails to save on auto-save (e.g. phone interrupt → background save path)

**H2 — Error state surfaced on save + review screens**
- `app/(app)/live/save.tsx` — added `error, clearError` from `useLiveSession()`, renders dismissible red banner when context error is set
- `app/(app)/live/review.tsx` — same pattern; covers inject and save-as-note failure paths
- Both screens now display LiveSessionContext errors that were previously invisible

**H3 — Audio recorder released on prepare failure (`services/openai-realtime.ts`)**
- Added `try { await this.recorder?.stop(); } catch {}` before nulling recorder on `prepareToRecordAsync` 3-retry exhaustion
- Prevents native audio resources staying allocated after repeated prepare failures

**M1 — Coaching signal response validated at runtime (`lib/contexts/LiveSessionContext.tsx`)**
- Added `typeof data.signal.headline === 'string' && typeof data.signal.content === 'string'` guard before calling `addCoachingSignal`
- Prevents malformed API responses from polluting coaching UI state

**M4 — `alertError` utility extracted (`lib/utils/errors.ts`)**
- New file: `lib/utils/errors.ts` — `alertError(err, fallback)` utility
- Replaced `Alert.alert('Error', err instanceof Error ? ...)` boilerplate in save.tsx, review.tsx (handleSaveNote, handleInject) with `alertError(...)` calls
- Fallback messages now include "Check your connection and try again."

**L1+L2 — Human voice copy fixes**
- `app/(app)/live/recording.tsx`: "This will end the session and process your transcript." → "We'll save and process your recording."
- `app/(app)/live/save.tsx`: "optionally link it to an account" → "attach it to an account if you like"

### Test results
- TypeScript: 0 errors
- Jest: 50 suites · 0 failures (524 passing + 12 todo)

### What's next
- Medium findings from code review not yet addressed (lower priority): coaching interval pause optimization, utterances array cap, 500ms delay constant, integration tests for record→finalize path
- Outstanding items from previous session: test on iOS simulator and physical iPhone 16 for phone call interruption

---

## Session: March 26, 2026 — Recording Reliability & Phone Call Handling

### What was done

**Fix 1 — `prepareToRecordAsync` retry (`services/openai-realtime.ts`)**
- Single-attempt replaced with retry loop (3 attempts, 300 ms / 600 ms back-off)
- A transient audio session glitch no longer permanently kills the session

**Fix 2 — Preserve utterances across background transitions (`lib/contexts/LiveSessionContext.tsx`)**
- `flushBuffer()` called immediately on AppState `background` transition
- `saveUtteranceBackup()` now includes `pendingBufferRef.current` alongside Zustand utterances
- 500 ms delayed `flushBuffer()` after returning to `active` (after chunk rotation fires)

**Fix 3 — Accidental stop prevention (`app/(app)/live/recording.tsx`)**
- Replaced `Alert.alert` with an inline absolute-positioned modal
- 2-second countdown before the "Stop & Save" button becomes active
- Cancel always active; countdown resets on cancel

**Fix 4 — Phone call interruption (`services/openai-realtime.ts`)**
- Added `recordingStatusUpdate` listener on `recorder` — fires `handleInterruption()` when iOS stops the recorder externally
- Added `mediaServicesDidReset` check in the 150 ms metering interval
- `intentionallyStopping` flag guards against false positives during normal chunk rotation
- Listener cleanup in `pause()` and `stop()`

**Tests: 50 test suites, 534 tests passing**
- Updated expo-audio mock with `addListener` and `mediaServicesDidReset`
- Fixed pre-existing background test flaw (improper `stop`/`start` sequence)
- Added tests: retry loop, status listener, `intentionallyStopping` guard, `mediaServicesDidReset`, listener cleanup

### What's next
- Test on iOS simulator (personal machine) and physical iPhone 16
- Verify phone call interruption banner appears and auto-resumes

---

## Session: March 22, 2026 — Chat History + AI Artifact Generation

**Dependencies added:** `react-native-markdown-display`, `expo-clipboard`

**`lib/api/chat.ts`** — Added `conversationId` param to `sendChatMessage`. Added `getConversations`, `getConversation`, `createConversation`, `deleteConversation` API functions. `ConversationSummary` + `ConversationDetail` types exported.

**`lib/hooks/useConversations.ts`** — New. `useConversations` (list), `useConversation(id)`, `useCreateConversation`, `useDeleteConversation`. Tests written first — 8 tests passing.

**`lib/hooks/useChat.ts`** — Updated: `useSendMessage` now accepts `conversationId`, invalidates conversation queries on success.

**`app/(app)/search.tsx`** — Full rewrite:
- Toolbar: "New" button + history icon (badge with conversation count)
- Conversation history: modal (pageSheet) showing list of past conversations with title, relative time, message count. Tap to load. Delete button per row.
- Message rendering: user messages = plain Text; AI messages = `Markdown` component from `react-native-markdown-display` with full theme-matched styles (tables, code blocks, blockquotes, headings, lists)
- Copy button (copy-outline icon) below each AI message. Checkmark feedback for 1.5s. `expo-clipboard.setStringAsync`
- Auto-select most recent conversation on mount
- Optimistic pending messages while API is in flight
- Suggestion chips updated to include "Draft a deal brief for my top opportunity"

**Tests:** 50 suites · 510 passing · 0 failures (+8 new).

---

## Session: March 22, 2026 — Push Notifications (Mobile Side)

**`lib/hooks/useNotificationPrefs.ts`** — New. `useNotificationPrefs` (GET) + `useUpdateNotificationPrefs` (PATCH, invalidates cache). Tests written first: 7 tests passing.

**`app/(app)/notification-prefs.tsx`** — New screen. 3 toggle rows: Meeting Summary Ready, Action Items Created, Deal Alerts. `makeStyles(colors)`, `Switch` from RN core.

**`app/(app)/profile.tsx`** — Replaced disabled Coming Soon with `TouchableOpacity → /(app)/notification-prefs`. Removed 4 unused styles.

**Tests:** 49 suites · 502 passing · 0 failures (+7).

---

## Session: March 22, 2026 — Feed Timezone + Analysing Spinner Fixes

### Root cause (both bugs): timezone mismatch

**Bug 1 — Perpetual "Analysing..." spinner**
`today = new Date().toISOString().split('T')[0]` returned the UTC date. Users west of UTC after ~7pm (UTC-5) or ~4pm (UTC-8) would see `today` as tomorrow's date in UTC while the meeting's `date` property held the local date — mismatch meant spinner could show incorrectly or stick forever. No timeout meant a meeting whose backend analysis silently failed would spin indefinitely.

**Fix:** Use `toLocalDateString()` (new export from `lib/utils/format.ts`) for the `today` comparison. Added a 10-minute `createdAt`-based timeout: if 10 minutes have passed with no summary, stop spinning.

**Bug 2 — Yesterday's meetings in "Today" section + wrong intra-day sort order**
Two separate issues in `lib/utils/feed-filter.ts`:
1. `timeBucket` used `(Date.now() - nodeTimestamp) / 86_400_000 < 1` — a rolling 24-hour window. A meeting from yesterday at 11pm (23h ago) fell into 'Today'. Conversely, today's meetings after ~7pm local time (where UTC midnight was > 24h ago) fell out of 'Today' into 'Last 7 days'.
2. `nodeTimestamp` returned `new Date(dateProp).getTime()` where `dateProp` is `'YYYY-MM-DD'` — all meetings on the same calendar day got the same UTC-midnight anchor, making intra-day sort order arbitrary.

**Fix:**
- Added `toLocalDateString(ms)` to `lib/utils/format.ts` — returns YYYY-MM-DD in LOCAL timezone
- `nodeTimestamp`: changed primary key to `createdAt` (actual creation time) for precise intra-day ordering; `date` property only as fallback
- `timeBucket`: replaced rolling-24h check with direct local date string comparison (`dateStr === localToday`) for 'Today'; midnight-UTC diff for the 7/30-day ranges

### Files changed
- `lib/utils/format.ts` — added `toLocalDateString(ms?)`
- `lib/utils/feed-filter.ts` — fixed `nodeTimestamp`, `timeBucket`; added `nodeDateString`
- `app/(app)/feed/index.tsx` — fixed `isAnalysing` (local date + 10-min timeout)

### Tests
- New: `__tests__/lib/utils/feed-filter.test.ts` (9 tests written before implementation)
- All tests pass: 495 passing, 0 failures

### Protocol check
- ✅ Tests written first, confirmed 3 failing before implementation
- ✅ Full suite: 495 passing, 0 failures (48 suites)
- ✅ SESSION_NOTES updated

---

## Session: March 22, 2026 — Full Test Coverage Pass

### Goal
Fix all pre-existing test failures and bring lib/ + services/ to 100% test coverage.

### Fix 1 — `useAuth.test.tsx` (fetch vs Axios mock conflict)
**Root cause:** `login()` in `lib/api/auth.ts` uses native `fetch()` via `fetchWithTimeout()`, not `apiClient` (Axios). Tests were mocking Axios via `MockAdapter` but `global.fetch` had no implementation, causing `undefined.ok` to throw.

**Fix:** Added `(global.fetch as jest.Mock).mockResolvedValue(...)` in `beforeEach` for happy paths. Per-test `mockResolvedValueOnce` / `mockRejectedValueOnce` overrides for error paths. Checked fetch mock call args (not `mock.history.post`) for credential-assertion tests.

### Fix 2 — `useBoards.test.tsx` + `useNotebook.test.tsx` (sync isSuccess/isError assertion)
**Root cause:** `isSuccess` / `isError` asserted synchronously immediately after `mutateAsync`. React Query mutation state update requires a render cycle.

**Fix:** Wrapped bare `expect(result.current.isSuccess/isError).toBe(true)` in `await waitFor(...)` in 4 tests across both files.

### New: `__tests__/lib/utils/push-notifications.test.ts` (13 tests)
Covers `registerPushToken` (permissions granted/requested/denied, token failure, API failure, correct token value) and `getNodeIdFromNotification` (present, absent, wrong type).

### New: `__tests__/lib/error-reporter.test.ts` (12 tests)
Covers `ErrorReporter.report()` (POST shape, auth header, mobile header, platform field, stack/path/metadata, deduplication window, different messages bypass dedupe, different levels bypass dedupe, non-fatal on fetch failure) and `ErrorReporter.init()` (registers Axios interceptor, interceptor re-rejects errors, skips report for 4xx).

### New: `__tests__/lib/contexts/live-session-lifecycle.test.ts` (24 tests)
Covers startSession (createSession called, store updated, realtime started), stopSession (stop called, isRecording=false, audioLevel cleared), finalize (assignSpeakers, finalizeSession with all args, notes passthrough, empty notes → undefined, skips assignSpeakers when no speakers, returns result), injectIntoMeeting (correct args, returns result), saveAsNote (correct args, undefined title, returns result), autoSave (stops first, uses titleOverride, generates default title, isRecording=false after).

### Test counts
- Start: 472 total (3 failing)
- End: 496 total, 486 passing (0 failing, 10 todo)
- New files: 3 (+49 tests)
- Fixed: 4 pre-existing failures

### Protocol check
- ✅ Root causes fixed, no overriding patches
- ✅ Full suite: 486 passing, 0 failures (47 suites)
- ✅ SESSION_NOTES updated

---

## Session: March 22, 2026 — Background Recording Fixes Round 3

### Fix 1 — `TranscribingBanner` crash (`ReferenceError: Property 'styles' doesn't exist`)
`BouncingDot` inside `TranscribingBanner.tsx` referenced `styles.dot` without calling `makeStyles(colors)`. The bug was latent — it only surfaced when `isTranscribing` became true. Added `const styles = makeStyles(colors)` to `BouncingDot`.

### Fix 2 — Spurious HTTP 400 transcription error on foreground return
**Root cause:** When the 12 s chunk timer fires while the app is backgrounded, `rotateChunk()` stops the current recorder and immediately calls `startChunk()` (recording a fresh chunk). If the user returns within milliseconds, `notifyAppState('active')` also calls `rotateChunk()` on this brand-new recorder, producing a near-empty M4A container that Whisper rejects with HTTP 400.

**Fix:** In `rotateChunk()` and `stop()`, capture `chunkDurationMs = Date.now() - chunkStartTime` before stopping. Skip `transcribeChunk()` if `chunkDurationMs < MIN_CHUNK_MS (1 s)`.

**Tests:** 2 new tests in `openai-realtime-background.test.ts` (sub-second chunk skipped; ≥ 1 s chunk is sent).

### Fix 3 — Background chunk rotation driven by metering interval
**Root cause:** `startChunk()` and `resume()` used `setTimeout(rotateChunk, 12_000)`. iOS throttles plain `setTimeout` calls when an app is backgrounded, even with `UIBackgroundModes: audio`. This caused all background audio to accumulate in a single large chunk only submitted to Whisper when the user returned.

**Fix:** Replaced the 12 s `setTimeout` with an elapsed-time check inside the existing 150 ms metering `setInterval`. The metering tick is backed by the native recorder state and fires reliably in the background. Applied to both `startChunk()` and `resume()`. `chunkTimer` field retained but no longer assigned (only `null`-cleared via `clearChunkTimer()`).

**Tests:** 3 new tests in `openai-realtime-background.test.ts`:
- Chunk rotates after 12 s via metering tick
- No rotation before 12 s
- Rotation works when `onLevel` is not provided

### Protocol check
- ✅ Tests written before implementation for Fix 2 and Fix 3
- ✅ Full suite: 462 passing (0 failures)
- ✅ SESSION_NOTES updated

### Test counts
- Start: 444 total
- End: 472 total (+28, including 5 new tests across both fixes)

---

## Session: March 22, 2026 — Post-refactor Code Cleanup

### Summary
Six-tier cleanup pass following the `useColors()` dark-mode refactor merge.

### Tier 1 — Deleted 8 dead Expo scaffold files
Removed `components/EditScreenInfo.tsx`, `ExternalLink.tsx`, `StyledText.tsx`, `Themed.tsx`, `useColorScheme.ts`, `useColorScheme.web.ts`, `useClientOnlyValue.web.ts`, and `constants/Colors.ts`. All confirmed zero active imports before deletion. Also cleared pre-existing TypeScript errors in those files.

### Tier 2 — Fixed double semicolons in 39 files
The `useColors` refactor left `import { useColors } from '@/lib/hooks/useColors';;` (double semicolon) across 39 files. Automated `str.replace(';;', ';')` pass.

### Tier 3 — Fixed hardcoded colors in board + live components
- Added `overlay: 'rgba(6,8,14,0.75)'` token to both `lightColors` and `darkColors` in `constants/theme.ts`
- Updated `boards/index.tsx`, `notebook/index.tsx`, `FeedbackSheet.tsx`, `TaskList.tsx` to use `colors.overlay`
- Replaced `PRIORITY_COLOR` static map in `KanbanCard.tsx` with dynamic `colors.*` lookups
- Replaced `COL_COLORS` static map in `KanbanColumn.tsx` with dynamic `colors.*` lookups
- Replaced `PRIORITY`/`COLS` hardcoded hex in `TaskList.tsx` with `getColColor()` / `getPriColor()` helpers
- Replaced `SPEAKER_COLORS` in `SpeakerAssignment.tsx` with `getSpeakerColor()` from theme (aligns to theme palette)

### Tier 4 — Extracted duplicate `formatTime` utility
- Created `lib/utils/format.ts` exporting `formatTime` and `formatDate`
- Removed local `formatTime` from `recording.tsx` and `TranscriptDisplay.tsx`
- Removed local `formatDate` from `feed/index.tsx`
- Tests: `__tests__/lib/utils/format.test.ts` (4 new tests written before implementation)

### Tier 5 — Added 15s timeout to bare `fetch()` in `lib/api/auth.ts`
Both `login()` and `requestPasswordReset()` bypassed the Axios client (circular dep) with no timeout. Wrapped both in `fetchWithTimeout()` using `AbortController` + `setTimeout(abort, 15_000)` with `clearTimeout` in finally.
- Tests: `__tests__/lib/api/auth-timeout.test.ts` (3 new tests written before implementation)

### Tier 6 — Performance: `React.memo` + Zustand selectors
- Wrapped `FeedCard` (`feed/index.tsx`) and `NodeRow` (`notebook/index.tsx`) with `React.memo()` — prevents 6–7 re-renders/sec during background recording (audioLevel updates every 150ms)
- Replaced bulk `useLiveStore()` destructuring in `recording.tsx` with individual selectors — only changed state slices trigger re-renders

### Protocol check
- ✅ Tests written first for Tiers 4 and 5, confirmed failing before implementation
- ✅ Full suite run after each tier
- ✅ SESSION_NOTES updated

### Test counts
- Start: 435 total (419 passing, 5 pre-existing failures + flaky useBoards)
- End: 442 total (425+ passing, 7 new tests added)

---

## Session: March 22, 2026 — Background Recording: Audio Session Recovery

### Problem
Recording terminated and UI stuck after backgrounding the app. Live session showed as ended on server while mobile still showed "recording active".

### Root causes found (3)

**1 — Audio session not re-initialised on foreground return** (`services/openai-realtime.ts` — `notifyAppState`)

iOS may suspend the AVAudioSession when the app backgrounds, even with `shouldPlayInBackground: true`. On return, any `prepareToRecordAsync` call fails because the session is stale. The old code called `rotateChunk()` directly without re-initialising the session first.

**Fix:** `notifyAppState('active')` now calls `setAudioModeAsync` (with all four background flags) before chunk work, using `.finally()` to sequence correctly.

**2 — Null recorder after background silently did nothing** (`notifyAppState`)

If iOS killed the recorder while backgrounded, `this.recorder` is null. The old code had `if (this.recorder) rotateChunk()` with no else — returning from background with a dead recorder did nothing. Service stayed `running = true`, UI stuck.

**Fix:** Added `else → void this.startChunk()` to restart the recorder from scratch.

**3 — `prepareToRecordAsync` failure silently zombied the service** (`startChunk`)

If the audio session was suspended and `prepareToRecordAsync` threw, the error propagated to `rotateChunk`'s catch, which called `startChunk` again — which threw again — silently, indefinitely. `running` stayed `true`, `isRecording` stayed `true`, nothing was captured.

**Fix:** Wrapped `prepareToRecordAsync` in try/catch. On failure: set `running = false`, call `onError` with a user-readable message so the recording screen can show an error and let the user restart.

### Files changed
- `services/openai-realtime.ts` — `notifyAppState` re-init + null recorder recovery; `startChunk` error surface
- `__tests__/services/openai-realtime-background.test.ts` — 3 new tests (9 total in file), written before implementation

### Protocol check
- ✅ Tests written first, confirmed failing before implementation
- ✅ Full suite run after — 419 passing, 3 pre-existing failures only
- ✅ SESSION_NOTES updated

### Test counts
- Start: 416 passing
- End: 419 passing (+3)

---

## Session: March 22, 2026 — Background Recording Fix + Safe Area + API URL

### Background recording stops when app is out of focus

**Root cause 1 — missing audio mode flags** (`services/openai-realtime.ts`)
`AudioModule.setAudioModeAsync` was missing `shouldPlayInBackground: true` and `allowsBackgroundRecording: true`. `UIBackgroundModes: ["audio"]` in app.json declares the capability but these flags are required at runtime to keep the audio session alive when backgrounded.

**Root cause 2 — home button treated as phone call** (`lib/contexts/LiveSessionContext.tsx`)
The AppState handler used a `< 3000ms` timing heuristic to distinguish phone calls from normal background. Both transitions (home button and answering a phone call) happen in ~100–500ms, so home button presses incorrectly triggered `handleInterruption()` → paused recording every time the user left the app.

**Fix:**
- Added `shouldPlayInBackground: true` + `allowsBackgroundRecording: true` to `setAudioModeAsync` on `start()` and `handleInterruptionEnd()`
- Removed the unreliable timing heuristic from `LiveSessionContext.tsx`
- AppState handler now always calls `notifyAppState('background')` for background and `handleInterruptionEnd()` + `notifyAppState('active')` for active
- If a real phone call interrupts the audio, the native OS stops the recorder; `rotateChunk`'s catch block handles recovery when returning

**Files changed:**
- `services/openai-realtime.ts` — added background flags to `setAudioModeAsync` (both `start()` and `handleInterruptionEnd()`)
- `lib/contexts/LiveSessionContext.tsx` — removed `wentInactiveAtRef` and phone-call heuristic; simplified AppState handler

**Tests added:**
- `__tests__/services/openai-realtime.test.ts` — added test asserting `shouldPlayInBackground` and `allowsBackgroundRecording` are set
- `__tests__/services/openai-realtime-background.test.ts` — NEW: 6 tests covering background invariants (no pause on home button, no-op handleInterruptionEnd without interruption, background audio flags restored after real phone call)

---

### Forgot password back button hidden behind status bar

**Root cause:** `/(auth)/forgot-password.tsx` renders its own back button (`headerShown: false`) but had hardcoded `paddingVertical: 24` — not enough to clear the iOS status bar (44–59px depending on device).

**Fix:** Added `useSafeAreaInsets`, changed `inner` style to `paddingBottom: 24` only, applied `paddingTop: insets.top + 16` inline to both scroll views (main form and success state).

---

### API URL pointing to unresolvable domain

**Root cause:** `.env.local` had `EXPO_PUBLIC_API_URL=https://okta.se-n-sei.com` — DNS not configured. Vercel deployment at `sensei-webapp-eta.vercel.app` also had Deployment Protection enabled, blocking all API calls.

**Fix:** User disabled Vercel Deployment Protection. Updated `.env.local` to `EXPO_PUBLIC_API_URL=https://sensei-webapp-eta.vercel.app`.

---

### Test counts
- Start of session: 394 passing
- End of session: 416 passing (+22)
- Pre-existing failures: useAuth, useNotebook, useBoards (fetch mock issue — do not fix)
