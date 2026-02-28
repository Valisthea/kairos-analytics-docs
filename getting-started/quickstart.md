# Quickstart

Live in 30 seconds. Two lines of code.

---

## 1. Get your App ID

1. Go to [kairosanalytics.org/register](https://kairosanalytics.org/register)
2. Sign up with Google, GitHub, or email
3. Complete the 3-step wizard (App ID, dApp type, plan)

## 2. Install the snippet

```html
<script>window.KAIROS_APP_ID = "your-app-id";</script>
<script src="https://kairosanalytics.org/snippet.js" async></script>
```

15+ event types auto-tracked including page views, clicks, wallet connections, and transactions.

## 3. Open your dashboard

Go to [kairosanalytics.org/dashboard](https://kairosanalytics.org/dashboard). Events appear in real-time.

---

## Auto-tracked events

| Category | Events |
|----------|--------|
| **Page** | page_view, navigation, route_change |
| **UI** | click, scroll_depth, form_submit, outbound_click, tab_hidden |
| **Wallet** | wallet_detected, wallet_connected, wallet_disconnected, chain_changed |
| **Transaction** | tx_submitted, tx_confirmed, tx_failed, swap_executed |
| **System** | performance, error |

## npm alternative

```bash
npm install kairos-analytics
```

```javascript
import { KairosAnalytics } from "kairos-analytics"
const kairos = new KairosAnalytics({ appId: "your-app-id" })
kairos.track("custom_event", { key: "value" })
```
