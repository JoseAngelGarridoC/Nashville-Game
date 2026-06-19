# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## What this is

A browser-based trainer for the **Nashville Number System** — translating between chord names and scale degrees. Spanish-language UI. Each page is a single self-contained HTML file (inline CSS + vanilla JS); there is **no build step, no package manager, no tests, and no dependencies installed locally**. Tailwind and the fonts load from CDNs at runtime.

## Running / developing

- Open `index.html` directly in a browser, or serve the folder statically (`python3 -m http.server`) and visit it. Live reload requires a server with that feature; otherwise just refresh.
- Deployment is GitHub Pages from `main` (push to `main` publishes). There is nothing to build.

## Files

- `index.html` — the game. All HTML, CSS (`<style>`) and JS (`<script>`) live in this one file.
- `blog.html` — a standalone editorial article about chord-to-number substitution. It must stay independent: it shares the *visual language* of the game but **none of its game logic**. The two pages cross-link (Blog link in the game header, Inicio/Home link in the blog header).
- `cambios.md` — informal Spanish changelog / feature notes.

## Game architecture (index.html)

The game is a small state machine driven by a single `state` object and a `keydown` listener.

- **Stages** (`state.stage`): `'inicial'` shows the chord (e.g. `Am`) and expects the Nashville degree (e.g. `6`); `'intermedia'` shows the degree and expects the chord's base letter. Each stage runs **levels 1→3**, and each level is **10 matches** (`matchCount`) before `advanceLevel()`.
- **Level mechanics** layered on top of the same gameplay (`setupLevelMechanics`): level 1 plain, level 2 adds a 60s timer (beating 0:30 triggers the "Good job, worshipper!" float bonus), level 3 adds a 4/4 metronome where input is only valid on beat 1 — off-beat input is a tempo error.
- **Scoring**: +100 XP per completed progression, −25 XP per wrong or off-tempo keypress.
- **Data**: `keyData` holds the 8 supported keys with their six diatonic degrees (chord / nashville number / base letter). `progressionPatterns` is a list of 4-chord patterns expressed as degree indices `0–5`; `generateProgression()` picks a key + pattern (avoiding the last few via `recentHistory`).
- **Input** comes from both physical keys (`handleInput`) and on-screen mobile buttons (`renderMobileKeys`), routed through the same `handleInput`.

## Conventions to preserve

- **Theme palette is duplicated** in both `index.html` and `blog.html` as CSS custom properties: light is the default in `:root`, warm dark is `body.theme-dark`. If you change colors/fonts, update **both files** to keep them consistent (Anthropic ivory/clay aesthetic, Fraunces serif for display, Inter for body, JetBrains Mono for chords/numbers).
- The theme toggle swaps `theme-light`/`theme-dark` on `<body>` and swaps the sun/moon SVG path inline — keep both pages' toggle logic in sync.
- When changing gameplay, treat `blog.html` as off-limits, and vice versa — a stated project constraint is that the blog must not modify or depend on the game.
- DOM is wired by hard-coded element IDs (`chord-N`, `ans-N`, `beat-N`, `progression-container`, etc.); renaming an ID means updating every reference in the same file.
