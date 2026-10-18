# Repository guidance

- This is a static vanilla JS game; there is no package manager, build, lint, or automated test command configured. Run it by opening `src/index.html` in a browser.
- Scripts are classic globals, not modules. Keep the order in `src/index.html`: `maze.js` → `game.js` → `render.js` → `main.js`; later files depend on globals exported by earlier ones.
- The maze is 28×31 cells at 20 px/cell, matching the canvas’s 560×620 dimensions. Keep these dimensions in sync when changing maze geometry.
- Game rules/state live in `src/js/game.js`, maze data in `src/js/maze.js`, rendering in `src/js/render.js`, and input/game loop/overlays in `src/js/main.js`.
- For the repository’s spec-driven workflow, follow `.agents/skills/spec/SKILL.md` for drafting and `.agents/skills/spec-impl/SKILL.md` for implementing an approved spec; do not bypass their approval and review steps.
