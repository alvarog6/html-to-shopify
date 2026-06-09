---
name: html-to-shopify
description: Método para reconstruir como theme de Shopify (Liquid) sitios entregados como HTML/CSS estático. Úsala cuando haya que migrar a Shopify un sitio HTML legacy, una maqueta HTML/CSS, una plantilla Bootstrap/Tailwind, o cualquier entrega de diseño en archivos HTML — leer el HTML/CSS con Read/Grep, **medir los estilos computados del HTML origen con Playwright** (no el CSS escrito), traducirlo a `sections/*.liquid` con su `{% schema %}`, y verificar comparando computados origen vs construido en `shopify theme dev` en todos los breakpoints. Dispara ante cualquier trabajo de HTML-a-Shopify / HTML-a-Liquid / theme de Shopify desde mockup HTML, aunque no se nombren explícitamente.
---

# HTML → Shopify Theme

Método reutilizable para convertir sitios HTML estáticos en themes de Shopify. Adapta el flujo validado de `html-elementor-skill` cambiando el backend de Elementor (DB + MCP que mueve elementos en runtime) a Shopify (archivos `.liquid` en disco + Shopify CLI para deploy).

## Estado

**v0.1.** Esqueleto del método. Estructura, pipeline y workflow definidos por extrapolación del flujo validado en `html-elementor-skill`. **No probado en theme de producción todavía** — los quirks específicos de Liquid/Shopify deben llenarse en la primera ejecución real. Marcar cada quirk descubierto en `references/liquid-quirks.md` y `references/section-schema-patterns.md`.

## Diferencia conceptual con html-elementor-skill

| Aspecto | Elementor | Shopify |
|---|---|---|
| Construcción | MCP API en runtime (`add-heading`, `update-container`) | Escribir archivos `.liquid` en disco |
| Datos del componente | Settings en DB del widget | `{% schema %}` JSON al final de cada section |
| Editor visual | Sí, primero (drag & drop) | Sí, secundario (Theme Editor consume schemas) |
| Despliegue | Plugin installer custom | **Shopify CLI** (`shopify theme push`) |
| MCP rol | Indispensable, mueve elementos | Opcional, solo consulta docs / valida GraphQL |
| Preview iterativo | WP Studio + browser | `shopify theme dev` con hot reload |

La filosofía "medir computados, no leer CSS escrito" se mantiene idéntica.

## Pipeline

| Etapa | Herramienta | Para qué |
|---|---|---|
| Leer diseño | **Read / Grep / Glob** + **Playwright** apuntado a `python -m http.server` local o `file://` | Mapear `<section>`s e `id`s, extraer tokens del CSS (`:root`), **medir estilos computados** (`getComputedStyle`) de cada elemento clave |
| Construir | **Edit / Write** sobre archivos del theme + **Shopify CLI** para scaffolding inicial | `shopify theme init`, crear `sections/*.liquid` con su `{% schema %}`, `templates/*.json` que ensambla secciones, `snippets/` reutilizables, `assets/` para CSS/JS/imágenes |
| Preview | **`shopify theme dev`** (hot reload local) | Servidor local que proxy a la tienda, refresca al guardar |
| Verificar | **Playwright** apuntado al preview de `theme dev` | Leer computados del construido y compararlos con los del origen en 3 breakpoints. Screenshot solo al final |
| Validar código | **Shopify Theme Check** (`shopify theme check`) | Linter de Liquid + JSON + schemas |
| Consultar docs / objetos Liquid | **MCP `@shopify/dev-mcp`** (opcional) | Buscar referencia de `product.*`, `cart.*`, `section.settings.*`, filtros, tags |
| Desplegar | **`shopify theme push --unpublished`** o `--theme=<id>` | Sube a la tienda. Primero a unpublished para review, luego publicar desde admin o `--live` |

La traducción del HTML al árbol de secciones es **manual y deliberada** — ahí está el criterio de diseño, y por eso no se automatiza.

## Workflow por página (medir → construir → comparar → screenshot)

Seguir estos pasos en orden. **Saltarse el 1.5 (medir computados del origen) o el 5 (comparar computados) es la causa principal de retrabajo** — el CSS escrito miente (cascade, herencia, `clamp()`, media queries) y solo los valores computados son la verdad.

**0. Consultar el método antes de construir.** Revisar `references/section-schema-patterns.md` para no improvisar una section que ya tiene forma validada (header, hero, product-grid, collection-list, testimonials, FAQ, footer).

**1. Reconnaissance del HTML.**
- `Read` del HTML completo de la página → mapa de `<section>`s e IDs.
- `Read` del CSS específico de la página + del CSS base (`styles.css` / `globals.css`) para mapear `:root` (colores, fuentes, spacing) → estos van a `config/settings_schema.json` como theme settings y a `assets/theme.css` como variables CSS.
- `Grep` por clases de componentes recurrentes (`.card`, `.hero`, `.product-*`) en **todas las páginas** para detectar repetidos antes de empezar — esos serán secciones reutilizables.

**1.5. Medir computados del HTML origen con Playwright.** Antes de construir cualquier section, abrir el HTML origen con Playwright (vía `file://` o `python -m http.server`) y leer `window.getComputedStyle(el)` de cada elemento clave: tipografía (font-family / size / weight / line-height), espaciados (padding / margin / gap), colores (background / color / border), dimensiones (width / height / `getBoundingClientRect()`), layout (`display` / `flex-direction` / `grid-template-columns`). Script template en `references/playwright-verification.md`.

**2. Inventario.** Antes de construir, ver qué secciones ya existen en el theme (`sections/`). Si una section ya cubre el bloque → reutilizar (agregar blocks/settings al schema). Si no → crear nueva `sections/<nombre>.liquid` con su `{% schema %}`.

**3. Assets.** Los assets ya están en disco. Copiarlos a `assets/` del theme (Shopify CDN sirve todo lo que esté ahí con URL predecible: `{{ 'mi-imagen.jpg' | asset_url }}`). Para SVGs, revisar y corregir `preserveAspectRatio` antes de copiar. Las fuentes pueden ir a `assets/` (self-host) o usar las del catálogo de Shopify (`{{ settings.body_font | font_face }}`).

**4. Construir la section.**
- Crear `sections/<nombre>.liquid`
- HTML semántico con clases (mantener nombres de clase del origen donde sea posible — facilita reutilizar el CSS)
- `{% schema %}` JSON al final con: `name`, `settings`, `blocks` (si la section tiene bloques repetibles), `presets`
- Cada elemento variable del diseño = un `setting` del schema (text/image_picker/color/range/select). Esto es lo que aparece en el Theme Editor para que el merchant edite sin tocar código.
- Si la página entera consume varias secciones → crear `templates/<tipo>.json` que las ensambla.

**5. Comparar computados origen vs construido.** Con `shopify theme dev` corriendo, apuntar Playwright al preview local y medir los mismos `getComputedStyle` que en el paso 1.5. Diff. Si difiere → ajustar CSS/schema hasta que coincida en 320px, 768px, 1280px.

**6. Screenshot final.** Solo al final, para imágenes/sombras/detalles que `getComputedStyle` no captura.

## Stack y setup inicial

Una sola vez por máquina:

```bash
npm install -g @shopify/cli @shopify/theme
shopify version    # verificar
shopify auth login # OAuth con Partners account
```

Una vez por proyecto:

```bash
# Opción A: theme nuevo en blanco
shopify theme init mi-theme --clone-url https://github.com/Shopify/dawn   # Dawn es la base recomendada

# Opción B: theme vacío total (para reconstruir desde HTML sin baseline)
mkdir mi-theme && cd mi-theme
# crear estructura mínima a mano (ver references/theme-structure.md)

# Dev loop
shopify theme dev --store=<mi-tienda>.myshopify.com
# Servidor local en http://127.0.0.1:9292 con hot reload
```

Para tienda de desarrollo gratis: crear **dev store** desde Shopify Partners dashboard (no cuesta nada, no caduca, sirve para construir themes sin pagar plan).

## Quirks a documentar mientras se construye

(Por llenar en la primera ejecución real, en `references/liquid-quirks.md`. Categorías esperables:)

- `{% sections %}` vs `{% section %}` — singular para una sola, plural para grupos (header/footer)
- `section.settings.X` vs `block.settings.X` — scoping interno
- `assign` vs `capture` — variables strings vs blocks de Liquid
- Filtros encadenables: `{{ 'foo' | t | escape | replace: 'X', 'Y' }}`
- `for` loops con `limit:`, `offset:`, `reversed`, `paginate`
- Ajax cart endpoints (`/cart/add.js`, `/cart.js`, `/cart/change.js`) si el theme usa drawer/sidebar
- `theme.css.liquid` vs `theme.css` — el primero procesa Liquid, el segundo es estático (más rápido en CDN)
- Schema presets vs default — los presets aparecen en el library del Theme Editor; default es la config al añadir
- Translations `{{ 'key.path' | t }}` que requieren entrada en `locales/<idioma>.json`

## Despliegue

```bash
# Subir a la tienda como unpublished (para review)
shopify theme push --unpublished --store=<tienda>.myshopify.com

# Subir y publicar directo (cuidado: reemplaza el theme live)
shopify theme push --live

# Subir reemplazando un theme específico ya existente
shopify theme push --theme=<ID>

# Bajar el theme actual para versionar
shopify theme pull --theme=<ID>
```

Gestionar versiones del theme con git en paralelo — Shopify guarda histórico limitado en admin, git es la fuente de verdad.

## Relación con html-elementor-skill

Skills hermanas. Mismo input (HTML/CSS), mismo método (medir → construir → comparar), distinto backend:

- [`html-elementor-skill`](https://github.com/alvarog6/html-elementor-skill) — backend WordPress + Elementor MCP
- `html-to-shopify` — backend Shopify + Liquid + Shopify CLI

Si el cliente quiere e-commerce escalable de pago, va Shopify. Si quiere flexibilidad editorial y control total del sitio, va WordPress + Elementor.
