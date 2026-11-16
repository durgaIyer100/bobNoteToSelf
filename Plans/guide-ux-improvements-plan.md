# Guide UX Improvements Plan

## Top-Level Overview

Four improvements to the AI guide in `note-to-self/index.html`:

1. **Split first message into 3 sequential bubbles** with a 3-second gap between each, **varied randomly each session**.
2. **Replace typed yes/skip with clickable buttons** that appear inline whenever the guide asks a yes/skip or yes/no question.
3. **Post-skip smart replies** — after the user skips onboarding and later asks about schedule, auto-replies, AI guide, or draw, the guide responds with a targeted, friendly walkthrough using the existing knowledge base and `getFeatureSteps()` content.

All changes are confined to `note-to-self/index.html`.

---

## Sub-Tasks

---

### Sub-Task 1 — Split the first guide message into 3 varied bubbles with 3-second gaps

**Status:** `[ ] pending`

**Intent:**
Right now `startGuide()` sends one large greeting as a single bubble. The goal is to split it into 3 natural messages sent 3 seconds apart, so the guide feels more conversational and less like a wall of text. Additionally, the opening line should be chosen randomly from a small pool each time the guide starts, so returning users (or users who reset) see a fresh greeting instead of the same line every time.

**Expected Outcomes:**
- Bubble 1 (immediate): randomly chosen from a pool of ~4 opening lines (all warm, welcoming, slightly different phrasing/emoji).
- Bubble 2 (3s later): randomly chosen from a pool of ~3 privacy/toggle context lines (same information, different wording).
- Bubble 3 (6s later): randomly chosen from a pool of ~3 feeling-check lines (e.g. "How are you feeling today? 🙂", "What's on your mind right now?", "How's your day going so far?") → sets `awaitingFeelingReply = true`.
- Each `setTimeout` ID is pushed to `guideTimers[]` so cancellation via `toggleGuide` still works.
- Variation is purely in-memory (`Math.random()`), no new storage needed.

**Todo List:**
1. In `startGuide()` (line 1454), define three small arrays — `openers[]`, `contextLines[]`, `feelingAsks[]` — each with 3–4 variant strings.
2. Pick one from each array using `Math.random()` at call time.
3. Replace the single `pushGuide(greeting, cb)` call with three sequenced `setTimeout` calls (0s, 3s, 6s), each pushing the selected variant.
4. The third timer's callback sets `awaitingFeelingReply = true` after pushing its message.
5. All three timer IDs must be pushed to `guideTimers`.

**Relevant Context:**
- `startGuide()` at line 1454
- `pushGuide(text, cb)` at line 1466
- `guideTimers[]` at line 929 — push all timer IDs here for cancellation safety
- `awaitingFeelingReply` at line 928

---

### Sub-Task 2 — Clickable yes/skip (and yes/no) quick-reply buttons

**Status:** `[ ] pending`

**Intent:**
Whenever the guide sends a message ending in `(yes / skip)` or `(yes / no)`, show clickable pill buttons beneath that bubble so the user does not have to type. Clicking a button should behave identically to typing that word and hitting Send.

**Expected Outcomes:**
- After any guide bubble whose text ends with `(yes / skip)` or `(yes / no)`, two styled pill buttons appear directly in the chat (not a floating bar).
- Clicking a button posts the user's typed message (shows in chat as a normal user bubble), then processes the choice through the existing `advanceOnboarding()` / `sendMessage()` logic.
- Buttons disappear as soon as one is clicked (they are part of that specific bubble's render, not a persistent UI element).
- The input field still works as before — typing is not disabled.

**Design note:**
The buttons are rendered as part of the `renderMessages()` output for that specific guide message. When a new message is added to the chat (by user or guide), the button row is naturally replaced/removed because the whole list re-renders.

**Todo List:**
1. In the CSS section, add a `.guide-choices` container style and `.guide-choice-btn` pill button style (small, rounded, accent-colored, inline-flex).
2. In `renderMessages()`, after rendering a guide message bubble, detect if `m.text` ends with `(yes / skip)` or `(yes / no)` and if it's the last guide message. If so, append a `.guide-choices` div with the two buttons inside the bubble wrapper.
3. Add a `handleGuideChoice(value)` JS function that:
   a. Removes all `.guide-choices` elements from the DOM.
   b. Sets `inp.value = value` and calls `sendMessage()`.
4. Wire each button's `onclick` to `handleGuideChoice('yes')` / `handleGuideChoice('skip')` or `handleGuideChoice('no')`.

**Relevant Context:**
- `renderMessages()` — the main render loop (search for `pushGuide` renders)
- Guide messages are tagged with `m.guide === true`
- `sendMessage()` at line 1199 — this is what clicking effectively calls
- Existing CSS uses CSS custom properties (e.g. `--accent`, `--surface`) for theming

---

### Sub-Task 3 — Post-skip smart replies for feature how-to questions

**Status:** `[ ] pending`

**Intent:**
When the user skips onboarding and later asks "how do I use the schedule?", "how does auto-reply work?", "how do I draw?", or "what is the AI guide?", the guide should give them a targeted, helpful walkthrough answer — not just the generic RAG response.

The existing `askAgent()` already detects these categories via `retrieve()` and opens the relevant panel. This sub-task adds richer, specific "how-to" answers for these four categories when the user has previously skipped onboarding (`guideDone === true`).

**Expected Outcomes:**
- After skipping, if the user types any message matching schedule / auto-reply / draw / AI guide intent, the guide responds with a clear, friendly how-to reply specific to that feature.
- The existing panel auto-open side-effect (`action`) from `askAgent()` is preserved (schedule picker opens, rules panel opens, draw dialog opens).
- No new state variables needed — this uses `guideDone` and existing `retrieve()` / `entry.category` logic.

**Implementation approach:**
- In `askAgent()`, after the RAG retrieval step, detect if `guideDone === true` AND `entry.category` is one of `schedule`, `auto-reply`, `draw`, or `app-help` (AI guide falls under `app-help`).
- If so, substitute a richer canned how-to reply (pulled from the existing `getFeatureSteps()` text, condensed into one clear response) instead of the generic template chunk.
- The `action` (open panel) is still applied as before.

**How-to reply content (one per feature):**
- **schedule**: Summarise the 3 steps — tap ⏰ Schedule, pick date/time, type message and send.
- **auto-reply**: Summarise — tap 💬 Auto-Replies, add a keyword + message, save and test.
- **draw**: Summarise — tap ✏️ Draw, draw with finger/mouse, tap Send.
- **app-help / AI guide**: Explain the guide is on by default, can be toggled top-right, walks through features, and answers questions in chat.

**Todo List:**
1. In `askAgent()` (line 2149), after `const entry = retrieve(input)`, add an override block: if `guideDone && ['schedule','auto-reply','draw','app-help'].includes(entry.category)`, build and return a feature-specific reply string instead of the generated template text.
2. Write the four canned reply strings (concise, friendly, 2–3 sentences each) directly in that block.
3. Ensure the existing `action` (panel open) is still set for schedule / auto-reply / draw — this requires the action-assignment block at lines 2191–2199 to still run. Restructure so only `text` is overridden, not `action`.

**Relevant Context:**
- `askAgent()` at line 2149
- `retrieve()` at line 2003 — returns a KB entry with `.category`
- `guideDone` variable at line 926
- `getFeatureSteps()` at line 1473 — existing step text to draw from for consistency
- KB category strings: `'schedule'`, `'auto-reply'`, `'draw'`, `'app-help'`

---

## Notes for Implementation

- All three sub-tasks are in the single file `note-to-self/index.html`.
- Sub-tasks should be done in order: 1 → 2 → 3.
- Sub-task 2 is the largest change (CSS + render logic + new JS function).
- Sub-task 3 is the smallest change (a few lines inside `askAgent()`).
- Sub-task 1 (including variation) requires only editing `startGuide()` — no new state, no storage.
- No new files, libraries, or dependencies are needed.

---

## Post-Implementation Steps

After all three sub-tasks are complete and verified:

### Step A — Git commit and push
- Stage: `note-to-self/index.html` and `Plans/guide-ux-improvements-plan.md`
- Commit message: `feat: split guide opening into 3 varied bubbles, add clickable yes/skip buttons, post-skip smart how-to replies`
- Push to `origin main`

### Step B — Update session report (`bob-task-session-report.html`)
The report is at version 10 (`v10`). Update it to `v11` with:

1. **Summary grid** — increment "Sub-tasks completed" from 25 → 28, "Lines in index.html" to actual new line count, "GitHub commits pushed" from 12 → 13.
2. **Commits badge** — update from `12 commits · main` → `13 commits · main`.
3. **Activity log** — add a new `<div class="activity-day">` entry for today (use today's date) with these bullet points:
   - Split guide opening into 3 sequential bubbles with 3-second gaps — varied randomly each session using `openers[]`, `contextLines[]`, `feelingAsks[]` pools
   - Added clickable yes/skip and yes/no pill buttons — render inline after any guide prompt ending in `(yes / skip)` or `(yes / no)`, clicking posts the choice as a user message
   - Added post-skip smart how-to replies — after skipping onboarding, asking about schedule/auto-replies/draw/AI guide returns a targeted feature summary and auto-opens the relevant panel
4. **Sub-task table** — add three new rows:
   - `26` / `Guide opening — 3 varied bubbles` / `startGuide() split into openers/context/feeling pools, 3s gaps, guideTimers` / Done
   - `27` / `Clickable yes/skip & yes/no buttons` / `guide-choice-btn pills, renderMessages injection, handleGuideChoice()` / Done
   - `28` / `Post-skip smart how-to replies` / `askAgent() override for guideDone + feature category match` / Done
5. **Commit log** — prepend a new `<li>` with the new commit hash (placeholder `HASH` — update after actual push) and today's date.
6. **Feature inventory** — add two new `.feature-item` entries:
   - `Guide opening — 3 varied randomised bubbles with 3-second gaps`
   - `Clickable yes/skip pill buttons — inline after every guide prompt`
   - `Post-skip smart how-to replies — targeted feature walkthroughs after onboarding skip`
7. **Remaining Work** — remove the first item ("Message bubble spacing") if it was resolved; keep or update the rest.
8. **Report label** — change `v10` → `v11` in the `.label` div.
9. **Project Assets table** — update `note-to-self/index.html` notes with new line count; add a row for `Plans/guide-ux-improvements-plan.md`.

### Step C — Open files in browser
- Open `note-to-self/index.html` in the default browser.
- Open `bob-task-session-report.html` in the default browser.
