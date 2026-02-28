# Quick Start — Snippet

Get Kairos Analytics running on your dApp in under 30 seconds.

## 1. Create Your Account

Go to [kairosanalytics.org/register](https://kairosanalytics.org/register.html) and sign up with Google, GitHub, or email. You'll choose an **App ID** (a unique identifier for your dApp) during onboarding.

## 2. Add the Snippet

Paste these 2 lines into your HTML, just before `</head>` or `</body>`:

```html
<script>window.KAIROS_APP_ID = "your-app-id";</script>
<script src="https://kairosanalytics.org/snippet.js" async></script>
```

Replace `your-app-id` with the App ID you chose during registration.

## 3. You're Live

That's it. Open your dApp in a browser, navigate around, and check your dashboard at [kairosanalytics.org/dashboard](https://kairosanalytics.org/dashboard).

The snippet immediately starts tracking:

* Page views (with URL, referrer, viewport)
* Clicks (with element selector, text, position)
* Scroll depth (25%, 50%, 75%, 100%)
* Session duration and tab visibility
* Performance metrics (load time, TTFB)
* JavaScript errors
* Wallet detection (if any Web3 wallet is installed)

## Framework Examples

### React / Next.js

Add to your `index.html` or `_document.tsx`:

```html
<script>window.KAIROS_APP_ID = "your-app-id";</script>
<script src="https://kairosanalytics.org/snippet.js" async></script>
```

Or load dynamically in a component:

```jsx
useEffect(() => {
  window.KAIROS_APP_ID = "your-app-id";
  const s = document.createElement('script');
  s.src = "https://kairosanalytics.org/snippet.js";
  s.async = true;
  document.head.appendChild(s);
}, []);
```

### Vue / Nuxt

In `nuxt.config.ts`:

```typescript
export default defineNuxtConfig({
  app: {
    head: {
      script: [
        { innerHTML: 'window.KAIROS_APP_ID = "your-app-id";' },
        { src: 'https://kairosanalytics.org/snippet.js', async: true }
      ]
    }
  }
})
```

### Plain HTML

```html
<!DOCTYPE html>
<html>
<head>
  <script>window.KAIROS_APP_ID = "your-app-id";</script>
  <script src="https://kairosanalytics.org/snippet.js" async></script>
</head>
<body>
  <!-- your dApp -->
</body>
</html>
```

## Verify It's Working

1. Open your dApp in a browser
2. Open DevTools → Network tab
3. Filter by `track` — you should see POST requests to the Kairos relayer
4. Go to your [dashboard](https://kairosanalytics.org/dashboard) — the Live Feed widget shows events in real time

## What's Next

* [Track custom events →](../sdk/tracking.md)
* [Enable Web3 wallet tracking →](../sdk/web3-detect.md)
* [Understand your dashboard →](../dashboard/app-dashboard.md)
