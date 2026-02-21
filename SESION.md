# Sesión: LEGO 3D Editor

## Objetivo

Generar un **editor LEGO 3D de calidad profesional** en un único archivo `index.html`, autónomo y ejecutable en navegador moderno, usando exclusivamente **Three.js** desde CDN.

---

## Requisitos principales

- Archivo único `index.html`, sin dependencias externas fuera de Three.js CDN
- Ejecución inmediata al abrir en navegador (sin build tools, sin backend)
- Experiencia premium: PBR, sombras suaves, microinteracciones, sonido WebAudio, vibración háptica
- Rejilla de studs, snap exacto, soporte stud↔tube, detección de colisiones
- UI responsiva con panel lateral plegable y barra de herramientas

---

## Resultado: Funcionalidades implementadas

### Piezas y colores
- **32 tipos de piezas** en 5 categorías: Bricks, Plates, Slopes, Tiles, Special
- **26 colores** tipo LEGO auténticos (incluyendo 4 transparentes)
- Previews renderizadas en 3D para cada pieza en el sidebar

### Motor 3D (Three.js)
- Iluminación PBR: directional, fill, hemisphere, ambient
- Sombras PCFSoft con shadow map 2048×2048
- Tone mapping ACES Filmic
- Fog exponencial para profundidad
- OrbitControls con damping suave

### Sistema de construcción
- **Snap** a rejilla de studs (X/Z) y niveles de altura (plate layers)
- **Ghost piece** con indicador visual verde (válido) / rojo (inválido)
- **Detección de colisiones** (no interpenetración)
- **Validación de soporte** stud↔tube al apilar
- Auto-cálculo de Y (apila sobre piezas existentes)

### Edición
- Selección simple (clic) y múltiple (Shift+drag para box select)
- Copiar/pegar/eliminar piezas
- Rotación con Q/E (pasos configurables: 15°/45°/90°)
- Recoloración instantánea del brick seleccionado
- Arrastrar con botón derecho para mover en plano
- **Undo/redo ilimitado** (hasta 200 entradas)

### Persistencia
- Guardar/cargar en localStorage (hasta 8 slots con miniatura automática)
- Importar/exportar JSON con validación de formato

### Templates predefinidos
- 6 plantillas listas: Car, Castle, Spaceship, Robot, Cat, Dog

### Configuración de escena
- Baseplate con tamaños conmutables: 16×16, 24×24, 32×32, 48×48
- Toggle snap on/off
- Toggle auto-focus al seleccionar

### Microinteracciones
- Animación de caída al colocar pieza
- Wobble sutil post-colocación
- Partículas al eliminar piezas
- Sonido de colocación generado con WebAudio API
- Vibración háptica (navigator.vibrate) en dispositivos compatibles

### Atajos de teclado
| Tecla | Acción |
|---|---|
| Q / E | Rotar pieza CCW / CW |
| Delete / Backspace | Eliminar seleccionados |
| Ctrl+Z (Cmd+Z en Mac) | Undo |
| Ctrl+Y / Ctrl+Shift+Z | Redo |
| Ctrl+C / Ctrl+V | Copiar / Pegar |
| Shift+Drag | Box select múltiple |
| Click derecho+drag | Mover pieza en plano |
| Tab | Toggle sidebar |
| H | Toggle ayuda |
| Escape | Deseleccionar todo |
| Mouse wheel | Zoom |
| Middle-click drag | Pan cámara |

---

## Problemas encontrados y correcciones

### Problema 1: Límite de tokens de salida

**Error:** `API Error: Claude's response exceeded the 32000 output token maximum`

**Causa:** El archivo completo excedía el límite de 32000 tokens de salida de Claude Code.

**Solución:** Se cambió la estrategia de generación. En vez de intentar generar todo en una sola respuesta, se escribió el archivo completo usando la herramienta `Write` que no tiene ese límite.

---

### Problema 2: Previews de piezas en blanco

**Síntoma:** Todas las tarjetas de piezas en el sidebar mostraban un canvas blanco sin renderizar.

**Causa:** La función `renderPiecePreview()` creaba un nuevo `WebGLRenderer` para cada una de las 32+ piezas. Los navegadores limitan los contextos WebGL activos a ~16. Los contextos excedentes fallaban silenciosamente, dejando los canvas en blanco.

**Solución:** Se reemplazó por un **único renderer offscreen compartido** (`UI._prevRenderer`). Renderiza cada pieza en un canvas oculto y luego copia los pixels al canvas visible con `ctx.drawImage()`.

```javascript
// ANTES (32+ contextos WebGL - falla silenciosamente)
renderPiecePreview(cvs, piece) {
    const prevRenderer = new THREE.WebGLRenderer({canvas: cvs});
    // ...render...
    prevRenderer.dispose(); // El contexto ya se perdió
}

// DESPUÉS (1 solo contexto compartido)
renderPiecePreview(cvs, piece) {
    if (!UI._prevRenderer) {
        UI._prevCanvas = document.createElement('canvas');
        UI._prevRenderer = new THREE.WebGLRenderer({canvas: UI._prevCanvas});
        // ... setup una sola vez ...
    }
    // render al canvas oculto, luego copiar
    UI._prevRenderer.render(UI._prevScene, UI._prevCam);
    const ctx = cvs.getContext('2d');
    ctx.drawImage(UI._prevCanvas, 0, 0);
}
```

---

### Problema 3: No se podían colocar piezas en el lienzo

**Síntoma:** Al hacer clic en la baseplate (zona verde), no se colocaba ninguna pieza. El ghost piece no aparecía.

**Causa:** El `#canvas-container` usaba `flex:1` con `margin-top:44px`, pero el sidebar era `position:absolute` y se superponía al canvas. Las coordenadas del mouse no correspondían con la geometría 3D visible.

**Solución:** Se cambió el canvas container a posicionamiento absoluto con coordenadas explícitas que respetan el sidebar:

```css
/* ANTES */
#canvas-container { flex:1; position:relative; margin-top:44px }

/* DESPUÉS */
#canvas-container { position:absolute; top:44px; left:280px; right:0; bottom:28px }
#sidebar.hidden ~ #canvas-container { left:0 }
```

También se añadió `ResizeObserver` para recalcular dimensiones automáticamente y se removió el `!important` en los estilos del canvas que interferían con Three.js.

---

### Problema 4: Copy/Paste no funcionaba

**Síntoma:** Ctrl+C/V no hacía nada.

**Causas múltiples:**
1. En Mac, se usa `Cmd` (`metaKey`) en vez de `Ctrl` (`ctrlKey`)
2. La función paste dependía de `State.ghostData` que solo existe si el mouse está sobre la escena
3. No había feedback visual cuando no había nada copiado

**Solución:**
- Añadido soporte para `e.metaKey` (Mac Cmd)
- Paste ahora calcula la posición Y automáticamente con `Grid.getPlacementY()`
- Mensajes toast informativos: "Nothing to paste", "Could not paste - collision"

---

### Problema 5: hasSupport demasiado estricto

**Síntoma:** Algunas piezas no se colocaban en la baseplate por fallar la validación de soporte.

**Causa:** La comparación `y < PLATE_H * 0.01` era demasiado sensible a errores de punto flotante.

**Solución:** Se relajó el umbral a `y < 0.05`.

---

### Fix adicional: Thumbnails al guardar

Se añadió `preserveDrawingBuffer: true` al renderer principal para que `canvas.toDataURL()` funcione correctamente al generar miniaturas para los save slots.

---

## Arquitectura del código

Todo dentro de un único `index.html` (~1387 líneas), organizado en módulos lógicos:

| Módulo | Líneas aprox. | Responsabilidad |
|---|---|---|
| CSS Styles | 1-134 | UI completa, responsiva, tema oscuro |
| HTML Structure | 136-249 | Toolbar, sidebar, canvas, modals |
| State | 264-271 | Estado global de la aplicación |
| Colors | 274-288 | Paleta de 26 colores LEGO |
| Piece Catalog | 291-326 | Definición de 32 piezas |
| Geometry Cache | 328-389 | Cache de geometrías y materiales |
| Three.js Setup | 391-436 | Scene, camera, renderer, lights |
| Grid & Baseplate | 438-561 | Rejilla, snap, studs, colisiones |
| Brick Mesh Builder | 563-615 | Generación de meshes por pieza |
| Ghost Piece | 617-632 | Preview de colocación |
| Placement | 634-664 | Lógica de colocar/remover piezas |
| Animations | 666-721 | Drop, wobble, partículas |
| History | 723-831 | Undo/redo, copy/paste, delete |
| Selection | 833-863 | Simple y multi-selección |
| Input | 865-1004 | Mouse, teclado, pointer events |
| Audio | 1006-1032 | WebAudio para sonidos |
| Haptics | 1034-1037 | Vibración háptica |
| Persistence | 1039-1113 | Save/load, import/export JSON |
| Scene Management | 1115-1129 | Clear scene |
| Templates | 1131-1216 | 6 plantillas predefinidas |
| UI | 1218-1330 | Inicialización de UI, previews, filtros |
| Render Loop | 1350-1387 | Animation loop, resize, modals |

## Dependencias externas

- `three@0.160.0` (Three.js core) via jsDelivr CDN
- `OrbitControls` desde `three/examples/jsm/` via jsDelivr CDN
- Nada más. Sonido, fuentes, e interacciones son 100% locales.
