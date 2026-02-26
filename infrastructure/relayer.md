# Relayer

The server-side middleware that bridges the SDK, Supabase, and the dashboard.

---

## What the relayer does

The relayer is a Node.js/Express service deployed on Railway. It serves as the API layer for the Kairos Analytics platform:

1. Accepts events from the SDK via `POST /track`
2. Manages sessions and validates App IDs
3. Serves event data and batch metadata to the dashboard
4. Exposes K-PPE security headers (`X-KA-HAP-*`) for the KA-HAP security architecture
5. Provides a `/kpe/health` endpoint for K-PPE Engine monitoring

---

## Endpoints

### `POST /track`

Receive an event from the SDK.

**Request body:**
```json
{
  "appId": "your-app-id",
  "event": "swap_attempted",
  "sessionId": "a3f9b2c1-...",
  "timestamp": 1700000000000,
  "clientHash": "0x7a3f...",
  "properties": {
    "fromToken": "ETH",
    "toToken": "USDC",
    "amount": 1.5
  }
}
```

**Response:**
```json
{
  "ok": true,
  "eventId": "evt-abc123"
}
```

The relayer validates the `appId` against the App Registry contract before accepting events.

---

### `GET /health`

Verify the relayer is running.

**Response:**
```json
{
  "status": "ok",
  "version": "4.0",
  "timestamp": "2025-01-01T00:00:00.000Z",
  "kppe": {
    "contract": "0x3B53F7044E47766769156bF210c2661F03Df45dd",
    "chain": "base",
    "chainId": 8453
  }
}
```

---

### `GET /kpe/health`

K-PPE Engine health and status endpoint.

**Response:**
```json
{
  "status": "ok",
  "engine": "K-PPE-v1",
  "contract": "0x3B53F7044E47766769156bF210c2661F03Df45dd",
  "chain": "base",
  "chainId": 8453,
  "latestBatchId": 42,
  "totalEventsAnchored": 1234,
  "latestRoot": "0xabc...",
  "lastAnchoredAt": "2025-01-01T12:00:00.000Z"
}
```

This endpoint is used by the dashboard to display the K-PPE status indicator and by monitoring tools to verify the engine is anchoring batches.

---

### `GET /events`

Query stored events for an app. Used by the dashboard.

**Query parameters:**

| Parameter | Type | Default | Description |
|---|---|---|---|
| `appId` | string | -- | **Required.** Filter by app ID |
| `limit` | number | 20 | Number of events to return |
| `sort` | string | `desc` | `asc` or `desc` by timestamp |
| `from` | ISO date | -- | Filter events after this timestamp |
| `to` | ISO date | -- | Filter events before this timestamp |

**Response:**
```json
{
  "events": [
    {
      "id": "evt-abc123",
      "appId": "your-app-id",
      "name": "swap_attempted",
      "sessionId": "a3f9b2c1",
      "timestamp": "2025-01-01T12:00:00.000Z",
      "clientHash": "0x7a3f...",
      "batchId": 42,
      "properties": { "fromToken": "ETH", "toToken": "USDC", "amount": 1.5 }
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
|---|---|---|
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

### `GET /batch/:txHash`

Fetch batch details including events for Merkle tree verification.

**Response:**
```json
{
  "ok": true,
  "batch": {
    "txHash": "0xdef...",
    "chain": "base",
    "eventCount": 12,
    "merkleRoot": "0xabc...",
    "createdAt": 1700000000000,
    "events": [ ... ]
  }
}
```

---

### `GET /batches`

List recent K-PPE batches.

**Query parameters:**

| Parameter | Type | Default | Description |
|---|---|---|---|
| `appId` | string | -- | Filter by app ID |
| `limit` | number | 10 | Number of batches to return |

**Response:**
```json
{
  "batches": [
    {
      "id": 42,
      "txHash": "0xdef...",
      "merkleRoot": "0xabc...",
      "eventCount": 12,
      "treeDepth": 4,
      "status": "confirmed",
      "createdAt": "2025-01-01T12:00:00.000Z"
    }
  ]
}
```

---

## K-PPE Security Headers

The relayer exposes KA-HAP (Kairos Analytics Hardened Application Protocol) security status via response headers. These headers are included on all responses and are used by the dashboard's security status display.

| Header | Description | Example Value |
|---|---|---|
| `X-KA-HAP-Status` | Overall KA-HAP security status | `operational` |
| `X-KA-HAP-Layers` | Number of active security layers | `7` |
| `X-KA-HAP-Crypto` | Cryptographic self-test result | `pass` |
| `X-KA-HAP-Anchor` | K-PPE anchor status | `active` |
| `X-KA-HAP-Timestamp` | Timestamp of last security check | ISO 8601 |

These headers are derived from dynamic K-PPE cryptographic self-tests, not static values. The dashboard reads them to render the real-time security status indicator.

---

## CORS configuration

The dashboard and SDK make browser requests to the relayer. CORS headers are required:

```javascript
app.use((req, res, next) => {
  res.header('Access-Control-Allow-Origin', '*')
  res.header('Access-Control-Allow-Headers', 'Content-Type, x-api-key')
  res.header('Access-Control-Allow-Methods', 'GET, POST, OPTIONS')
  res.header('Access-Control-Expose-Headers',
    'X-KA-HAP-Status, X-KA-HAP-Layers, X-KA-HAP-Crypto, X-KA-HAP-Anchor, X-KA-HAP-Timestamp')
  if (req.method === 'OPTIONS') return res.sendStatus(200)
  next()
})
```

Note the `Access-Control-Expose-Headers` line -- without it, the browser will not allow the dashboard JavaScript to read the `X-KA-HAP-*` headers.

---

## Environment variables

| Variable | Description | Required |
|---|---|---|
| `PRIVATE_KEY` | Private key of the wallet that signs on-chain batches | Yes |
| `REGISTRY_ADDRESS` | App Registry contract address on Base mainnet | Yes |
| `KPPE_CONTRACT_ADDRESS` | KPPEAnchor contract address on Base mainnet | Yes |
| `RPC_URL` | Base mainnet RPC endpoint | Yes |
| `SUPABASE_URL` | Supabase project URL | Yes |
| `SUPABASE_SERVICE_ROLE_KEY` | Supabase service role key | Yes |
| `PORT` | Server port (Railway sets this automatically) | Auto |
| `RELAYER_API_KEY` | API key for authenticated endpoints | Optional |
| `BATCH_INTERVAL_MS` | How often to commit event batches on-chain | Optional |

---

## Monitoring

Access real-time logs from your Railway project: **Railway > your service > Logs tab**

Key log lines to look for:

```
Server running on port 3000
Event received: swap_attempted from your-app-id | hash: 0x7a3f...
K-PPE health check: latestBatchId=42, totalAnchored=1234
AppId not registered: unknown-app
RPC error: could not connect to Base mainnet
```

### Recommended alerts

Set up Railway alerts for:
- Service crash or restart
- Memory usage above 80%
- Response time above 2 seconds
- `/health` or `/kpe/health` returning non-200 status
