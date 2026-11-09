# Plan: Fix Message Bubble Spacing & Mobile Toggle Overlap

## Overview

Two UI issues to fix in `note-to-self/index.html`:

1. **Desktop bubble gap** — On wide screens, short messages sit far apart because the chat container expands to 64–80rem but bubbles are only `max-width: 65–75%`. The visual gap in the center is too large.
2. **Mobile toggle overlap** — The dark/light mode toggle is `position: fixed; top: 0.6rem; right: 0.6rem`, which overlaps the settings gear icon in the header on mobile viewports.

---

## Sub-Task 1: Fix Desktop Message Bubble Gap

**Intent**
Reduce the max-width of the `.chat-messages-inner` container on desktop so that left and right bubbles don't spread across the full 64–80rem width. This makes short messages feel like a natural conversation rather than bubbles shouting across a room.

**Expected Outcomes**
- On desktop (≥1024px), message bubbles sit closer together — a short message on either side no longer leaves a huge empty strip in the center.
- Mobile layout is unchanged.

**Todo List**
- [ ] In the `@media (min-width: 1024px)` block (~line 557), change `.chat-messages-inner { max-width: 64rem; }` → `max-width: 48rem`.
- [ ] In the `@media (min-width: 1280px)` block (~line 571), change `.chat-messages-inner { max-width: 80rem; }` → `max-width: 48rem`.
- [ ] Leave `.chat-header`, `.chat-subheader`, `.chat-settings-bar`, and `.chat-form-inner` max-widths unchanged — those should still span wide.

**Relevant Context**
- File: `note-to-self/index.html`
- Lines 556–573: `@media (min-width: 1024px)` and `@media (min-width: 1280px)` blocks
- Line 373: Base `.chat-messages-inner { max-width: 42rem; margin: 0 auto; }` — this is the mobile/base value; desktop overrides it wider

**Status:** `[x] done`

---

## Sub-Task 2: Fix Mobile Dark/Light Mode Toggle Overlapping Settings

**Intent**
On mobile, `#theme-toggle-fixed` (position: fixed, top: 0.6rem, right: 0.6rem) sits directly over the settings gear button in the header. Rather than moving the toggle, add `padding-right` to `.chat-header` on mobile so the header's right-side content (gear + Guide toggle) steps inward, away from the fixed toggle. The toggle stays at `top: 0.6rem` on all screen sizes — consistent with desktop.

**Expected Outcomes**
- On mobile, the dark/light mode toggle no longer overlaps the settings gear icon.
- The toggle position (`top: 0.6rem; right: 0.6rem`) is unchanged on all screen sizes — consistent with desktop behavior.

**Todo List**
- [ ] In `.chat-header` base styles (~line 299), change `padding: 1rem 1rem` → `padding: 1rem 4.5rem 1rem 1rem`. This reserves ~4.5rem of right padding (toggle width 3.75rem + 0.6rem offset) so the gear icon is never obscured.

**Relevant Context**
- File: `note-to-self/index.html`
- Lines 111–117: `#theme-toggle-fixed { position: fixed; top: 0.6rem; right: 0.6rem; z-index: 9999; }` — toggle is 3.75rem wide
- Line 299: `.chat-header` base padding — currently `padding: 1rem 1rem`
- Line 301: `@media (min-width: 640px) { .chat-header { padding: 1rem 2rem; } }` — desktop already has enough horizontal room; only the base (mobile) padding needs updating

**Status:** `[x] done`
