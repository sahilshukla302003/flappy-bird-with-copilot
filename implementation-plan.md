# Implementation Plan

## Requirement

Approved requirement: ENH-001 — Implement core Flappy Bird gameplay (game loop, physics, pipes, scoring, collisions, restart)

Note: No Jira story key was provided in the request. This plan uses the approved enhancement ID (ENH-001) as the requirement reference.

## Objective

Convert the current Vite + React template landing page into a playable Flappy Bird single-page game with:

- a clear game state machine (Start → Playing → Game Over)
- a requestAnimationFrame-driven game loop with delta time
- bird physics (gravity, flap impulse)
- moving pipe pairs with random gaps
- collision detection (bird vs pipes/ground/ceiling)
- scoring when passing pipes

- restart flow (game over screen + replay/reset)

## Current Implementation

- The app is a default Vite + React template-like single page.
- `Src/App.js` renders:
  - hero images (src/assets/hero.png, src/assets/react.svg, src/assets/vite.svg)
  - a counter button using `useState`
  - static sections for Documentation/Social links
- No game Loop, no canvas, and no gameplay modules or components exist yet.
- Styling is contained in `src/App.css` and `src/index.css`.

Relevant code paths:
- `src/App.jsx`
- `src/App.css`
- `src/main.jsx`
- `vite.config.js`
- `package.json`

## Proposed Changes

### Frontend

- Replace the current landing-template UI in `src/App.jsx` with a game shell component that renders:
  - a main game viewport (canvas element or a DOM layer using absolute-positioned divs)
  - HUD for score and state
  - Action prompts: "Click/Tap to Start", [Space] to flap, "Restart" on Game Over

- Introduce a small game architecture under `src/game/` to keep logic testable and components thin:
  - State machine: `src/game/gameState.js` with states `inactive | ready | playing | gameOver` (exact names TBD)
  - Pure update loop: `{state, d} = update(state, dt)` in `src/game/update.js`
  - Physics constants and utilities: `src/game/physics.js`
  - Collision: axis-aligned bounding box (aABB) helpers in `src/game/collision.js`
  - Pipe generation: `src/game/pipes.js` (spawn, move, remove offscreen, score gate)
  - Rendering: a
    - Canvas renderer in `src/game/renderConvas.js`, or 
    - DOM-based rendering ina new `src/components/GameViewport.jsx`
  
  Important: The codebase currently has dependencies only for React/Vite; the plan avoids adding a game engine library.

- Input handling:
  - Add listeners for keydown (Space/ArrowUp) and pointerdown (tap/click) to trigger flap in Playing state, and to transition from Start/GameOver as appropriate.
  - Centralize input gating by game state to avoid input bugs (e.g., flap during game over).

- Game loop integration in React:
  - In a container component (e.g. `Game.jsx`) use `requestAnimationFrame` with `ref` storage for last frame time.
  - Update game state on each tick with clamped dt (e.g. max 50-66ms) to avoid large jumps on tab switch.
  - Avoid storing every frame in React state if using canvas; prefer `ref` for mutable state and only sync high-level UI (score, state) to React state at a lower frequency. (If DOM rendering, state updates every frame may be acceptable for this small game, but perf should be monitored.)

- Styling:
  - Replace or add new CSS in `src/App.css` for a simple game layout (centered viewport, minh-height, background, hud overlay).
  - Remove or archive the template sections (Docs/Social) from the main view as they're out of scope for core gameplay.

### Backend

Not in scope. This repo is a pure frontend Vite + React app with no server code or API.

### Database

Not in scope. No database exists in the current codebase.

### API

Not in scope. No API exists in the current codebase.

## Files to Modify

- `src/App.jsx` (replace template UI with game shell)
- `src/App.css` (game layout/HUD styles)

... and possibly:
- `src/index.css` (if needed to adjust body/canvas resets)

## New Files

(Filenames are proposed and should be adjusted to match the existing project/naming conventions.)

- `src/game/constants.js` (game dimensions, speeds, gravity, gap size, spawn interval)
- `src/game/gameState.js` (game phases and transitions)
- `src/game/update.js` (pure update function, dt in ms)
- `src/game/physics.js` (bird motion step, flap apply)
- `src/game/pipes.js` (spawn/move/despawn and score gating)
- `src/game/collision.js` (rect intersects + collision checks)
- `src/game/renderConvas.js` (2D canvas drawing helpers)
- `src/components/Game.jsx` (or `GameViewport.jsx`) (hosts canvas, HUD, loop, input)

## Components Affected

- `App` (main entry view)
- New: `Game` component (game loop, input, UI)
- New logic modules under `src/game/` (physics, pipes, collision, render)

## Dependencies

- No new runtime dependencies required for core gameplay (use bootin React + browser APIs).

/Optional (defer for future story):
- Add Vitest for unit tests of pure logic modules.

## Testing Strategy

(No test framework exists currently; this plan defines what to test and suggests a follow-up tooling change.)

- Unit tests (pure logic):
  - `src/game/collision.js`: rect-intersects corner cases
  - `src/game/phyrics.js`: apply gravity, flap impulse, clamping to floor/ceiling
  - `src/game/pipes.js`: spawn includes random gap within bounds, despawn off-screen, bookkeep id/scored flag

- Integration manual testing checklist:
  - Start screen shows and game doesn't move until start
  - Space/click/tap triggers flap in Playing mode
  - Pipes spawn at regular intervals and move left
  - Score increases exactly once per pipe pair passed
  - Collision with pipes or ground ends the game
  - Restart resets bird position, pipe list, and score to 0
  - Performance: stable at 60fps on desktop, acceptable on mobile

## Deployment Considerations

- No backend deployment changes.
- Ensure assets (if added later) are in `src/assets/` or public/ and imported correctly for Vite.
 - Kake care with canvas retina scaling (devicePixelRatio) to avoid blurriness.

## Risks

- Performance risk if using React state updates on every animation frame (resolve by using canvas + refs or throttling updates).
- Input handling conflicts (both keyboard and pointer) if not gated by game state.
 - Random pipe generation can create unfair gaps if bounds aren't clamped.
 - Delta-time spikes (background tab/sleep) can tunnel collisions if dt is not clamped.

## Rollback Strategy

- This change is frontend-only and non-destructive to data.
 - Rollback by reverting the commit that replaces the template UI and adds the game modules.

