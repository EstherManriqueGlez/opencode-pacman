# AGENTS.md

## Project

Pac-Man clone built with **Vanilla JS, HTML, CSS**. No build tools, bundlers, package managers, tests, or linters. Just open `src/index.html` in a browser to play.

## How to run

Open `src/index.html` in any modern browser. No server or build step required.

## Architecture

Four JS files loaded in strict order via `<script>` tags in `src/index.html`. Each attaches to `window` — there are no modules.

| File | Global exports | Responsibility |
|---|---|---|
| `src/js/maze.js` | `MAZE`, `TUNNEL_ROW`, `PACMAN_START`, `GHOST_STARTS` | 28×31 grid definition, constants |
| `src/js/game.js` | `createGame`, `update`, `DIRS` | Game state, movement, collision |
| `src/js/render.js` | `draw` | Canvas rendering (walls, dots, Pac-Man, ghosts, HUD) |
| `src/js/main.js` | (runs loop) | Keyboard input, overlay, `requestAnimationFrame` loop |

**Order matters**: `maze.js` → `game.js` → `render.js` → `main.js`. Removing or reordering scripts breaks the game.

### Key constants

- Maze is 28 columns × 31 rows. Origin is top-left `(x, y)`.
- Tile size is 20px (`TILE` in render.js). Canvas is 560×620.
- Tunnel row is `TUNNEL_ROW = 14` (index 14 in the grid array).
- Pac-Man starts at `(13, 23)` facing left.
- Ghosts start at `(13, 14)` and `(14, 14)` inside the pen.
- Wall `1`, dot `2`, door `3`, empty `0`.
- Pac-Man speed: `0.125` px/frame (aligns every 8 frames). Ghost speed: `0.1`.

## Spec-driven development

This repo uses a **spec-driven** workflow via `.agents/skills/`:

- **`/spec`** — Create a new spec in `specs/NN-slug.md`. Specs follow the template in `.agents/skills/spec/template.md`. States: `Draft`, `In review`, `Approved`, `Implemented`, `Obsolete`.
- **`/spec-impl`** — Implement an approved spec. Requires state `Approved` (or equivalent in any language).
- The `specs/` directory does not exist yet — it will be created on first `/spec`.
- Branch creation is controlled by `specs/.spec-config.yml` (`AutoCreateBranch`, defaults `true`).
- The spec skill reads project-memory files in order: `CLAUDE.md`, `AGENTS.md`, `GEMINI.md`, `README.md`.

## Coding conventions

- Comments are in **Spanish** throughout (`// Estado y reglas`, `// Bucle, teclado y pantallas`, etc.).
- Code is **not modularized** — everything is global, attached to `window`.
- `game.js` uses globals from `maze.js` (`MAZE`, `TUNNEL_ROW`, `PACMAN_START`, `GHOST_STARTS`).
- `render.js` uses `game.grid` (not `MAZE`) so eaten dots are reflected.
- `main.js` calls `createGame()` at top level, so the game starts immediately on page load.
- CSS uses `Press Start 2P` font (fallback to `Courier New`).

## Gotchas

- **No package.json exists** — there is no `npm test`, `npm run lint`, or any npm command.
- **No test framework, no CI/CD, no linter, no typechecker.** Verification is manual: open the HTML file and play.
- **`skills-lock.json`** references `skills/engineering/spec/SKILL.md` but actual files live under `.agents/skills/`. This is just a lock file for skill versioning.
- The `game.js` `update()` function mutates `game.grid` directly — `maze.js`'s `MAZE` is never mutated because `createGame()` copies it with `.slice()`.
