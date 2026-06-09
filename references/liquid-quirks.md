# Liquid quirks (por llenar con uso real)

Gotchas, sorpresas y workarounds específicos de Shopify/Liquid descubiertos al construir themes.

## Sintaxis básica recordatoria

```liquid
{% assign x = 'hola' %}                {# variable string #}
{% capture html %}<p>foo</p>{% endcapture %}   {# variable con bloque #}
{% if product.available %} ... {% endif %}
{% for item in collection.products limit: 8 %} ... {% endfor %}
{% render 'snippet-name', producto: product %}
{% comment %} ... {% endcomment %}

{{ variable | filter1 | filter2: 'arg' }}
{{ 'mi-img.jpg' | asset_url }}                         {# resuelve a CDN URL #}
{{ product.featured_image | image_url: width: 800 }}   {# imagen responsive #}
{{ 'general.checkout' | t }}                           {# traducción #}
{{ product.price | money }}                            {# formato moneda #}
```

## Quirks esperables (validar)

### `section.settings` vs `block.settings`

Dentro del mismo `.liquid`:
- Fuera de `{% for block in section.blocks %}` → solo accedes `section.settings.X`
- Dentro → accedes `block.settings.X` PARA EL BLOCK Y `section.settings.X` igual

Confundirlos es error común. `block.settings.heading` fuera del loop = `nil`.

### `.json` template no acepta Liquid

`templates/index.json` es JSON puro — no puedes meter `{% %}`. Si necesitas lógica, ponla en una `section.liquid`.

### `assign` no procesa Liquid dentro de strings

```liquid
{% assign x = "{{ product.title }}" %}   {# x es el string literal, NO el title #}
{% capture x %}{{ product.title }}{% endcapture %}   {# x ES el title — usar capture #}
```

### `theme.css` vs `theme.css.liquid`

- `theme.css` → archivo estático servido por CDN (rápido, cacheable)
- `theme.css.liquid` → procesado por Liquid antes de servir (lento, no cacheable). Solo usar si necesitas `{{ settings.color_brand }}` dentro del CSS.

Alternativa más rápida: definir las variables CSS en un `<style>` del `theme.liquid` y mantener `theme.css` estático.

### Sections groups (header / footer)

En Online Store 2.0, `header` y `footer` se definen en `sections/header.liquid` + `sections/footer.liquid` Y en `config/header-group.json` + `config/footer-group.json` para que sean editables como grupo.

### `customer.tags` y `product.tags` son arrays

```liquid
{% if product.tags contains 'nuevo' %} ... {% endif %}
{% for tag in product.tags %} ... {% endfor %}
```

### Ajax cart endpoints (cuando hay drawer/sidebar)

- `POST /cart/add.js` — agregar item
- `GET /cart.js` — obtener cart actual (JSON)
- `POST /cart/change.js` — cambiar cantidad
- `POST /cart/clear.js` — vaciar

Respuesta JSON con cart entero (items, total_price, item_count).

### Pagination

```liquid
{% paginate collection.products by 12 %}
  {% for product in collection.products %} ... {% endfor %}
  {{ paginate | default_pagination }}
{% endpaginate %}
```

Liquid solo permite paginar **dentro** de `{% paginate %}`.

(Por agregar conforme se descubren)
