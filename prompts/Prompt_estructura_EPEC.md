# Notas para humanos

> - **E** = Enunciado del problema (qué resolver + por qué importa + contexto dentro de lo provisto)
> - **P** = Perfil del usuario (quién lo usará + necesidades/limitaciones/comportamientos **derivables** del prompt)
> - **E** = Estructura funcional (funciones, flujos, y componentes/arquitectura)
> - **C** = Criterios de validación (umbrales de aceptación y condiciones de entrega)

# PROMPT PARA EL MODELO (COPIAR Y PEGAR)

Usa exclusivamente el texto entre BEGIN_PROMPT y END_PROMPT. Ignora todo lo demás.

BEGIN_PROMPT

## E — Enunciado del problema (qué necesitas resolver, por qué importa y contexto)

### Qué se debe resolver

Generar un **editor LEGO 3D de calidad profesional** en un **único archivo** ⁠ `index.html` ⁠, **autónomo** y ejecutable al abrirse en un navegador moderno, usando **exclusivamente Three.js** y sus módulos oficiales desde CDN (incluyendo ⁠ `examples/jsm` ⁠ si es necesario).

### Por qué importa (según el prompt)

El resultado debe ofrecer una experiencia de creación LEGO 3D **premium**:

- **Precisa** (rejilla de studs, snap exacto, soporte stud↔️tube)
- **Fluida** (interacción suave en equipos de gama media)
- **Visualmente atractiva** (PBR, sombras suaves, brillo/transparencia sutil)
- **Intuitiva** (mouse/táctil, UI plegable, atajos visibles, ayuda rápida)
- Con sensación “**mágica**” de juguete físico (microinteracciones, sonido local, vibración háptica)

### Contexto de entrega y restricciones (condiciones no negociables)

- **Respuesta/Salida:** responder **solo** con el HTML responsive final completo (sin explicaciones, sin markdown, sin texto extra).
- **Ejecución inmediata:** debe funcionar al guardar como ⁠ `index.html` ⁠ y abrir en navegador.
- **Completitud:** sin TODOs, placeholders, bloques incompletos ni funciones vacías.
- **Dependencias:** solo módulos CDN oficiales de Three.js; prohibidas librerías externas distintas a Three.js.
- **Rendimiento:** interacción suave en equipos de gama media; reutilización de geometrías/materiales cuando sea posible; raycasting eficiente y actualización incremental de escena/UI.

---

## P — Perfil del usuario (quién lo usará, necesidades, limitaciones y comportamientos esperados)

### Quién lo usará

Usuario final que construye modelos LEGO en un navegador moderno, interactuando mediante:

- **Mouse** (incluye acciones con botón derecho y teclado)
- **Pantalla táctil**
- **Teclado** (atajos Q/E y modificadores como Shift)

### Necesidades clave

- Construcción precisa y confiable: **snap** exacto, rejilla por studs y niveles de altura, soporte real stud ↔️tube, sin colisiones.
- Feedback antes de colocar: **ghost piece** con estado válido/inválido.
- Edición eficiente: seleccionar (simple y múltiple), copiar/pegar/eliminar, recolorear, mover en plano, rotar con incrementos configurables.
- Navegación 3D cómoda: cámara orbit con zoom/pan/orbit, opción de enfoque automático.
- Organización y continuidad de trabajo: guardar/cargar en localStorage con miniatura, importar/exportar JSON con validación.
- Curva de aprendizaje asistida: plantillas listas (coche, castillo, nave espacial, robot) y ejemplos (gato, perro, carro, robot) con armado paso a paso.
- UI clara y no intrusiva: panel lateral plegable y barra superior ocultable; atajos visibles y ayuda rápida; diseño responsivo.

### Limitaciones y expectativas operativas (derivables)

- Debe rendir bien en **equipos de gama media**.
- No depende de recursos externos (imágenes, audios, fuentes, modelos, APIs) fuera de Three.js (sonido debe generarse localmente con WebAudio).
- Debe funcionar “out-of-the-box” sin instalación, build tools ni backend.

---

## E — Estructura funcional (cómo funciona el sistema, flujos principales y arquitectura de componentes)

### Flujos principales

1. **Elegir pieza y color**
   - Buscar/filtrar en la paleta lateral por **categoría** y **tamaño**
   - Seleccionar color (mínimo 20+ colores tipo LEGO, paleta auténtica aproximada)

2. **Colocar pieza en escena (con validación)**
   - Arrastrar desde paleta a escena + clic para colocar
   - Previsualizar con **ghost piece** (válido/inválido)
   - Ajuste por **snap** a rejilla de studs (X/Z) y niveles “plate/layer”
   - Validar soporte stud↔️tube al apilar
   - Detectar colisiones (no interpenetración)

3. **Editar y transformar**
   - Rotar con **Q/E** (pasos configurables, toggle de incrementos 15°/45°/90°)
   - Arrastrar con botón derecho para mover en plano
   - Recoloración instantánea del ladrillo seleccionado
   - Selección simple y multiselección (Shift + arrastrar para caja)
   - Acciones: copiar, pegar, eliminar
   - Undo/redo ilimitado (historial robusto)

4. **Configurar escena**
   - Baseplate con tamaños conmutables (varios presets)
   - Clear Scene con confirmación
   - Toggle snap on/off (ajuste de rejilla)

5. **Navegar**
   - Cámara orbit suave con soporte mouse + táctil
   - Zoom/pan/orbit cómodos
   - Enfoque automático opcional al seleccionar

6. **Persistir, compartir y recuperar**
   - Guardar/cargar proyectos en localStorage
   - Generar miniatura automática al guardar (captura de canvas)
   - Importar/exportar JSON
   - Manejo de errores y validación de formato JSON

7. **Cargar contenido guiado**
   - Galería de plantillas: coche, castillo, nave espacial, robot (mínimo 4)
   - Ejemplos predeterminados: animales (gato, perro) y cosas (carro, robot) armables paso a paso

8. **Sensación premium (microinteracciones)**
   - Caída breve al colocar, tambaleo suave, partículas al eliminar
   - Sonido de colocación sutil generado localmente (WebAudio)
   - Vibración háptica cuando esté disponible (navigator.vibrate)

### Componentes y arquitectura interna

Código organizado por módulos lógicos dentro del mismo HTML:

- **estado global**
- **motor de rejilla**
- **catálogo de piezas**
- **render**
- **input**
- **historial**
- **persistencia**
- **UI**

Calidad de código:

- Nombres claros, comentarios breves de alto valor
- Sin duplicación innecesaria
- Manejo defensivo de errores
- Consistencia en unidades y convenciones de coordenadas

---

## C — Criterios de validación (cómo sabemos que funciona, métricas de éxito y umbrales)

### Criterios de aceptación

Se considera correcto solo si:

1. Permite construir estructuras válidas por snap stud/tube sin colisiones.
2. Incluye biblioteca amplia de piezas y 20+ colores.
3. Implementa selección múltiple, copiar/pegar/eliminar y undo/redo ilimitado.
4. Guarda/carga con miniatura en localStorage y soporta JSON import/export.
5. Provee UX fluida con cámara orbit, ghost placement y panel plegable.
6. Entrega un ⁠ `index.html` ⁠ único, totalmente funcional, sin dependencias externas fuera de Three.js CDN.

### Umbrales de entrega (condiciones de verificación)

- La respuesta debe ser **solo** el HTML final completo (sin texto adicional).
- Debe ejecutarse inmediatamente al abrir en navegador (sin build tools, sin backend).
- No debe contener TODOs, placeholders, bloques incompletos ni funciones vacías.
- Debe usar únicamente Three.js y módulos oficiales desde CDN (core + addons oficiales).

Devuelve únicamente el código HTML final completo.

END_PROMPT
