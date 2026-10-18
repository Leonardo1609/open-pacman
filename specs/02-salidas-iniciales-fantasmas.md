# SPEC 02 — Salidas iniciales diferenciadas de los fantasmas

> **Estado:** Implemented
> **Depende de:** SPEC 01
> **Fecha:** 2026-10-18
> **Objetivo:** Hacer que los cuatro fantasmas se dispersen por rutas iniciales asignadas antes de retomar sus estrategias habituales.

## Alcance

**Incluye:**

- Asignar una ruta de apertura a cada tipo de fantasma, dirigida a un sector distinto del laberinto.
- Hunter se dirige al sector superior izquierdo, ambusher al superior derecho, patroller al inferior derecho y erratic al inferior izquierdo.
- Permitir que las rutas compartan el tramo obligatorio de salida de la jaula, pero hacer que se separen al alcanzar las ramas exteriores que conducen a sus sectores.
- Retomar la estrategia habitual de cada fantasma después de que tome la rama hacia su sector.
- Repetir las rutas de apertura al comenzar una partida y después de perder una vida.
- Mantener las posiciones iniciales, velocidades, colores y estrategias actuales una vez terminada la apertura.

**Fuera de alcance (para futuras especificaciones):**

- Cambiar la geometría del laberinto o las posiciones iniciales.
- Cambiar los colores, velocidades o estrategias habituales de los fantasmas.
- Mantener rutas asignadas durante el resto de la partida.

## Modelo de datos

En `src/js/game.js`, los destinos de apertura se definen por tipo usando los puntos de esquina ya existentes en `PATROL_POINTS`:

```js
const GHOST_OPENING_TARGETS = {
  hunter: { x: 1, y: 1 },
  ambusher: { x: 26, y: 1 },
  patroller: { x: 26, y: 29 },
  erratic: { x: 1, y: 29 },
};
```

Cada fantasma tendrá un indicador `openingActive`, que empieza en `true` y vuelve a `true` al restaurar sus posiciones. Mientras esté activo, el fantasma sigue la ruta legal más corta hacia su destino de apertura. Lo desactiva después de tomar la primera rama exterior que lo orienta hacia su sector; desde entonces se usa su estrategia habitual.

## Plan de implementación

1. Añadir los destinos de apertura por tipo y el estado `openingActive` al crear fantasmas. Comprobar que la partida sigue creando cuatro fantasmas con las velocidades y posiciones actuales.
2. Integrar la elección de ruta de apertura con `findPath` y las reglas de movimiento existentes para que los fantasmas avancen legalmente hacia sus sectores.
3. Cambiar cada fantasma a su estrategia habitual después de tomar la rama exterior asignada. Comprobar que los cuatro conservan su comportamiento actual una vez dispersos.
4. Restaurar `openingActive` al reiniciar posiciones tras perder una vida. Comprobar manualmente una nueva partida y una reaparición.

## Criterios de aceptación

- [ ] Al iniciar una partida, los cuatro fantasmas conservan sus posiciones, velocidades y colores actuales.
- [ ] Hunter, ambusher, patroller y erratic toman rutas hacia los sectores superior izquierdo, superior derecho, inferior derecho e inferior izquierdo, respectivamente.
- [ ] Las rutas pueden compartir el tramo obligatorio dentro de la jaula, pero se separan en las ramas exteriores hacia sus sectores.
- [ ] Cada fantasma retoma su estrategia habitual después de tomar su rama exterior asignada, sin esperar un temporizador fijo.
- [ ] Ningún fantasma intenta avanzar a una pared ni queda detenido por no encontrar su ruta de apertura.
- [ ] Al perder una vida, los cuatro fantasmas repiten sus rutas de apertura desde sus posiciones iniciales.
- [ ] La partida funciona al abrir `src/index.html` y no presenta errores en la consola.

## Decisiones

- **Sí:** Asignar destinos por tipo de fantasma y no por color. Los tipos ya son parte del estado del juego y los colores no determinan el comportamiento.
- **Sí:** Permitir compartir únicamente el tramo obligatorio de salida; se prioriza la dispersión en las primeras ramas disponibles sin alterar el laberinto.
- **Sí:** Usar rutas legales más cortas a cuatro sectores y reanudar la IA existente después de tomar la rama correspondiente.
- **No:** Cambiar la jaula para crear salidas físicas adicionales. No es necesario para diferenciar las rutas tras la salida común.
- **No:** Mantener la ruta de apertura durante toda la partida. Los perfiles definidos en SPEC 01 vuelven a controlar a cada fantasma tras la dispersión inicial.

## Riesgos

| Riesgo | Mitigación |
| --- | --- |
| Un destino de apertura puede dejar de ser alcanzable si cambia el laberinto. | Reutilizar `findPath` y el ajuste a celdas alcanzables existente; si no hay ruta, desactivar la apertura y usar la estrategia habitual. |
| La transición a la IA habitual puede ocurrir antes de que el fantasma tome una rama distinta. | Verificar la transición en las bifurcaciones exteriores y mantener la apertura activa hasta completar la primera rama hacia el sector asignado. |

## Qué **no** incluye esta especificación

- Cambios a la geometría del laberinto o a las posiciones iniciales.
- Cambios en colores, velocidades o estrategias habituales.
- Rutas de apertura permanentes o una revisión general de la IA de fantasmas.
