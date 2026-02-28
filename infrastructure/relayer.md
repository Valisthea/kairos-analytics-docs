# Relayer API — 54 Endpoints

The Kairos relayer is the backend service that processes events, manages data, and communicates with the blockchain. It exposes 54 HTTP endpoints grouped by authentication level.

**Base URL**: `https://kairos-relayer-production.up.railway.app`

## Public Endpoints (No Auth)

These endpoints require no authentication.

| Method | Endpoint | Description |
|--------|----------|-------------|
| GET | `/health` | Relayer health check. Returns uptime, version, event count. |
| GET | `/status` | Detailed system status: relayer, Supabase, Base RPC connectivity. |
| GET | `/verify/batch` | Get batch info by batchId. Returns Merkle root, event count, tx hash. |
| GET | `/verify/proof` | Get Merkle proof for an event hash within a batch. |
| GET | `/verify/root` | Verify a Merkle root against the on-chain anchor. |

## User Endpoints (Supabase JWT)

These endpoints require a valid Supabase JWT token in the `Authorization: Bearer <token>` header. The user must own the appId being queried.

### Event Tracking

| Method | Endpoint | Description |
|--------|----------|-------------|
| POST | `/track` | Submit an analytics event. Body: `{ appId, type, meta }` |
| POST | `/track/batch` | Submit multiple events at once. Body: `{ appId, events: [...] }` |

### Dashboard Data

| Method | Endpoint | Description |
|--------|----------|-------------|
| GET | `/stats` | Aggregated stats: DAU, sessions, events, page views. Params: `appId`, `range`. |
| GET | `/sessions` | Session list with duration, pages, wallet info. Params: `appId`, `range`, `limit`. |
| GET | `/pages` | Top pages by view count. Params: `appId`, `range`. |
| GET | `/events` | Recent events feed. Params: `appId`, `limit`, `type` (filter). |
| GET | `/timeline` | Event volume over time. Params: `appId`, `range`, `interval` (hour/day/week). |
| GET | `/top-events` | Most frequent event types. Params: `appId`, `range`. |
| GET | `/devices` | Device breakdown (browser, OS, screen). Params: `appId`, `range`. |
| GET | `/funnel` | Funnel analysis. Params: `appId`, `steps` (comma-separated event types). |
| GET | `/breakdown` | Metric breakdown by dimension. Params: `appId`, `metric`, `dimension`. |
| GET | `/heatmap` | Click coordinates for heatmap overlay. Params: `appId`, `url`. |
| GET | `/errors` | JavaScript errors grouped by signature. Params: `appId`, `range`. |
| GET | `/geo` | Geographic distribution. Params: `appId`, `range`. |
| GET | `/volume` | Event volume and plan usage. Params: `appId`. |
| GET | `/wallets` | Wallet analytics summary. Params: `appId`, `range`. |
| GET | `/wallet-detail` | Individual wallet behavior. Params: `appId`, `address`. |
| GET | `/tx-analytics` | Transaction analytics. Params: `appId`, `range`. |
| GET | `/wallet-retention` | Wallet cohort retention. Params: `appId`, `range`. |
| GET | `/alerts` | Active alerts for the app. Params: `appId`. |
| GET | `/live-count` | Currently active users. Params: `appId`. |
| GET | `/button-map` | Button click analysis. Params: `appId`, `url`. |

### Account & Billing

| Method | Endpoint | Description |
|--------|----------|-------------|
| GET | `/plan` | Current plan info for an appId. Returns plan, eventsUsed, eventsLimit. |
| GET | `/user-exists` | Check if an email has an account. Returns appId and plan if found. |
| GET | `/user-appid` | Get the appId associated with an email. |
| POST | `/register-user` | Create a new account during onboarding. Body: `{ appId, plan, interests, dappType }` |
| POST | `/register-app` | Legacy registration endpoint (kept for compatibility). |
| POST | `/create-payment-intent` | Create a Stripe subscription. Returns clientSecret for Payment Element. |
| GET | `/billing` | Current subscription and invoice info. |
| POST | `/cancel-subscription` | Cancel the active Stripe subscription. |

### K-PPE

| Method | Endpoint | Description |
|--------|----------|-------------|
| GET | `/kppe/batches` | List proof batches for an appId. Params: `appId`, `limit`. |
| GET | `/kppe/status` | K-PPE engine status: queue length, last anchor, next anchor. |
| GET | `/health-score` | Composite dApp health score. Params: `appId`. |
| GET | `/anomalies` | Detected anomalies. Params: `appId`, `range`. |
| GET | `/wallet-score` | Wallet reputation score. Params: `appId`, `address`. |

## Admin Endpoints (Admin JWT)

These endpoints require a Supabase JWT where the user's email is in the `ADMIN_EMAILS` whitelist.

| Method | Endpoint | Description |
|--------|----------|-------------|
| GET | `/admin/apps` | List all registered apps with stats. |
| GET | `/admin/stats` | Global platform statistics. |
| GET | `/admin/billing` | All subscriptions, invoices, MRR. |
| GET | `/admin/logs` | Server logs and recent errors. |
| GET | `/admin/feed` | Global event feed across all apps. |
| POST | `/update-plan` | Change an app's plan tier. Body: `{ appId, plan }` |
| POST | `/admin/set-tags` | Set tags on an app. Body: `{ appId, tags }` |
| POST | `/admin/set-custom-limit` | Override event limit. Body: `{ appId, limit }` |
| DELETE | `/admin/delete-app` | Delete an app and all associated data. Body: `{ appId }` |
| POST | `/admin/view-as` | View dashboard as a specific app (impersonation for support). |

## Stripe Webhook

| Method | Endpoint | Description |
|--------|----------|-------------|
| POST | `/stripe/webhook` | Receives Stripe webhook events. Verified via webhook signing secret. |

Handled events:
* `invoice.payment_succeeded` — Activates or confirms paid plan
* `customer.subscription.deleted` — Downgrades to free
* `checkout.session.completed` — Activates plan after first checkout

## Error Responses

All endpoints return JSON. Error format:

```json
{
  "ok": false,
  "error": "Description of what went wrong"
}
```

Common HTTP status codes:

| Status | Meaning |
|--------|---------|
| 200 | Success |
| 400 | Bad request (invalid params or body) |
| 401 | Unauthorized (missing or invalid JWT) |
| 403 | Forbidden (not an admin, or doesn't own the appId) |
| 404 | Not found (app or resource doesn't exist) |
| 409 | Conflict (appId already taken) |
| 429 | Rate limited |
| 500 | Server error |
