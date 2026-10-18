# SPEC 03 — Power Pellets y fantasmas vulnerables

> **Estado:** Implemented
> **Depende de:** SPEC 01, SPEC 02
> **Fecha:** 2026-10-18
> **Objetivo:** Añadir cuatro Power Pellets que permitan a Pac-Man comer temporalmente a los fantasmas.

## Alcance

**Incluye:**

- Colocar Power Pellets en las celdas transitables `(1,1)`, `(26,1)`, `(1,29)` y `(26,29)`, reemplazando el punto normal de cada celda.
- Hacer que cada pellet cuente como coleccionable para completar el nivel y otorgue 50 puntos.
- Activar el poder durante 8 segundos al comer un pellet; comer otro durante el efecto renueva los 8 segundos y reinicia la secuencia de bonificación.
- Mientras el poder esté activo, permitir que Pac-Man coma fantasmas en vez de perder una vida al colisionar con ellos.
- Al ser comido, devolver cada fantasma a su posición inicial y reiniciar su estado de salida y patrulla según las reglas existentes.
- Otorgar 200, 400, 800 y 1600 puntos por fantasma comido en una activación, en ese orden. Mantener la bonificación en 1600 para fantasmas adicionales comidos durante la misma activación.
- Cancelar el poder y reiniciar la secuencia de bonificación cuando Pac-Man pierde una vida. Los pellets ya comidos permanecen consumidos durante esa partida.
- Dibujar los pellets y mostrar los fantasmas vulnerables en azul, haciéndolos parpadear durante los últimos 2 segundos del poder.

**Fuera de alcance (para futuras especificaciones):**

- Cambiar la geometría o las dimensiones del laberinto.
- Añadir otros tipos de poderes, vidas o coleccionables.
- Cambiar las estrategias, velocidades o colores habituales de los fantasmas fuera del indicador temporal de vulnerabilidad.
- Guardar el progreso entre partidas o sesiones.

## Modelo de datos

En `src/js/maze.js`, el carácter `o` de `MAZE_STR` representa un Power Pellet y se convierte en el valor de tile `4`. Las cuatro coordenadas de colocación reemplazan puntos normales.

En `src/js/game.js`, el estado de partida incluye:

```js
powerUntil: 0, // instante de expiración en milisegundos; 0 significa inactivo
ghostEatChain: 0, // fantasmas comidos en la secuencia actual, limitado a 4
```

`dotsRemaining` cuenta tanto los puntos normales (`2`) como los Power Pellets (`4`). Comer un pellet lo cambia a vacío, suma 50 puntos, establece `powerUntil` a ocho segundos después del instante actual y reinicia `ghostEatChain` a `0`. La duración se mide con tiempo real, no con una cantidad fija de fotogramas.

`ghostEatChain` selecciona la bonificación `[200, 400, 800, 1600]`. Después de alcanzar cuatro, las siguientes capturas en esa activación siguen otorgando 1600 puntos.

## Plan de implementación

1. Añadir el tile `4` y las cuatro posiciones en `src/js/maze.js`; contar y consumir esos tiles en `src/js/game.js`; dibujarlos en `src/js/render.js`. Comprobar manualmente que cada uno se ve, otorga 50 puntos y reduce `dotsRemaining` una sola vez.
2. Añadir el temporizador de poder a `src/js/game.js`. Comer un pellet activa o renueva los 8 segundos y reinicia la secuencia de bonificación. Comprobar que el estado inicial no tiene poder y que cada pellet inicia o renueva el efecto.
3. Cambiar el manejo de colisiones en `src/js/game.js`: sin poder, conservar la pérdida de vida actual; con poder, sumar la bonificación correspondiente y devolver el fantasma a su posición inicial, reiniciando `openingActive` y `patrolIndex` cuando corresponda. Al perder una vida, cancelar poder y bonificación sin restaurar los pellets consumidos. Comprobar ambos tipos de colisión y la reaparición del fantasma.
4. En `src/js/render.js`, mostrar los fantasmas azules mientras el poder esté activo y hacerlos parpadear en los últimos 2 segundos. Comprobar visualmente la activación, el parpadeo y la expiración.

## Criterios de aceptación

- [ ] Hay cuatro Power Pellets en `(1,1)`, `(26,1)`, `(1,29)` y `(26,29)`, sustituyendo los puntos normales de esas celdas.
- [ ] Cada Power Pellet se dibuja, se puede consumir, suma 50 puntos y reduce `dotsRemaining` en uno.
- [ ] Comer un Power Pellet activa el poder durante 8 segundos; comer otro durante el efecto renueva el tiempo y reinicia la secuencia de bonificación.
- [ ] Si Pac-Man colisiona con un fantasma sin poder activo, pierde una vida como antes.
- [ ] Si Pac-Man colisiona con un fantasma con poder activo, no pierde una vida; el fantasma vuelve a su posición inicial y reanuda su salida y estrategia.
- [ ] Las capturas sucesivas durante una activación suman 200, 400, 800 y 1600 puntos, y las capturas adicionales suman 1600 cada una.
- [ ] Perder una vida cancela el poder y reinicia la bonificación, pero no restaura los pellets ya consumidos.
- [ ] Los fantasmas vulnerables se ven azules y parpadean durante los últimos 2 segundos; al expirar el poder recuperan su apariencia habitual.
- [ ] Una partida nueva restaura los cuatro Power Pellets.
- [ ] El juego funciona al abrir `src/index.html` y no presenta errores en la consola.

## Decisiones

- **Sí:** Colocar cuatro pellets en las esquinas transitables existentes. No es necesario cambiar el laberinto.
- **Sí:** Sustituir puntos normales por pellets que valen 50 puntos y cuentan para terminar el nivel.
- **Sí:** Usar un efecto temporizado de 8 segundos; un nuevo pellet renueva el tiempo y reinicia la secuencia.
- **Sí:** Devolver los fantasmas comidos a sus posiciones iniciales y reanudar su salida definida en SPEC 02.
- **Sí:** Limitar la bonificación progresiva a 1600 puntos por captura tras el cuarto fantasma de una activación.
- **Sí:** Cancelar el poder al perder una vida, manteniendo consumidos los pellets durante la partida.
- **No:** Cambiar las estrategias habituales de los fantasmas ni añadir otros poderes.

## Riesgos

| Riesgo | Mitigación |
| --- | --- |
| Los Power Pellets podrían no contarse como puntos restantes y permitir completar el nivel antes de recogerlos. | Contar explícitamente los tiles `2` y `4` en la inicialización y disminuir el total al consumir cualquiera de ellos. |
| El temporizador puede expirar mientras el navegador suspende la pestaña. | Comparar el instante de expiración con el tiempo real al reanudar la actualización; el poder no se extiende por fotogramas omitidos. |
| Un fantasma reaparecido podría conservar estado de su estrategia anterior. | Restaurar su posición, `openingActive` y `patrolIndex` junto con su dirección inicial. |

## Qué **no** incluye esta especificación

- Cambios a la geometría o dimensiones del laberinto.
- Otros poderes, coleccionables o cambios a la IA habitual de los fantasmas.
- Persistencia del progreso entre partidas o sesiones.
