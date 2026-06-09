# Estructura mínima de un theme Shopify

Esqueleto que `shopify theme dev` necesita para correr. Crear esto antes de empezar a portar HTML.

```
mi-theme/
├── assets/                    # CSS / JS / imágenes / SVG (servidos por CDN)
│   ├── theme.css              # styles globales (puede ser .css.liquid si necesita variables Liquid)
│   ├── theme.js
│   └── (assets del HTML origen)
├── config/
│   ├── settings_schema.json   # theme settings globales (colores, fuentes, logo)
│   └── settings_data.json     # valores actuales — generado/editado en Theme Editor
├── layout/
│   └── theme.liquid           # wrapper master: <html>, <head>, {{ content_for_header }}, {{ content_for_layout }}
├── locales/
│   └── es.default.json        # traducciones — claves accesibles con {{ 'key' | t }}
├── sections/                  # componentes reutilizables con su {% schema %}
│   ├── header.liquid
│   ├── footer.liquid
│   ├── hero.liquid
│   ├── featured-products.liquid
│   └── ...
├── snippets/                  # partials chicos (sin schema), llamables con {% render 'nombre' %}
│   └── price.liquid
└── templates/                 # qué secciones consume cada tipo de página
    ├── index.json             # home — JSON con array de sections
    ├── product.json           # ficha de producto
    ├── collection.json        # listado de productos
    ├── page.json              # páginas estáticas (About, Contact)
    ├── cart.json
    └── 404.json
```

## Archivos mínimos imprescindibles para que `shopify theme dev` arranque

- `layout/theme.liquid` con al menos `<html><head>{{ content_for_header }}</head><body>{{ content_for_layout }}</body></html>`
- `templates/index.json` con un sections array (puede estar vacío `{ "sections": {}, "order": [] }`)
- `config/settings_schema.json` con al menos `[{ "name": "theme_info", "theme_name": "Mi Theme" }]`

## Diferencia `.json` vs `.liquid` en `templates/`

- **`.json`** (recomendado): template es un JSON que lista qué secciones renderizar y en qué orden. El **merchant puede agregar/quitar/reordenar secciones desde el Theme Editor**. Es lo que Dawn y todos los themes modernos usan.
- **`.liquid`**: template es código Liquid fijo. Menos flexible, sin Theme Editor para reordenar.

Para themes nuevos: siempre `.json`.

## Settings vs blocks vs sections — jerarquía

- **Theme settings** (`config/settings_schema.json`): globales del theme (color de marca, fuente, logo). Accesibles con `{{ settings.X }}` en cualquier `.liquid`.
- **Section settings** (`{% schema %}` dentro de la section): específicos a esa instancia. `{{ section.settings.X }}`.
- **Block settings** (dentro de `blocks: []` del schema de la section): items repetibles dentro de una section. `{{ block.settings.X }}` dentro de `{% for block in section.blocks %}`.

Ejemplo: section "Featured products" con 3 blocks tipo "product card" — cada card es un block con sus propios settings (producto seleccionado, badge, etc.).

## Recursos de referencia

- Base recomendada: **Dawn** (theme oficial de Shopify, open source) — https://github.com/Shopify/dawn
- Dawn como punto de partida: `shopify theme init mi-theme --clone-url https://github.com/Shopify/dawn`
- Liquid reference: https://shopify.dev/docs/api/liquid
- Theme architecture: https://shopify.dev/docs/storefronts/themes/architecture
