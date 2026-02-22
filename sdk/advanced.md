# Advanced Patterns

Patterns for more complex integration scenarios.

---

## Tracking events from existing functions

If you have existing JavaScript functions you cannot rewrite, dispatch custom DOM events from them and let the SDK listen separately. This keeps your tracking code decoupled from your business logic.

**Step 1 — Dispatch a custom event inside the existing function:**

```javascript
// Your existing function — add one line
async function processPayment(amount) {
  const result = await submitToChain(amount)

  // Add this line after the action completes
  window.dispatchEvent(new CustomEvent('app:payment:success', {
    detail: { amount, tx_hash: result.hash }
  }))
}
```

**Step 2 — Listen for it in your analytics snippet:**

```javascript
window.addEventListener('app:payment:success', (e) => {
  _k.track('payment_completed', 'transaction', e.detail)
})
```

This approach means you can add tracking to any part of your app without touching the core logic.

---

## Tracking single-page app route changes

In SPAs, `page()` must be called on every route change, not just on initial load.

**React Router:**
```javascript
import { useLocation } from 'react-router-dom'
import { useEffect } from 'react'

function Analytics() {
  const location = useLocation()
  useEffect(() => {
    window._k?.track && page(location.pathname)
  }, [location.pathname])
  return null
}

// Add <Analytics /> inside your Router
```

**Vue Router:**
```javascript
router.afterEach((to) => {
  page(to.path)
})
```

**Vanilla JS history API:**
```javascript
const originalPushState = history.pushState
history.pushState = function(...args) {
  originalPushState.apply(this, args)
  page(window.location.pathname)
}
window.addEventListener('popstate', () => page(window.location.pathname))
```

---

## Tracking before a redirect

If you need to track an event immediately before the user is redirected away:

```javascript
function goToCheckout(planId) {
  _k.track('checkout_redirect', 'transaction', { plan: planId })
  // Small delay to ensure the event dispatches before navigation
  setTimeout(() => {
    window.location.href = '/checkout?plan=' + planId
  }, 100)
}
```

---

## Waiting for the SDK before calling track

If your scripts run before the SDK has finished initializing, events may be lost. The safest pattern:

```javascript
// Option 1 — wrap your script in a DOMContentLoaded listener
// (the SDK snippet loads at the end of <body>, so it runs before this fires)
document.addEventListener('DOMContentLoaded', () => {
  _k.track('page_interactive', 'page')
})

// Option 2 — check _k exists before calling
function safeTrack(event, category, props) {
  if (window._k?.track) {
    _k.track(event, category, props)
  }
}
```

---

## Identifying wallet connections

When a user connects their wallet, you can associate subsequent events with that wallet address by passing it in the properties:

```javascript
_k.track('wallet_connected', 'wallet', {
  address: userWalletAddress,
  chain_id: chainId,
  provider: 'metamask'   // 'metamask', 'walletconnect', 'coinbase', etc.
})
```

The dashboard groups events by wallet address when this property is present, enabling per-wallet funnel analysis.

---

## Debouncing high-frequency events

For events that fire frequently (search input, scroll, resize), debounce before tracking:

```javascript
let searchTimeout
searchInput.addEventListener('input', (e) => {
  clearTimeout(searchTimeout)
  searchTimeout = setTimeout(() => {
    if (e.target.value.length > 2) {
      _k.track('search_performed', 'ui', { query: e.target.value })
    }
  }, 800)
})
```

This avoids flooding the relayer with partial keystrokes.
