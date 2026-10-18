# SPEC 01 — Cuatro fantasmas inteligentes

> **Estado:** Implemented
> **Depende de:** Ninguna
> **Fecha:** 2026-10-18
> **Objetivo:** Añadir cuatro fantasmas con estrategias distintas, incluido uno que persigue a Pacman por la ruta transitable más corta.

## Alcance

**Incluye:**

- Cuatro fantasmas activos desde el comienzo, con posiciones iniciales distintas dentro de la jaula.
- Cuatro estrategias: perseguidor, emboscador, patrullero y errático.
- La misma velocidad actual para los cuatro; se diferencian por su comportamiento.
- Mantener la apariencia actual y los cuatro colores ya asignados.

**Fuera de alcance (para futuras especificaciones):**

- Fases globales de persecución y dispersión.
- Aumentos de velocidad, poderes o cambios visuales en las formas de los fantasmas.

## Modelo de datos

En `src/js/maze.js`, `GHOST_STARTS` tendrá cuatro posiciones distintas y transitables dentro de la jaula:

```js
const GHOST_STARTS = [
  { x: 12, y: 14, kind: 'hunter' },
  { x: 13, y: 14, kind: 'ambusher' },
  { x: 14, y: 14, kind: 'patroller' },
  { x: 15, y: 14, kind: 'erratic' },
];
```

En `src/js/game.js`, el estado de cada fantasma conserva `x`, `y`, `dir`, `speed` y `kind`. El patrullero también conserva `patrolIndex`, que empieza en `0` y señala su próximo destino.

Los puntos del patrullero se recorren en este orden: `(1,1)`, `(26,1)`, `(26,29)` y `(1,29)`.

## Plan de implementación

1. Añadir las cuatro posiciones y tipos a `GHOST_STARTS`, y reiniciar el índice de patrulla al restaurar posiciones. Comprobar que la partida sigue iniciando con cuatro fantasmas.
2. Añadir a `src/js/game.js` una búsqueda de rutas por celdas que considere paredes, puertas, y el túnel. Mantener el movimiento funcional mientras se conecta la búsqueda.
3. Hacer que el perseguidor siga la ruta transitable más corta hacia Pacman. Hacer que el emboscador siga la ruta hacia la celda situada cuatro pasos delante de Pacman.
4. Hacer que el patrullero alcance sus puntos en ciclo y que el errático elija aleatoriamente un giro legal en cada cruce.
5. Si un objetivo cae en una pared o fuera del laberinto, ajustarlo a la celda transitable alcanzable más cercana antes de calcular la ruta.

## Criterios de aceptación

- [ ] La partida crea cuatro fantasmas en posiciones distintas dentro de la jaula.
- [ ] Los cuatro tienen colores distintos y conservan la apariencia existente.
- [ ] El perseguidor busca la ruta transitable más corta hacia Pacman y tiene la misma velocidad que los otros fantasmas.
- [ ] El emboscador apunta cuatro celdas delante de Pacman según la dirección en la que se mueve.
- [ ] El patrullero visita los cuatro puntos definidos en orden y reinicia el ciclo.
- [ ] El errático solo elige movimientos legales y puede continuar moviéndose por el laberinto.
- [ ] El reinicio restaura las posiciones iniciales y el recorrido del patrullero.
- [ ] El juego funciona al abrir `src/index.html` y no presenta errores en la consola.

## Decisiones

- **Sí:** Diferenciar a los fantasmas por estrategia, no por velocidad.
- **Sí:** Activar los cuatro desde el inicio y mantener los colores actuales.
- **Sí:** Usar las rutas más cortas para el perseguidor, el emboscador y el patrullero.
- **No:** Alternar fases globales; el perseguidor debe perseguir continuamente.
- **No:** Implementar las fórmulas y fases clásicas completas; se usarán perfiles simplificados.

## Riesgos

| Riesgo | Mitigación |
| --- | --- |
| El cálculo de ruta podría ignorar el túnel o la puerta de la jaula y dejar un fantasma atascado. | La búsqueda tratará el túnel como conexión y aplicará las reglas de paso de los fantasmas, no las de Pacman. |
| Un objetivo calculado puede caer en una pared o fuera del tablero. | Ajustar el objetivo a una celda transitable alcanzable cercana antes de buscar la ruta. |

## Qué **no** incluye esta especificación

- Poderes para asustar o comer fantasmas.
- Aumentos de velocidad o cambios visuales en las formas.
- Ciclos clásicos de persecución y dispersión.
