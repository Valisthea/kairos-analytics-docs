# Snippet v2.0

The Kairos snippet is a lightweight JavaScript file that auto-tracks 15+ event types in the browser with zero configuration. It's the recommended integration method for all browser-based dApps.

## Installation

```html
<script>window.KAIROS_APP_ID = "your-app-id";</script>
<script src="https://kairosanalytics.org/snippet.js" async></script>
```

## Configuration

Set these globals **before** loading the snippet:

| Variable | Required | Default | Description |
|----------|----------|---------|-------------|
| `window.KAIROS_APP_ID` | ✅ | — | Your unique app identifier |
| `window.KAIROS_ENDPOINT` | ❌ | Kairos relayer | Custom relayer URL (for self-hosted) |
| `window.KAIROS_DEBUG` | ❌ | `false` | Logs events to console when `true` |
| `window.KAIROS_DISABLE_AUTO` | ❌ | `false` | Disables auto-tracking (manual only) |

Example with debug mode:

```html
<script>
  window.KAIROS_APP_ID = "your-app-id";
  window.KAIROS_DEBUG = true;
</script>
<script src="https://kairosanalytics.org/snippet.js" async></script>
```

## Auto-Tracked Events

These events are captured automatically without any code:

### Web2 Events

| Event | Trigger | Data Captured |
|-------|---------|---------------|
| `page_view` | Page load | URL, referrer, title, viewport size |
| `click` | Any click | Element tag, class, ID, text, coordinates |
| `scroll_depth` | Scroll milestones | Percentage (25%, 50%, 75%, 100%) |
| `form_submit` | Form submission | Form ID, action URL (no field values) |
| `outbound_click` | Click on external link | Destination URL, link text |
| `tab_hidden` | Tab loses focus | Duration visible |
| `tab_visible` | Tab regains focus | Duration hidden |
| `performance` | Page fully loaded | Load time, TTFB, DOM ready time |
| `error` | JavaScript error | Message, file, line, column, stack |
| `session_start` | New session detected | Session ID, referrer, UTM params |
| `session_end` | Tab close / navigate away | Session duration, pages visited |

### Web3 Events

| Event | Trigger | Data Captured |
|-------|---------|---------------|
| `wallet_detected` | Wallet provider found | Provider name, type |
| `wallet_connected` | User connects wallet | Address (truncated), chain ID, provider |
| `wallet_disconnected` | User disconnects | Previous address, provider |
| `chain_changed` | User switches chain | Previous chain, new chain |
| `tx_submitted` | Transaction sent | Hash, from, to, value, method |
| `tx_confirmed` | Transaction mined | Hash, block number, gas used |
| `tx_failed` | Transaction reverted | Hash, error message |
| `swap_executed` | DEX swap detected | Token pair, amounts, DEX name |

## Custom Events

### Using `data-track` Attribute

Add `data-track` to any HTML element to track interactions without JavaScript:

```html
<button data-track="swap-confirm">Confirm Swap</button>
<a data-track="docs-link" href="/docs">Documentation</a>
<input data-track="search-input" type="text" placeholder="Search...">
```

When the user clicks (or focuses, for inputs) the element, Kairos fires a custom event with the `data-track` value as the event type.

### Using JavaScript API

The snippet exposes a global function for programmatic tracking:

```javascript
// Basic custom event
window.kairos?.track('custom_event', {
  key: 'value',
  amount: 42
});

// Track a conversion
window.kairos?.trackConversion({
  type: 'purchase',
  value: 29.99,
  currency: 'USD'
});

// Track a transaction manually
window.kairos?.trackTransaction({
  hash: '0x123...',
  from: '0xabc...',
  to: '0xdef...',
  value: '1.5',
  chain: 'base'
});
```

> **Note**: Always use optional chaining (`window.kairos?.track`) because the snippet loads asynchronously. If you call `track` before the snippet has loaded, the optional chaining prevents errors.

## Privacy

The snippet does **not**:

* Set cookies
* Collect IP addresses
* Capture form field values (only form IDs and action URLs)
* Track keystrokes
* Fingerprint users with canvas or WebGL
* Send data to any third party (only to the Kairos relayer)

Wallet addresses are truncated in the dashboard display (e.g., `0x1234...abcd`) but stored in full for analytics purposes.

## Performance

The snippet is designed to have minimal impact on your dApp:

* Loads asynchronously — never blocks page render
* Debounced scroll tracking — fires at milestones, not on every pixel
* Batched network requests — events are queued and sent in batches
* Tiny footprint — external script, not bundled into your app

## Troubleshooting

**Events not appearing in dashboard?**

1. Check DevTools Console for errors
2. Verify `KAIROS_APP_ID` matches your registered app ID (case-sensitive)
3. Check Network tab for POST requests to the relayer — look for 200 responses
4. Enable `KAIROS_DEBUG = true` to see events logged to console

**Wallet events not firing?**

Wallet detection runs on page load. If the wallet extension injects after the snippet runs, the detection may miss it. The snippet retries detection for up to 5 seconds after load.
