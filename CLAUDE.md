# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project

Tetris clásico en JavaScript vanilla con HTML5 Canvas. Sin build, sin dependencias, sin `package.json`. Todo el proyecto son 3 archivos: `index.html`, `style.css`, `game.js`.

## Running

No hay proceso de build ni test suite. Para probar cambios, servir el directorio y abrir en el navegador:

```bash
python3 -m http.server 8000   # o: npx serve .
```

Luego abrir `http://localhost:8000`. También se puede abrir `index.html` directamente con `xdg-open index.html`.

## Architecture

Toda la lógica vive en `game.js` (un solo archivo, sin módulos). Puntos clave para entender el flujo antes de tocar código:

- **Estado global**: variables sueltas (`board`, `current`, `next`, `score`, `lines`, `level`, `paused`, `gameOver`, `dropInterval`, etc.) declaradas arriba del archivo y mutadas directamente por las funciones — no hay clases ni un objeto de estado central.
- **Tablero**: matriz `ROWS × COLS`, cada celda es `0` (vacía) o un índice `1–7` que identifica el color/tipo de pieza fijada.
- **Piezas**: matrices cuadradas en `PIECES`. La rotación (`rotateCW`) transpone + invierte filas; no usa SRS, usa wall kicks simples (`tryRotate` prueba offsets `[0,-1,1,-2,2]`).
- **Loop de juego**: `loop()` corre vía `requestAnimationFrame`, acumula delta time y baja la pieza cuando supera `dropInterval`; `dropInterval` se recalcula en `clearLines()` según el nivel (`max(100, 1000 - (level-1)*90)`).
- **Ciclo de una pieza**: `spawn()` → cae por `loop()`/`softDrop()`/`hardDrop()` → `lockPiece()` (merge en el tablero + `clearLines()` + `spawn()` de la siguiente). Si la pieza recién generada colisiona al aparecer, se dispara `endGame()`.
- **Rendering**: todo con Canvas 2D (`draw()` para el tablero principal, `drawNext()` para el preview). La ghost piece se dibuja proyectando `ghostY()` hacia abajo con `globalAlpha = 0.2`.
- **Input**: un único listener `keydown` en `document` despacha por `e.code` (flechas, `KeyX` para rotar, `Space` para hard drop, `KeyP` para pausa).
- **HUD**: `updateHUD()` sincroniza `score`/`lines`/`level` con el DOM tras cada cambio relevante.

El README.md documenta el flujo completo con más detalle (incluye diagrama ASCII de `init()`/`loop()`) y una tabla de constantes tuneables (`COLS`, `ROWS`, `BLOCK`, `COLORS`, `LINE_SCORES`, `dropInterval`) — consultarlo si hace falta más contexto antes de modificar mecánicas del juego.

**Importante**: si se cambia `COLS`, `ROWS` o `BLOCK`, hay que ajustar también `width`/`height` del `<canvas id="board">` en `index.html` para que coincidan (`COLS × BLOCK` × `ROWS × BLOCK`).
