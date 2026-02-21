# LEGO 3D Editor

Editor de construcción LEGO 3D que corre directamente en el navegador. Un solo archivo HTML, sin instalación, sin backend.

![Three.js](https://img.shields.io/badge/Three.js-0.160-black?logo=threedotjs)
![HTML](https://img.shields.io/badge/Single_File-HTML-orange)
![License](https://img.shields.io/badge/License-MIT-green)

## Uso rápido

1. Descarga `index.html`
2. Ábrelo en tu navegador
3. Listo. Ya puedes construir.

No necesitas instalar nada. No hay dependencias locales, build tools ni servidor.

## Cómo construir

1. **Elige una pieza** en el panel izquierdo (32 tipos disponibles)
2. **Elige un color** en la paleta (26 colores LEGO)
3. **Mueve el mouse** sobre la baseplate verde — verás un "ghost" de la pieza
4. **Haz clic** para colocarla
5. Repite. Las piezas se apilan automáticamente con snap a la rejilla de studs

## Controles

| Acción | Control |
|---|---|
| Colocar pieza | Clic izquierdo sobre la baseplate |
| Seleccionar pieza | Clic izquierdo sobre una pieza colocada |
| Seleccionar varias | Shift + arrastrar |
| Mover pieza | Clic derecho + arrastrar (sobre pieza seleccionada) |
| Rotar pieza | Q (izquierda) / E (derecha) |
| Eliminar | Delete o Backspace |
| Copiar / Pegar | Ctrl+C / Ctrl+V (Cmd en Mac) |
| Deshacer / Rehacer | Ctrl+Z / Ctrl+Y |
| Orbitar cámara | Clic izquierdo + arrastrar (en zona vacía) |
| Zoom | Scroll del mouse |
| Pan cámara | Clic medio + arrastrar |
| Toggle panel | Tab |
| Ayuda | H |

## Funcionalidades

- **32 piezas** — bricks, plates, slopes, tiles, cilindros, conos, arcos
- **26 colores** — paleta auténtica LEGO incluyendo transparentes
- **Snap inteligente** — rejilla de studs con validación de soporte stud↔tube
- **Detección de colisiones** — no permite interpenetración de piezas
- **Undo/Redo ilimitado** — hasta 200 acciones
- **Guardar/Cargar** — localStorage con miniatura automática (hasta 8 slots)
- **Importar/Exportar** — formato JSON para compartir modelos
- **6 plantillas** — Car, Castle, Spaceship, Robot, Cat, Dog
- **Baseplate configurable** — 16×16, 24×24, 32×32, 48×48
- **Microinteracciones** — animación de caída, wobble, partículas al eliminar
- **Sonido** — click sutil al colocar (WebAudio, sin archivos externos)
- **Vibración háptica** — en dispositivos móviles compatibles

## Stack técnico

- **Three.js 0.160** (CDN) — renderizado 3D, PBR, sombras
- **OrbitControls** — navegación de cámara
- **WebAudio API** — sonidos generados proceduralmente
- **localStorage** — persistencia de proyectos
- Nada más. Todo en un archivo HTML de ~1400 líneas.

## Estructura del proyecto

```
lego/
├── index.html   ← La aplicación completa
├── README.md    ← Este archivo
└── SESION.md    ← Bitácora de desarrollo y bugs resueltos
```

## Licencia

MIT
