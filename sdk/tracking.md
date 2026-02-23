# Tracking Events

How to capture user actions across your dApp.

---

## Automatic tracking

Once the snippet is installed, these events fire with zero extra code:

- `page_view` — path, referrer, device type
- `click` — element tag, text, position on page
- `session_start` / `session_end` — duration, pages visited, depth

---

## Manual tracking functions

### `trackSwap(fromToken, toToken, amount)`

Track a swap attempt in a DEX:

```javascript
// On swap button click
KairosAnalytics.trackSwap('ETH', 'USDC', 1.5);

// With success/failure
try {
  await executeSwap();
  KairosAnalytics.trackEvent('swap_success', { pair: 'ETH/USDC', amount: 1.5 });
} catch (err) {
  KairosAnalytics.trackEvent('swap_failed', { pair: 'ETH/USDC', reason: err.message });
}
```

---

### `trackTransaction(txHash, amount, success)`

Track an on-chain transaction confirmation:

```javascript
const receipt = await tx.wait();
KairosAnalytics.trackTransaction(
  receipt.hash,
  amount,
  receipt.status === 1  // true = success, false = reverted
);
```

---

### `trackWalletConnect(address, walletType)`

Track when a user connects their wallet to your dApp:

```javascript
// After wallet connection succeeds
KairosAnalytics.trackWalletConnect('0xabc...', 'MetaMask');
```

> ℹ️ **Note:** This tracks the user's wallet connection event in your analytics. It has nothing to do with on-chain proof — proof is handled server-side by the relayer without any wallet interaction from your users.

---

### `trackPageView(path)`

For single-page apps (React, Vue, etc.) — call on every route change:

```javascript
// React Router
useEffect(() => {
  KairosAnalytics.trackPageView(location.pathname);
}, [location.pathname]);

// Vue Router
router.afterEach((to) => {
  KairosAnalytics.trackPageView(to.path);
});
```

---

### `trackEvent(name, properties)`

Track any custom event:

```javascript
// UI interaction
KairosAnalytics.trackEvent('filter_applied', { filter: 'last_7_days' });

// Feature usage
KairosAnalytics.trackEvent('csv_exported', { rows: 1500 });

// Error
KairosAnalytics.trackEvent('api_error', { endpoint: '/swap', code: 429 });
```

---

## Event naming conventions

Use `snake_case`. Structure as `object_action`:

```
✅ Good
  page_view
  swap_initiated
  wallet_connected
  transaction_failed
  modal_opened

❌ Avoid
  pageView        (camelCase)
  swap-initiated  (hyphens)
  SWAP            (uppercase)
  click           (too vague)
```

---

## Properties best practices

- **Flat** — no deeply nested objects
- **Consistent** — same key names across events
- **Non-identifying** — never include emails, names, passwords

```javascript
// ✅ Good
KairosAnalytics.trackEvent('plan_selected', {
  plan: 'builder',
  price_usd: 29,
  billing: 'monthly'
});

// ❌ Avoid
KairosAnalytics.trackEvent('plan_selected', {
  user_email: 'john@...',      // PII — never
  data: { plan: { id: 1 } }   // nested — hard to query
});
```

---

## Tracking a conversion funnel

A funnel is a sequence of events. Track each step and the dashboard computes drop-off automatically:

```javascript
// Step 1 — user views pricing
KairosAnalytics.trackPageView('/pricing');

// Step 2 — user selects a plan
KairosAnalytics.trackEvent('plan_selected', { plan: 'builder' });

// Step 3 — user enters checkout
KairosAnalytics.trackEvent('checkout_started', { plan: 'builder', price: 29 });

// Step 4 — payment confirmed
KairosAnalytics.trackEvent('checkout_completed', { plan: 'builder' });
```

The Conversion Funnel widget in the dashboard visualizes drop-off between any event sequence.
