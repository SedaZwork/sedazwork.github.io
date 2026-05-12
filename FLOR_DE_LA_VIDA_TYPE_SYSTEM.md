# Sistema integral para un generador tipográfico basado en la Flor de la Vida

## 1) Fundamentos y reglas geométricas

### 1.1 Geometría base
- **Unidad primaria**: círculo de radio `r`.
- **Retícula madre**: red hexagonal/triangular obtenida por empaquetado de círculos con centros separados por `r`.
- **Flor de la Vida mínima útil**: 19 círculos (1 central + 6 + 12) para repertorio latino básico; expandible por coronas.
- **Sistema de coordenadas**:
  - Cartesiano para exportación tipográfica.
  - Axial/cúbico hexagonal para lógica constructiva (`q,r,s` con `q+r+s=0`).

### 1.2 Proporciones tipográficas derivadas
Definir altura y anchos sobre múltiplos de `r`:
- `xHeight = 4r`
- `capHeight = 6r`
- `ascender = 7r`
- `descender = -2r`
- `baseline = 0`
- `overshoot_round = 0.08r a 0.18r` (paramétrico)

### 1.3 Ejes y simetrías
- Ejes principales: 0°, 60°, 120°.
- Ejes secundarios: 30°, 90°, 150° (para modulaciones caligráficas).
- Simetrías permitidas:
  - Reflexión bilateral para glifos simétricos.
  - Rotación de orden 2/3/6 para signos geométricos.

### 1.4 Curvas canónicas
- Arcos de circunferencia centrados en nodos de la retícula.
- Bézier cúbicas aproximando arco solo en exportación (si el entorno no admite arco real).
- Splines de transición para unión orgánica entre arcos no concéntricos.

---

## 2) Normas de tangencialidad

### 2.1 Condiciones C0/C1/C2
- **C0**: continuidad posicional obligatoria en toda unión.
- **C1**: continuidad tangencial obligatoria en contornos visibles.
- **C2**: opcional en estilos orgánicos/variables para suavidad de peso.

### 2.2 Regla de unión tangente entre círculos
Dados círculos `C1(c1,r1)` y `C2(c2,r2)`:
- Punto de unión `p` válido si:
  - `p` pertenece a ambas primitivas de construcción (arco o curva puente).
  - vectores tangentes `t1(p)` y `t2(p)` son colineales y mismo sentido local.
- Para puentes Bézier:
  - Manecillas (`h1`,`h2`) alineadas con normales de círculos en los puntos de contacto.
  - Longitud de manecilla base `k = alpha * r`, con `alpha` en `[0.45, 0.62]`.

### 2.3 Colisión y clearance
- Espesor mínimo interno: `counterMin = max(0.7*stroke, 0.35r)`.
- Separación mínima entre contornos no conectados: `gapMin = 0.2r`.
- Intersecciones permitidas solo si se resuelven por operación booleana explícita.

---

## 3) Gramática constructiva (shape grammar)

### 3.1 Alfabeto de primitivas
- `O`: círculo completo.
- `A(theta1, theta2)`: arco.
- `S(p1,p2,w)`: segmento estructural (esqueleto).
- `T(p,dir,len)`: tangente constructiva.
- `B(p1,h1,h2,p2)`: puente Bézier.
- `K(region)`: contraforma.

### 3.2 Reglas de reescritura
- `O -> A + A` (descomposición por sectores funcionales).
- `A + A (tangentes) -> loop` (cierre de ojo/contraforma).
- `S -> offset(stroke/2)` (expansión de trazo).
- `loop - K -> glyphPart` (sustracción de contraforma).
- `glyphPart + glyphPart -> glyph` (composición final).

### 3.3 Estados de construcción
1. **Skeleton** (estructura)
2. **Stroke** (expansión)
3. **Boolean** (limpieza)
4. **Optical** (correcciones)
5. **Hint/Export** (salida)

---

## 4) Métodos de construcción tipográfica

### 4.1 Método radial (ideal para O, C, G, Q)
1. Seleccionar nodo central.
2. Construir anillo base con radio `R = n*r`.
3. Recortar sector angular según glifo.
4. Añadir terminales tangenciales (rectos o gota).

### 4.2 Método de costillas hexagonales (H, E, F, A, M, N)
1. Definir columna vertebral en líneas 90° o 60°/120°.
2. Añadir travesaños en nodos de intersección hex.
3. Redondear encuentros con arcos `0.5r` a `1.2r`.

### 4.3 Método caligráfico tangencial
1. Trazar esqueleto con dirección de pluma virtual.
2. Modulación de grosor por función angular:
   - `w(theta)=wMin + (wMax-wMin)*|cos(theta - penAngle)|^gamma`
3. Resolver uniones con curvas C1/C2.

### 4.4 Método fractal geométrico
1. Construir glifo base legible.
2. Sustituir segmentos seleccionados por micro-módulos de Flor (`depth` controlado).
3. Aplicar solo en tamaño display; activar fallback simplificado en texto.

---

## 5) Sistema paramétrico

### 5.1 Parámetros globales
- `r`: unidad geométrica.
- `stroke`: grosor principal.
- `contrast`: 0..1.
- `roundness`: 0..1.
- `tension`: rigidez de curvas.
- `slant`: inclinación.
- `aperture`: apertura de contraformas.
- `inkTrap`: tamaño de trampa.
- `fractalDepth`: nivel recursivo.
- `organicNoise`: perturbación controlada.

### 5.2 Parámetros por glifo
- `widthClass[g]`, `weightClass[g]`, `nodeBias[g]`, `terminalStyle[g]`.
- Anclas para diacríticos ligadas a nodos hex (`top`, `bottom`, `ogonek`, etc.).

### 5.3 Restricciones
- Legibilidad: área de contraforma mínima por glifo (`>= threshold`).
- Consistencia: desviación angular máxima respecto a familia.
- Manufactura: radio mínimo de herramienta para CNC/laser.

---

## 6) Pseudocódigo generativo

```pseudo
initSystem(params):
    grid = buildHexGrid(params.r, params.rings)
    flower = buildFlowerOfLife(grid)
    metrics = deriveMetrics(params.r)
    return Context(grid, flower, metrics, params)

buildGlyph(char, ctx):
    spec = grammarSpecFor(char)
    skeleton = []

    for rule in spec.rules:
        skeleton.append(applyRule(rule, ctx.grid, ctx.params))

    strokeOutline = expandStroke(skeleton, ctx.params.stroke, ctx.params.contrast)
    strokeOutline = enforceTangency(strokeOutline, mode="C1")
    strokeOutline = resolveBooleans(strokeOutline)
    strokeOutline = applyOpticalCorrections(strokeOutline, ctx.metrics)
    strokeOutline = enforceConstraints(strokeOutline, ctx.params)

    return strokeOutline

buildFont(charset, params):
    ctx = initSystem(params)
    font = newFont(ctx.metrics)

    for ch in charset:
        glyphOutline = buildGlyph(ch, ctx)
        font.addGlyph(ch, glyphOutline)

    font.kerning = autoKernByShapeDistance(font)
    font.features = buildOpenTypeFeatures(font)
    return font
```

---

## 7) Procedimientos modulares

### 7.1 Módulos core
- `grid-core`: retícula hex + consultas espaciales.
- `geom-core`: intersecciones, tangencias, offsets.
- `grammar-engine`: reglas y derivaciones.
- `glyph-builder`: construcción por clases de glifo.
- `optical-engine`: overshoot, compensaciones, traps.
- `export-engine`: UFO/GLIF/OTF/TTF/variable.

### 7.2 Pipeline recomendado
1. Definición paramétrica
2. Generación skeleton
3. Stroke expansion
4. Boolean clean
5. QA geométrico
6. QA tipográfico
7. Export + pruebas de texto

### 7.3 QA automatizable
- Detección de auto-intersecciones.
- Detección de nodos redundantes.
- Control de extremos casi colineales.
- Validación de winding direction.
- Pruebas de rasterización a tamaños 9–14 px.

---

## 8) Extensiones para Variable Fonts e IA

### 8.1 Ejes de variación sugeridos
- `wght`: grosor.
- `wdth`: ancho.
- `opsz`: tamaño óptico.
- `ROND`: redondez geométrica.
- `APTR`: apertura.
- `ORGN`: organicidad/noise controlado.
- `FRCL`: profundidad fractal (display).

### 8.2 Compatibilidad de masters
- Topología idéntica de nodos por glifo entre masters.
- Orden de contorno estable.
- Reglas de interpolación para arcos (preferir parámetros, no puntos libres).

### 8.3 IA asistida
- **IA de propuesta**: sugiere combinaciones de reglas por glifo.
- **IA de evaluación**: puntúa legibilidad, ritmo y color tipográfico.
- **IA de corrección**: optimiza kerning y espaciado lateral.
- **Control humano** obligatorio: aprobación de caracteres críticos (`a,e,s,n,r,g,0,1,2`).

### 8.4 IA + CAD/CAM
- Exportar curvas limpias con tolerancia configurable.
- Convertir contornos a trayectorias de fabricación (CNC/laser/plotter).
- Modo stencil para corte físico (puentes automáticos en contraformas cerradas).

---

## 9) Enfoques estilísticos

### 9.1 Orbital / orgánico
- Curvas dominantes, transiciones suaves C2.
- Ligera perturbación procedural (`organicNoise` bajo).
- Terminales bulbosos o gota.

### 9.2 Hexagonal modular
- Construcción estricta sobre segmentos de 0/60/120°.
- Menor cantidad de curvas libres.
- Alta repetibilidad para sistemas de identidad.

### 9.3 Caligráfico tangencial
- Contraste marcado por ángulo de herramienta.
- Terminales con salida dinámica.
- Priorizar ritmo sobre simetría exacta.

### 9.4 Fractal geométrico
- Detalle recursivo en extremos/serifas/espinas internas.
- Activación por tamaño (`opsz > displayThreshold`).
- Versión text simplificada automática.

---

## 10) Implementación por plataforma

### 10.1 Glyphs / RoboFont
- Generar UFO con capas:
  - `skeleton`, `construction`, `final`.
- Scripts Python para:
  - colocar nodos en coordenadas hex,
  - aplicar offsets de stroke,
  - validar compatibilidad para variable font.
- Integrar con fontmake/varLib en etapa final.

### 10.2 Processing / p5.js
- Usar sistema como laboratorio visual en tiempo real.
- Controles UI (sliders) para parámetros globales.
- Exportar SVG por glifo o specimen animado de ejes variables.

### 10.3 Blender Geometry Nodes
- Grafo nodal:
  - generación de retícula,
  - instanciación de círculos,
  - recortes y conversiones curva/malla.
- Útil para versiones 3D, motion y lettering volumétrico.

### 10.4 Houdini
- HDA para construir glifos procedurales por reglas.
- Atributos por primitiva para controlar peso, roundness, fractal depth.
- PDG para lote de glifos y variaciones masivas.

### 10.5 Grasshopper
- Definición paramétrica con componentes geométricos.
- Integración con Rhino para revisión y manufactura.
- Excelente para señalética, grabado y corte arquitectónico.

### 10.6 AI + CAD/CAM
- Pipeline:
  1. IA propone variantes
  2. motor geométrico valida reglas
  3. CAD ajusta tolerancias
  4. CAM fabrica prototipos
- Guardrails:
  - no permitir curvas inválidas,
  - respetar espesores mínimos de producción,
  - registrar trazabilidad de parámetros.

---

## 11) Hoja de ruta de desarrollo

1. **MVP**: mayúsculas A–Z + números con estilo hexagonal modular.
2. Añadir minúsculas y signos básicos en modo orbital.
3. Incorporar ejes `wght`, `wdth`, `opsz`.
4. Integrar módulo IA para propuestas y QA.
5. Extender a estilos caligráfico y fractal display.
6. Empaquetar SDK (Python + JS) con presets por estilo.

## 12) Entregables recomendados
- Documento de reglas (este sistema).
- Librería de primitivas geométricas.
- Scripts de generación para Glyphs/RoboFont.
- Prototipo interactivo p5.js.
- HDA de Houdini / definición Grasshopper.
- Set de masters para variable font + pruebas de interpolación.
