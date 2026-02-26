# Tracking Events

How to capture user actions across your dApp with cryptographic receipts.

---

## The `track()` method

Every event is tracked with the `track()` method, which returns a promise resolving to an `EventReceipt`:

```javascript
const receipt = await kairos.track(eventType, data)
```

| Parameter | Type | Description |
|---|---|---|
| `eventType` | string | Name of the event (e.g., `'swap_attempted'`, `'page_view'`) |
| `data` | object | Event properties (flat key-value pairs) |

**Returns:** `Promise<EventReceipt>`

```typescript
interface EventReceipt {
  eventId: string      // Unique event identifier
  clientHash: string   // keccak256 hash computed client-side
  timestamp: number    // Unix timestamp (ms)
  eventType: string    // The event type passed to track()
  appId: string        // Your App ID
  sessionId: string    // Current session ID
}
```

---

## How client-side hashing works

When you call `track()`, the SDK performs the following steps before sending anything to the server:

1. **Assemble the event object** -- combines `eventType`, `data`, `appId`, `sessionId`, and `timestamp`
2. **Canonicalize** -- sorts the object keys alphabetically and serializes with `JSON.stringify(event, Object.keys(event).sort())`
3. **Hash** -- computes `keccak256(canonicalJSON)` using ethers v6
4. **Send** -- writes the event data plus the `client_hash` to Supabase
5. **Return receipt** -- returns an `EventReceipt` containing the `clientHash`

```javascript
// What the SDK does internally:
const event = {
  appId: 'your-app-id',
  data: { pair: 'ETH/USDC', amount: 1.5 },
  eventType: 'swap_attempted',
  sessionId: 'a3f9b2c1-...',
  timestamp: 1700000000000,
}

// Keys sorted alphabetically: appId, data, eventType, sessionId, timestamp
const canonical = JSON.stringify(event, Object.keys(event).sort())
const clientHash = ethers.keccak256(ethers.toUtf8Bytes(canonical))
// clientHash = 0x7a3f...
```

### Deterministic hashing

The hash is deterministic: the same data always produces the same hash, regardless of the order in which properties are specified in your code:

```javascript
// These produce the SAME hash:
kairos.track('swap_attempted', { amount: 1.5, pair: 'ETH/USDC' })
kairos.track('swap_attempted', { pair: 'ETH/USDC', amount: 1.5 })
```

This is because the SDK sorts keys before serialization. This ensures that key ordering in your JavaScript objects does not affect the cryptographic hash.

---

## Automatic tracking

Once the SDK is initialized, these events fire without any extra code:

| Event | Trigger | Properties |
|---|---|---|
| `page_view` | Every page load | `path`, `referrer`, `device` |
| `click` | Every button/link click | `tag`, `text`, `position` |
| `session_start` | New browser session | `sessionId` |
| `session_end` | Tab close or 30 min inactivity | `sessionId`, `duration`, `eventCount` |

---

## Common tracking patterns

### Track a swap

```javascript
// On swap button click
const receipt = await kairos.track('swap_attempted', {
  fromToken: 'ETH',
  toToken: 'USDC',
  amount: 1.5,
})

// After swap succeeds
await kairos.track('swap_success', {
  fromToken: 'ETH',
  toToken: 'USDC',
  amount: 1.5,
  txHash: '0xabc...',
})

// If swap fails
await kairos.track('swap_failed', {
  fromToken: 'ETH',
  toToken: 'USDC',
  reason: 'slippage_exceeded',
})
```

### Track a transaction

```javascript
const txReceipt = await tx.wait()
await kairos.track('transaction_confirmed', {
  txHash: txReceipt.hash,
  amount: 500,
  success: txReceipt.status === 1,
})
```

### Track a wallet connection

```javascript
await kairos.track('wallet_connected', {
  address: '0xabc...',
  walletType: 'MetaMask',
  chain: 'base',
})
```

### Track page views in a SPA

```javascript
// React Router
useEffect(() => {
  kairos.track('page_view', { path: location.pathname })
}, [location.pathname])

// Vue Router
router.afterEach((to) => {
  kairos.track('page_view', { path: to.path })
})

// Next.js App Router
const pathname = usePathname()
useEffect(() => {
  kairos.track('page_view', { path: pathname })
}, [pathname])
```

### Track a custom event

```javascript
// UI interaction
await kairos.track('filter_applied', { filter: 'last_7_days' })

// Feature usage
await kairos.track('csv_exported', { rows: 1500 })

// Error
await kairos.track('api_error', { endpoint: '/swap', code: 429 })
```

---

## Receipt storage

Every `track()` call produces an `EventReceipt`. Receipts are stored locally based on your `receiptStorage` configuration:

| Setting | Storage location | Persists across sessions |
|---|---|---|
| `'localStorage'` (default) | `window.localStorage` | Yes |
| `'sessionStorage'` | `window.sessionStorage` | No (cleared on tab close) |
| `'none'` | Not stored | N/A (use `onReceipt` callback) |

Receipts are stored under the key `kairos_receipts` as a JSON array. You can retrieve them:

```javascript
// Get all stored receipts
const receipts = kairos.getReceipts()

// Get receipt for a specific event
const receipt = kairos.getReceipt('evt-abc123')
```

These receipts are what you use to verify events later. See [Trustless Verification](verification.md).

---

## Event naming conventions

Use `snake_case`. Structure as `object_action`:

```
Good:
  page_view
  swap_attempted
  swap_success
  wallet_connected
  transaction_failed
  modal_opened
  trade_opened
  trade_closed

Avoid:
  pageView        (camelCase -- not consistent)
  swap-initiated  (hyphens -- not standard)
  SWAP            (uppercase -- hard to query)
  click           (too vague -- what was clicked?)
```

---

## Properties best practices

### Keep it flat

No deeply nested objects. Properties should be simple key-value pairs:

```javascript
// Good
kairos.track('trade_opened', {
  pair: 'BTC/USDT',
  amount: 1000,
  leverage: 10,
  direction: 'long',
})

// Avoid
kairos.track('trade_opened', {
  trade: {
    pair: { base: 'BTC', quote: 'USDT' },
    params: { amount: 1000 },
  },
})
```

### Be consistent

Use the same key names across related events:

```javascript
// Good -- same keys for the same concepts
kairos.track('trade_opened', { pair: 'BTC/USDT', amount: 1000 })
kairos.track('trade_closed', { pair: 'BTC/USDT', pnl: 42.5 })

// Avoid -- inconsistent naming
kairos.track('trade_opened', { token_pair: 'BTC/USDT', size: 1000 })
kairos.track('trade_closed', { pair: 'BTC/USDT', profit: 42.5 })
```

### Never include PII

Never pass personally identifiable information in event properties:

```javascript
// Good
kairos.track('plan_selected', {
  plan: 'builder',
  price_usd: 29,
  billing: 'monthly',
})

// Never do this
kairos.track('plan_selected', {
  user_email: 'john@example.com',   // PII
  full_name: 'John Doe',            // PII
  credit_card: '4242...',           // sensitive
})
```

---

## Tracking a conversion funnel

A funnel is a sequence of events. Track each step and the dashboard computes drop-off automatically:

```javascript
// Step 1 -- user views pricing
await kairos.track('page_view', { path: '/pricing' })

// Step 2 -- user selects a plan
await kairos.track('plan_selected', { plan: 'builder' })

// Step 3 -- user enters checkout
await kairos.track('checkout_started', { plan: 'builder', price: 29 })

// Step 4 -- payment confirmed
await kairos.track('checkout_completed', { plan: 'builder' })
```

The Conversion Funnel widget in the dashboard visualizes drop-off between any event sequence.
