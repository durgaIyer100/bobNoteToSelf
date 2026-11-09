# Plan: Flatpickr + Responsive/Mobile-First UI Overhaul

## Overview

Two parallel improvements to `note-to-self/index.html`:

1. **Flatpickr date picker** — replace the broken custom wheel picker with Flatpickr (CDN,
   ~12 kb), giving a Google-Material-style calendar + time picker that works consistently
   on all devices (scroll wheel on iOS, calendar grid on desktop).

2. **Responsive layout overhaul** — the app currently has only two breakpoints (640px, 768px)
   and narrow max-widths (42–56rem) that create excessive side whitespace on desktop. Apply
   mobile-first design: base styles target phones, then progressively enhance for tablet
   (768px), desktop (1024px), and wide desktop (1280px).

No structural HTML changes beyond the schedule input. All fixes are CSS-only except for the
Flatpickr initialisation (3 lines of JS).

---

## Problem Summary

| Screen | Issue |
|--------|-------|
| Home — all sizes | `max-width: 56rem` + `margin: 0 auto` leaves huge dead space on wide screens |
| Home — cards | Single column on mobile is correct; two-column grid kicks in at 768px but cards are under-padded |
| Chat — messages | `max-width: 42rem` on `.chat-messages-inner` makes the chat feel cramped on desktop |
| Chat — header/form | `max-width: 48rem` / `42rem` — correct on mobile but too narrow on 1280px+ |
| Chat — bubbles | `max-width: 75%` is fine on mobile; on wide desktop bubbles should allow up to 60% of a wider container |
| Chat — toolbar | `justify-content: center` with `gap: 0.5rem` — buttons bunch together; needs spacing on desktop |
| Dialogs | `max-width: 28rem` on a 1440px screen looks tiny; should grow to ~36rem on desktop |
| Both screens | No `1024px` or `1280px` breakpoints — desktop is treated same as 640px |

---

## Sub-Tasks

---

### Sub-Task 1 — Remove the custom wheel picker

**Intent**
Strip all custom wheel code so there are no conflicts when Flatpickr is added.

**Expected Outcomes**
- The `/* ─── WHEEL PICKER DIALOG ───*/` CSS block deleted (~65 lines).
- The `#wheel-picker` dialog HTML block deleted (~40 lines).
- `<button id="schedule-display">` + `<input type="hidden" id="schedule-input">` replaced
  with `<input type="text" id="schedule-input" class="schedule-input" placeholder="Pick date & time" readonly />`.
- JS functions `wpFill`, `wpSelected`, `openWheelPicker`, `closeWheelPicker`,
  `confirmWheelPicker`, and the `WP_MONTHS` constant deleted (~90 lines).
- `cancelSchedule()` no longer references `schedule-display`.

**Todo List**
1. Delete the `/* ─── WHEEL PICKER DIALOG ───*/` CSS block (lines ~492–555).
2. Delete the `#wheel-picker` dialog HTML block (lines ~746–785).
3. Replace `<button id="schedule-display">` + hidden input with plain text input.
4. Delete the `/* ── Wheel date/time picker ───*/` JS section (lines ~1286–1369).
5. Clean `cancelSchedule()` — remove `schedule-display` reference.

**Relevant Context**
- CSS: lines ~492–555
- HTML: lines ~704–711 (schedule row), ~746–785 (wheel dialog)
- JS: lines ~1278–1369

**Status** — [ ] pending

---

### Sub-Task 2 — Add Flatpickr CDN, theme overrides, and initialise

**Intent**
Load Flatpickr from CDN and wire it to `#schedule-input`. Override Flatpickr's default
styles to match the app's oklch color tokens in both light and dark mode.

**Expected Outcomes**
- `<link>` for Flatpickr CSS added in `<head>`.
- `<script>` for Flatpickr JS added before `</body>`.
- CSS override block inside `<style>` maps Flatpickr classes to app CSS variables.
- `const fp = flatpickr('#schedule-input', { enableTime, dateFormat, minDate, minuteIncrement, time_24hr, disableMobile: false })` initialised in `DOMContentLoaded`.
- `cancelSchedule()` calls `fp.clear()`.
- `sendMessage()` unchanged — already reads `.value` and parses with `new Date()`.
- Tapping the input opens the calendar. Picking a future date+time stores it as
  `YYYY-MM-DDTHH:MM` in the input value.

**Todo List**
1. Add `<link rel="stylesheet" href="https://cdn.jsdelivr.net/npm/flatpickr/dist/flatpickr.min.css" />` in `<head>`.
2. Add `<script src="https://cdn.jsdelivr.net/npm/flatpickr"></script>` before `</body>`.
3. Add Flatpickr theme override CSS block inside `<style>` targeting `.flatpickr-calendar`,
   `.flatpickr-day`, `.flatpickr-day.selected`, `.flatpickr-day.today`, `.flatpickr-time`,
   `.flatpickr-months`, `.numInputWrapper`, nav arrows — all using CSS variables.
4. In `DOMContentLoaded`, declare `let fp;` at module scope, then initialise:
   `fp = flatpickr('#schedule-input', { enableTime: true, dateFormat: "Y-m-d\\TH:i", minDate: "today", minuteIncrement: 5, time_24hr: false, disableMobile: false })`.
5. Update `cancelSchedule()` to call `fp.clear()` instead of `el.value = ''`.

**Relevant Context**
- `<head>` ends at line ~11
- `</body>` is the last line
- `:root` and `.dark` CSS variable blocks at lines ~18–80
- `DOMContentLoaded` listener starts at line ~1030
- `cancelSchedule()` at line ~1278

**Status** — [ ] pending

---

### Sub-Task 3 — Mobile-first responsive layout

**Intent**
Apply a proper mobile-first breakpoint system. Base styles are already mobile-friendly;
this sub-task adds `1024px` and `1280px` breakpoints and widens max-widths so desktop users
see a full, well-proportioned layout instead of a narrow column floating in whitespace.

**Expected Outcomes**

Home screen:
- On mobile (< 768px): single column, `padding: 2rem 1rem`, cards full-width — unchanged.
- On tablet (768px+): two-column card grid — unchanged.
- On desktop (1024px+): `max-width` of `.home-inner` expands to `72rem`; home padding
  increases to `4rem 2rem`; card grid gap increases to `2rem`.
- On wide desktop (1280px+): `.home-inner` expands to `88rem`; `how-grid` becomes
  3-column if 3+ cards exist, otherwise stays 2-column with wider cards.

Chat screen:
- On mobile: unchanged (full-width, no sidebar).
- On desktop (1024px+): `.chat-header`, `.chat-messages-inner`, `.chat-form-inner`
  max-widths expand from `42–48rem` to `64rem`.
- On wide desktop (1280px+): these expand to `80rem`.
- `.msg-bubble` max-width stays `75%` on mobile; on 1024px+ increases to `65%` of the
  (now wider) container — bubbles feel proportionate, not wall-to-wall.
- `.toolbar` buttons: on 1024px+ use `justify-content: flex-start` with more spacing
  so they don't bunch in the centre of a wide form area.
- `.chat-form` horizontal padding increases on 1024px+.

Dialogs:
- `.dialog-box` `max-width` increases from `28rem` to `36rem` on 1024px+.

**Todo List**
1. Add `@media (min-width: 1024px)` block with:
   - `#screen-home { padding: 4rem 2rem; }`
   - `.home-inner { max-width: 72rem; }`
   - `.how-grid { gap: 2rem; }`
   - `.chat-header, .chat-subheader, .chat-settings-bar { max-width: 64rem; }`
   - `.chat-messages-inner { max-width: 64rem; }`
   - `.chat-form-inner { max-width: 64rem; }`
   - `.chat-form { padding: 0.75rem 2.5rem 1.5rem; }`
   - `.dialog-box { max-width: 36rem; }`
   - `.toolbar { justify-content: flex-start; gap: 1rem; }`
2. Add `@media (min-width: 1280px)` block with:
   - `#screen-home { padding: 5rem 3rem; }`
   - `.home-inner { max-width: 88rem; }`
   - `.chat-header, .chat-subheader, .chat-settings-bar { max-width: 80rem; }`
   - `.chat-messages-inner { max-width: 80rem; }`
   - `.chat-form-inner { max-width: 80rem; }`
   - `.chat-form { padding: 0.75rem 3rem 1.5rem; }`
3. Add `@media (min-width: 1024px)` rule: `.msg-bubble { max-width: 65%; }`.

**Relevant Context**
- `.home-inner` at line ~233
- `.chat-header` at line ~279
- `.chat-messages-inner` max-width at line ~352
- `.chat-form-inner` max-width at line ~396
- `.toolbar` at line ~416
- `.dialog-box` at line ~445
- Existing breakpoints: 640px (home padding, card padding, chat padding), 768px (card grid)

**Status** — [ ] pending

---

### Sub-Task 4 — Commit and verify

**Intent**
Push all changes and confirm both Flatpickr and the responsive layout work correctly.

**Expected Outcomes**
- All changes committed and pushed to `main`.
- On mobile (375px): layout unchanged from before.
- On tablet (768px): two-column card grid, wider chat.
- On desktop (1280px): home fills width properly, chat conversation area is wide, no
  excessive whitespace on sides, toolbar buttons spread naturally.
- Schedule picker opens Flatpickr calendar, picks a future date, sends correctly.
- Light and dark mode both look correct in the Flatpickr calendar.

**Todo List**
1. `git add note-to-self/index.html`
2. `git commit -m "Add Flatpickr date picker, mobile-first responsive layout for desktop"`
3. `git push origin main`

**Status** — [ ] pending
