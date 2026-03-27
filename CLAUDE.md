# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project Overview

This is a collection of self-contained browser games. Each game is a **single HTML file** with all CSS and JavaScript inlined — no build system, no dependencies, no npm. Open any file directly in a browser to run it.

## Files

- `index.html` — GitHub Pages landing page linking to both games
- `aura.html` — Main game: pixel art side-scroller in a Tamagotchi device shell
- `tictactoe.html` — Two-player Tic Tac Toe

## Running & Testing

Open directly in a browser:
```bash
open aura.html
open tictactoe.html
```

No build step, no server required.

## Deployment

The site is live at **https://cassmcxi.github.io/aura-game/** via GitHub Pages (serving from `main` branch root). Every push to `main` deploys automatically.

After any change:
```bash
git add <file>
git commit -m "description of change"
git push
```

## AURA Architecture (`aura.html`)

The entire game lives in one `<script>` block, organized into labeled sections:

| Section | Purpose |
|---|---|
| A: Constants | Canvas size (160×144), physics values, level palettes |
| B: Audio | Web Audio API chiptune engine — `initAudio()`, `tone()`, `scheduleMusicChunk()` (look-ahead scheduler), SFX functions |
| C: Pixel Fonts | Bitmask glyph arrays + `drawText()` / `drawTitle()` renderers |
| D: Sprites | 2D color arrays for every animation frame. `drawSprite()` and `drawSpriteFlip()` render them as 1×1 `fillRect` calls |
| E: Background | `drawBackground()` — sky, procedural castle towers, animated bats, ground tiles |
| F: Player | Physics state object, `updatePlaying()` (gravity/input/collision), `setAnim()` / `tickPlayerAnim()` |
| G: Level System | `startTransition()` / `updateTransition()` / `renderTransition()` — slide animation between levels |
| H: Game State | State machine: `TITLE → PLAYING → TRANSITION`. `renderTitle()`, `renderPlaying()`, `drawHUD()` |
| I: Input | `keydown`/`keyup` listeners populating a `keys` map |
| J: Game Loop | Fixed-step `requestAnimationFrame` loop (16.67ms steps, dt capped at 100ms) |

**Canvas:** 160×144 native, scaled 3× to 480×432 via CSS `image-rendering: pixelated`. All coordinates use native space.

**Sprites:** Defined as 2D arrays of hex color strings (or `null` for transparent). The constant `T = null` is used throughout sprite definitions as shorthand for transparent pixels.

**Audio:** `initAudio()` must be called on first user keypress (browser autoplay policy). Music uses a look-ahead scheduler — `scheduleMusicChunk()` is called every frame but self-throttles by checking `audioCtx.currentTime`.

**Levels:** 4 palettes cycling across 8 levels. Walking off the right edge → `startTransition('right')`, left edge → `startTransition('left')`. The transition slides the old and new scenes simultaneously.
