# Plan: Standalone Note to Self — Single HTML File

## Overview

Convert the Lovable/TanStack MVP into a **single self-contained `index.html`** file that
implements the agreed feature set using only vanilla HTML, CSS, and JavaScript —
no build tools, no framework, no npm. The file must work by simply opening it in any
modern browser (Chrome, Safari, Firefox, Edge).

Source reference: `note-to-self-src/note-to-self/`

### Agreed Scope (finalised)

| # | Sub-Task | Included | Coins |
|---|---|---|---|
| 1 | Design System & Shell | ✅ | ~15 |
| 2 | Chat Core — Send, bubbles, delete | ✅ | ~10 |
| 3 | Auto-Replies Panel | ✅ | ~12 |
| 4 | Scheduled Messages | ✅ | ~8 |
| 5 | Drawing Panel | ✅ | ~12 |
| 6 | Voice Transcription (Whisper CDN) | ❌ Cut | — |
| 7 | Profile Photo, Settings & localStorage | ❌ Cut | — |
| 8 | AI Onboarding Guide Agent | ✅ | ~18 |
| 9 | Polish & Accessibility | ✅ | ~5 |
| **Total** | | | **~80 coins** |

---

## Sub-Tasks

---

### Sub-Task 1 — Design System & Shell

**Intent**
Reproduce the visual design token system from `src/styles.css` (oklch colors, chat bubbles,
radius scale, dark-mode variables) in plain `<style>` inside the HTML file, and lay out the
two-screen shell: a **Home** screen and a **Chat** screen shown/hidden via a single CSS class
toggle — no router needed.

**Expected Outcomes**
- Light and dark color themes match the Lovable app.
- Home screen renders with heading, subtitle, "Open Note to Self" button, and How-it-works cards.
- Chat screen renders the header (avatar, title, settings icon), message list area, and bottom
  toolbar — correctly laid out at all viewport sizes.
- Switching between screens works.

**Todo List**
1. Create `note-to-self/index.html` with `<!DOCTYPE html>` boilerplate and a `<style>` block.
2. Port all CSS custom properties from `styles.css` (`:root` and `.dark` blocks) verbatim.
3. Add utility classes for bubbles, pills, cards, and buttons used in the Lovable UI.
4. Write the Home screen HTML matching `src/routes/index.tsx` layout and copy.
5. Write the Chat screen HTML skeleton matching `src/routes/chat.tsx` layout.
6. Add a `showScreen(id)` JS function that toggles visibility between `#screen-home` and
   `#screen-chat`.

**Relevant Context**
- `src/styles.css` — full token set to copy.
- `src/routes/index.tsx` — home screen layout and exact copy.
- `src/routes/chat.tsx` — chat screen shell and header.

**Status** `[ ] pending`

---

### Sub-Task 2 — Chat Core: Send, Receive, Delete

**Intent**
Implement the core messaging loop: typing a text message, pressing Send, showing it as a
right-aligned bubble with a timestamp, and deleting individual messages with a trash icon.
Store the message array in a JS variable (in-memory only for now — persistence comes in
Sub-Task 4).

**Expected Outcomes**
- Typing in the input and pressing Send (or Enter) adds a right-aligned bubble.
- Each bubble shows the time and a delete icon (visible on hover / always on mobile).
- Clicking delete removes that bubble.
- The message list scrolls to the latest message automatically.

**Todo List**
1. Define a `messages = []` array and a `renderMessages()` function.
2. Wire the Send button and form `submit` event to `pushMsg({text})` → `renderMessages()`.
3. Implement `renderMessages()` to build bubble HTML for each message item.
4. Add delete button handler that removes by `id` and re-renders.
5. Auto-scroll the list container to the bottom after every render.

**Relevant Context**
- `src/routes/chat.tsx` `send()`, `push()`, `bubble()`, message map JSX.
- `src/lib/chat-storage.ts` `Msg` type shape (`id`, `text`, `at`, `auto`, `scheduled`, `image`).

**Status** `[ ] pending`

---

### Sub-Task 3 — Auto-Replies Panel

**Intent**
Implement the Auto-Replies dialog: add keyword→message rules (text or drawing type),
list existing rules, delete rules, save to `localStorage`, and fire a matching auto-reply
700 ms after a matching outbound message is sent.

**Expected Outcomes**
- "Auto-Replies" toolbar button opens a modal dialog.
- User can type keywords (comma-separated) and a message, choose Text or Drawing type, and
  click "Add auto-reply" to append to the list.
- Existing rules are listed with a delete button.
- "Save" persists rules to `localStorage`; rules reload on page open.
- Sending a text message that contains a keyword fires the auto-reply bubble after 700 ms.

**Todo List**
1. Port `matchReply()` from `src/lib/auto-replies.ts` verbatim as a JS function.
2. Build the Auto-Replies dialog HTML (rule list + add-rule form + Text/Drawing toggle).
3. Wire "Add auto-reply" to push a rule into a `rules[]` array and re-render the list.
4. Wire "Save" to `localStorage.setItem("self-rules", JSON.stringify(rules))` and close.
5. On app load, read rules from `localStorage`.
6. After `pushMsg` in Sub-Task 2, call `matchReply` and schedule auto-reply with
   `setTimeout(..., 700)`.

**Relevant Context**
- `src/lib/auto-replies.ts` — `Rule` type and `matchReply()` to port.
- `src/routes/chat.tsx` `RulesPanel` component and `send()` auto-reply wiring.

**Status** `[ ] pending`

---

### Sub-Task 4 — Scheduled Messages

**Intent**
Add the Schedule toolbar button that shows a `datetime-local` input. When a time is set and
Send is pressed, the message is added to a `scheduled[]` array and a `setTimeout` fires to
deliver it at the right moment — matching the `deliverDue()` logic from `chat-storage.ts`.

**Expected Outcomes**
- Clicking the Schedule button shows/hides a "Schedule for" datetime row above the input bar.
- With a future time set, pressing Send adds a green scheduled-bubble placeholder and shows
  "N scheduled" count.
- The message appears in the chat at the scheduled time (left-aligned, green, labelled
  "scheduled from you").
- The datetime input and schedule row close after scheduling.

**Todo List**
1. Add the "Schedule for" row HTML (datetime input + cancel button) above the input bar.
2. Toggle it with the Schedule toolbar button.
3. In `pushMsg`, detect `when` is set: push to `scheduled[]` and compute `setTimeout` delay.
4. The `setTimeout` callback calls `deliverMsg(m)` which pushes to `messages[]` and re-renders.
5. Show a "N scheduled" counter next to the save-toggle when `scheduled.length > 0`.

**Relevant Context**
- `src/routes/chat.tsx` `schedule()`, `showSchedule` state, scheduled bubble rendering.
- `src/lib/chat-storage.ts` `deliverDue()`.

**Status** `[ ] pending`

---

### Sub-Task 5 — Drawing Panel

**Intent**
Implement the Draw toolbar button that opens a modal with a `<canvas>` for freehand
drawing, a schedule field, an optional drawing-category selector (populated from drawing-type
auto-reply rules), and a Send button that posts the canvas image as a `data:image/png` message.

**Expected Outcomes**
- Clicking Draw opens a dialog with a white canvas that supports freehand pointer drawing.
- "Clear" wipes the canvas.
- Category dropdown is populated with drawing types from existing rules.
- Optional schedule datetime works the same as Sub-Task 4.
- Sending embeds the PNG data URL as an `<img>` inside a message bubble.

**Todo List**
1. Add draw dialog HTML: canvas, category select, schedule input, Clear/Cancel/Send buttons.
2. Implement pointer-event drawing (pointerdown, pointermove, pointerup) matching
   `DrawPanel` in `chat.tsx`.
3. Populate the category `<select>` from `rules.filter(r => r.type === "drawing")`.
4. On Send: call `toDataURL("image/png")`, push `{image: dataUrl}` message or schedule it.
5. Wire drawing-type auto-replies: call `matchReply(rules, category, "drawing")` and fire reply.

**Relevant Context**
- `src/routes/chat.tsx` `DrawPanel` component — canvas pointer logic and send wiring.

**Status** `[ ] pending`

---

### Sub-Task 6 — AI Onboarding Guide Agent

**Intent**
Add a scripted onboarding guide bot that fires only on a user's first visit. It greets the
user, asks how they are feeling, responds empathetically to their answer, then walks them
step-by-step through creating their first Auto-Reply. The guide uses a distinct avatar,
appears as left-aligned "guide" bubbles with a gentle typing delay, and goes permanently
silent after the walkthrough completes. No external AI API is needed — the guide is a
self-contained conversation tree.

**Expected Outcomes**
- On first open (checked via `localStorage` flag), the guide fires automatically after 800 ms.
- Guide sends: "Hi! How are you feeling today? 🙂"
- User types a reply; the guide reads sentiment keywords (sad, happy, anxious, good, etc.)
  and responds with an empathetic message tailored to that tone.
- Guide then sends four onboarding steps — one bubble at a time with 1.2 s delays between each.
- Step 4 pre-fills the keyword field in the Auto-Replies panel with the user's feeling word
  when the panel is opened from the guide prompt.
- After the final message the guide sets `localStorage` flag and never fires again.
- Guide bubbles are visually distinct: ✨ avatar, soft green background, labelled "your guide".
- A **"Guide: On/Off" toggle** is visible in the chat header. Turning it **off** immediately
  silences any in-progress guide sequence and sets the done flag so the guide never starts
  again. Turning it back **on** clears the flag so the guide restarts on the next page load,
  and shows a small note: "Guide will restart on your next visit."

**Conversation Tree**

```
Guide: Hi! How are you feeling today? 🙂
  ↓ user replies
Guide: [empathetic response based on sad/happy/anxious/other]
Guide: Let me show you how to set up an Auto-Reply for moments like this.
Guide: Step 1 — Tap "Auto-Replies" in the toolbar below ↓
Guide: Step 2 — Type a keyword like "[user's feeling word]" and a kind message to yourself.
Guide: Step 3 — Tap "Add auto-reply" then "Save".
Guide: You're all set! Try sending that word now and watch what happens 🙂
  → Guide goes silent
```

**Todo List**
1. Define a `guideScript[]` array of step objects: `{text, delay}`.
2. On app load, check `localStorage.getItem("self-guide-done")`; if absent, call
   `startGuide()` after 800 ms.
3. `startGuide()` pushes the first guide bubble (`{text, guide: true}`) and sets a flag
   `awaitingFeelingReply = true`.
4. In `pushMsg`, if `awaitingFeelingReply` is true, capture the user's message, detect
   sentiment, push the tailored empathetic response, then queue the 4 step bubbles with
   `setTimeout` delays.
5. Store the user's feeling word in a variable so Sub-Task 3's Auto-Replies panel can
   optionally pre-fill the keyword input when opened during the guide flow.
6. After the final guide bubble, set `localStorage.setItem("self-guide-done", "true")`.
7. Style guide bubbles with a distinct background, ✨ avatar, and "your guide" label.
8. Add a **"Guide" labelled toggle** (checkbox or switch) to the chat header next to the
   settings icon.
   - **Toggle off**: cancel all pending guide `setTimeout` timers, reset
     `awaitingFeelingReply = false`, set `localStorage.setItem("self-guide-done", "true")`.
   - **Toggle on**: `localStorage.removeItem("self-guide-done")`, show a one-line note
     "Guide will restart on your next visit." The guide does not re-fire mid-session to
     avoid interrupting an active chat.
   - Persist the toggle preference itself to `localStorage` under `"self-guide-enabled"` so
     the header reflects the correct state on reload.

**Relevant Context**
- `src/routes/chat.tsx` auto-reply bubble rendering (`auto: true`) — reuse the same
  left-aligned bubble pattern for guide bubbles.
- `src/lib/auto-replies.ts` `matchReply()` — sentiment detection uses the same
  keyword-matching pattern.

**Status** `[ ] pending`

---

### Sub-Task 7 — Polish & Accessibility

**Intent**
Ensure the finished file is accessible, visually faithful to the Lovable original, and works
as a zero-dependency drop-in. Validate all features end-to-end.

**Expected Outcomes**
- All interactive elements have `aria-label` or visible labels.
- Dark mode works (respects `prefers-color-scheme`).
- The file opens correctly from `file://`.
- No console errors on load.
- Privacy wording on the Home screen matches the original exactly.

**Todo List**
1. Add `@media (prefers-color-scheme: dark)` to toggle `.dark` class on `<html>`.
2. Audit all buttons and inputs for `aria-label` / `title` attributes.
3. Verify `file://` open works for all features.
4. Manual end-to-end test: send, auto-reply, schedule, draw, guide onboarding, clear.

**Relevant Context**
- `src/routes/index.tsx` privacy section copy — replicate verbatim.
- `src/routes/chat.tsx` aria attributes throughout.

**Status** `[x] done`

---

### Sub-Task 8 — Agentic RAG AI Guide

**Intent**
Replace the scripted onboarding guide with a fully local **Agentic RAG** (Retrieval-Augmented
Generation) system. No external API, no model download, no internet required — everything
runs in the browser using plain JavaScript.

The system has three layers:
1. **Knowledge base** — a curated set of app help + emotional support chunks stored as JS arrays
2. **Retriever** — fuzzy keyword scorer that finds the most relevant chunk for the user's message
3. **Agent** — an intent classifier that decides not just *what to say* but *what action to take*
   (open a dialog, pre-fill a field, highlight a button)

This makes the guide feel genuinely intelligent and interactive rather than just chatty.

**Architecture**

```
User message
     ↓
[RETRIEVER]  score all knowledge chunks → pick best match (cosine-like keyword overlap)
     ↓
[INTENT CLASSIFIER]  what does the user need?
   → "app-help"       → explain feature + optionally trigger UI action
   → "schedule"       → explain + call showScheduleRow()
   → "auto-reply"     → explain + call openRules() with pre-filled keyword
   → "draw"           → explain + call openDraw()
   → "emotional"      → warm empathetic response + suggest Auto-Reply
   → "positive"       → affirm + encourage
   → "unknown"        → warm fallback
     ↓
[RESPONDER]  compose reply from retrieved chunk + intent template
     ↓
Guide bubble appears after 800ms delay (feels natural)
```

**Expected Outcomes**
- Guide stays active as a persistent assistant after onboarding, as long as Guide toggle is on.
- Handles paraphrasing — "how do I set a reminder?" and "can I send a future message?" both
  trigger the scheduling response, because the retriever scores keyword overlap not exact match.
- Agent takes real UI actions: opening the schedule row, pre-filling keywords in Auto-Replies,
  opening the draw dialog — based on what the user is asking about.
- Emotional messages receive warm, varied empathetic replies drawn from the knowledge base.
- Truly unknown inputs get a gentle fallback: "I'm not sure about that, but I'm here. Try
  sending yourself a kind note 🙂"
- If Guide is toggled off, the agent goes completely silent — normal chat resumes.
- Conversation context: the agent remembers the last 3 turns so follow-up questions
  ("how?" after "you can schedule messages") resolve correctly.

**Knowledge Base Categories**
| Category | Example triggers | Agent action |
|---|---|---|
| Scheduling | schedule, remind, future, later, time, clock | Opens schedule row |
| Auto-Replies | auto, keyword, reply, trigger, word | Opens Auto-Replies panel, pre-fills keyword |
| Drawing | draw, sketch, picture, canvas, doodle | Opens draw dialog |
| App overview | what is, what can, how does, help, guide | Explains the app |
| Privacy | private, data, saved, server, track | Reads privacy section copy |
| Emotional — sad | sad, down, hurt, lonely, lost, bad day | Empathetic reply + Auto-Reply suggestion |
| Emotional — anxious | anxious, nervous, worried, stress, panic | Calming reply + breathing tip |
| Emotional — tired | tired, exhausted, burnt out, overwhelmed | Gentle reply + schedule a rest note |
| Positive | happy, great, good, excited, grateful | Affirming reply |
| Unknown | anything else | Warm fallback |

**Todo List**
1. Build `knowledgeBase[]` — array of `{ id, category, keywords[], chunks[] }` objects.
   Each chunk is a short natural-language response string (2–4 sentences).
2. Write `retrieve(input)` — scores each knowledge base entry by counting keyword overlaps
   with the lowercased input; returns the top-scoring entry (or null if score = 0).
3. Write `classifyIntent(entry, input)` — maps the retrieved entry's category to an
   intent string: `"schedule" | "auto-reply" | "draw" | "app-help" | "privacy" |
   "emotional-sad" | "emotional-anxious" | "emotional-tired" | "positive" | "unknown"`.
4. Write `agentAct(intent, input)` — picks a response chunk from the retrieved entry
   (rotate through chunks[] for variety) and schedules any UI side-effect:
   - `"schedule"` → `setTimeout(showScheduleRow, 1800)`
   - `"auto-reply"` → `setTimeout(() => openRules(feelingWord), 1800)`
   - `"draw"` → `setTimeout(openDraw, 1800)`
   - others → response only, no UI action
5. Write `askAgent(input)` — orchestrates retrieve → classifyIntent → agentAct.
   Returns the reply string.
6. Maintain `agentContext[]` — a rolling window of the last 3 `{ role, text }` turns.
   On each call, prepend context keywords to the retrieval query so follow-up questions
   resolve correctly (e.g. "how?" after a scheduling explanation re-scores scheduling).
7. In `sendMessage()`, after the user's message is pushed, if `guideEnabled` and not
   `awaitingFeelingReply`, call `askAgent(input)` and push the reply as a guide bubble
   after 800 ms. Auto-replies still fire independently after.
8. In `openRules()`, accept an optional `prefillKeyword` parameter so the agent can
   pre-fill the keyword field when it opens the panel mid-conversation.

**Relevant Context**
- `note-to-self/index.html` — `sendMessage()`, `pushGuide()`, `openRules()`, `openDraw()`,
  `showScheduleRow()` / `toggleSchedule()` — all need small updates to wire the agent in.
- `src/lib/auto-replies.ts` `matchReply()` — the keyword-scoring approach is the same idea,
  extended to score full entries rather than single rules.

**Estimated Coins** ~30

**Status** `[ ] pending`

---

### Sub-Task 9 — Manual Dark / Light Mode Toggle

**Intent**
Add a sun/moon toggle button visible on both the Home screen and the Chat screen header.
Clicking it manually switches between light and dark mode, persists the preference to
`localStorage`, and overrides the OS `prefers-color-scheme` setting.

**Expected Outcomes**
- A sun icon (☀️) in light mode / moon icon (🌙) in dark mode appears in both screens.
- Clicking toggles the `.dark` class on `<html>` immediately.
- Preference is saved to `localStorage` under `"self-theme"` and restored on next load.
- Manual preference always overrides the OS setting; if no manual preference is set, the
  OS preference is still respected as before.

**Todo List**
1. Add a theme toggle button to the Home screen (top-right corner).
2. Add a theme toggle button to the Chat screen header (next to Settings).
3. Write `toggleTheme()` — flips `.dark` on `<html>`, saves to `localStorage`.
4. In `applyDark()` (dark mode init), check `localStorage.getItem("self-theme")` first;
   only fall back to `prefers-color-scheme` if no manual preference is stored.
5. Update button icon dynamically (☀️ when dark, 🌙 when light).

**Relevant Context**
- Existing `applyDark()` IIFE in `note-to-self/index.html` — extend, don't replace.

**Status** `[ ] pending`

---

### Sub-Task 10 — Real Screenshots in Home Screen Cards

**Intent**
Replace the placeholder `<div class="how-card-img">` elements in the Home screen "How it
works" section with actual screenshots of the finished app — matching the Lovable MVP
exactly. Screenshots are captured manually (or via browser DevTools), cropped, and embedded
as base-64 PNGs directly in the HTML so the file stays self-contained.

**Expected Outcomes**
- The "Note to Self" card shows a real screenshot of the chat interface with bubbles,
  the toolbar (Auto-Replies, Draw, Schedule), and the input bar.
- The "Auto-Replies" card shows a real screenshot of the Auto-Replies dialog with a rule
  listed and the add-rule form visible.
- Images are `h-96 w-full object-contain` matching the Lovable original.
- No external image URLs — screenshots are embedded as `data:image/png;base64,...`.

**Todo List**
1. Finish all other sub-tasks first (screenshots must show the final UI).
2. Open `note-to-self/index.html` in Chrome, navigate to Chat, send a few messages,
   open Auto-Replies and add a rule — then use DevTools device toolbar to set 390×844
   (iPhone-sized) viewport.
3. Capture screenshot of the chat view → save as PNG → convert to base-64.
4. Capture screenshot of the Auto-Replies dialog → save as PNG → convert to base-64.
5. Replace both `<div class="how-card-img">...</div>` placeholders in `index.html` with
   `<img src="data:image/png;base64,..." alt="..." class="how-card-img" />`.

**Relevant Context**
- `src/routes/index.tsx` lines 32–42 — original `<img>` elements and their classes.
- `note-to-self/index.html` Home screen section — find the two `how-card-img` divs.

**Status** `[ ] pending`

---

### Sub-Task 11 — Commit to GitHub

**Intent**
Push the finished `note-to-self/index.html` to the `durgaIyer100/bobNoteToSelf` GitHub
repository so it is publicly accessible and shareable.

**Todo List**
1. `git add note-to-self/index.html note-to-self-standalone-plan.md`
2. `git commit -m "Add standalone Note to Self app (single HTML file)"`
3. `git push origin main`

**Status** `[ ] pending`

---

## Coin Estimate

### Original build (Sub-Tasks 1–7) — completed

| Sub-Task | Complexity | Coins |
|---|---|---|
| 1 — Design System & Shell | Medium | ~15 |
| 2 — Chat Core | Low–Medium | ~10 |
| 3 — Auto-Replies Panel | Medium | ~12 |
| 4 — Scheduled Messages | Low | ~8 |
| 5 — Drawing Panel | Medium | ~12 |
| 6 — AI Onboarding Guide Agent | Medium–High | ~18 |
| 7 — Polish & Accessibility | Low | ~5 |
| **Subtotal** | | **~80 coins** |

### Remaining build (Sub-Tasks 8–11)

| Sub-Task | Complexity | Coins |
|---|---|---|
| 8 — Agentic RAG AI Guide | High | ~30 |
| 9 — Dark/Light Mode Toggle | Low | ~5 |
| 10 — Real Screenshots | Low | ~8 |
| 11 — GitHub Commit | Trivial | ~1 |
| **Subtotal** | | **~44 coins** |

### Grand total

| Scope | Coins |
|---|---|
| Sub-Tasks 1–7 (built) | ~80 |
| Sub-Tasks 8–11 (remaining) | ~44 |
| **Grand total** | **~124 coins** |
| **Your balance** | **126.73 coins** |
| **Left after completion** | **~2.73 coins** |

> Voice transcription (Whisper) and profile photo/localStorage persistence were cut from
> this build per user decision. These can be added as future sub-tasks for ~27 additional coins.
