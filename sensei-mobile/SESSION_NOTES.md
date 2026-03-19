# sensei-mobile — Session Notes
_Last updated: March 18, 2026_

---

## Session: March 18, 2026 — Recording destination picker, AI analysis screen, push notifications, Feed tab

### Summary
Overhauled the recording save flow, added AI analysis surfacing on meeting notes, push notifications when analysis completes, and a new chronological Feed as the primary tab.

---

### 1. Recording destination picker (`1e0ece9`)

**review.tsx — full redesign:**
- Removed speaker assignment (Whisper doesn't reliably diarize)
- Added destination picker: **Note / Meeting / Inject into existing**
- Note: saves immediately as note node, navigates to it
- Meeting: proceeds to save.tsx (title + parent — unchanged)
- Inject: shows meeting list (last 7 days), optional account/opp filter, appends transcript

**recording.tsx:**
- Button label: "Stop & Transcribe" → "Stop & Save Transcription"

**lib/api/live.ts:**
- `injectSession(sessionId, meetingNodeId)` — POST `.../inject`
- `saveSessionAsNote(sessionId, title?, parentNodeId?)` — POST `.../save-as-note`

**LiveSessionContext.tsx:**
- `injectIntoMeeting(meetingNodeId)` — flushes buffer then calls inject
- `saveAsNote(title?, parentNodeId?)` — flushes buffer then calls save-as-note

---

### 2. AI analysis on meeting screen + push notifications (`0c68c9f`)

**app/(app)/notebook/[nodeId].tsx:**
- Meeting nodes: AI summary + action items (read-only list) shown at top
- "Analysing your recording…" spinner while analysis pending
- Transcript collapsed by default (tap to expand), shows char count
- Auto-polls `/api/notebook` every 3s until summary property appears, then stops
- AI-generated keys filtered from standard properties table

**expo-notifications installed:**
- `lib/utils/push-notifications.ts`: `registerPushToken()` + `getNodeIdFromNotification()`
- `app.json`: expo-notifications plugin + Android permissions

**app/(app)/_layout.tsx:**
- Registers push token on login (`user?.id` dep)
- Notification tap handler → navigates to the meeting node (`nodeId` from notification data)

---

### 3. Feed tab as primary screen (`ffa63d5`)

**app/(app)/feed/index.tsx (new primary tab):**
- Replaces "Accounts" as tab 1
- Flat chronological list: meetings + notes, newest first
- Search bar: searches title, AI summary, action items, transcript
- FeedCard: type pill, date, parent account/opp name, summary preview, action item count badge, analysing spinner
- Header right: notebook icon → pushes notebook tree view

**_layout.tsx:**
- `feed/index` is now tab 1 (`initialRouteName: 'feed/index'`)
- `notebook/index` hidden from tabs, accessible from Feed header

**lib/utils/feed-filter.ts:**
- `filterFeedNodes(nodes, query)` — filters meetings+notes, searches 4 fields, sorts newest first
- `parseActionItems(node)` — safely parses actionItems JSON

---

### Commits

| Description | Hash |
|---|---|
| feat: recording destination picker — note, meeting, inject | `1e0ece9` |
| feat: AI analysis on meeting screen + push notifications | `0c68c9f` |
| feat: Feed tab as primary screen — meetings/notes feed with global search | `ffa63d5` |
| fix: review screen safe area + background recording chunk rotation | `53d0383` |
| design: light theme — direction B (matches sensei-webapp) | `2387ca1` |

---

### Next Session — Where to Pick Up

- **EC2 env var needed**: Add `RESEND_API_KEY` + `EMAIL_FROM` on EC2 for emails to work
- **Expo push** requires a physical device to test (simulator doesn't get tokens)
- **Feed pagination**: currently loads all nodes — add pagination if dataset grows large
- **sensei-mobile has no SESSION_NOTES in the repo itself** — this file lives in workspace-docs only

---

## Architecture Notes

### Recording flow (current)
```
recording → review (transcript + destination picker) →
  Note: inject endpoint → navigate to note
  Meeting: save.tsx (title + parent) → finalize → navigate to meeting
  Inject: meeting picker → inject endpoint → navigate to meeting
```

### Meeting node detail (current)
- AI Summary at top (from `summary` NodeProperty)
- Action items read-only list (from `actionItems` NodeProperty, JSON array)
- "Analysing…" spinner while pending (polls every 3s)
- Transcript collapsed by default

### Push notification flow
1. Mobile registers Expo push token → `POST /api/user/push-token` → stored in `UserIntegration`
2. User records → finalize → analyze kicks off in background with `sendNotification: true`
3. Analyze completes → `sendExpoPush()` → Expo push API → device receives notification
4. User taps notification → `addNotificationResponseReceivedListener` → navigate to meeting nodeId
