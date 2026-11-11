# Mapa del código de Apex GT

Guía para trabajar con una IA (DeepSeek, ChatGPT, etc.) sin romper el juego.
Pégale este archivo al inicio de la conversación y después **solo la sección** que quieres cambiar.

## Cómo está armado

- Todo el juego vive en **`index.html`** (~2,650 líneas): HTML, CSS y un solo `<script>`.
- Usa **Three.js r128** (desde cdnjs) y su **GLTFLoader** (desde jsdelivr). No cambiar de versión.
  Ojo: en r128 ya no existe `THREE.Geometry` (solo `BufferGeometry`) y no hay `import`;
  todo se usa con el global `THREE`. Ejemplos de internet con versiones nuevas pueden no funcionar.
- Archivos que el juego carga (no cambiar sus rutas):
  - `assets/sport-car.glb`: modelo 3D del auto.
  - `assets/sfx/engine-mid.wav`, `skid.wav`, `impact.wav`: sonidos de motor, derrape y choque.
- `server/server.mjs`: servidor de salas en línea (está en Render). No se toca al cambiar el juego.

## Secciones de `index.html`

Cada sección empieza con un comentario así: `// ---------- Nombre ----------`.
Busca ese texto en el archivo para encontrarla.

| Sección | Qué contiene | Riesgo al cambiarla |
|---|---|---|
| **Catálogo** | `CARS` (autos: velocidad `vmax` en m/s, `accel`, `grip`, color), `RIVALS` (rivales de IA), `HALF` (medio ancho de pista), `WALL`, `LAP_OPTS` (opciones de vueltas) | Bajo. Ideal para ajustar autos. `vmax` × 3.6 = km/h |
| **Render** | Renderizador, cámara, luces, cielo | Medio |
| **Pistas** | `TRACKS`: las 3 pistas (`costa`, `ciudad`, `desierto`) con puntos `ctrl`, colores, niebla, sol. Funciones `computeTrack`, `sampleAt`, `nearest` | Medio. Cambiar `ctrl` cambia la forma de la pista |
| **Decorado** | Terreno, nubes, árboles, edificios, cactus, rocas, gradas (`buildGround`, `buildSkyExtras`, `buildDecor`, `buildTrack`) | Bajo-medio. Solo se ve, no afecta la física |
| **Obstáculos** | Turbos, aceite, rampas, conos y barriles (`buildObstacles`, `vertical` para saltos) | **Alto**: ver regla de `seeded` abajo |
| **Autos** | Carga del modelo 3D, `makeCar`, ruedas | Medio |
| **Sonido** | Motor, derrape, choques y la **música** del menú (sintetizada, sin derechos) | Bajo |
| **Estado del juego** | Variables globales: `player`, `bots`, `state` (`menu`, `count`, `race`, `done`) | Alto |
| **Red (PeerJS)** | Constantes de red, `buildCars`, `resetRace` | **Alto** |
| **Física** | `roadAssist` (estabilidad), `updatePlayer` (aceleración, giro, nitro, aceite), `updateBots`, `collide`, `trackPlayer` (paredes, vueltas) | Alto. Probar las 3 pistas con el auto más rápido |
| **Minimapa / HUD** | Mapa de la esquina, velocímetro, posición, vueltas | Bajo |
| **Bucle** | `update(dt)` y `loop()`: lo que corre en cada cuadro | Alto |
| **Pantallas** | Menú, resultados, pausa, `startSolo`, `beginRace` | Medio |
| **Sala en línea** | Crear/unirse a sala con PeerJS, mensajes entre jugadores | **No tocar** salvo que sea para el modo en línea |
| **Salas por servidor propio** | `NET_SERVER` (dirección de Render), `createRoomWS`, `joinRoomWS`, `netTick` | **No tocar** |
| **Progreso y desbloqueos** | `UNLOCKS` (qué hay que ganar para cada auto), selector de autos | Bajo |
| **Controles** | Teclado (`KEYMAP`) y botones táctiles | Medio |
| **Control por inclinación** | Sensor del celular, calibración, `TILT_MAX` (24° = giro completo) | Medio |
| **Selector de pista** | Botones de pista y de vueltas | Bajo |
| **Arranque** | Lo que corre al abrir la página | Alto |

## Reglas para no romper nada

1. **Nunca pedir el archivo completo.** Es demasiado largo y la IA lo corta. Pedir los cambios así:
   ```
   BUSCAR: (texto exacto que ya existe)
   REEMPLAZAR POR: (texto nuevo)
   ```
2. No renombrar funciones, variables ni `id` del HTML que ya existen (otras partes los usan).
3. No agregar librerías nuevas ni cambiar la versión de Three.js.
4. **Modo en línea:** no cambiar `NET_SERVER`, `PREFIX`, ni el formato de los mensajes
   (`hello`, `lobby`, `start`, `st`, `snap`, `bye`, `left`). Todos los jugadores deben tener
   la misma versión del juego; después de publicar, que todos recarguen la página.
5. **Obstáculos:** `buildObstacles` usa `seeded()` y llama a `rnd()` en un orden fijo para que
   los obstáculos salgan **iguales para todos** en línea. No usar `Math.random()` para colocar
   obstáculos, y si se agrega uno nuevo, ponerlo **al final** de la función.
6. Los datos guardados usan claves que empiezan con `apexgt-` en `localStorage`
   (progreso, auto, pista, vueltas, nombre). No cambiarlas o se pierde el progreso.
7. Las velocidades están en **m/s** (multiplicar por 3.6 para km/h) y los ángulos en **radianes**.
8. Si la IA no está segura de dónde va un cambio, que pregunte antes de inventarlo.

## Cómo probar antes de publicar

1. Hacer los cambios en una copia, `prueba.html`, en el mismo repositorio.
   Se ve en `https://juegos444-lang.github.io/apex-gt/prueba.html`.
2. Abrir la página, presionar **F12** → **Console**: no debe haber errores en rojo.
3. Jugar una carrera completa. Si se tocó física, autos u obstáculos: probar las 3 pistas.
4. Si se tocó algo de red: abrir dos pestañas, crear sala en una y unirse con la otra.
5. Si todo funciona, copiar el contenido de `prueba.html` a `index.html`.

## Si algo se rompe

En GitHub, abrir `index.html` → **History** → elegir una versión que funcionaba → **Raw** →
copiar todo y pegarlo de vuelta en `index.html`.
