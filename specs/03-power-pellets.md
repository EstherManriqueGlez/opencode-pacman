# SPEC 03 — Power Pellets y fantasmas vulnerables

> **Estado:** Implemented
> **Depende de:** SPEC 01, SPEC 02
> **Fecha:** 2026-10-01
> **Objetivo:** Añadir Power Pellets que permiten a Pac-Man comer fantasmas vulnerables durante una ventana temporal.

## Alcance

**Dentro:**

- Añadir 4 Power Pellets al laberinto como nuevo valor `4` en `src/js/maze.js`.
- Colocar los 4 Power Pellets en celdas transitables cerca de las esquinas del laberinto; el implementador escogerá coordenadas válidas revisando `MAZE`.
- Consumir un Power Pellet suma 50 puntos, borra la celda del `game.grid` y activa el modo vulnerable durante 6 s.
- Si Pac-Man come otro Power Pellet mientras el modo vulnerable sigue activo, el temporizador se reinicia a 6 s desde ese consumo.
- Dibujar todos los fantasmas vulnerables como azul sólido en `src/js/render.js`.
- Permitir que Pac-Man coma fantasmas vulnerables con `outside === true`.
- Comer un fantasma vulnerable suma 200 puntos fijos, no mata a Pac-Man y devuelve ese fantasma a la pen para que vuelva a salir usando las reglas de SPEC 02.
- Cancelar el modo vulnerable al perder una vida y reiniciar posiciones.
- Contar los Power Pellets como comida necesaria para ganar la partida.

**Fuera de alcance (para futuros specs):**

- IA de huida o movimiento aleatorio durante el modo vulnerable.
- Parpadeo azul/blanco al final del temporizador.
- Puntuación escalada 200/400/800/1600 al comer varios fantasmas en la misma ventana.
- Sonidos, animaciones de ojos, pausa visual o transición especial al comer un fantasma.
- Nuevos niveles o cambios de velocidad por nivel.

## Modelo de datos

Se amplían las estructuras existentes:

```js
// src/js/maze.js — nuevo tipo de celda
// 0 = vacío, 1 = pared, 2 = dot, 3 = puerta, 4 = Power Pellet
const MAZE = [/* contiene exactamente 4 celdas con valor 4 */];
```

```js
// src/js/game.js — constantes nuevas
const POWER_PELLET_SCORE = 50;
const EAT_GHOST_SCORE = 200;
const FRIGHTENED_DURATION_MS = 6000;
```

```js
// src/js/game.js — campos nuevos en el estado de partida
const game = {
  frightenedUntil: 0, // ms (performance.now()) hasta que los fantasmas son vulnerables
};
```

```js
// src/js/game.js — cada ghost gana un campo derivado de la ventana vulnerable
const g = {
  x, y, dir, speed, kind,
  released: false,
  releaseAt: 0,
  outside: false,
  eatenDuringFrightened: false, // evita comer el mismo fantasma repetidamente antes de resetearlo
};
```

Convenciones:

- Coordenadas: origen arriba-izquierda, `x ∈ [0,27]`, `y ∈ [0,30]`.
- Los temporizadores se cuentan en ms reales con `performance.now()`, igual que la liberación escalonada de fantasmas.
- Un fantasma es vulnerable solo si `performance.now() < game.frightenedUntil`, `g.outside === true` y `g.eatenDuringFrightened === false`.

## Plan de implementación

1. En `src/js/maze.js`, añadir el valor de celda `4` para exactamente 4 Power Pellets en posiciones transitables cercanas a las esquinas. Prueba manual: abrir `src/index.html`; no debe haber errores de consola y el juego debe seguir cargando.

2. En `src/js/game.js`, añadir `POWER_PELLET_SCORE`, `EAT_GHOST_SCORE`, `FRIGHTENED_DURATION_MS`, `frightenedUntil` en `createGame()` y `eatenDuringFrightened:false` en cada ghost. Prueba manual: abrir el juego y comprobar que el comportamiento observable aún no cambia salvo por la presencia de los nuevos pellets si ya se renderizan como comida pendiente.

3. Actualizar la lógica de consumo de comida en `update()` para tratar `grid[y][x] === 4` como Power Pellet: sumar 50 puntos, poner la celda a `0` y establecer `frightenedUntil = performance.now() + FRIGHTENED_DURATION_MS`. Prueba manual: comer un Power Pellet aumenta el score en 50 y desaparece del mapa.

4. Ajustar la condición de victoria para que queden pendientes tanto dots (`2`) como Power Pellets (`4`). Prueba manual: la partida no se gana si queda al menos un Power Pellet sin comer.

5. En `render.js`, dibujar las celdas `4` como pellets más grandes que los dots normales y dibujar en azul sólido los fantasmas vulnerables. Prueba manual: los 4 Power Pellets se distinguen visualmente y los fantasmas fuera de la pen se ven azules durante 6 s tras consumir uno.

6. Actualizar la colisión Pac-Man/fantasma en `update()`: si el fantasma es vulnerable, sumar 200 puntos, marcarlo como comido y reiniciarlo en su posición de pen para que vuelva a salir usando `released:false`, `outside:false` y `releaseAt` actualizado. Prueba manual: Pac-Man toca un fantasma azul fuera de la pen, no pierde vida, gana 200 puntos y el fantasma reaparece en la pen.

7. Actualizar `resetPositions()` para cancelar `frightenedUntil` y limpiar `eatenDuringFrightened` en todos los fantasmas al perder una vida. Prueba manual: tras perder una vida, ningún fantasma queda azul y todos vuelven a salir según las reglas de SPEC 02.

8. Verificar el reinicio del temporizador: comer un segundo Power Pellet durante la ventana vulnerable debe llevar el final del efecto a 6 s desde el segundo consumo. Prueba manual: consumir dos Power Pellets seguidos mantiene a los fantasmas azules durante 6 s tras el segundo pellet, no suma duraciones.

## Criterios de aceptación

- [ ] `MAZE` contiene exactamente 4 celdas con valor `4` y todas están en posiciones transitables cercanas a esquinas del laberinto.
- [ ] Los Power Pellets se renderizan más grandes que los dots normales.
- [ ] Comer un Power Pellet suma exactamente 50 puntos y borra su celda del `game.grid`.
- [ ] Comer un Power Pellet activa el modo vulnerable durante 6 s medidos con `performance.now()`.
- [ ] Comer otro Power Pellet durante el modo vulnerable reinicia el temporizador a 6 s desde ese segundo consumo.
- [ ] Durante el modo vulnerable, los fantasmas con `outside === true` se dibujan azul sólido.
- [ ] Los fantasmas dentro de la pen o en modo salida no pueden ser comidos por Pac-Man por efecto del Power Pellet.
- [ ] Tocar un fantasma vulnerable fuera de la pen suma exactamente 200 puntos y no quita vida a Pac-Man.
- [ ] Un fantasma comido vuelve a la pen y sale otra vez usando la lógica de salida de SPEC 02.
- [ ] Tocar un fantasma no vulnerable mantiene el comportamiento actual: Pac-Man pierde una vida o la partida termina.
- [ ] Perder una vida cancela cualquier modo vulnerable activo.
- [ ] La partida solo se gana cuando no quedan dots (`2`) ni Power Pellets (`4`) en el `game.grid`.
- [ ] No hay errores en la consola al cargar `src/index.html`.

## Decisiones

- **Sí:** nuevo valor de celda `4` para Power Pellets. Permite distinguirlos de dots normales sin crear una estructura paralela.
- **Sí:** 4 Power Pellets cerca de esquinas transitables. Respeta el patrón clásico sin fijar coordenadas inválidas antes de revisar el mapa real.
- **Sí:** duración vulnerable de 6 s. Es suficiente para jugar y probar sin introducir escalado por nivel.
- **Sí:** Power Pellet = 50 puntos y fantasma comido = 200 puntos fijo. Mantiene el alcance simple.
- **Sí:** comer otro Power Pellet reinicia el temporizador a 6 s. Es predecible y evita ventanas acumuladas excesivas.
- **Sí:** azul sólido para fantasmas vulnerables. Es claro y requiere poca lógica visual.
- **Sí:** solo fantasmas `outside === true` son comibles. Evita casos raros dentro de la pen o durante la salida forzada.
- **Sí:** los fantasmas comidos vuelven a la pen y reutilizan la salida de SPEC 02. Integra la mecánica con el flujo existente.
- **Sí:** perder una vida cancela vulnerabilidad. Evita reinicios ambiguos.
- **No:** IA de huida durante el modo vulnerable. Queda para otro spec porque cambia decisiones de movimiento.
- **No:** puntuación escalada 200/400/800/1600. Se descarta para mantener una primera versión mínima.
- **No:** parpadeo al final del efecto. Puede añadirse después como mejora visual independiente.

## Riesgos

| Riesgo | Mitigación |
| --- | --- |
| Elegir coordenadas de Power Pellets sobre paredes o puertas rompería consumo/render. | El plan exige escoger celdas transitables revisando `MAZE` y validar que existen exactamente 4 valores `4`. |
| La colisión vulnerable puede entrar en conflicto con la colisión mortal existente. | La lógica de colisión debe comprobar primero si el fantasma es vulnerable y solo aplicar muerte cuando no lo es. |
| Reaparecer un fantasma comido puede desincronizar `released`, `outside` y `releaseAt`. | Reutilizar el mismo patrón de reinicio usado por `resetPositions()` y SPEC 02. |

## Lo que **no** está en este spec

- IA de huida o movimiento aleatorio durante el modo vulnerable.
- Parpadeo de advertencia al final del modo vulnerable.
- Puntuación escalada por múltiples fantasmas comidos.
- Sonidos, animaciones de ojos o efectos especiales.
- Nuevos niveles o cambios de velocidad por nivel.
