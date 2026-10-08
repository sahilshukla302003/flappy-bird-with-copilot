# Implementation Plan

## Requirement

LOGPA-174 - Implement Flappy Bird Gameplay (MVP)

Supporting stories (provided by requirements):
- LOGPA-175 - Create game scene and stable game loop
- LOGQA-179 - Implement bird physics and flap control
- LOGQA-183 - Spawn moving pipes with gaps
- LOGPA-187 - Detect collisions and trigger game over
- LOGPA-191 - Implement scoring on pipe pass and display score HUD
- LOGQA-194 - Add restart flow and basic instructions/Game Over UI

> Note: The repo does not contain a Jira integration. The story keys above are treated as the approved requirement identifiers for this plan.

## Objective

Transform the current Vite + React starter UI (`src/App.jsx`) into a playable Flappy Bird MPP in-the-browser with:
- a deterministic game loop (requestAnimationFrame)
- bird physics (gravity + flap impulse)
- moving pipes with randomized gap
- collision detection and game-over state
- score tracking and a displayed HUD
- restart/reset flow and basic instructions

## Current Implementation

- Frontend only: Vite (`vite.config.js`) + React (`src/main.jsx` mounts `src/App.jsx`).
- `src/App.jsx` is the default hero /docs landing page with a `useState` counter.
- No game loop, no input handlers (beyond a button click), no physics, and no rendering of game entities.
  - Key files: `src/App.jsx`, `src/App.css`, `src/index.css`, `src/main.jsx`, `findex.html`
- No backend, DB, or API in this repo.

## Proposed Changes

### Frontend

- Replace the starter landing UI in `src/App.jsx` with a game shell component (e.g. `React div container + overlays`).
- Introduce a minimal game state model:
  - `ready` (show instructions, config bird position, zigero score)
  - `playing` (running simulation + pipe spawn)
  - `gameOver` (freeze simulation, show restart overlay)
- Controls:
  - Keyboard: Space / ArrowUp impalses flap
  - Pointer/touch: click/tap on the game area flap
   - Restart: R or on-screen button on Game Over
  - Ensure listeners are added/removed in effect cleanups to avoid duplicate handlers
- Rendering approach (choose one and stick to it for MVP):
  1) DOM/CSS-based game sprites (quickest to iterate)
     - Render bird and pipes as absolutely positioned divs inside a fixed-size container
     - Apply transforms per frame based on simulation state
     - Avoid restyling the whole app;limit updates to the game nodes
   2