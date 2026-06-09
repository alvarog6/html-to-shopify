# Shopify CLI — workflow de dev / push / pull / publicar

## Instalación

```bash
npm install -g @shopify/cli @shopify/theme
shopify version
```

(En Windows si Node está bien instalado, debería funcionar directo. Si hay problemas de PATH, agregar `%APPDATA%\npm` al PATH.)

## Auth

```bash
shopify auth login
# OAuth en navegador, login con Partners account
```

Para una dev store específica:

```bash
shopify theme dev --store=mi-dev-store.myshopify.com
# Primera vez pedirá auth a esa tienda
```

## Crear dev store (una vez)

1. Login a https://partners.shopify.com (cuenta gratis)
2. Stores → Add store → Development store
3. Anotar el dominio `<nombre>.myshopify.com`

## Theme nuevo

### Opción A — clonar Dawn (recomendado)

```bash
shopify theme init mi-theme --clone-url https://github.com/Shopify/dawn
cd mi-theme
shopify theme dev --store=mi-dev-store.myshopify.com
```

### Opción B — desde HTML origen (lo que la skill cubre)

```bash
mkdir mi-theme && cd mi-theme
# Crear estructura mínima (ver references/theme-structure.md)
shopify theme dev --store=mi-dev-store.myshopify.com
```

## Dev loop

```bash
shopify theme dev --store=<tienda>.myshopify.com
# Sirve en http://127.0.0.1:9292 con hot reload
# Cambios en sections/, snippets/, assets/, templates/ → refresh automático
# Cambios en config/settings_schema.json → requieren restart
```

## Validar antes de push

```bash
shopify theme check
# Linter de Liquid + JSON. Falla en errores, advierte en warnings.
```

## Push a la tienda

```bash
# Subir como theme unpublished nuevo (para review)
shopify theme push --unpublished --store=<tienda>.myshopify.com

# Subir a un theme existente (sobrescribe)
shopify theme push --theme=<ID> --store=<tienda>.myshopify.com

# Subir Y publicar directo (CUIDADO: reemplaza live)
shopify theme push --live --store=<tienda>.myshopify.com

# Solo subir archivos modificados (incremental)
shopify theme push --only sections/hero.liquid --store=<tienda>.myshopify.com
```

## Pull desde la tienda

```bash
# Bajar el theme live
shopify theme pull --live --store=<tienda>.myshopify.com

# Bajar uno específico
shopify theme pull --theme=<ID> --store=<tienda>.myshopify.com

# Solo bajar config/settings_data.json (cuando el merchant editó settings en admin)
shopify theme pull --only config/settings_data.json --store=<tienda>.myshopify.com
```

## Listar themes de la tienda

```bash
shopify theme list --store=<tienda>.myshopify.com
# Devuelve ID y nombre de cada theme (live + unpublished)
```

## Versionado

`shopify` no versiona — usar git en paralelo:

```bash
git init && git add . && git commit -m "Initial theme commit"
# Cualquier cambio se commitea localmente
# Push a la tienda con shopify theme push después de commitear
```

## Despliegue a producción

1. Cliente da acceso de Staff con permiso de Themes (o tu cuenta es Partner colaborador)
2. `shopify auth login` con esa cuenta
3. `shopify theme push --unpublished --store=cliente.myshopify.com` — sube como unpublished
4. Cliente revisa en `Online Store → Themes` y, cuando aprueba, click "Publish" desde admin
5. (No usar `--live` en producción salvo emergencia — siempre review primero)

## Reglas operativas

- **Nunca tocar el theme live directamente** desde admin (eso desincroniza con el repo)
- Cambios siempre vía push desde repo
- Si el cliente edita `settings_data.json` desde admin → `shopify theme pull --only config/settings_data.json` y commitear
- Para themes con varios entornos (staging + producción), usar 2 dev stores distintas o 2 themes (live + unpublished) en la misma tienda
