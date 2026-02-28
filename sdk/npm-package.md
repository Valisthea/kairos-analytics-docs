# npm Package

The `@kairoslab/analytics-sdk` package provides programmatic access to Kairos Analytics from JavaScript/TypeScript environments.

## Installation

```bash
npm install @kairoslab/analytics-sdk
# or
yarn add @kairoslab/analytics-sdk
```

## Initialization

```javascript
import { KairosAnalytics } from '@kairoslab/analytics-sdk';

const kairos = new KairosAnalytics({
  appId: 'your-app-id',
  endpoint: 'https://kairos-relayer-production.up.railway.app',
  debug: false  // set true for console logging
});
```

## Methods

### `kairos.track(eventType, metadata)`

Track any custom event.

```javascript
await kairos.track('user_action', { action: 'claim', amount: 100 });
```

### `kairos.trackTransaction(txData)`

Track a blockchain transaction with full lifecycle support.

```javascript
await kairos.trackTransaction({
  hash: '0x...',
  from: '0x...',
  to: '0x...',
  value: '1000000000000000000',
  chain: 'base'
});
```

### `kairos.trackSwap(swapData)`

Track a DEX swap event.

```javascript
await kairos.trackSwap({
  tokenIn: 'ETH',
  tokenOut: 'USDC',
  amountIn: '1.0',
  amountOut: '2650.00',
  dex: 'uniswap-v3'
});
```

### `kairos.trackConversion(conversionData)`

Track a business conversion event.

```javascript
await kairos.trackConversion({
  type: 'deposit',
  value: 5000,
  currency: 'USDC'
});
```

### `kairos.identify(walletAddress)`

Associate subsequent events with a wallet address.

```javascript
kairos.identify('0x1234...abcd');
```

## Server-Side Usage (Node.js)

```javascript
const { KairosAnalytics } = require('@kairoslab/analytics-sdk');

const kairos = new KairosAnalytics({
  appId: 'your-app-id',
  endpoint: 'https://kairos-relayer-production.up.railway.app'
});

// Track backend events
app.post('/api/swap', async (req, res) => {
  // ... process swap ...
  await kairos.track('swap_backend', {
    user: req.user.wallet,
    pair: 'ETH/USDC',
    amount: req.body.amount
  });
  res.json({ ok: true });
});
```

## When to Use Snippet vs npm

Use the **snippet** for browser-based dApps (automatic tracking, zero config).
Use the **npm SDK** for server-side tracking, custom integrations, or when you need full programmatic control.

Both can be used together — the snippet handles auto-tracking in the browser while the SDK handles server-side events. They share the same app ID and data flows into the same dashboard.
