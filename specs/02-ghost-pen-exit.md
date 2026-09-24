# SPEC 02 — Salida de los fantasmas de la pen

> **Estado:** Approved
> **Depende de:** SPEC 01
> **Fecha:** 2026-09-23
> **Objetivo:** Hacer que cada fantasma salga de la pen por la puerta al liberarse, sin quedar atrapado en la jaula, y que no pueda volver a entrar.

## Alcance

**Dentro:**

- Modo salida para los 4 fantasmas: al liberarse, ignoran su IA de comportamiento y se dirigen forzadamente hacia la puerta del pen (filas 12–13, cols 13–14) para salir al mapa.
- La puerta del pen pasa a ser de un solo sentido para los fantasmas: solo se cruza hacia arriba (saliendo), nunca hacia abajo.
- Aplicar la misma salida en cada reaparición tras perder una vida, reutilizando `resetPositions`.
- Nueva constante para documentar la geometría de salida en un solo lugar.

**Fuera de alcance (para futuros specs):**

- Colisiones o separación entre fantasmas dentro de la pen (hoy los fantasmas no colisionan entre sí; comportamiento actual).
- Rutas de salida elaboradas con waypoints "clásicos".
- Power-pellets y modo asustado (Frightened).
- Cambios de velocidad o niveles adicionales.

## Modelo de datos

No se introducen estructuras globales nuevas. Se amplían las existentes:

```js
// src/js/game.js — constantes nuevas
const GHOST_DOOR_COLS = [13, 14]; // celdas de la puerta del pen (fila 12)
const GHOST_PEN_EXIT_Y = 11;      // fila justo encima de la puerta: "fuera del pen"
```

```js
// cada ghost gana un campo, inicializado en createGame() y resetPositions()
const g = {
  x, y, dir, speed, kind,
  released: false,
  releaseAt: 0,
  outside: false, // false = modo salida: se dirige a la puerta y sube hasta la fila 11
};
```

Convenciones:

- Coordenadas: origen arriba-izquierda, `x ∈ [0,27]`, `y ∈ [0,30]`.
- La puerta (`3` en `MAZE`) sigue siendo transitable para fantasmas; solo cambia la dirección permitida.

## Plan de implementación

1. En `src/js/game.js`, añadir `GHOST_DOOR_COLS` y `GHOST_PEN_EXIT_Y`, el campo `outside:false` en `createGame()` y su reinicio en `resetPositions()`. Prueba manual: `src/index.html` carga sin errores; el juego se comporta igual (el bug persiste hasta el paso 2).

2. Añadir en `decideGhost()` una rama de modo salida al inicio, **antes** de las ramas de comportamiento:
   - Si `!g.outside`: si `x < 13` → `dir='right'`; si `x > 14` → `dir='left'`; si `x ∈ {13,14}` → `dir='up'`.
   - Al alinearse en `(x, GHOST_PEN_EXIT_Y)` con `x ∈ {13,14}`, marcar `outside=true` (a partir del siguiente cruce usa su IA normal).

   Prueba manual: los 4 fantasmas abandonan el pen escalonadamente (hunter → ambusher → patrol → random cada 1.5 s) sin atascarse, y al llegar a la fila 11 retoman su comportamiento.

3. En `canMove()`, bloquear para `actor==='ghost'` el cruce hacia abajo a través de una celda de puerta (`grid[ty][tx] === 3 && dir === 'down'`). Prueba manual: ningún fantasma liberado vuelve a entrar al pen (observar al `random` ≈60 s).

## Criterios de aceptación

- [ ] Al iniciar la partida, cada fantasma abandona la pen tras su liberación y llega al mapa (filas ≤ 11) sin quedarse atascado.
- [ ] El `hunter` (13,14) sube recto por la puerta sin desplazarse lateralmente.
- [ ] El `ambusher` (12,14) se desplaza a la columna 13 y sube por la puerta.
- [ ] El `patrol` (15,14) se desplaza a la columna 14 y sube por la puerta.
- [ ] El `random` (14,14) sube recto por la puerta.
- [ ] Ningún fantasma permanece atrapado en las filas 13–15 del pen durante los primeros 30 s de partida.
- [ ] Tras salir, ningún fantasma vuelve a entrar en la pen (la puerta solo se cruza hacia arriba).
- [ ] Al perder una vida (con ≥1 vida restante), los 4 reinician en la pen y vuelven a salir escalonadamente por la puerta.
- [ ] El resto del juego no cambia: dots, colisión con fantasmas, estados `won`/`lost`, túnel en la fila 14.
- [ ] No hay errores en la consola al cargar `src/index.html`.

## Decisiones

- **Sí:** camino forzado hacia la puerta durante el modo salida (ignora la IA). Determinista y mínimo; coincide con la decisión de SPEC 01 ("cada fantasma sale en línea recta hacia la puerta").
- **Sí:** fin del modo salida al alcanzar la fila 11 sobre la puerta. Punto claro y verificable.
- **Sí:** puerta de un solo sentido (bloquear bajada en `canMove`). Evita que el bug reaparezca por reentrada del `random` u otros.
- **Sí:** reaplicar el modo salida en cada reaparición. Comportamiento consistente; reutiliza `resetPositions`.
- **Sí:** constantes `GHOST_DOOR_COLS` y `GHOST_PEN_EXIT_Y` en `game.js`. La geometría de salida se documenta en un solo lugar.
- **No:** colisiones/separación entre fantasmas en la pen. Los fantasmas no colisionan entre sí hoy; otro spec si se desea.
- **No:** rutas de salida con waypoints clásicos más elaborados.

## Riesgos

| Riesgo | Mitigación |
| --- | --- |
| El camino horizontal forzado asume que las celdas interiores del pen (filas 13–15, cols 11–16) son transitables. Si `MAZE` cambia, el supuesto se rompe. | Geometría documentada en constantes únicas (`GHOST_DOOR_COLS` y `MAZE` en `maze.js`) y cubierta por los criterios de aceptación manuales. |

## Lo que **no** está en este spec

- Colisiones o separación entre fantasmas dentro de la pen.
- Rutas de salida elaboradas con waypoints clásicos.
- Power-pellets y modo asustado (Frightened) — otro spec.
- Cambios de velocidad o niveles adicionales.