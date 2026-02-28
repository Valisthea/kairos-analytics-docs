# Event Tracking

The snippet auto-tracks 15+ event types. You can also send custom events.

---

## Auto-tracked Events

| Category | Events |
|----------|--------|
| **Page** | page_view, navigation, route_change |
| **UI** | click, scroll_depth, form_submit, outbound_click, tab_hidden |
| **Wallet** | wallet_detected, wallet_connected, wallet_disconnected, chain_changed |
| **Transaction** | tx_submitted, tx_confirmed, tx_failed, swap_executed |
| **System** | performance, error |

## Wallet Auto-detection

Supported wallets: MetaMask, Coinbase Wallet, Rabby, Brave Wallet, Trust Wallet, Phantom.

No configuration required. The snippet detects wallet providers automatically.

## Custom Events

    kairos.track("custom_event", {
      key: "value",
      amount: 100,
      token: "ETH"
    })

## Event Payload

Each event sent to the relayer includes:
- event_type: The event name
- category: Derived from event type (page, ui, wallet, transaction, error, custom)
- timestamp: Millisecond timestamp
- session_id: Unique session identifier
- properties: Custom key-value data
- page: Current page URL
