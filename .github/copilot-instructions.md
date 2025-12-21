# Brothers Solutions — Instrucciones para Agentes de IA

## Descripción general
**Brothers Solutions** es una landing page + wizard de PC builder en HTML/CSS/JS puro, sin frameworks. Ofrece servicios de armado de PCs, reparaciones de consolas, flasheos y desarrollo web.

- **Landing (`index.html`)**: Hero, servicios (6 ofertas), contacto con WhatsApp
- **PC Builder (`pc-builder.html`)**: Wizard paso-a-paso de 5 pasos que genera cotización prellenada

## Arquitectura y patrones

### 1. Diseño de componentes
- **Temática gaming/retro** con gradientes neon (verde, rosa, azul, violeta, dorado)
- **Token de diseño centralizado** en `:root` (variables CSS globales)
- **Card/panel reutilizable**: `background: linear-gradient(...); border: 1px solid rgba(255,255,255,.12); box-shadow: var(--shadowSoft)`
- **Botones estilo power-up**: gradiente amarillo→rosa + sombra efecto gema
- **Animaciones suaves**: `ease` (0.22s–0.75s) y `ease2` para entrada hero

### 2. Flujo de datos (PC Builder)
El wizard usa **estado centralizado** (`state` object):
```javascript
const state = {
  currency, budget, usage, cpuBrand, platform,
  cpuModel, mobo, ramCap, ramSpeed, gpu,
  storageType, storageCap, psu, caseSize, cooling,
  peripherals, rgb, wifi, notes
}
```

**Dependencias entre pasos:**
- Step 2 (Plataforma) → filtra CPUs y motherboards disponibles
- Step 3 (CPU/GPU) → recalcula PSU recomendado según GPU seleccionada
- Step 5 (Resumen) → construye mensaje WhatsApp prellenado

### 3. WhatsApp como CTA final
- **Número configurado** en constante `WHATSAPP_NUMBER` (sin `+`)
- **Helper**: `waLink(msg)` encapsula construcción de URL `https://wa.me/{NUMERO}?text={MSG}`
- **Botones inteligentes**: cada servicio en landing pre-rellena el tipo de problema
- **Builder**: mensaje completo incluye spec técnica + recomendaciones

### 4. Validación y hints
- **Step 1** → presupuesto (slider/input dual), uso → guía recomendación GPU
- **Step 2** → plataforma obligatoria para poblador de CPUs
- **Step 3** → PSU se ajusta automático si GPU requiere más watts (`bumpPsuToRecommended`)
- **Step 4** → notas libres, selecciones binarias (RGB/WiFi)
- **Hints contextuales**: DDR4 vs DDR5, compatibilidad exacta ⚠️

## Descubrimientos clave

### Patrones de código
1. **Selectores mínimos**: `const $ = sel => document.querySelector(sel)`
2. **Datos simplificados en JS**: Arrays de objetos en `DATA` (CPUs, motherboards, GPUs) — fácil expandir
3. **IntersectionObserver** para scroll-reveal (landing)
4. **No frameworks**: Vanilla JS con control completo de animaciones

### Ajustes esperados
- **Número WhatsApp**: reemplazar `WHATSAPP_NUMBER` en ambos HTML
- **Datos de componentes**: expandir `DATA.cpus`, `DATA.mobos`, `DATA.gpus` con opciones reales
- **Presupuestos regionales**: `BUDGET` configurable por moneda (COP/USD default)
- **Placeholder horarios/contacto**: llenar en footer de `index.html`

### Convenciones
- **Clases CSS**: `btn`, `card`, `panel`, `chip`, `step`, `reveal`
- **atributos data-**: `data-step`, `data-value`, `data-service` para estado UI
- **aria-labels**: accesibilidad básica en nav, forms, botones
- **Lenguaje**: es (Spanish) — textos y labels en español

## Tareas comunes

### Agregar servicio nuevo en landing
1. Duplicar `.serviceCard` en section `#servicios` (index.html, línea ~1000)
2. Actualizar `data-service` con nombre del servicio
3. Cambiar icono SVG
4. Handler automático: botón `.js-wa` genera pre-fill `"Quiero consultar por: *[NOMBRE]*"`

### Expandir opciones de CPU/Motherboard/GPU
1. Editar `DATA.cpus`, `DATA.mobos`, `DATA.gpus` en `pc-builder.html` (línea ~1570+)
2. Estructura: `{ brand, platform, name, tier }` (CPUs); `{ platform, name, size, mem }` (Mobos)
3. Automático: `populateCpuAndMobo()` repobla selects filtrando por plataforma

### Cambiar colores / tema
1. `:root` CSS: variables `--g` (verde), `--r` (rosa), `--y` (dorado), `--b` (azul), `--p` (violeta)
2. Gradientes: `rgba(HEXR,HEXG,HEXB,.OPACITY)` reutilizable
3. Mantener contraste (fondo `--bg0: #070A12` → texto `--txt: #EAF0FF`)

### Modificar wizard (agregar/eliminar pasos)
- Cambiar `steps.forEach(s => Number(s.dataset.step) === currentStep)` si se alteran pasos
- Actualizar labels `"Paso X de 5"` al cambiar total de pasos
- Validación en `validateStep(step)` — agregar lógica de requerimientos por paso

## Testing mental checklist
- ✅ WhatsApp abre con URL válida y mensaje legible
- ✅ Responsivo a 560px (móvil) — botones flotantes visibles
- ✅ Formulario landing: validación email + alerta accesible (no alert intrusivo)
- ✅ Builder: navegación prev/next mantiene estado
- ✅ Presupuesto: cambiar moneda recalcula range
- ✅ GPU → PSU: cambio automático si recomendación > selección actual

## Dependencias externas
- **Google Fonts**: Rubik + Space Grotesk (enlace preconnect en `<head>`)
- **SVG inline**: iconos nativos (no hay archivos de asset)
- **IntersectionObserver**: nativo en navegadores modernos

---

**Nota**: Este es un proyecto estático sin backend. Toda lógica es client-side. Los "datos de componentes" en `DATA` están hardcodeados — para integrar un catálogo real, considerar fetch/API al paso 2–3 del builder.
