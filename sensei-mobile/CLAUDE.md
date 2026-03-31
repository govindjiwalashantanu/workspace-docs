# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Working Protocol

All development follows the protocol at `/Users/shantanu.govindjiwala/Documents/WORKING_PROTOCOL.md`. Read it before any task. Every feature must clear the full persona pipeline before being presented to the owner.

At the start of each session, also read:
- `SESSION_NOTES.md` — prior session history and what's next (What's Next section at top)

## Current State
_Update this section at the end of every session._

**Last session:** March 30, 2026
**Tests:** 534 passing · 0 failing (50 suites)
**TypeScript:** Clean
**Uncommitted:** 7 files (code review fixes — pending commit approval)
**Pending physical test:** Phone call interruption flow (iOS simulator + iPhone 16)

## Commands

```bash
npm start                # Expo dev server
npm run ios              # iOS simulator
npm run android          # Android emulator
npm test                 # Run Jest tests (--forceExit)
npm run test:coverage    # Coverage report
npm run test:watch       # Watch mode
```

Run a single test file:
```bash
npx jest --testPathPattern="path/to/file" --forceExit
```

## Architecture

React Native 0.81.5 + Expo ~54 + TypeScript. File-based routing via Expo Router. Connects to the sensei-webapp backend at `EXPO_PUBLIC_API_URL`.

### Directory Layout

- `app/(app)/` — Authenticated screens behind the tab navigator (5 visible tabs)
- `app/(app)/live/` — Core recording flow: `index` → `recording` → `review` → `save`
- `app/(auth)/` — Login, register, forgot-password
- `components/` — Shared UI; `components/live/` for recording-specific components
- `lib/api/` — Modular Axios clients per domain (`client.ts` sets base URL + JWT interceptors)
- `lib/store/` — Zustand stores (`auth-store.ts`, `live-store.ts`)
- `lib/contexts/LiveSessionContext.tsx` — Singleton mic/session service shared across all live screens
- `lib/hooks/` — React Query wrappers (`useNotebook`, `useBoards`, `useAuth`, `useChat`, `useLiveSession`)
- `services/openai-realtime.ts` — OpenAI Realtime API for live transcription
- `constants/theme.ts` — **single source of truth** for all styles
- `__mocks__/` — Expo module mocks for the test environment

### State Management

**Zustand stores:**
- `auth-store.ts` — JWT + user profile (`id`, `email`, `name`, `orgId`, `orgSlug`, `role`). Token persisted in Expo SecureStore. `loadFromStorage()` called at app launch.
- `live-store.ts` — Recording session state: `sessionId`, `isRecording`, `isPaused`, `utterances[]`, `speakers[]`, `notes`, `liveError`.

**React Query (TanStack):** Queries for `['notebook']`, `['boards']`, `['chat']`. Config: `retry: 1`, `staleTime: 30s`, `networkMode: 'always'`.

**LiveSessionContext:** Holds a single `realtimeRef` (prevents duplicate mic instances). Manages app interruptions (phone calls → auto-pause), utterance backup, and the finalize/inject/save-as-note operations.

### API Layer

Single Axios client in `lib/api/client.ts`:
- Base URL: `EXPO_PUBLIC_API_URL`
- Request interceptor: attaches `Authorization: Bearer <token>` from auth store
- Response interceptor: clears auth on 401

Modular API functions: `auth.ts`, `notebook.ts`, `boards.ts`, `live.ts`, `chat.ts`.

## Key Patterns

- All fonts: `fonts.body` (system sans-serif) or `fonts.mono` (timestamps/technical text only)
- Speaker colors: always via `getSpeakerColor()` in `constants/theme.ts` — never hardcode
- Shadow opacity capped at 0.25, `shadowRadius` capped at 10
- Progress bars minimum 6px height

## Safe Area Rules

Screens with `headerShown: false` **must** handle safe area manually:
- Import `useSafeAreaInsets` from `react-native-safe-area-context`
- Apply `paddingTop: insets.top + N` to the first visible element
- Apply `paddingBottom: Math.max(N, insets.bottom + N)` to footer/controls
- Screens using this: `recording.tsx`, `review.tsx`, `save.tsx`, `feed/index.tsx`
- Screens using default Expo Router header (safe area handled automatically): boards, search, profile, notebook

## Tab Bar

- Height: 72px — raised from 60 to fit Record circle + label without overlap
- Record tab: 42px circular icon — do not increase
- Notebook tab (`app/(app)/notebook/`): `href: null` — hidden from tab bar, navigated to programmatically

## Live Session Architecture

- `LiveSessionContext` is the single source of truth for the recording service
- `useLiveSession` is a re-export of `useLiveSessionContext` — use either interchangeably
- `setRecording(false)` is in the `finally` block of `stopSession` — clears state even if mic stop throws
- Feed filter excludes: nodes with `source: poc_snapshot` property + titles starting with "POC"
- Analyzing spinner only shows for meetings from today (capped by date, not stuck on old meetings)

## Testing

**Environment:** `node` + `ts-jest` — no component rendering. Tests cover store mutations, hook logic, pure utilities, and service wrappers only.

**Mocks:** Expo module mocks in `__mocks__/` (secure-store, audio, router, constants, notifications). Global fetch mock + `console.warn` silencer in `jest.setup.ts`.

**Status:** All tests passing — 47 suites, 486 tests, 0 failures. `login()` in `lib/api/auth.ts` uses native fetch (not apiClient), so those tests mock `global.fetch`, not MockAdapter.

**Coverage collected from:** `lib/**/*.{ts,tsx}`, `services/**/*.{ts,tsx}`.
