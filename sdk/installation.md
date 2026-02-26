# Installation

Add Kairos Analytics to any web application with npm or a single script tag. The entire setup takes about 3 minutes.

**Package:** `@kairoslab/analytics-sdk`

---

## What the SDK does

The SDK runs entirely in the user's browser. When you call `track()`, it:

1. Canonicalizes the event data (deterministic JSON serialization)
2. Hashes the canonical data with **keccak256** -- the hash is born client-side and never modified by any server
3. Sends the event data and `client_hash` to Supabase
4. Returns an `EventReceipt` containing the `clientHash` and `eventId`

Later, the relayer's built-in Merkle tree builder batches those `client_hash` values into Merkle trees and anchors the root on Base mainnet via `KPPEAnchor.anchorBatch()`. Only the 32-byte root goes on-chain -- your event data stays **private**.

---

## npm install (recommended)

For applications using a bundler (Vite, Webpack, Next.js, etc.):

```bash
npm install @kairoslab/analytics-sdk
```

Or with yarn / pnpm:

```bash
yarn add @kairoslab/analytics-sdk
pnpm add @kairoslab/analytics-sdk
```

Then in your app entry point:

```javascript
import { KairosAnalytics } from '@kairoslab/analytics-sdk'

const kairos = new KairosAnalytics({
  appId: 'your-app-id',
  supabaseUrl: 'https://your-project.supabase.co',
  supabaseAnonKey: 'your-supabase-anon-key',
})

// Track a page view
kairos.track('page_view', { path: window.location.pathname })
```

The SDK uses ethers v6 internally for keccak256 hashing. It is bundled with the package -- you do not need to install ethers separately.

---

## Browser script tag (zero build step)

For static HTML pages or applications without a bundler:

```html
<!-- Kairos Analytics -->
<script src="https://cdn.jsdelivr.net/npm/@kairoslab/analytics-sdk/dist/browser.min.js"></script>
<script>
  const kairos = new window.KairosAnalytics({
    appId: 'your-app-id',
    supabaseUrl: 'https://your-project.supabase.co',
    supabaseAnonKey: 'your-supabase-anon-key',
  });

  // Track a page view
  kairos.track('page_view', { path: window.location.pathname });
</script>
```

When loaded via script tag, the SDK is available as `window.KairosAnalytics`.

---

## Configuration options

```javascript
const kairos = new KairosAnalytics({
  appId: 'your-app-id',               // Required
  supabaseUrl: 'https://...co',       // Required
  supabaseAnonKey: 'eyJhbG...',       // Required
  debug: false,                        // Optional
  onReceipt: (receipt) => { ... },     // Optional
  receiptStorage: 'localStorage',      // Optional
})
```

| Option | Type | Default | Description |
|---|---|---|---|
| `appId` | string | -- | **Required.** Your registered App ID |
| `supabaseUrl` | string | -- | **Required.** Your Supabase project URL |
| `supabaseAnonKey` | string | -- | **Required.** Your Supabase anonymous key |
| `debug` | boolean | `false` | Enable verbose logging to the browser console |
| `onReceipt` | function | -- | Callback fired after each event is tracked, receives `EventReceipt` |
| `receiptStorage` | string | `'localStorage'` | Where to store receipts: `'localStorage'`, `'sessionStorage'`, or `'none'` |

### appId

Your registered application identifier. Must match an App ID registered in the App Registry contract. Lowercase letters, numbers, and hyphens only, 3 to 32 characters.

### supabaseUrl

The URL of your Supabase project. Found in your Supabase dashboard under Project Settings > API.

### supabaseAnonKey

The anonymous (public) key for your Supabase project. This key is safe to expose in client-side code -- it only has access to the `analytics_events` table via Row Level Security policies.

### debug

When `true`, the SDK logs detailed information to the browser console:

```
[Kairos] SDK initialized | appId: your-app-id
[Kairos] Event hashed client-side: 0x7a3f... (swap_attempted)
[Kairos] Event stored in Supabase | eventId: evt-abc123
[Kairos] Receipt saved to localStorage
```

### onReceipt

A callback function called every time an event is tracked successfully. Receives the `EventReceipt` object:

```javascript
const kairos = new KairosAnalytics({
  appId: 'your-app-id',
  supabaseUrl: '...',
  supabaseAnonKey: '...',
  onReceipt: (receipt) => {
    console.log('Event tracked:', receipt.eventId)
    console.log('Client hash:', receipt.clientHash)
    // Send to your backend, store in state, etc.
  },
})
```

### receiptStorage

Controls where `EventReceipt` objects are stored for later verification:

| Value | Behavior |
|---|---|
| `'localStorage'` | Receipts persist across browser sessions (default) |
| `'sessionStorage'` | Receipts cleared when the tab closes |
| `'none'` | Receipts not stored; use `onReceipt` callback instead |

---

## Placement on the page

For the script tag approach, place the snippet before `</body>` as the last script on the page to avoid blocking rendering:

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
    <script src="https://cdn.jsdelivr.net/npm/@kairoslab/analytics-sdk/dist/browser.min.js"></script>
    <script>
      const kairos = new window.KairosAnalytics({ ... });
      kairos.track('page_view', { path: window.location.pathname });
    </script>
  </body>
</html>
```

---

## Framework examples

### React

```javascript
// analytics.js
import { KairosAnalytics } from '@kairoslab/analytics-sdk'

export const kairos = new KairosAnalytics({
  appId: 'your-app-id',
  supabaseUrl: process.env.REACT_APP_SUPABASE_URL,
  supabaseAnonKey: process.env.REACT_APP_SUPABASE_ANON_KEY,
})
```

```javascript
// App.jsx
import { useEffect } from 'react'
import { useLocation } from 'react-router-dom'
import { kairos } from './analytics'

function App() {
  const location = useLocation()

  useEffect(() => {
    kairos.track('page_view', { path: location.pathname })
  }, [location.pathname])

  return <div>...</div>
}
```

### Next.js

```javascript
// lib/analytics.ts
import { KairosAnalytics } from '@kairoslab/analytics-sdk'

export const kairos = typeof window !== 'undefined'
  ? new KairosAnalytics({
      appId: 'your-app-id',
      supabaseUrl: process.env.NEXT_PUBLIC_SUPABASE_URL!,
      supabaseAnonKey: process.env.NEXT_PUBLIC_SUPABASE_ANON_KEY!,
    })
  : null
```

```javascript
// app/layout.tsx
'use client'
import { usePathname } from 'next/navigation'
import { useEffect } from 'react'
import { kairos } from '@/lib/analytics'

export default function Layout({ children }) {
  const pathname = usePathname()

  useEffect(() => {
    kairos?.track('page_view', { path: pathname })
  }, [pathname])

  return <html><body>{children}</body></html>
}
```

### Vue

```javascript
// plugins/analytics.js
import { KairosAnalytics } from '@kairoslab/analytics-sdk'

export const kairos = new KairosAnalytics({
  appId: 'your-app-id',
  supabaseUrl: import.meta.env.VITE_SUPABASE_URL,
  supabaseAnonKey: import.meta.env.VITE_SUPABASE_ANON_KEY,
})
```

```javascript
// router/index.js
import { kairos } from '../plugins/analytics'

router.afterEach((to) => {
  kairos.track('page_view', { path: to.path })
})
```

---

## Comparison with Google Analytics 4

| | GA4 | Kairos Analytics |
|---|---|---|
| Installation | 1 snippet | 1 snippet or npm install |
| Track event | `gtag('event', 'name', {...})` | `kairos.track('name', {...})` |
| Page view | Automatic | `kairos.track('page_view', { path })` |
| Client-side hashing | No | Yes (keccak256) |
| On-chain proof | No | Yes (Base mainnet) |
| Receipts | No | Yes (EventReceipt with hash) |
| Data ownership | Google | You (Supabase + on-chain proof) |
| Blockable by ad blockers | Often | Rarely |
| Works without cookies | No | Yes |

---

## Key SDK methods

### `track(eventName, properties)` -- returns `EventReceipt`

```javascript
const receipt = await kairos.track('swap_attempted', {
  fromToken: 'ETH',
  toToken: 'USDC',
  amount: 1.5,
})

console.log(receipt.clientHash) // 0x7a3f... (keccak256 hash, computed in browser)
console.log(receipt.eventId)    // evt-abc123
```

Every `track()` call returns an `EventReceipt` containing the event's `clientHash`. Keep this receipt -- you will use it to verify your event on-chain.

### `verify(receipt)` -- checks Merkle proof on-chain

```javascript
const result = await kairos.verify(receipt)

console.log(result.hashMatch)  // true -- your client hash matches
console.log(result.proofValid) // true -- Merkle proof is valid
console.log(result.anchored)   // true -- root is on Base mainnet
console.log(result.txHash)     // 0xdef... (Basescan link)
```

The `verify()` method retrieves the Merkle proof from Supabase and checks it against the on-chain root stored in the KPPEAnchor contract. No permission is required -- verification is fully permissionless.
