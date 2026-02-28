# Quick Start — npm SDK

For developers who prefer a programmatic integration over the snippet.

> **Recommendation**: For most dApps, the [2-line snippet](quickstart-snippet.md) is faster and simpler. Use the npm SDK when you need fine-grained control over event tracking or want to integrate Kairos into a Node.js backend.

## Installation

```bash
npm install @kairoslab/analytics-sdk
```

## Setup

```javascript
import { KairosAnalytics } from '@kairoslab/analytics-sdk';

const kairos = new KairosAnalytics({
  appId: 'your-app-id',
  endpoint: 'https://kairos-relayer-production.up.railway.app'
});
```

## Track Events

```javascript
// Track a custom event
kairos.track('button_click', {
  button: 'swap-confirm',
  token: 'ETH',
  amount: '1.5'
});

// Track a transaction
kairos.trackTransaction({
  hash: '0x123...',
  from: '0xabc...',
  to: '0xdef...',
  value: '1000000000000000000',
  chain: 'base'
});

// Track a swap
kairos.trackSwap({
  tokenIn: 'ETH',
  tokenOut: 'USDC',
  amountIn: '1.0',
  amountOut: '2650.00',
  dex: 'uniswap-v3'
});

// Track a conversion
kairos.trackConversion({
  type: 'signup',
  value: 0,
  metadata: { referrer: 'twitter' }
});
```

## Differences from Snippet

| Feature | Snippet | npm SDK |
|---------|---------|---------|
| Auto-tracking (15+ events) | ✅ | ❌ Manual |
| Web3 wallet auto-detect | ✅ | ❌ Manual |
| Custom events | ✅ via `data-track` | ✅ Programmatic |
| Server-side tracking | ❌ | ✅ |
| Bundle size | 0 (external script) | ~12KB |
| Setup time | 30 seconds | 5 minutes |

## When to Use the npm SDK

* You need server-side event tracking (Node.js backend)
* You want to track events that the snippet can't detect (e.g., API calls, backend processes)
* You need conditional tracking logic (track only for specific users or conditions)
* You're building a React Native or non-browser application

For browser-based dApps, the snippet is almost always the better choice.
