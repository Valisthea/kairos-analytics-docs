# Tracking Events

Kairos tracks events at two levels: automatic (zero-config) and manual (custom code). This page covers both.

## Event Structure

Every event sent to Kairos has this structure:

```json
{
  "appId": "your-app-id",
  "type": "page_view",
  "timestamp": 1709142000000,
  "sessionId": "s_a1b2c3d4",
  "url": "https://yourdapp.com/swap",
  "meta": {
    "referrer": "https://twitter.com",
    "viewport": "1920x1080",
    "userAgent": "Mozilla/5.0..."
  }
}
```

The `meta` object varies by event type and can contain any key-value pairs for custom events.

## Automatic Events

The snippet auto-tracks these events with no configuration. See the [Snippet documentation](snippet.md) for the complete list and data captured for each.

To disable specific auto-tracked events:

```javascript
window.KAIROS_DISABLE_EVENTS = ['scroll_depth', 'performance'];
```

## Custom Events

### Simple Tracking

```javascript
window.kairos?.track('event_name', { key: 'value' });
```

Examples:

```javascript
// User clicked a specific CTA
window.kairos?.track('cta_click', { 
  button: 'hero-start-free', 
  location: 'homepage' 
});

// User viewed a specific section
window.kairos?.track('section_view', { 
  section: 'pricing', 
  plan_highlighted: 'builder' 
});

// User completed a form step
window.kairos?.track('form_step', { 
  step: 3, 
  total_steps: 5, 
  form: 'onboarding' 
});
```

### Transaction Tracking

For blockchain transactions, use the dedicated method that captures the full lifecycle:

```javascript
window.kairos?.trackTransaction({
  hash: '0x123abc...',
  from: '0xUserWallet...',
  to: '0xContractAddress...',
  value: '1000000000000000000', // in wei
  chain: 'base',
  method: 'swap',              // optional: contract method name
  status: 'submitted'          // submitted | confirmed | failed
});
```

The snippet automatically tracks `tx_confirmed` and `tx_failed` if you only track `tx_submitted` — it watches the transaction hash for confirmation.

### Swap Tracking

For DEX swaps:

```javascript
window.kairos?.trackSwap({
  tokenIn: 'ETH',
  tokenOut: 'USDC',
  amountIn: '1.0',
  amountOut: '2650.00',
  dex: 'uniswap-v3',
  pool: '0xPoolAddress...',
  slippage: '0.5'
});
```

### Conversion Tracking

For business-critical actions:

```javascript
window.kairos?.trackConversion({
  type: 'signup',       // signup | purchase | swap | deposit | custom
  value: 29.99,         // monetary value (optional)
  currency: 'USD',      // currency code (optional)
  metadata: {           // any additional context
    plan: 'builder',
    referrer: 'twitter-campaign-q1'
  }
});
```

## HTML Attribute Tracking

For simple click tracking without JavaScript, use `data-track`:

```html
<button data-track="confirm-swap">Confirm</button>
<a data-track="external-docs" href="https://docs.example.com">Docs</a>
```

You can add metadata with `data-track-*` attributes:

```html
<button 
  data-track="add-liquidity" 
  data-track-pool="ETH-USDC" 
  data-track-amount="1000"
>
  Add Liquidity
</button>
```

This fires an event with type `add-liquidity` and meta `{ pool: 'ETH-USDC', amount: '1000' }`.

## Event Naming Conventions

We recommend using **snake_case** for event names and keeping them descriptive:

| ✅ Good | ❌ Avoid |
|---------|---------|
| `swap_confirmed` | `SwapConfirmed` |
| `wallet_connected` | `connect` |
| `liquidity_added` | `event1` |
| `page_view_pricing` | `pv` |

## Rate Limits

The relayer accepts up to **100 events per second per app ID**. Events beyond this limit are queued and sent on the next batch. If your dApp generates more than 100 events/second sustained, consider sampling or batching on the client side.

## Event Verification

Every event you track is hashed and included in a Merkle batch. You can verify any event's integrity — see [Verification API](verification.md).
