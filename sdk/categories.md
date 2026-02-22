# Event Categories

Categories group your events by type and control how they appear in the dashboard.

---

## Built-in categories

| Category | Use for | Dashboard color |
|----------|---------|-----------------|
| `page` | Page views and navigation | Blue |
| `ui` | Button clicks, modals, filters, UI interactions | Amber |
| `transaction` | Purchases, payments, on-chain actions | Green |
| `wallet` | Wallet connect, disconnect, signature requests | Cyan |
| `custom` | Everything else that doesn't fit above | Purple |

---

## When to use each category

### `page`
Reserved for navigation events. The `page()` function uses this automatically. You rarely need to use it manually.

```javascript
page('/dashboard')   // automatically categorized as 'page'
```

### `ui`
Any interaction that changes what the user sees, but doesn't move money or affect chain state.

```javascript
_k.track('search_performed', 'ui', { query: 'defi' })
_k.track('filter_applied', 'ui', { filter: 'price_asc' })
_k.track('modal_opened', 'ui', { modal: 'onboarding' })
_k.track('tab_switched', 'ui', { from: 'overview', to: 'history' })
```

### `transaction`
Any event involving a financial action — whether on-chain or off-chain.

```javascript
_k.track('swap_initiated', 'transaction', { from: 'ETH', to: 'USDC', amount: 0.5 })
_k.track('swap_completed', 'transaction', { from: 'ETH', to: 'USDC', tx_hash: '0x...' })
_k.track('deposit_submitted', 'transaction', { amount: 100, token: 'USDC' })
_k.track('withdrawal_failed', 'transaction', { reason: 'slippage_exceeded' })
```

### `wallet`
Wallet lifecycle events. Useful for understanding where users drop off during onboarding.

```javascript
_k.track('wallet_connect_clicked', 'wallet')
_k.track('wallet_connected', 'wallet', { address: '0x...', chain_id: 8453 })
_k.track('wallet_disconnected', 'wallet')
_k.track('signature_requested', 'wallet', { action: 'login' })
_k.track('signature_rejected', 'wallet', { action: 'login' })
```

### `custom`
For domain-specific events that don't fit the above categories.

```javascript
_k.track('game_started', 'custom', { mode: 'competitive', players: 6 })
_k.track('nft_minted', 'custom', { collection: 'genesis', token_id: 42 })
_k.track('referral_used', 'custom', { code: 'LAUNCH2025' })
```

---

## Custom categories

You can use any string as a category — the built-in ones are just conventions. If your app has a domain-specific concept that warrants its own category, use it:

```javascript
_k.track('match_ended', 'game', { winner: 'player_a', duration_seconds: 180 })
_k.track('trade_executed', 'market', { pair: 'ETH/USDC', side: 'buy' })
```

Custom categories appear in the Event Breakdown widget with their own color.
