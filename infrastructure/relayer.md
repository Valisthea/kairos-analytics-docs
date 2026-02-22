# Relayer

The server-side middleware that bridges the SDK and the blockchain.

---

## What the relayer does

The relayer sits between your users' browsers and Base mainnet. It:

1. Accepts events from the SDK via `POST /track`
2. Validates that the sending app is registered in the Registry contract
3. Stores events in a local cache (for dashboard queries)
4. Batches accumulated events periodically
5. Signs and submits each batch to the Relayer contract on Base mainnet

---

## Endpoints

### `POST /track`
Receive an event from the SDK.

**Request body:**
```json
{
  "appId": "your-app-id",
  "event": "button_clicked",
  "category": "ui",
  "sessionId": "a3f9b2c1",
  "timestamp": 1700000000000,
  "properties": {
    "label": "hero_cta"
  }
}
```

**Response:**
```json
{ "ok": true }
```

---

### `GET /health`
Verify the relayer is running.

**Response:**
```json
{
  "status": "ok",
  "version": "1.0",
  "timestamp": "2025-01-01T00:00:00.000Z"
}
```

---

### `GET /events`
Query stored events for an app. Used by the dashboard.

**Query parameters:**

| Parameter | Type | Default | Description |
|-----------|------|---------|-------------|
| `appId` | string | — | **Required.** Filter by app ID |
| `limit` | number | 20 | Number of events to return |
| `sort` | string | `desc` | `asc` or `desc` by timestamp |
| `from` | ISO date | — | Filter events after this timestamp |
| `to` | ISO date | — | Filter events before this timestamp |

**Response:**
```json
{
  "events": [
    {
      "id": "evt_abc123",
      "appId": "your-app-id",
      "name": "button_clicked",
      "category": "ui",
      "sessionId": "a3f9b2c1",
      "timestamp": "2025-01-01T12:00:00.000Z",
      "properties": { "label": "hero_cta" }
    }
  ],
  "count": 1
}
```

---

### `GET /sessions`
Query active sessions for an app. Used by the dashboard.

**Query parameters:**

| Parameter | Type | Description |
|-----------|------|-------------|
| `appId` | string | **Required.** Filter by app ID |

**Response:**
```json
{
  "sessions": [
    {
      "session_id": "a3f9b2c1",
      "current_page": "/pricing",
      "event_count": 12,
      "duration_seconds": 340,
      "last_seen": "2025-01-01T12:05:00.000Z"
    }
  ]
}
```

---

## CORS configuration

The dashboard makes browser requests to the relayer. CORS headers are required:

```javascript
app.use((req, res, next) => {
  res.header('Access-Control-Allow-Origin', '*')
  res.header('Access-Control-Allow-Headers', 'Content-Type')
  res.header('Access-Control-Allow-Methods', 'GET, POST, OPTIONS')
  if (req.method === 'OPTIONS') return res.sendStatus(200)
  next()
})
```

---

## Environment variables

| Variable | Description | Required |
|----------|-------------|----------|
| `PRIVATE_KEY` | Private key of the wallet that signs on-chain batches | ✅ |
| `REGISTRY_ADDRESS` | Registry contract address on Base mainnet | ✅ |
| `RELAYER_ADDRESS` | Relayer contract address on Base mainnet | ✅ |
| `RPC_URL` | Base mainnet RPC endpoint | ✅ |
| `PORT` | Server port (Railway sets this automatically) | Auto |
| `BATCH_INTERVAL_MS` | How often to commit event batches on-chain | Optional |
| `DATABASE_URL` | PostgreSQL connection string for event cache | Optional |

---

## Monitoring

Access real-time logs from your Railway project:

**Railway → your service → Logs tab**

Key log lines to look for:

```
✅ Server running on port 3000
✅ Event received: page_view from your-app-id
✅ Batch of 12 events submitted — tx: 0x...
⚠️  AppId not registered: unknown-app
❌ RPC error: could not connect to Base mainnet
```
