# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Proyecto

Juego estilo Asteroids en HTML5 Canvas puro: sin dependencias, sin bundler, sin build, sin tests ni linter. Toda la lógica vive en `game.js`; `index.html` solo monta un `<canvas id="canvas" width="800" height="600">` y carga el script. Textos de UI y comentarios están en español.

## Cómo correr

Abrir `index.html` directamente en el navegador, o servir el directorio:

```bash
npx serve .   # luego http://localhost:3000
```

No hay paso de compilación: los cambios en `game.js` se ven recargando la página.

## Arquitectura de `game.js`

- **Loop**: `loop(ts)` vía `requestAnimationFrame` llama `update(dt)` y luego `draw()`. `dt` está en segundos y se limita a 0.05 s; todas las velocidades están en px/s (o rad/s) y se multiplican por `dt`.
- **Estado global** en variables de módulo (`ship`, `bullets`, `asteroids`, `particles`, `score`, `lives`, `level`, `state`). `initGame()` reinicia todo; `nextLevel()` se dispara cuando `asteroids` queda vacío y genera `3 + level` asteroides grandes.
- **Máquina de estados** `state`: `'playing'` → `'dead'` (tras `killShip()`, temporizador `deadTimer` de 2 s, luego `ship.reset()`) → `'gameover'` cuando `lives` llega a 0 (Espacio reinicia). `update()` tiene una rama por estado.
- **Entidades** (`Bullet`, `Asteroid`, `Ship`, `Particle`) comparten la convención `update(dt)` / `draw()` / flag `dead`. La limpieza se hace filtrando arrays (`arr.filter(x => !x.dead)`) en `update()`, no dentro de las clases.
- **Espacio toroidal**: las posiciones se envuelven con `wrap()` (excepto partículas). Las colisiones usan `dist()` circular simple y **no** consideran el envolvimiento de bordes.
- **Asteroides**: `size` 3/2/1 (grande/mediano/pequeño) indexa los arrays `RADII`, `SPEEDS`, `POINTS` (índice 0 sin usar). `split()` devuelve dos asteroides de `size - 1`.
- **Input**: `keys[code]` para teclas mantenidas; `pressed(code)` para detección de flanco (consume el `justPressed`). Usar `pressed()` para acciones de una sola vez (disparar, reiniciar).
- **Nave**: invencibilidad de 3 s al reaparecer (con parpadeo en `draw()`), cooldown de disparo de 0.2 s, colisión con `ship.radius + a.radius * 0.82`.
- Todo el dibujo usa el `ctx` global; el HUD y overlays están en `drawHUD()` / `drawOverlay()`. `W`/`H` están hardcodeados (800×600) y deben coincidir con el tamaño del canvas en `index.html`.

## Notas

- El README todavía menciona power-ups y una "estrella fugaz", pero esas funciones se eliminaron del código (commit `13e713f`). No asumir que existen.
