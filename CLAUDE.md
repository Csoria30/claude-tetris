# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project

Tetris implementado en JavaScript vanilla (ES6+) con HTML5 Canvas. Sin dependencias, sin `package.json`, sin bundler ni transpilador: todo el proyecto son 3 archivos (`index.html`, `style.css`, `game.js`).

## Running the game

No hay build ni instalación. Basta con abrir `index.html` en el navegador, o servirlo con cualquier servidor estático:

```bash
python3 -m http.server 8000
# o
npx serve .
```

No hay tests, linter ni CI configurados en el repo.

## Architecture

Toda la lógica vive en `game.js` (un único archivo, sin módulos). Piezas clave:

- **Estado global**: `board` (matriz `ROWS × COLS`, cada celda es `0` o un índice de color 1–7), `current`/`next` (pieza activa y siguiente), `score`/`lines`/`level`, flags `paused`/`gameOver`.
- **Piezas**: definidas en `PIECES` como matrices cuadradas fijas (índice = color en `COLORS`). `rotateCW` rota transponiendo + invirtiendo filas; no hay estado de rotación separado, se rota la matriz de la pieza directamente.
- **Colisión y wall kicks**: `collide(shape, ox, oy)` es la única función de detección de colisión, reutilizada para movimiento, rotación, ghost piece y hard drop. `tryRotate` prueba una serie de desplazamientos (`kicks`) tras rotar antes de descartar el giro.
- **Game loop**: un único `loop(ts)` en `requestAnimationFrame` acumula delta time (`dropAccum`) contra `dropInterval`; no hay lógica de fixed-timestep más allá de eso.
- **Ciclo de vida de una pieza**: `spawn()` promueve `next` a `current` y genera la siguiente; si la nueva pieza colisiona al aparecer, dispara `endGame()`. `lockPiece()` (llamado desde soft/hard drop o el loop al no poder bajar más) hace `merge()` → `clearLines()` → `spawn()`.
- **Rendering**: `draw()` redibuja todo el canvas cada frame (grid, tablero fijo, ghost piece con `globalAlpha`, pieza actual). `drawNext()` hace lo mismo en el canvas secundario `next-canvas`. No hay dirty-rect ni capas: todo se limpia y redibuja por completo.
- **HUD**: `updateHUD()` sincroniza `score`/`lines`/`level` con el DOM tras cualquier cambio de puntuación.

Al modificar `COLS`, `ROWS` o `BLOCK` en `game.js`, hay que ajustar también `width`/`height` del `<canvas id="board">` en `index.html` (`COLS × BLOCK` y `ROWS × BLOCK`).
