# Minimal 20-Min Timer — Project Summary

## The Goal

This wasn't about building just another timer. Several timers already exist on the web — customisable, feature-rich, and functional. The goal here was different: **to push Claude to its limit in replicating a precise Figma design pixel by pixel**, and to prove that AI-assisted development can go beyond "50% there" when given the right constraints and attention.

The secondary goal was deeply personal — a **20-minute timer built for personal use**, since that's the duration most reached for in daily focus sessions.

---

## Why This Project

The core frustration with most vibe-coding and no-code platforms: **functions work, but design doesn't follow**. Given a detailed brief, most tools will produce something that works mechanically but drifts significantly from the intended visual. Tweaking that drift is painful and often leads to compromises.

This project chose a small, controlled scope — a single-purpose timer — precisely so that **every element, every animation, every color, and every interaction could be pinpointed and corrected**. Nothing was left vague. The design was not minimal by accident — every choice was deliberate and curated.

---

## How It Started

The project began with a detailed Figma design shared as annotated screenshots, with a full styling guide covering:

- **Every element named and numbered** — progress arc, ghost ring, minute hand, second hand, digital display, play/pause button, cancel button
- **Exact color values** — background `#1B1B1B`, arc white to `#E73131` (last minute), cancel button `#F9A9A9`, icon red `#E73131`
- **Typography** — DM Mono Medium 96px for minutes, DM Mono Light 96px at 20% opacity for seconds
- **Sizes** — 551×551 containing rectangle, 96.1 ratio, stroke widths, button sizes (84px), margins
- **Behavioural rules** — arc direction (anti-clockwise from top), minute hand starting at 20-min clock position (120° from 12), second hand completing full 360° per minute on top of all elements

---

## Build Process

### Phase 1 — Core Timer
Built the HTML/CSS/JS foundation: the SVG clock face, progress arc, both hands, digital display, and play/pause/cancel buttons. Font loaded via Google Fonts (DM Mono). Fully responsive using `min()` and `clamp()` for sizing, and container query units (`cqw`) for the digital display to scale with the clock.

### Phase 2 — The Progress Arc Struggle
The most technically involved part of the project. Getting the arc to:
- Start exactly at **12 o'clock (top)**
- Sweep **anti-clockwise**
- Shrink correctly as time counted down

...required careful reasoning about SVG coordinate systems, `stroke-dashoffset`, and transform order. Several approaches were tried:
- `rotate(-90)` + `scale(-1,1)` — started at wrong position
- Adding a `314.16` base offset — overcorrected
- Step-by-step manual tuning with the user controlling each transform incrementally

The final working solution: `rotate(90 220 220) scale(-1 -1) translate(-440 -440)` with `dashoffset` calculated cleanly from `CIRC * (1 - remaining/TOTAL)`.

### Phase 3 — UI Polish
- **Pause button styling** — while running: ghost background (`rgba(255,255,255,0.08)`) + white icon (reduced visual priority). While paused: solid white + black icon (high priority to resume). Transition uses a spring cubic-bezier for a "bop" feel.
- **Font swap under 1 minute** — minutes de-emphasised, seconds bolded at the `00:59` mark with a smooth 0.4s crossfade
- **Responsive layout** — clock, text, and buttons all scale fluidly across screen sizes with buttons always visible in viewport

### Phase 4 — Done Screen
When the timer completes:
- Clock face and digits fade out
- Done overlay fades in with a custom alarm-check SVG icon (uploaded directly and embedded inline) and "Timer Done" in Inter Medium
- Play button collapses to `width: 0` so the restart pill centers naturally
- Cancel button morphs into a full-width pill **RESTART** button with the custom restart icon and Inter SemiBold label
- Return to idle resets everything back cleanly

### Phase 5 — Icons
Initial attempts used Material Icons CDN and GitHub raw URLs — both failed (CDN missing glyphs, GitHub SVGs not fetchable as images). Solution: user uploaded both SVG files directly into the chat, and they were embedded as **inline SVG code** — zero external dependencies, works offline and on any host.

### Phase 6 — Sound
Web Audio API used to generate the done sound — no external files, no licensing issues. Pattern: `beep · beep · · · beep · beep` repeated twice, using sine wave oscillators at 880Hz (A5) with exponential gain fade for a clean, non-harsh tone.

### Phase 7 — Intro Animation
A fully staggered loading sequence, all running in a single shared `requestAnimationFrame` loop with time offsets:

| Element | Delay | Duration | Easing |
|---|---|---|---|
| Progress arc sweep | 0ms | 1600ms | ease-in-out |
| Second hand grows from center | 300ms | 400ms | ease-out |
| Second hand 360° spin | 700ms | 1000ms | ease-in-out |
| Minute hand grows (stroke-width 0→8) | 900ms | 1000ms | ease-out |
| Numbers slide up + fade | 1300ms | 600ms | ease-out |
| Controls fade in | 1700ms | 400ms | ease-out |
| Buttons unlock | 2100ms | — | — |

The minute hand dot artifact (caused by `stroke-linecap="round"` rendering a circle at zero length) was fixed by starting `stroke-width` at `0` and animating it alongside the length.

### Phase 8 — START Button State
The idle state was redesigned to show a single wide pill **▶ START** button with no restart button visible — cleaner and more intentional. On first click:
- Pill morphs into round ghost pause button
- Restart button fades in beside it

On cancel or restart: everything resets to the START pill state.

### Phase 9 — Background Tab Fix
The original `requestAnimationFrame` loop paused when the tab was hidden (browser throttling). Fixed by switching to an **absolute end time approach** using `setInterval`:
- On start: `endTime = Date.now() + remaining * 1000`
- Every 500ms: `remaining = Math.round((endTime - Date.now()) / 1000)`
- Timer stays accurate across tab switches, sleeps, and backgrounding

---

## Dev Tooling
A **SKIP button** (top-right, Micro 5 font, ghost styling) was added during development to jump to 1:03 for quickly testing the last-minute font swap, the red arc, the done screen, and the sound — without waiting 20 minutes each time. Removed before final release.

---

## Stack
- **Pure HTML/CSS/JS** — no frameworks, no build tools
- **SVG** for all clock graphics
- **Web Audio API** for sound
- **Google Fonts** — DM Mono, Inter, Micro 5
- **Inline SVGs** for all icons (no CDN dependency)
- **Hosted on Netlify** via drag-and-drop

---

## Key Takeaway

The design gap in AI-assisted development is real — but it's closeable. The combination of a precise Figma spec, annotated screenshots, element-by-element naming, and iterative correction produced a result that matched the original vision far more faithfully than typical vibe-coding workflows. The constraint of a small, controlled project made that level of precision achievable.
