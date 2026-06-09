# html-to-shopify

Skill de Claude Code: método reutilizable para construir themes de Shopify (Liquid) a partir de HTML/CSS estático.

## Qué es

Encapsula el pipeline **HTML/CSS → Liquid → Shopify**: leer el HTML con Read/Grep, **medir los estilos computados del HTML origen con Playwright** (no el CSS escrito), traducirlo a `sections/*.liquid` con su `{% schema %}`, y verificar comparando computados origen vs construido con `shopify theme dev`. Adapta el flujo validado de `html-elementor-skill` para target Shopify.

Cubre migración de sitios HTML legacy, maquetas HTML/CSS, plantillas Bootstrap/Tailwind, y cualquier entrega de diseño en archivos HTML hacia un theme de Shopify.

## Instalación

La skill se carga clonando este repo en el directorio de skills de Claude Code:

```
git clone https://github.com/alvarog6/html-to-shopify.git ~/.claude/skills/html-to-shopify
```

Claude Code la detecta automáticamente vía `SKILL.md`.

## Estado

**v0.1** — esqueleto del método extrapolado de `html-elementor-skill`. Pipeline y workflow definidos; quirks específicos de Liquid/Shopify por llenar en la primera ejecución real.

## Estructura

- `SKILL.md` — el método (lo que Claude Code carga en contexto).
- `references/` — detalle fino (esqueletos por completar mientras se usa la skill):
  - `theme-structure.md` — esqueleto mínimo de un theme Shopify
  - `section-schema-patterns.md` — patrones de `{% schema %}` por tipo de sección
  - `liquid-quirks.md` — gotchas de Liquid descubiertos en uso real
  - `playwright-verification.md` — script template para medir computados
  - `shopify-cli-workflow.md` — comandos de CLI para dev / push / pull / publicar

## Stack necesario

- **Shopify CLI** (`npm i -g @shopify/cli @shopify/theme`)
- Shopify Partners account + **dev store** (gratis, no caduca)
- **Playwright** (igual que las skills hermanas)
- Opcional: **`@shopify/dev-mcp`** para consultar docs / validar GraphQL

## Relación con html-elementor-skill

Skills hermanas. Mismo método (medir → construir → comparar), distinto backend:

- [`html-elementor-skill`](https://github.com/alvarog6/html-elementor-skill) — WordPress + Elementor MCP
- `html-to-shopify` — Shopify + Liquid + Shopify CLI
