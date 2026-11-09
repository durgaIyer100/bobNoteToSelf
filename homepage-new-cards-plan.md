# Plan: Add Draw, Schedule, and AI Guide Cards to Homepage

## Overview

Add three new "How it works" cards to the homepage alongside the existing Note to Self and Auto-Replies cards. Each card reuses the existing `.how-card` / `.how-card-img` / `.how-card ol` components. The `how-grid` already uses `grid` with `1fr 1fr` at ≥768px, so four cards will naturally flow into a 2×2 layout — no grid CSS change needed.

Screenshots for the three new cards will be embedded as `<img>` tags with base64 data URIs. Since the screenshots are not yet on disk, the img src will be set to the existing asset files as a placeholder, to be swapped once real captures are saved.

---

## Sub-Task 1 — Add Draw card

**Intent:** Insert a new `.how-card` article for the Draw feature after the Auto-Replies card, reusing all existing card markup patterns.

**Expected Outcomes:**
- A third card titled "Draw" appears in the `how-grid`
- It contains a screenshot placeholder (`how-card-img`) and a 3-step `<ol>` explaining the draw feature
- Visual appearance is identical to the existing two cards

**Todo List:**
1. Locate the closing `</article>` of the Auto-Replies card in `note-to-self/index.html`
2. Insert a new `<article class="how-card">` block with title "Draw", a `<div class="how-card-img">` placeholder, and an `<ol>` with the draw steps
3. Steps should describe: opening Draw from toolbar, drawing on the canvas, optionally picking a category for an auto-reply, and sending or scheduling

**Relevant Context:**
- `note-to-self/index.html` lines 615–624 — Auto-Replies card to insert after
- `.how-card`, `.how-card-img`, `.how-card ol` CSS at lines 244–263

**Status:** [ ] pending

---

## Sub-Task 2 — Add Schedule card

**Intent:** Insert a new `.how-card` article for the Schedule feature after the Draw card.

**Expected Outcomes:**
- A fourth card titled "Schedule" appears in the `how-grid`
- Screenshot placeholder and 3-step `<ol>` explaining the schedule feature

**Todo List:**
1. Insert after the Draw card
2. Steps should describe: tapping the Schedule icon in the toolbar, picking a date and time with the calendar picker, typing a message and sending, and the message arriving at the scheduled time

**Relevant Context:**
- Same card markup pattern as Sub-Task 1

**Status:** [ ] pending

---

## Sub-Task 3 — Add AI Guide card

**Intent:** Insert a new `.how-card` article for the AI Guide feature after the Schedule card.

**Expected Outcomes:**
- A fifth card titled "AI Guide" appears in the `how-grid`
- Screenshot placeholder and 3-step `<ol>` explaining the guide feature

**Todo List:**
1. Insert after the Schedule card
2. Steps should describe: the guide appearing on first visit, how it walks through features step by step, how to toggle it off for a fully private unguided experience

**Relevant Context:**
- Same card markup pattern as Sub-Tasks 1 and 2

**Status:** [ ] pending

---

## Sub-Task 4 — Update how-grid layout for 5 cards

**Intent:** With 5 cards, a strict `1fr 1fr` grid leaves the last card spanning the full width. Update the media query so the grid uses `repeat(2, 1fr)` at 768px (unchanged) but adds a wider breakpoint at ≥1024px with `repeat(3, 1fr)` so the 5 cards sit 3+2 rather than the last one stretching full width.

**Expected Outcomes:**
- On mobile: single column (unchanged)
- On tablet (≥768px): 2 columns, cards wrap 2+2+1
- On desktop (≥1024px): 3 columns, cards sit 3+2

**Todo List:**
1. Add a `@media (min-width: 1024px)` rule setting `.how-grid { grid-template-columns: repeat(3, 1fr); }`
2. The last card in the 3-column row can be centered with `justify-items: center` or left as-is depending on preference

**Relevant Context:**
- `.how-grid` CSS at line 239–243
- Existing 1024px breakpoint block at line 531

**Status:** [ ] pending
