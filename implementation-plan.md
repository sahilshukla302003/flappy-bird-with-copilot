# Implementation Plan

## Requirement

JARA-UNKNOWN: Implement core Flappy Bird gameplay (MVP)

## Objective

Convert the current Vite + React template app Into a playable Flappy Bird MVP: a
- game loop
- bird physics (gravity + flap impulse)
- pipe obstacle spawning with a gap
- collision detection (bird vs pipes and ground/ceiling)
- scoring when passing pipes
A minimal user experience: Start overlay -> Playing -> Game Over -> Restart.

## Current Implementation

The repo is a standard Vite + React starter.

- `src/main.jsx` mounts `<App/>` to `# root`.
- `src/App.jsx` shows a static hero section with logos and hero image, a counter button, plus template docs/social links.
- No game loop, no canvas, and no game state model.
- No backend, API, or database exists in this project (static frontend only).

## Proposed Changes

### Frontend

- Replace the template UI in `src/App.jsx` with a game root component (e.g., `<FlappyBirdGame />`).
  - Keep `App.jsx` as a thin shell or rename/reorganize if desired, but the gap should be unmaintained for clarity.


- Decide a rendering strategy for the game:
  - (Plan A): html <canvas> rendering (recommended for performance and simplicity).
  - (Plan B): DOM divs for bird/pipes with CSS transforms.
   - Economics: canvas reduces dom churn and simplifies collision bounding box calcs.

- Introduce a game state model and loop:
  - Game states: `"ready"| "playing" | "game-over"| ("paused" optional)*.
  - Use requestAnimationFrame (not setInterval) for the main loop.
  - Timestep: calculate dt from `performance.now()`, clamp dt to avoid tunneling after tab switch.

- Bird physics:
  - State: y position, velocity vy, constant x (world scrolls via pipe movement).
  - Params: gravity, flap impulse, max vy, clamps. Place in a `constants` module for tuning.

- Pipes:
  - Spawn pipe pairs on a timer (e.g. every N seconds) or by distance traveled.
  - Pipe state: x x, y gapCenterY, gapHeight, width, and a `scored` boolean.
   - Move pipes left with scrollSpeed; remove when off-screen to bound memory.

- Collision detection:
  - Define bounding boxes for bird and two pipe rects (top and bottom).
  - Detect ground/ceiling hit.
  - On hit: transition to "game-over" and freeze world updates.
 
- Scoring:
  - Increment score when a pipe pair passes bird x and it hasn't been scored yet.
  - display score overlay ("current") while playing and ("final") on game over.

- Overlays (Ready/GameOver):
  - Ready: nudge controls ("press space/tar to flap") and a Start call-to-action.
  - Game over: show final score and a Restart button. Also support space/tap to restart.

- Input handling:
  - Keyboard: Space/ArrowUp to flap/start/restart.
  - Pointer/touch: tap/click on the game container to flap/start/restart.
  - Prevent default touch scroll in the game area using CSS (touch-action) and event preventDefault as needed.

- Styling:/Layout:
  - Create a game viewport with fixed aspect ratio and responsive scaling.
  - Ensure only 1 rAF loop runs at a time (stable refs). 

### Backend

- No backend changes (project is frontend-only).

### Database

- No database changes (any high score persistence via localStorage would be a separate story).


### API

- No API changes.

## Files to Modify

- `src/App.jsx`
- `src/App.css` (or migrate to a new game CSS file)
- `src/index.css` (only if needed)

## New Files

- `src/game/FlappyBirdGame.jsx`
- `src/game/useAnimationFrame.js`
- `src/game/engine/constants.js`
- `src/game/engine/step.js`
- `src/game/engine/collision.js`
- `src/game/engine/pipes.js`
- `src/game/render/canvasRender.js`
   * (only if canvas rendering is chosen)
- `src/game/game.css`

## Components Affected

- `App`
- New: `game/FlappyBirdGame`
- New: `game/engine/*
` (pure game logic)

## Dependencies

- No new dependencies required for MVP game logic. (Optional: add Vitest in a fature story to test pure functions.)

## Testing Strategy

- Unit tests:
  - (Optional follow-up story) introduce Vitest and test `step()`, `collision` intersection, and `pipe spawner` behavior.
- Integration/ui tests:
  - Manual checklist: start, flap, score increments, collision ends game, restart resets state, input works on mobile (touch).
- Performance:
  - Ensure reafs loop is stable and does not leak on restart.

## Deployment Considerations

- Static frontend build via `npm run build` remains unchanged.
- Canvas rendering should be tested on low-end mobile browsers for fps drops.

## Risks

- Delta-time and rAF loop bugs can cause inconsistent physics if dt is not clamped.
- Input handling on mobile can conflict with scrolling if touch defaults aren't prevented.
- Canvas scaling and hi-DPI rendering can lead to blurriness if not adjusted (to be handled in renderer).


## Rollback Strategy

- This PRonly adds a plan file and does not change production code.
- To rollback: revert the commit or close the PR without merging.