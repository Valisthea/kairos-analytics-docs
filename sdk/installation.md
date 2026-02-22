# Installation & Snippet

Add Kairos Analytics to any web application with a single snippet — no build step required.

---

## The universal snippet

Paste this before `</body>` in any HTML page:

```html
<!-- Kairos Analytics -->
<script type="module">
  import { init, page, track } from 'https://esm.sh/@valisthea/analytics'
  ;(async () => {
    try {
      await init({ appId: 'your-app-id', mode: 'offchain' })
      page(window.location.pathname)
      window._k = { track }
    } catch (e) {
      window._k = { track: () => {} }
    }
  })()
</script>
```

The `try/catch` wrapping ensures that if the SDK fails for any reason (network issue, CDN outage), your application continues to work normally. The fallback `window._k = { track: () => {} }` silently absorbs any `_k.track()` calls elsewhere on the page.

---

## npm install (for bundled apps)

If you use a bundler (Vite, Webpack, etc.):

```bash
npm install @valisthea/analytics
```

Then in your app entry point:

```javascript
import { init, page, track } from '@valisthea/analytics'

await init({ appId: 'your-app-id', mode: 'offchain' })
page(window.location.pathname)

// Export for use anywhere
export { track }
```

---

## Configuration options

```javascript
await init({
  appId: 'your-app-id',   // Required. Your registered app identifier.
  mode: 'offchain',        // 'offchain' (recommended) or 'onchain'
  debug: false,            // true = verbose console logs
  relayerUrl: '...',       // Optional. Override the default relayer URL.
})
```

| Option | Type | Default | Description |
|--------|------|---------|-------------|
| `appId` | string | — | **Required.** Your registered App ID |
| `mode` | string | `'offchain'` | `'offchain'` sends via HTTP. `'onchain'` signs from user wallet. |
| `debug` | boolean | `false` | Enable verbose logging to the console |
| `relayerUrl` | string | Default relayer | Override if self-hosting |

---

## `page()` — tracking page views

Call `page()` once per page load with the current path:

```javascript
// Static HTML — on each page
page(window.location.pathname)

// Single-page app — call on every route change
router.on('navigate', (path) => page(path))

// React Router example
useEffect(() => {
  page(location.pathname)
}, [location.pathname])
```

---

## Placement on the page

The snippet must load **after** your page content, before `</body>`. It should be the last script on the page to avoid blocking rendering.

```html
<html>
  <head>
    <!-- your head content -->
  </head>
  <body>
    <!-- your page content -->

    <!-- your other scripts -->
    <script src="app.js"></script>

    <!-- Kairos Analytics last -->
    <script type="module">
      import { init, page, track } from 'https://esm.sh/@valisthea/analytics'
      ...
    </script>
  </body>
</html>
```

---

## Comparison with Google Analytics 4

| | GA4 | Kairos Analytics |
|---|---|---|
| Installation | 1 snippet | 1 snippet |
| Track event | `gtag('event', 'name', {...})` | `_k.track('name', 'category', {...})` |
| Page view | Automatic | `page(pathname)` |
| Data ownership | Google | You (Base mainnet) |
| Blockable by ad blockers | Often | Rarely |
| Works without cookies | No | Yes |
