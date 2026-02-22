# Tracking Events

How to capture user actions across your application.

---

## The `track()` function

```javascript
_k.track(eventName, category, properties)
```

| Parameter | Type | Required | Description |
|-----------|------|----------|-------------|
| `eventName` | string | ✅ Yes | Name of the event in `snake_case` |
| `category` | string | ✅ Yes | Event category — see [Event Categories](categories.md) |
| `properties` | object | No | Any key-value pairs relevant to the event |

---

## Basic examples

**Button click:**
```javascript
button.addEventListener('click', () => {
  _k.track('cta_clicked', 'ui', { label: 'hero_signup' })
})
```

**Form submission:**
```javascript
form.addEventListener('submit', () => {
  _k.track('form_submitted', 'ui', { form_id: 'onboarding' })
})
```

**Wallet connection:**
```javascript
async function connectWallet() {
  _k.track('wallet_connect_initiated', 'wallet')
  try {
    const address = await provider.connect()
    _k.track('wallet_connected', 'wallet', { address, chain: chainId })
  } catch (err) {
    _k.track('wallet_connect_failed', 'wallet', { reason: err.message })
  }
}
```

**Transaction flow:**
```javascript
// Initiated
_k.track('purchase_initiated', 'transaction', { item_id: 'item_123', price: 10.00 })

// Confirmed
_k.track('purchase_completed', 'transaction', { item_id: 'item_123', tx_hash: '0x...' })

// Failed
_k.track('purchase_failed', 'transaction', { item_id: 'item_123', reason: 'insufficient_balance' })
```

---

## Naming conventions

Use `snake_case` for all event names. Structure names as `noun_verb` or `object_action`:

```
✅ Good:
  page_view
  button_clicked
  wallet_connected
  purchase_completed
  modal_opened
  filter_applied

❌ Avoid:
  pageView         (camelCase)
  clicked-button   (hyphens)
  PURCHASE         (uppercase)
  click            (too vague)
```

---

## Properties best practices

Properties are free-form key-value pairs. Keep them:

- **Flat** — avoid deeply nested objects
- **Consistent** — use the same key names for the same concepts across events
- **Descriptive** — prefer `payment_method: 'crypto'` over `method: 1`
- **Non-identifying** — never include emails, real names, or passwords

```javascript
// Good properties
_k.track('item_purchased', 'transaction', {
  item_id: 'plan_pro',
  price_usd: 49.00,
  payment_method: 'crypto',
  currency: 'USDC'
})

// Avoid
_k.track('item_purchased', 'transaction', {
  user_email: 'john@...',   // PII — never do this
  data: { nested: { deeply: true } }  // hard to query
})
```

---

## Tracking a conversion funnel

A funnel is just a sequence of events. Define the steps you care about, then track each one:

```javascript
// Step 1 — user lands on the page
page('/pricing')

// Step 2 — user interacts with the product
_k.track('pricing_plan_viewed', 'ui', { plan: 'pro' })

// Step 3 — user initiates purchase
_k.track('checkout_started', 'transaction', { plan: 'pro', price: 49 })

// Step 4 — user completes purchase
_k.track('checkout_completed', 'transaction', { plan: 'pro', tx_hash: '0x...' })
```

The dashboard's Conversion Funnel widget automatically computes drop-off rates between any sequence of events.

---

## Error tracking

Track failures the same way you track successes — just use a consistent naming pattern:

```javascript
try {
  await submitTransaction()
  _k.track('transaction_success', 'transaction', { amount, token })
} catch (err) {
  _k.track('transaction_failed', 'transaction', {
    amount,
    token,
    error_code: err.code,
    reason: err.message.slice(0, 100)  // truncate long messages
  })
}
```

The Admin Panel's Alerts section automatically surfaces spikes in `_failed` events.
