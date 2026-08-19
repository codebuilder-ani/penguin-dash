# CLAUDE.md — Penguin Dash

Context for any future Claude session working on this project.

**Author:** Anirudh V · **Started:** August 2026

---

## What this is

A 2D infinite-runner browser game. An Emperor penguin runs across an
Antarctic ice field, jumping icebergs, ducking sawblades and collecting
snowflakes. Built for a 9-year-old — **Anirudh is the player and the
owner of this project**, and he gives feedback directly. His dad handles
hosting and accounts.

Playable on desktop (keyboard) and phone/tablet (taps), portrait or
landscape.

## The single most important constraint

**Everything lives in one file: `penguin-dash.html`.**

No build step, no bundler, no `npm install`, no external assets. Open the
file in a browser and it runs. This is deliberate — it makes the game
trivial to host, easy to email, and possible for a kid to poke at.

Do not split it into modules, add a framework, or introduce a
dependency without asking first.

Exception: the Google Fonts `<link>` in the head. Fallback font stacks
are in place, so the game still looks right offline.

## Hard rules

- **No `localStorage` / `sessionStorage`.** They fail in the Claude
  artifact viewer, which is where the game gets tested. The high score
  lives in a plain variable and resets on reload. If persistence is ever
  wanted, it has to be a deliberate change made after leaving the
  artifact environment.
- **No blood or gore.** The crash is a cartoon tumble with X eyes,
  snow puffs and stars. Keep it that way.
- **Ducking must be the only way past a sawblade.** See "Tuning" below.
- **The penguin never speeds up.** Difficulty comes from obstacle
  density only. This was an explicit design decision.

## File layout

```
penguin-dash.html
├── header comment   — author, date, how the game works
├── <style>          — menus, buttons, control cluster
├── <body>           — <canvas> + HTML overlay screens
└── <script>         — the entire game, in labelled sections:
     SETUP · MUSIC · INPUT · MENU WIRING · WORLD OBJECTS ·
     UPDATE · DRAWING (background / penguin / objects) ·
     HUD · MAIN LOOP
```

## Core architecture

- The canvas is a fixed **960×440 logical space**. The browser scales it
  to any screen. All game maths uses those coordinates, so behaviour is
  identical everywhere.
- **The penguin never moves horizontally.** He sits at x=150; the world
  scrolls left past him. That's why one `SPEED` value drives obstacles,
  snowflakes and scenery.
- `state` is the master switch: `menu | howto | playing | paused |
  dying | over`. Almost everything checks it first.
- The main loop is `frame()` → `update(dt)` then draw. `dt` is seconds
  since the last frame, clamped to 1/30 so a stalled tab can't teleport
  the penguin through an obstacle.
- Menus are **real HTML overlays**, not canvas-drawn. Keeps them
  accessible and tappable.

## Tuning numbers (change these carefully)

| Constant | Value | Why |
|---|---|---|
| `SPEED` | 385 px/s | Never changes during a run, by design |
| `GRAV` / `JUMP_V` | 2150 / −735 | Gives a ~125px, ~0.68s jump |
| min spawn gap | 0.92s | **Floor.** A jump takes 0.68s — anything tighter is unfair |
| switch gap | +0.38s | Extra room when a jump obstacle is followed by a duck one, or vice versa |
| sawblade y | `GROUND_Y − 78` | Low enough that a standing penguin collides |
| chain hitbox | ±10px, y=0 to blade | Makes jumping over a sawblade impossible |

If you move the sawblade or change the jump, **re-verify the duck rule**:
simulate the hitbox at every jump height from 0 to 200px and confirm no
height clears the blade.

## Audio

Two music tracks, both **generated in the browser** with oscillators —
no audio files. Each track is four rows of numbers (`bass`, `lead`,
`chime`, `kick`) read as a grid of eighth notes; values are semitones
from C, `null` is a rest. Editing the tune means editing those arrays.

The scheduler books notes ~0.25s ahead of time so the music doesn't
stutter when the game is busy drawing. Don't replace it with a naive
timer.

Signal path: sfx and music → their own gain nodes → `masterGain` →
speakers. **Mute silences everything** via `masterGain`, which is what
Anirudh asked for. Music "Off" in the chooser leaves sound effects on.

Audio can't be created until the user interacts — browsers block it. So
`initAudio()` is called lazily on the first click, tap or key press.

Adding real mp3 tracks later is a known future option: the chooser is
already structured as a list, so a URL or local filename can slot in
alongside the generated ones.

## Controls

| Input | Action |
|---|---|
| `W` / `↑` | Jump |
| `S` / `↓` | Duck (hold) |
| `P` | Pause |
| `M` | Mute |
| `Enter` | Start / restart |
| `Esc` | Back to menu |
| Tap top of screen | Jump |
| Tap + hold bottom | Duck |

Touch zones span the **whole viewport**, split at 56% of the canvas
height — not just the canvas — so portrait play works when the game is a
band in the middle of the screen. Buttons swallow their own taps so
using the controls doesn't make the penguin jump.

## Hosting

GitHub Pages. `index.html` in the repo root, Pages source set to
`main` / `/ (root)`. Rename `penguin-dash.html` → `index.html` on
upload. Public repo required for the free tier. Netlify Drop is the
fallback if GitHub feels like too much setup.

Note: GitHub requires account holders to be 13+, so the account is his
dad's.

## Testing

There's no test suite. Verify by:

1. Syntax check — extract the script block and `new Function(js)` it.
   Careful: the header comment contains the literal text `<script>`, so
   use `lastIndexOf`, not `indexOf`.
2. Simulate the hitbox maths in Node when touching collisions or
   obstacle positions.
3. Check the music arrays are all the same length within a track.
4. Grep for `localStorage` — should return nothing.

## Working with Anirudh

He's 9 and tests his own game, so his feedback is concrete and usually
right ("you can jump over the sawblade" was his catch). Answer him
directly and in plain language — he's the one making the decisions about
his game. Explain the reasoning behind changes; he's interested in how
it works, not just that it works.

He has asked for discussion before coding on bigger changes. Respect
that.

## Ideas raised but not built

- Saved high score between visits (needs a non-artifact environment)
- Real mp3 tracks alongside the generated ones
- A proper zoomed-in portrait layout, if the current band feels cramped
- More power-ups (the spec left this open for future versions)
