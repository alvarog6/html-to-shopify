# Verificación con Playwright

Mismo flujo que `html-elementor-skill`. Cambia solo la URL del target: del preview de WP Studio al preview local de `shopify theme dev` (típicamente `http://127.0.0.1:9292`).

## Setup mínimo en Windows

(Reutilizar el setup ya hecho para html-elementor-skill — Playwright Core + Chrome del sistema + `--use-system-ca`.)

```js
// shot.cjs
const { chromium } = require('playwright-core');
(async () => {
  const browser = await chromium.launch({
    executablePath: 'C:/Program Files/Google/Chrome/Application/chrome.exe'
  });
  const page = await browser.newPage();
  await page.goto(process.argv[2], { waitUntil: 'networkidle' });
  await page.setViewportSize({ width: parseInt(process.argv[4]) || 1280, height: 900 });
  await page.screenshot({ path: process.argv[3], fullPage: true });
  await browser.close();
})();
```

## Workflow medir → construir → comparar

### Paso 1.5 — medir computados del HTML origen

```js
// measure-source.cjs <ruta-archivo-html>
const { chromium } = require('playwright-core');
(async () => {
  const browser = await chromium.launch({ executablePath: 'C:/Program Files/Google/Chrome/Application/chrome.exe' });
  const page = await browser.newPage();
  await page.goto('file:///' + process.argv[2]);
  for (const vw of [320, 768, 1280]) {
    await page.setViewportSize({ width: vw, height: 900 });
    await page.waitForTimeout(200);
    const data = await page.evaluate(() => {
      const selectors = ['.hero h1', '.hero p', '.btn', '.product-card', '.product-card h3', '.product-card .price'];
      return selectors.map(sel => {
        const el = document.querySelector(sel);
        if (!el) return { sel, error: 'not found' };
        const cs = getComputedStyle(el);
        const r = el.getBoundingClientRect();
        return {
          sel,
          font: `${cs.fontFamily} ${cs.fontWeight} ${cs.fontSize} / ${cs.lineHeight}`,
          color: cs.color,
          bg: cs.backgroundColor,
          padding: cs.padding,
          margin: cs.margin,
          width: r.width,
          height: r.height,
        };
      });
    });
    console.log(`=== ${vw}px ===`);
    console.log(JSON.stringify(data, null, 2));
  }
  await browser.close();
})();
```

### Paso 5 — comparar computados origen vs theme dev

Mismo script, target `http://127.0.0.1:9292/` (preview de `shopify theme dev`). Diff visual o `diff` de salidas JSON.

## Auth / consent

`shopify theme dev` puede redirigir a una pantalla de password si la tienda es de desarrollo cerrada. Ignorar visualmente o configurar **storefront password** y agregar handling al script (relleno + click) si es necesario.

## Breakpoints recomendados a medir

| Breakpoint | Por qué |
|---|---|
| 320px | mobile small (límite duro de la mayoría de targets) |
| 768px | tablet portrait |
| 1280px | desktop estándar |

Si el HTML origen tiene un breakpoint a otra resolución específica (1440, 1920), agregar.

## Screenshot final

Solo cuando los computados ya matchean. Si la imagen difiere y los computados no → probablemente es font-rendering del OS, no del CSS.
