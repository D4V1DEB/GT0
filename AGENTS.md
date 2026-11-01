# GT0 — Bitácora del proyecto

> **Regla para el agente:** al terminar cada prompt, añade una entrada en *Registro* y actualiza
> *Estado actual*. No borres entradas viejas: son el contexto que evita repetir trabajo.
> Todo en español. Sin dependencias nuevas, sin build, sin assets externos.

## Qué es

Juego de automovilismo **para navegador** ambientado en Perú. Campeonato ficticio con circuitos
inspirados en geografía peruana. Tres principios innegociables:

1. **Realismo** — el coche y las reglas deben ser creíbles.
2. **Accesibilidad** — sin volante ni conocimientos avanzados; teclado y listo.
3. **Identidad peruana** — escenarios, nombres y ambientación propios.

Hoy esto es un *vertical slice*, no el juego completo: una pista, un coche, vueltas cronometradas.

## Stack

- **three.js r149** (`lib/three.min.js`, build UMD local) → se abre con doble clic, sin servidor,
  sin internet, sin bundler. Si algún día hace falta un módulo moderno, hay que servir por HTTP.
- Un solo archivo: `index.html` (HTML + CSS + JS en línea).
- Texturas y geometría 100 % procedurales (canvas / `BufferGeometry`), cero archivos de imagen.
- Sin sombras dinámicas (sombra falsa bajo el coche) por rendimiento.

## Estructura

```
index.html        juego completo (escena, física, HUD, cámara)
lib/three.min.js  three.js r149 (no tocar)
AGENTS.md         este archivo
```

## Estado actual (2026-11-01)

Jugable de principio a fin: `Circuito Costa Verde` (~2,19 km, desierto costeño).

- **Física arcade** en `updateCar()`: vector de velocidad descompuesto en longitudinal/lateral;
  el agarre lateral se amortigua (`exp(-grip*dt)`), menos con freno de mano o fuera de pista.
  Todo el *tuning* vive ahí: `maxF` 70 m/s (≈245 km/h), motor 13,5 m/s², freno 34 m/s²,
  `grip` 9,5 en pista / 3,2 fuera / 2,4 con freno de mano.
- **Cámaras** con `C`: persecución (suavizada, FOV según velocidad) y primera persona a 1,5 m.
- **Cronometraje** por índice de muestra sobre la curva (`checkLap`): la vuelta solo cuenta al
  cruzar meta hacia adelante *después* de pasar el 55 % del trazado. `R` recoloca en pista.
- **HUD** HTML: vuelta, tiempo actual, mejor, última, velocímetro y aviso de fuera de pista.
- Escenario: dunas/cerros truncados, rocas y cactus en `InstancedMesh`, pórtico con cartel,
  tribuna, meta a cuadros.
- **Rendimiento:** ~60 draw calls, sin post-proceso, `pixelRatio` máximo 2.

## Cómo probarlo

Doble clic en `index.html`. Controles: `W` acelerar, `S` frenar/reversa, `A`/`D` girar,
`ESPACIO` freno de mano, `C` cámara, `R` volver a pista.

## Pendiente (en orden sugerido)

1. **Tacto de conducción**: falta que un humano lo conduzca y ajuste `updateCar()`.
2. Separar en módulos (`track.js`, `car.js`, `hud.js`) — solo cuando haya una segunda pista.
3. Más circuitos por región (sierra, selva, costa) reutilizando el generador de cinta.
4. IA rival + reglamento (banderas, salida, penalizaciones) → el cronómetro ya sirve de base.
5. Menú de campeonato y categorías.
6. Sonido (motor, viento) y volante/mando.
7. Multijugador — solo después de separar estado de simulación de la vista.

## Decisiones y trampas conocidas

- `dt` se recorta a `[0, 0.05]`: un `dt` negativo (reloj que retrocede) producía NaN en la física.
- El cronómetro usa `THREE.Clock` (tiempo real). Si la pestaña se suspende, el tiempo de vuelta
  se infla; pendiente pausar por `visibilitychange`.
- Los UV de la carretera van en `dist/9`; la textura de asfalto se repite cada 9 m.
- `renderer.outputEncoding = sRGBEncoding` con materiales `MeshLambert` (barato).
- three.js es r149 a propósito: es la última con build UMD, y eso permite `file://`.

## Registro

- **2026-11-01** — Nace el proyecto. Vertical slice del Circuito Costa Verde: trazado por spline
  cerrada (`CatmullRomCurve3`), cinta de asfalto con líneas de borde en la textura, física arcade,
  dos cámaras, cronometraje de vueltas, HUD, paisaje costeño y zona de meta.
  Verificado con piloto automático (2+ vueltas sin salirse ni NaN) y renders forzados de las tres
  vistas. Corregido en el proceso: `dt` negativo, capó tapando la cámara, cartel espejado,
  panel de controles sin `position:absolute`, coche sin colocar hasta el primer frame.
  Repo movido a `GT0`.
