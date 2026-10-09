# Implementation Plan

## Requirement

ENH-001 — Implement core Flappy Bird gameplay (MVP)

> Note: No Jira project/keys were provided/created in the connected Jira instance. This plan is based on the approved requirement text above and the existing repository code.

## Objective

Replace the current Vite/React starter landing page with a playable Flappy Bird MVP in the browser, including:

- Game states: Start → Playing → Game Over → Restart
- requestAnimationFrame-driven game loop using delta time
- Bird physics: gravity + flap impulse via Space / Click / Tap
- Pipes: spawn at interval, move left, gap randomization, cleanup
- Collision detection and scoring
- Basic rendering (Canvas recommended) with minimal placeholder visuals

## Current Implementation

- The app is a default Vite + React SPA.
- `src/main.jsx` mounts `<App />`.
- `src/App.jsx` renders a static landing page with hero images/logos, documentation links, and a simple counter button using `useState`.
- No game loop, physics, entities, input abstraction, or rendering pipeline exists.
- No backend, API, or database layers exist in this repo.

## Proposed Changes

### Frontend

#### Architecture (keep game logic testable)

- Introduce a small “game engine” set of modules that:
  - Holds the authoritative game state (bird, pipes, score, phase)
  - Advances the simulation via a pure `step(state, dt, inputs)` function
  - Produces renderable primitives (or allow renderer to read state)
- Keep React responsible for:
  - Lifecycle (starting/stopping animation loop)
  - Capturing input events and turning them into discrete actions
  - Rendering: Canvas element + overlay UI (start/game-over/hud)

#### Game loop

- In React (likely in `App.jsx` or a new `Game` component):
  - Use `requestAnimationFrame`.
  - Track `lastTimestamp` and compute `dtMs`, clamp `dtMs` (e.g., max 50ms) to avoid tunneling after tab switching.
  - Only step simulation when phase is `PLAYING`.
  - Keep RAF running for rendering and overlay updates; alternatively pause RAF on non-playing phases.

#### Input handling

- Support:
  - Keyboard: Space to flap during `PLAYING`; Space/click/tap to start from `START`; Space/click/tap to restart from `GAME_OVER`.
  - Pointer/touch: click/tap on the game surface to flap.
- Implementation approach:
  - Create a small input module that exposes a transient “flap requested” boolean that is consumed/reset each frame.
  - Use `keydown` listener with `event.code === 'Space'` and `preventDefault()`.
  - Use `pointerdown` on the canvas container and `preventDefault()`.

#### Entities and physics

- Define constants in one place (gravity, flap impulse, pipe speed, spawn interval, gap size, bird size, world width/height).
- Bird:
  - State: `{ x, y, vy }`
  - Step: `vy += gravity * dt; y += vy * dt`
  - Flap: set `vy = flapImpulse` (negative upward) or `vy += flapImpulse` depending on tuning.
- Pipes:
  - State per pipe pair: `{ x, gapCenterY, scored }`
  - Move: `x -= pipeSpeed * dt`
  - Spawn: every `spawnIntervalMs` create new pipe with randomized `gapCenterY` within bounds.
  - Cleanup: remove when `x + pipeWidth < 0`.

#### Collision and scoring

- Use axis-aligned bounding box (AABB) rectangles:
  - Bird rectangle derived from `{x,y}` and bird dimensions.
  - Each pipe pair forms 2 rectangles (top pipe and bottom pipe) based on `gapCenterY` and `gapSize`.
- Collision triggers `GAME_OVER` when:
  - Bird intersects any pipe rect
  - Bird goes out of world bounds (y < 0 or y + birdHeight > worldHeight)
- Scoring:
  - When bird passes pipe centerline (e.g., `pipe.x + pipeWidth < bird.x`) and `scored === false`, increment score and set scored.

#### Rendering (Canvas)

- Add a Canvas component that:
  - Sets canvas width/height based on a logical resolution (e.g., 360x640) and scales via CSS for responsiveness.
  - Uses `devicePixelRatio` to render crisp lines.
  - Draws background, pipes, bird, and HUD text.
- Keep visuals minimal:
  - Bird: filled circle/rounded rect.
  - Pipes: green rectangles.
  - Ground/sky: solid fills.
- Add overlay UI (React DOM) positioned over the canvas:
  - Start: title + “Press Space / Tap to start”
  - Game Over: score + “Press Space / Tap to restart”
  - HUD: score displayed during playing.

#### Accessibility / UX

- Ensure the game surface is focusable (`tabIndex=0`) so keyboard users can start.
- Provide visible instructions and a restart control (button) in overlays.
- Prevent page scroll on Space and on touch interactions during play.

### Backend

- None. This project is a static frontend-only app.

### Database

- None.

### API

- None.

## Files to Modify

- `src/App.jsx` (replace starter UI with game container/component usage)
- `src/App.css` (styles for canvas container + overlays; remove unused starter styles as needed)
- `src/index.css` (global styles to support full-viewport game surface and disable selection/scroll during play if needed)

## New Files

> Exact file structure can be adjusted during implementation; below is a concrete proposed layout consistent with this repo.

- `src/game/constants.js` — Tunable constants (world size, gravity, speeds, sizes)
- `src/game/state.js` — Initial state factory and phase enum
- `src/game/step.js` — Pure simulation step function: `(state, dtMs, actions) => nextState`
- `src/game/collision.js` — AABB helpers and collision checks
- `src/game/pipes.js` — Pipe spawn/randomization helpers
- `src/game/input.js` — Input action normalization (optional; can also live in component)
- `src/components/GameCanvas.jsx` — Canvas setup + render routine
- `src/components/Overlay.jsx` — Start/GameOver overlays + HUD

## Components Affected

- `App` (becomes the game host)
- New: `GameCanvas`, `Overlay`
- New: `game/*` modules for engine logic

## Dependencies

- No new runtime dependencies required (Canvas API is built-in).
- Optional (if adding tests later): Vitest (not part of this requirement).

## Testing Strategy

Because the current repo has no test framework configured, testing is described at two levels:

- **Unit tests (recommended to add later)**
  - Make `step`, `collision`, `pipes` deterministic/pure so they can be tested.
  - Candidate tests:
    - Gravity integration consistency for varying `dtMs`.
    - Flap sets/changes velocity as expected.
    - Pipe spawn interval correctness.
    - Off-screen pipe cleanup.
    - Collision detection against known rectangles.
    - Score increments once per pipe.
- **Manual functional checks (required for MVP delivery)**
  - Start screen appears on load.
  - Space / click / tap starts the game.
  - During play, flap works on all inputs.
  - Pipes spawn and move; gap varies.
  - Collision ends the game.
  - Restart resets state (bird, pipes, score) reliably.
  - Score increments when passing pipes, not on collision.
  - Works across refresh, resize, and tab switch (dt clamping).

## Deployment Considerations

- No special deployment changes needed for a static Vite build.
- Ensure canvas scaling and responsive layout work on typical viewport sizes.
- If hosting under a subpath (e.g., GitHub Pages), confirm `vite.config.js` `base` value is correct (currently unknown/unchanged in this plan).

## Risks

- Physics tuning: gravity/flap/pipe speed must feel playable; may need iteration.
- Delta-time spikes can cause “tunneling” through pipes; mitigate via `dt` clamp and/or sub-stepping.
- Input handling on mobile can trigger scroll/zoom; must prevent default and use proper CSS (`touch-action: none`).
- Canvas sizing with devicePixelRatio can cause blurry rendering if mishandled.

## Rollback Strategy

- This change is isolated to frontend files.
- Rollback options:
  1. Revert the commit that introduces the game modules and restores the original `App.jsx` starter UI.
  2. If multiple commits are used, revert the merge commit of the implementation PR.
  3. Keep the old starter UI in a separate component during development to allow quick toggling if needed.
