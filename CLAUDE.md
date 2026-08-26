# CLAUDE.md

Este archivo proporciona orientación a Claude Code (claude.ai/code) cuando se trabaja con código en este repositorio.

## Descripción General

**Tetris** es una implementación del juego clásico en JavaScript vanilla (ES6+), usando HTML5 Canvas y CSS3, sin dependencias externas ni proceso de build. Solo abrir `index.html` en un navegador y jugar.

## Estructura del Proyecto

```
03-tetris/
├── index.html     # DOM: canvas, panel lateral (score, level, next), overlay (pausa/game over)
├── style.css      # Dark theme retro arcade (flexbox, backdrop-filter, variables CSS)
├── game.js        # Lógica completa (~305 líneas)
└── README.md      # Documentación del usuario
```

## Cómo Ejecutar

### Opción 1: Servidor local (recomendado)
```bash
# Python 3
python3 -m http.server 8000

# Node.js
npx serve .

# PHP
php -S localhost:8000
```
Luego abre `http://localhost:8000` en el navegador.

### Opción 2: Abrir directo
```bash
start index.html       # Windows
open index.html        # macOS
xdg-open index.html    # Linux
```

## Arquitectura de Alto Nivel

### Modelo del Tablero (`game.js`)

- **Matriz 10×20**: array 2D donde cada celda = `0` (vacía) o índice de color (1–7) que identifica la pieza.
- **Piezas**: 7 tipos (I, O, T, S, Z, J, L), cada una almacenada como matriz cuadrada en `PIECES`.
- **Estado actual**: objeto `current` con `{ type, shape, x, y }`.
- **Vista previa**: objeto `next` con la siguiente pieza.

### Game Loop

```
requestAnimationFrame(loop)
  ├─ Acumula tiempo delta (dt)
  ├─ Si dt ≥ dropInterval → baja pieza o fija (lockPiece)
  ├─ Dibuja tablero + ghost + pieza actual
  └─ Loop siguiente
```

La velocidad de caída se calcula como `max(100, 1000 - (level - 1) * 90)` ms.

### Funciones Clave

| Función | Propósito |
|---------|-----------|
| `createBoard()` | Matriz vacía 20×10 |
| `randomPiece()` | Genera pieza aleatoria con x centrado |
| `collide(shape, x, y)` | Detecta si la forma choca con bordes o bloques fijos |
| `rotateCW(shape)` | Rota 90° horario (transpuesta + reverso de filas) |
| `tryRotate()` | Intenta rotar; si falla, usa wall kicks (±1, ±2 columnas) |
| `merge()` | Fija la pieza actual en el tablero |
| `clearLines()` | Detecta y elimina filas completas (de abajo hacia arriba) |
| `ghostY()` | Proyecta dónde aterrizará la pieza actual (hardDrop visual) |
| `hardDrop()` | Caída instantánea; suma 2 puntos por celda recorrida |
| `softDrop()` | Bajada acelerada; suma 1 punto por fila |
| `spawn()` | Mueve `next` a `current`, genera nueva `next`. Dispara Game Over si hay colisión |
| `draw()` | Renderiza grid + bloques fijos + ghost + pieza actual |
| `loop(ts)` | Bucle del juego (timestamp-driven) |

### Sistema de Puntuación

- **Líneas únicas**: 100 × nivel
- **Dobles**: 300 × nivel
- **Triples**: 500 × nivel
- **Tetris (4 líneas)**: 800 × nivel
- **Soft drop**: +1 por fila bajada
- **Hard drop**: +2 por celda recorrida

El nivel sube cada 10 líneas.

### Canvas Rendering

- **Canvas principal** (`#board` 300×600 px): tablero 10×20 bloques de 30×30 px.
- **Next canvas** (`#next-canvas` 120×120 px): vista previa centrada.
- **Efectos visuales**: highlight blanco en la parte superior de cada bloque, alpha=0.2 para ghost piece.

## Controles

| Tecla | Acción |
|-------|--------|
| ← / → | Mover |
| ↑ / X | Rotar |
| ↓ | Soft drop |
| Espacio | Hard drop |
| P | Pausa |

## Parámetros Tuneables

En `game.js`, modificar estas constantes:

| Constante | Defecto | Notas |
|-----------|---------|-------|
| `COLS` | 10 | Ancho del tablero |
| `ROWS` | 20 | Alto del tablero |
| `BLOCK` | 30 | Tamaño de cada celda (px). Actualizar también `<canvas>` en HTML |
| `COLORS` | Array 7 | Colores hex por tipo de pieza |
| `LINE_SCORES` | `[0,100,300,500,800]` | Puntos por 1, 2, 3, 4 líneas |

## Desarrollo

### Agregar una Mecánica Nueva

1. **Entender el game loop**: `loop()` acumula `dt` y dispara eventos cada `dropInterval` ms.
2. **Acceso al tablero**: usar `board[row][col]`.
3. **Acceso a la pieza actual**: `current.shape`, `current.x`, `current.y`, `current.type`.
4. **Detectar colisiones**: llamar a `collide(shape, x, y)`.
5. **Renderizar**: agregar código en `draw()` con Canvas 2D API.
6. **Actualizar HUD**: llamar a `updateHUD()` si cambios score/lines/level.

### Workflow Típico

```
1. Modificar constante o lógica en game.js
2. Guardar archivo
3. Refrescar navegador (F5) en http://localhost:8000
4. Jugar para verificar el comportamiento
```

### Puntos Delicados

- **Wall kicks**: `tryRotate()` prueba 5 desplazamientos `[0, -1, 1, -2, 2]` antes de descartar.
- **Limpieza de líneas**: recorre de abajo arriba (`ROWS - 1` → `0`) para evitar saltarse filas.
- **Spawn y Game Over**: si la pieza nueva ya colisiona, `endGame()` se dispara inmediatamente.
- **Ghost piece**: se calcula cada frame en `ghostY()` buscando la y más baja sin colisión.
- **Pausa**: cancela el `animId` y rehace el loop cuando se reanuda (reset de `lastTime`).

## Nota Técnica

**Sin transpilador ni bundler**: solo ES6+ nativo (arrow functions, const/let, spread, Array.from, template literals). Los navegadores modernos los soportan completamente.
