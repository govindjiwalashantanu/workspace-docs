# Echo — Roadmap

> **Product:** Echo — voice-first, privacy-first AI thought companion
> **Repos:** `echo-backend` (Next.js webapp + API) · `echo-mobile` (Expo React Native)
> **Last updated:** March 16, 2026

---

## ✅ Completed

### Phase 0 — Backend Infrastructure
- Live session recording, utterance batching, speaker diarisation
- AI reflection pipeline (summary, follow-up question, patterns, action items)
- Per-user billing via RevenueCat (mobile) and Stripe (web)
- NextAuth v4 with Okta, Google, Credentials providers
- Multi-tenancy (organization-scoped data)
- Free tier limits (10 sessions, 15 min max)

---

### Phase 1 — Per-User Encryption at Rest ✅ March 16, 2026
Zero-knowledge AES-256-GCM encryption for all user-generated content.

**Delivered:**
- `lib/user-encrypt.ts` — `ensureUserEncryptionKey`, `encryptField`, `safeDecrypt`, `safeDecryptArray`
- Per-user keys generated on first request, encrypted with global `ENCRYPTION_KEY`, stored in `UserIntegration`
- In-memory key cache per serverless invocation (no cross-request leakage)
- Routes updated: utterances POST (encrypt write), live-sessions GET (decrypt read), reflect POST, finalize POST, sessions list GET
- `safeDecrypt` passthrough for legacy plaintext rows — fully backward compatible
- Migration script: `scripts/encrypt-existing-data.ts` — backs up all rows to `backups/` before encrypting
- `tsconfig.scripts.json` for running scripts with `npx tsx`
- Encrypted fields: `Utterance.text`, `Reflection.summary`, `Reflection.followUpQuestion`, `Reflection.patterns[]`, `Reflection.actionItems[]`
- **Phase 2 (deferred):** `NotebookNode.content`, `SpeakerProfile.name/email`

**To activate:** Set `ENCRYPTION_KEY` (64-char hex) in production env, then run migration script once.

---

### Echo Design System ✅ March 16, 2026
Consistent brand identity across webapp, mobile, docs, and pitch deck.

**Delivered:**
- New Echo logo mark: three arc-waves + voice source dot (gradient blue→cyan)
- `public/brand/`: `icon.svg`, `logo-dark.svg`, `logo-light.svg`, `favicon.svg`, `design-tokens.css`, `design-system.html`
- `design-system.html` — comprehensive reference page (identity, colors, typography, components, platform guidelines)
- `globals.css` — surface colors aligned to design system, DM Mono added
- `globals-redesign.css` — logo text spacing updated
- `components/Icons.tsx` — `AppIcon` and `BrandName` updated to Echo arc-wave mark
- `components/AppShell.tsx` — header logo updated
- All "senSEi" references replaced with "echo" across all UI, email templates, API routes, auth pages, docs, pitch page

---

### Echo Mobile — Design System Applied ✅ March 16, 2026
Applied design system and UX improvements to the React Native app.

**Delivered (echo-mobile):**
- `components/ui/EchoIcon.tsx` — Echo arc-wave mark in pure React Native (no SVG lib needed)
- `components/ui/HeaderLeft.tsx` — shared back + logo component for every screen
- Logo on every page header: main tabs (logo only), sub-screens (chevron + logo)
- `constants/theme.ts` — `fonts.body` (SF Pro / Roboto), `colors.warm` (#C9A96E) for AI content, `colors.warmSoft/warmDim`
- Typography overhaul: Courier New replaced with system sans-serif for all UI text; mono kept only for transcript text, timestamps, version numbers
- `letterSpacing` reduced from 1.5 → 0.5 across all label styles
- Warm accent applied to all AI/reflection content (Insights tab, session detail, reflection cards, sparkle icons)
- `KeyboardAvoidingView` added to recording, review, and notebook detail screens (notes input no longer covered by keyboard)
- `SpeakerAssignment` redesigned — large 40px circular icons removed, compact numbered badge + single-line input
- Login/register screens updated with Echo icon replacing mic ring

---

## 🚧 In Progress / Up Next

### Multi-Language + Multi-Provider Transcription ✅ March 18, 2026

Echo's AI value is currently English-locked despite Groq Whisper supporting 99 languages. No competitor has built a private AI voice journal targeting non-English markets. This plan makes three things happen: (1) users pick languages at signup and in settings, (2) the AI reflection responds in that language, (3) transcription routes to the best provider per language.

**Implementation is fully scoped — all files identified, all code patterns decided. Pick up and build.**

---

#### Supported Languages v1 (10)

| Code | Language | Native | Flag | Best Provider |
|------|----------|--------|------|---------------|
| `en` | English | English | 🇬🇧 | Groq |
| `hi` | Hindi | हिन्दी | 🇮🇳 | Deepgram Nova-2 |
| `ar` | Arabic | العربية | 🇸🇦 | Deepgram Nova-2 |
| `es` | Spanish | Español | 🇪🇸 | Groq |
| `fr` | French | Français | 🇫🇷 | Groq |
| `pt-BR` | Brazilian Portuguese | Português | 🇧🇷 | Groq |
| `id` | Indonesian | Bahasa Indonesia | 🇮🇩 | AssemblyAI Universal-3 |
| `bn` | Bengali | বাংলা | 🇧🇩 | Groq (fallback) |
| `tr` | Turkish | Türkçe | 🇹🇷 | AssemblyAI Universal-3 |
| `ja` | Japanese | 日本語 | 🇯🇵 | Deepgram Nova-2 |

Provider routing activates per language only when that provider's API key is in env — gracefully falls back to Groq otherwise. Zero breaking changes.

---

#### Part 1 — User Language Preference

**1a. Prisma schema — `echo-backend/prisma/schema.prisma`**
Add to User model:
```prisma
preferredLanguages String[] @default(["en"])
```
Migration SQL: `ALTER TABLE "User" ADD COLUMN "preferredLanguages" TEXT[] NOT NULL DEFAULT '{en}'`

**1b. New shared constants — `echo-backend/lib/language-constants.ts`**
```ts
export const SUPPORTED_LANGUAGES = [
  { code: 'en',    name: 'English',              nativeName: 'English',          flag: '🇬🇧' },
  { code: 'hi',    name: 'Hindi',                nativeName: 'हिन्दी',            flag: '🇮🇳' },
  { code: 'ar',    name: 'Arabic',               nativeName: 'العربية',           flag: '🇸🇦' },
  { code: 'es',    name: 'Spanish',              nativeName: 'Español',           flag: '🇪🇸' },
  { code: 'fr',    name: 'French',               nativeName: 'Français',          flag: '🇫🇷' },
  { code: 'pt-BR', name: 'Brazilian Portuguese', nativeName: 'Português',         flag: '🇧🇷' },
  { code: 'id',    name: 'Indonesian',           nativeName: 'Bahasa Indonesia',  flag: '🇮🇩' },
  { code: 'bn',    name: 'Bengali',              nativeName: 'বাংলা',             flag: '🇧🇩' },
  { code: 'tr',    name: 'Turkish',              nativeName: 'Türkçe',            flag: '🇹🇷' },
  { code: 'ja',    name: 'Japanese',             nativeName: '日本語',            flag: '🇯🇵' },
];
export const LANGUAGE_NAMES: Record<string, string> = Object.fromEntries(
  SUPPORTED_LANGUAGES.map(l => [l.code, l.name])
);
export const SUPPORTED_CODES = new Set(SUPPORTED_LANGUAGES.map(l => l.code));
```

**1c. Reflect route — `echo-backend/app/api/agent/reflect/route.ts`**

Add `detectedLanguage` to BodySchema. After billing check, fetch user's preferredLanguages and resolve lang:
```ts
const userRecord = await prisma.user.findUnique({
  where: { id: user.id }, select: { preferredLanguages: true }
});
const userLangs = userRecord?.preferredLanguages ?? ['en'];
const lang = detectedLang && userLangs.includes(detectedLang) ? detectedLang : userLangs[0];
const langInstruction = lang !== 'en'
  ? `IMPORTANT: Respond entirely in ${LANGUAGE_NAMES[lang]}. Every field MUST be in ${LANGUAGE_NAMES[lang]}. Do not use English.\n\n`
  : '';
```
Prepend `langInstruction` to the system prompt.

Also add model tier selection:
```ts
const isHighTier = billing?.mobileEntitlement === 'echo_thinker' || billing?.mobileEntitlement === 'echo_pro';
const model = isHighTier
  ? (process.env.LITELLM_MODEL ?? 'claude-sonnet-4-5')
  : (process.env.LITELLM_MODEL_HAIKU ?? 'claude-haiku-4-5');
```
Pass `model` to `callLiteLLM`. New env var: `LITELLM_MODEL_HAIKU`.

**1d. Profile PATCH — `echo-backend/app/api/user/profile/route.ts`**

Extend Zod UpdateSchema:
```ts
preferredLanguages: z.array(z.string().refine(v => SUPPORTED_CODES.has(v))).min(1).max(10).optional()
```
Add to `updateData` and the "nothing to update" check. Add to the prisma update.

**1e. Mobile-login response — `echo-backend/app/api/auth/mobile-login/route.ts`**

In the user query, add `preferredLanguages: true` to select. Add `preferredLanguages: user.preferredLanguages` to the returned `user` object.

**1f. Mobile auth store — `echo-mobile/lib/store/auth-store.ts`**

Add `preferredLanguages: string[]` to the `AuthUser` interface.

**1g. SetupWizard — `echo-mobile/components/onboarding/SetupWizard.tsx`**

Insert new Step 1 (Language Picker) between role (step 0) and use case (now step 2). Total 4 steps — update progress dots to `[0,1,2,3]`.

New step UI:
- Multi-select chips: `flag + nativeName` for all 10 languages
- Cyan highlight when selected, at least 1 required, English pre-selected
- Subheading: "Select all languages you speak — Echo will adapt to each recording"
- Add `onLanguagesSelected: (langs: string[]) => void` prop to `Props`
- Call it on "Next" from step 1; parent sends `PATCH /api/user/profile`
- Navigation: step 0 → step 1 → step 2 → step 3

**1h. Profile screen — `echo-mobile/app/(app)/profile.tsx`**

New "Languages" row in Account Settings section (before Reset Password):
- Shows selected flags inline, e.g. `🇬🇧 🇮🇳 >`
- Tapping opens a Modal with the same multi-select chip picker
- On confirm: `PATCH /api/user/profile` + update `useAuthStore`

---

#### Part 2 — Multi-Provider Transcription Routing

**2a. New file — `echo-backend/lib/transcription-router.ts`**

```ts
type Tier = 'free' | 'personal' | 'thinker' | 'pro';

const ACCURACY_ROUTING: Record<string, 'groq-turbo' | 'groq-v3' | 'deepgram' | 'assemblyai'> = {
  hi: 'deepgram', ar: 'deepgram', ja: 'deepgram', ko: 'deepgram',
  id: 'assemblyai', tr: 'assemblyai',
};

export function selectProvider(input: { lang: string; tier: Tier }) {
  if (input.tier === 'free') {
    return { provider: 'groq', model: 'whisper-large-v3-turbo', costPerMin: 0.00067 };
  }
  const preferred = ACCURACY_ROUTING[input.lang];
  if (preferred === 'deepgram' && process.env.DEEPGRAM_API_KEY) {
    return { provider: 'deepgram', model: 'nova-2', costPerMin: 0.0043 };
  }
  if (preferred === 'assemblyai' && process.env.ASSEMBLYAI_API_KEY) {
    return { provider: 'assemblyai', model: 'universal-3', costPerMin: 0.0062 };
  }
  return { provider: 'groq', model: 'whisper-large-v3', costPerMin: 0.00185 };
}
```

**v1 routing decision (final):** All tiers → Groq Turbo initially (healthy margins globally, 99 languages). The router infrastructure is in place for when Deepgram/AssemblyAI keys are added.

**2b. Transcribe route — `echo-backend/app/api/transcribe/route.ts`**

- Accept optional `lang` query param
- Determine tier from billing (for now: default to 'personal' — gate already checks Pro)
- Call `selectProvider({ lang, tier })`
- Branch: Groq path (existing), Deepgram path (new), AssemblyAI path (new)
- Each path returns `{ text: string, detectedLanguage?: string }`

**Deepgram implementation:**
```ts
// POST to https://api.deepgram.com/v1/listen?model=nova-2&language=hi&punctuate=true&smart_format=true
// Headers: Authorization: Token DEEPGRAM_API_KEY
// Body: raw audio binary
// Response: results.channels[0].alternatives[0].transcript
```

**AssemblyAI implementation:**
```ts
// 1. POST audio to https://api.assemblyai.com/v2/upload → get upload_url
// 2. POST { audio_url: upload_url, language_code: 'id' } to /v2/transcript → get id
// 3. Poll GET /v2/transcript/{id} until status === 'completed' (max ~60s with 2s intervals)
// 4. Return text + language_code
```

**2c. Mobile — pass language to transcribe — `echo-mobile/services/openai-realtime.ts`**

In `transcribeChunk`, resolve lang:
1. `useAuthStore.getState().user?.preferredLanguages?.[0]`
2. `Localization.locale` from `expo-localization` (trim to base code)
3. `'en'` fallback

Append `?lang={code}` to the fetch URL: `` `${API_BASE}/api/transcribe?lang=${lang}` ``

**2d. Auto-detect on first recording (nice-to-have, implement after core)**
If `detectedLanguage` comes back from transcription and user's `preferredLanguages` is still `['en']` (default), silently PATCH profile.

**2e. Arabic RTL on mobile**
When `preferredLanguages[0] === 'ar'`, add `writingDirection: 'rtl', textAlign: 'right'` to text-heavy screens (recording detail, reflections display, chat). Not needed for controls/nav.

**2f. New env vars:**
```
DEEPGRAM_API_KEY=        # Nova-2 for Hindi, Arabic, Japanese
ASSEMBLYAI_API_KEY=      # Universal-3 for Indonesian, Turkish
LITELLM_MODEL_HAIKU=     # Cheaper model alias for free + personal tiers
```

---

#### Part 3 — Localized Hero Page

**3a. `echo-backend/next.config.ts`** — Add i18n config:
```ts
i18n: {
  locales: ['en', 'hi', 'ar', 'id'],
  defaultLocale: 'en',
  localeDetection: true,
},
```
Note: `app/page.tsx` uses App Router. Since `useRouter().locale` is Pages Router, detect locale client-side via `navigator.language` in a `useEffect`, mapped to `['en','hi','ar','id']`.

**3b. New file — `echo-backend/lib/hero-content.ts`**
```ts
export const HERO_CONTENT = {
  en: {
    headline: 'Private AI for any conversation worth remembering.',
    subheadline: 'Speak freely. Echo instantly structures your thoughts into summaries, action items, and insights. Zero-knowledge encryption means only you can read them.',
    ctaIOS: 'Download for iOS',
    ctaAndroid: 'Download for Android',
    privacyBadge: '🔒 AES-256 per-user encryption · No AI training on your data',
    rtl: false,
  },
  hi: {
    headline: 'हर ज़रूरी बातचीत के लिए निजी AI।',
    subheadline: 'बोलिए बेझिझक। Echo आपके विचारों को तुरंत सारांश, कार्य-सूची और अंतर्दृष्टि में बदल देता है। ज़ीरो-नॉलेज एन्क्रिप्शन — सिर्फ आप पढ़ सकते हैं।',
    ctaIOS: 'iOS के लिए डाउनलोड करें',
    ctaAndroid: 'Android के लिए डाउनलोड करें',
    privacyBadge: '🔒 प्रति-उपयोगकर्ता AES-256 एन्क्रिप्शन · आपका डेटा AI प्रशिक्षण में उपयोग नहीं',
    rtl: false,
  },
  ar: {
    headline: 'ذكاء اصطناعي خاص لكل محادثة تستحق التذكر.',
    subheadline: 'تحدث بحرية. يحوّل Echo أفكارك فوراً إلى ملخصات وخطوات عمل ورؤى. تشفير بدون معرفة — أنت فقط من يمكنه القراءة.',
    ctaIOS: 'تنزيل لنظام iOS',
    ctaAndroid: 'تنزيل لنظام Android',
    privacyBadge: '🔒 تشفير AES-256 لكل مستخدم · بياناتك لا تُستخدم لتدريب الذكاء الاصطناعي',
    rtl: true,
  },
  id: {
    headline: 'AI Pribadi untuk Setiap Percakapan yang Berharga.',
    subheadline: 'Bicara bebas. Echo langsung mengubah pikiran Anda menjadi ringkasan, daftar tugas, dan wawasan. Enkripsi zero-knowledge — hanya Anda yang bisa membacanya.',
    ctaIOS: 'Unduh untuk iOS',
    ctaAndroid: 'Unduh untuk Android',
    privacyBadge: '🔒 Enkripsi AES-256 per pengguna · Data Anda tidak digunakan untuk melatih AI',
    rtl: false,
  },
};
```

**3c. `echo-backend/app/page.tsx`**
- Import `HERO_CONTENT`
- Add `useEffect` to detect locale from `navigator.language`, map to `['en','hi','ar','id']`, default to `'en'`
- Replace hardcoded headline, subheadline, CTA text, privacy badge with `content[locale].*`
- For Arabic (`rtl: true`): add `dir="rtl"` to hero section root div, flip flex to `row-reverse`

---

#### Files to touch (complete list)

| File | Change |
|------|--------|
| `echo-backend/prisma/schema.prisma` | Add `preferredLanguages String[] @default(["en"])` to User |
| `echo-backend/lib/language-constants.ts` | **NEW** |
| `echo-backend/lib/transcription-router.ts` | **NEW** |
| `echo-backend/lib/hero-content.ts` | **NEW** |
| `echo-backend/app/api/agent/reflect/route.ts` | Language-aware prompt + model tier selection |
| `echo-backend/app/api/user/profile/route.ts` | Accept `preferredLanguages` array |
| `echo-backend/app/api/auth/mobile-login/route.ts` | Return `preferredLanguages` |
| `echo-backend/app/api/transcribe/route.ts` | Provider routing + `lang` query param |
| `echo-backend/next.config.ts` | Add i18n locale config |
| `echo-backend/app/page.tsx` | Locale-aware copy from hero-content.ts + RTL |
| `echo-mobile/lib/store/auth-store.ts` | Add `preferredLanguages: string[]` to AuthUser |
| `echo-mobile/services/openai-realtime.ts` | Append `?lang=` to transcribe URL |
| `echo-mobile/components/onboarding/SetupWizard.tsx` | New language picker step (step 1 of 4) |
| `echo-mobile/app/(app)/profile.tsx` | Languages row + picker modal + PATCH |

---

#### Verification checklist (run after build)
1. `cd /Documents/echo-backend && npx vitest run` — all pass
2. Register → pick Hindi → send test transcript → reflection returns Hindi
3. Set `DEEPGRAM_API_KEY` → Hindi recording → verify Deepgram called in server logs
4. Remove `DEEPGRAM_API_KEY` → Hindi recording → graceful Groq fallback
5. Visit `/hi` → Hindi copy; visit `/ar` → Arabic RTL
6. Change language to Japanese in Profile → next reflection returns Japanese
7. Save Hindi/Arabic reflection → reload → decrypts correctly

---

### Phase 2 — Encryption for NotebookNode.content
Extend encryption to meeting transcript content stored in `NotebookNode.content`.

**Why deferred:** Too many interdependent routes read/write `NotebookNode.content` (POC guide, AI agents, search, export). Needs careful route audit before encrypting.

**Scope:**
- Encrypt `NotebookNode.content` for `type: meeting` nodes
- Decrypt in all read paths: notebook GET, search, POC extract, AI agents, DOCX/PDF export
- Migration script update to include notebook nodes

---

### Scramble Mode (Pro)
Distribute audio chunks across multiple transcription providers so no single provider hears the full recording.

**Status:** Designed, not implemented.

---

### App Icon PNG Export
`assets/images/icon.svg` in `echo-mobile` has been updated to the new Echo arc-wave mark but the `.png` files still contain the old SE lettermark.

**Action needed:** Export from the SVG in Figma/Sketch:
- `icon.png` — 1024×1024
- `splash-icon.png` — 200×200
- `android-icon-foreground.png` — 1024×1024
- `android-icon-background.png` — 1024×1024

---

## 🔮 Backlog

### Mobile UX
- **Onboarding flow** — first-run experience after registration
- **Offline recording** — queue utterances locally when network is unavailable, sync on reconnect
- **Search** — full-text search across all sessions and reflections
- **Pattern view** — surface recurring themes across multiple recordings over time

### Webapp
- **Sessions page** — currently basic list; add filtering by date, tag, reflection status
- **Transcript view** — better formatting with speaker colors, timestamps, playback markers
- **Export** — PDF/DOCX export for sessions and reflections

### Infrastructure
- **Redis key cache** — replace in-memory key cache with Redis for better cold-start performance
- **Phase 2 encryption** — `NotebookNode.content` (see above)
- **Rate limiting** — protect transcription and AI routes from abuse
- **Audit log** — track key creation and encryption events per user

### Growth
- **Referral system** — invite a friend for extra recording hours
- **Team plan** — shared workspace for small teams
- **Zapier/Make integration** — trigger workflows from reflection action items

---

## Known Issues

| Issue | Severity | Notes |
|---|---|---|
| App icon PNGs still show old SE mark | Medium | Need manual PNG export from updated icon.svg |
| `utils.test.ts` date test flakes in UTC | Low | Pre-existing timezone issue, unrelated to Echo |

---

## Environment

- **Backend:** Next.js 16, TypeScript, Prisma, PostgreSQL, NextAuth v4, Vercel
- **Mobile:** Expo ~54, React Native 0.81, expo-router ~6, Zustand, TanStack Query
- **AI:** LiteLLM proxy → Claude Sonnet
- **Billing:** Stripe (web) + RevenueCat (mobile)
- **Encryption:** AES-256-GCM, per-user keys in UserIntegration

## Repos

- `echo-backend` → `github.com/govindjiwalashantanu/echo-backend`
- `echo-mobile`  → `github.com/govindjiwalashantanu/echo-mobile`
