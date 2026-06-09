# Patrones de `{% schema %}` por tipo de section

Cada `sections/<nombre>.liquid` termina con un bloque `{% schema %}...{% endschema %}` con JSON que define:
- `name`: nombre que ve el merchant en el Theme Editor
- `settings`: campos editables a nivel de section
- `blocks`: items repetibles dentro de la section (opcional)
- `presets`: configuraciones predeterminadas que aparecen en el library

Patrones probados/esperables (por validar en uso real).

## Patrón hero (single instance)

```liquid
<section class="hero" style="background-image: url({{ section.settings.bg_image | image_url: width: 2000 }});">
  <div class="hero__inner">
    <h1>{{ section.settings.heading }}</h1>
    <p>{{ section.settings.subheading }}</p>
    {% if section.settings.cta_text != blank %}
      <a class="btn" href="{{ section.settings.cta_link }}">{{ section.settings.cta_text }}</a>
    {% endif %}
  </div>
</section>

{% schema %}
{
  "name": "Hero",
  "settings": [
    { "type": "image_picker", "id": "bg_image", "label": "Background image" },
    { "type": "text", "id": "heading", "label": "Heading", "default": "Título" },
    { "type": "textarea", "id": "subheading", "label": "Subheading" },
    { "type": "text", "id": "cta_text", "label": "Button text" },
    { "type": "url", "id": "cta_link", "label": "Button link" }
  ],
  "presets": [{ "name": "Hero" }]
}
{% endschema %}
```

## Patrón con blocks repetibles (ej. testimonials)

```liquid
<section class="testimonials">
  <h2>{{ section.settings.heading }}</h2>
  <div class="testimonials__grid">
    {% for block in section.blocks %}
      <article class="testimonial" {{ block.shopify_attributes }}>
        <p>"{{ block.settings.quote }}"</p>
        <footer>
          <strong>{{ block.settings.name }}</strong>
          <span>{{ block.settings.role }}</span>
        </footer>
      </article>
    {% endfor %}
  </div>
</section>

{% schema %}
{
  "name": "Testimonials",
  "settings": [
    { "type": "text", "id": "heading", "label": "Heading", "default": "Lo que dicen nuestros clientes" }
  ],
  "blocks": [
    {
      "type": "testimonial",
      "name": "Testimonial",
      "settings": [
        { "type": "textarea", "id": "quote", "label": "Quote" },
        { "type": "text", "id": "name", "label": "Name" },
        { "type": "text", "id": "role", "label": "Role" }
      ]
    }
  ],
  "max_blocks": 9,
  "presets": [
    {
      "name": "Testimonials",
      "blocks": [
        { "type": "testimonial" },
        { "type": "testimonial" },
        { "type": "testimonial" }
      ]
    }
  ]
}
{% endschema %}
```

`{{ block.shopify_attributes }}` es **obligatorio** en el elemento root de cada block — habilita el editor visual del Theme Editor (seleccionar block para editarlo).

## Patrón featured products (con product picker)

```liquid
<section class="featured-products">
  <h2>{{ section.settings.heading }}</h2>
  <div class="grid">
    {% for block in section.blocks %}
      {% assign product = all_products[block.settings.product] %}
      {% if product != blank %}
        <a class="card" href="{{ product.url }}" {{ block.shopify_attributes }}>
          <img src="{{ product.featured_image | image_url: width: 600 }}" alt="{{ product.title }}">
          <h3>{{ product.title }}</h3>
          <span>{{ product.price | money }}</span>
        </a>
      {% endif %}
    {% endfor %}
  </div>
</section>

{% schema %}
{
  "name": "Featured products",
  "settings": [
    { "type": "text", "id": "heading", "label": "Heading", "default": "Destacados" }
  ],
  "blocks": [
    {
      "type": "product",
      "name": "Product",
      "settings": [
        { "type": "product", "id": "product", "label": "Product" }
      ]
    }
  ],
  "presets": [{ "name": "Featured products" }]
}
{% endschema %}
```

## Types de settings más usados

| Type | Para qué |
|---|---|
| `text` / `textarea` | Texto corto / largo |
| `richtext` | Editor WYSIWYG con `<p>`, `<strong>`, `<em>`, `<a>` |
| `image_picker` | Selección de imagen del media library |
| `color` | Color picker |
| `range` | Slider numérico (con `min`, `max`, `step`, `unit`) |
| `select` | Dropdown con `options: [{ value, label }]` |
| `checkbox` | Boolean |
| `radio` | Radio buttons |
| `url` | Link (interno o externo) |
| `product` / `collection` / `blog` / `article` / `page` | Resource picker |
| `font_picker` | Selección de fuente del catálogo de Shopify |
| `video_url` | URL de YouTube/Vimeo |

## Restricciones a recordar

- `name` ≤ 25 caracteres
- `id` solo alfanumérico y `_`
- `max_blocks` por section: 50 absoluto (en práctica, 20 es razonable)
- `presets` define **qué versión aparece al añadir desde Theme Editor**; sin presets, la section solo se puede usar si está hardcoded en `templates/*.json`

(Por agregar mientras se construyen themes reales)
