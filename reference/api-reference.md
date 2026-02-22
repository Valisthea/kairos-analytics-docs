# Full API Reference

Complete reference for all SDK functions and configuration options.

---

## SDK Functions

### `init(config)`

Initialize the SDK. Must be called before `page()` or `track()`.

```javascript
await init({
  appId: 'your-app-id',   // Required
  mode: 'offchain',        // 'offchain' | 'onchain'
  debug: false,            // boolean
  relayerUrl: '...',       // string (optional override)
})
```

**Returns:** `Promise<void>`

**Throws:** If `appId` is missing or empty.

---

### `page(path)`

Track a page view.

```javascript
page(window.location.pathname)
page('/dashboard')
page('/products/item-123')
```

**Parameters:**
| Parameter | Type | Description |
|-----------|------|-------------|
| `path` | string | The current page path |

**Returns:** `void`

**Notes:** Automatically attaches the current session ID and timestamp.

---

### `track(eventName, category, properties?)`

Track a custom event.

```javascript
track('button_clicked', 'ui', { label: 'signup' })
track('purchase_completed', 'transaction', { amount: 49.00, currency: 'USDC' })
track('wallet_connected', 'wallet', { address: '0x...', chain_id: 8453 })
```

**Parameters:**
| Parameter | Type | Required | Description |
|-----------|------|----------|-------------|
| `eventName` | string | ✅ | Event name in `snake_case` |
| `category` | string | ✅ | Event category (`page`, `ui`, `transaction`, `wallet`, `custom`) |
| `properties` | object | No | Additional key-value data |

**Returns:** `void`

---

## Event payload shape

Every event sent to the relayer has the following structure:

```json
{
  "appId": "your-app-id",
  "name": "button_clicked",
  "category": "ui",
  "sessionId": "a3f9b2c1d4e5",
  "timestamp": 1700000000000,
  "properties": {
    "label": "signup"
  }
}
```

---

## Relayer API

### `POST /track`

```
POST https://your-relayer-url/track
Content-Type: application/json
```

**Body:** Event payload (see above)

**Response:**
```json
{ "ok": true }
```

---

### `GET /health`

```
GET https://your-relayer-url/health
```

**Response:**
```json
{ "status": "ok", "version": "1.0" }
```

---

### `GET /events`

```
GET https://your-relayer-url/events?appId=your-app-id&limit=20&sort=desc
```

**Query parameters:**

| Parameter | Type | Default | Description |
|-----------|------|---------|-------------|
| `appId` | string | — | Required |
| `limit` | number | 20 | Max results |
| `sort` | string | `desc` | `asc` or `desc` |
| `from` | ISO timestamp | — | Filter from date |
| `to` | ISO timestamp | — | Filter to date |

**Response:**
```json
{
  "events": [ ...event objects... ],
  "count": 20
}
```

---

### `GET /sessions`

```
GET https://your-relayer-url/sessions?appId=your-app-id
```

**Response:**
```json
{
  "sessions": [
    {
      "session_id": "a3f9b2c1",
      "current_page": "/pricing",
      "event_count": 8,
      "duration_seconds": 210,
      "last_seen": "2025-01-01T12:05:00.000Z"
    }
  ]
}
```

---

## Event category reference

| Category | Typical events |
|----------|---------------|
| `page` | `page_view` |
| `ui` | `button_clicked`, `modal_opened`, `filter_applied`, `search_performed` |
| `transaction` | `purchase_initiated`, `purchase_completed`, `purchase_failed`, `deposit_submitted` |
| `wallet` | `wallet_connect_clicked`, `wallet_connected`, `wallet_disconnected`, `signature_rejected` |
| `custom` | Any domain-specific event |
