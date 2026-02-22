# Data API

The `kairos-analytics-api.js` client library used by the dashboards.

---

## Overview

`kairos-analytics-api.js` is a shared JavaScript module that wraps all network calls made by the dashboards. It handles:

- Authentication (login, session persistence, logout)
- Queries to the relayer (behavioral events, sessions)
- Queries to your application's own API (business KPIs, transactions)
- Relayer health checks

---

## Loading the client

Include it in any dashboard page before using it:

```html
<script src="/kairos-analytics-api.js"></script>
```

Then access it via `window.KairosAnalyticsAPI`:

```javascript
const API = window.KairosAnalyticsAPI
```

---

## Authentication

```javascript
// Login with an access code
const result = await API.loginWithBetaCode('YOUR_ACCESS_CODE')
if (result.ok) {
  console.log('Logged in as', API.getUser().username)
}

// Login with a wallet address
const result = await API.loginWithWallet('0x...')

// Check if logged in
API.isConnected()  // → true / false

// Check if admin
API.isAdmin()      // → true / false

// Get current user
API.getUser()      // → { username, wallet_address, is_admin, ... }

// Logout
API.logout()
```

---

## Relayer endpoints

```javascript
// Recent events for an app (live feed)
const { ok, events } = await API.getRecentEvents('your-app-id', 20)

// Query events with filters
const { ok, events } = await API.getEvents('your-app-id', {
  limit: 50,
  sort: 'desc',
  from: '2025-01-01T00:00:00Z',
  to: '2025-01-31T23:59:59Z'
})

// Active sessions
const { ok, sessions } = await API.getActiveSessions('your-app-id')

// Relayer health
const { ok } = await API.getRelayerHealth()
```

---

## Application API endpoints

These call your application's own backend, not the relayer. Configure your API base URL in the client initialization.

```javascript
// Global KPIs (admin-dashboard action)
const { ok, data } = await API.getAdminDashboard()
// data.kpi → { total_users, active_sessions, total_revenue, ... }

// Transactions list
const { ok, data } = await API.getAdminTransactions(limit, offset, type)
// data.transactions → [{ id, tx_type, amount, created_at, ... }]

// Users list
const { ok, data } = await API.getAdminUsers(limit, offset, search)
// data.users → [{ id, username, wallet_address, level, ... }]

// Online user count
const { ok, data } = await API.getOnlineCount()
// data.count → number
```

---

## Low-level access

For custom queries not covered by the helper methods:

```javascript
// Direct call to your application API
const result = await API.supabaseApi('any-action', { param: 'value' })

// Direct GET to the relayer
const result = await API.relayerGet('/any-path?param=value')
```

---

## Error handling

All methods return `{ ok: true, data: ... }` on success and `{ ok: false, error: '...' }` on failure. Always check `ok` before using the response:

```javascript
const result = await API.getRecentEvents('your-app-id')

if (!result.ok) {
  console.error('Failed to load events:', result.error)
  return
}

console.log('Events:', result.events)
```
