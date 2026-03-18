# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Working Protocol

All development in this repo follows the protocol defined at:
`/Users/shantanu.govindjiwala/Documents/WORKING_PROTOCOL.md`

Read it before starting any task. Every feature must clear the full persona pipeline before being presented to the owner.

## Commands

```bash
npm start          # Start Expo dev server
npm run ios        # iOS simulator
npm run android    # Android emulator
npm test           # Run Jest tests
```

## Architecture

React Native app built with Expo. Voice-first thought capture app.
Connects to echo-backend at EXPO_PUBLIC_API_URL.

Core loop: Record → Transcribe (Groq Whisper) → Reflect (LiteLLM/Claude) → Save

- `app/(app)/live/` — Recording flow (record → review → insights → save)
- `app/(app)/notes/` — Session history and detail
- `app/(app)/insights/` — Cross-session patterns and action items
- `app/(auth)/` — Auth screens
- `lib/` — API clients, hooks, store, contexts
- `services/openai-realtime.ts` — Audio recording + transcription service
- `constants/theme.ts` — Design tokens

## Key Patterns

- Per-user AES-256-GCM encryption: utterances and reflections are encrypted at rest
- All API calls use Bearer JWT auth (mobile-login endpoint)
- `safeDecrypt()` used for backward compatibility with unencrypted legacy data
- RevenueCat handles mobile IAP subscriptions
