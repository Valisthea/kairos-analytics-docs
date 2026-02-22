# Admin Panel

A centralized view across all registered applications. For platform operators only.

---

## Access requirements

The Admin Panel requires your account to have admin privileges. See [Authentication](authentication.md) for setup.

---

## Sidebar views

### Global Overview

The landing view when you open the Admin Panel.

- **Global KPI cards** — Total Apps, Total Events, Active Sessions, Total Revenue, Error Rate
- **Combined event chart** — 7-day event volume chart with one line per registered app
- **App summary cards** — One card per registered app with at-a-glance stats. Click any card to open the app detail modal.

### All Apps

A full table of all registered applications:

| Column | Description |
|--------|-------------|
| App Name | Display name |
| App ID | Unique identifier used in the SDK |
| Status | Active / Inactive |
| Events | Total event count |
| Sessions | Total session count |
| Revenue | Revenue associated with this app (if connected) |
| Last Event | Timestamp of the most recent event received |
| Actions | View details, Suspend, Delete |

Click any row to open the **App Detail Modal**.

### App Detail Modal

Opens when you click an app in the table or overview:

- Header with app name, ID, and a **"View Dashboard →"** link that opens the full App Dashboard
- 4 summary stats: Total Events, Active Sessions, Unique Wallets, Revenue
- Top events list — ranked by volume with a bar chart

### Revenue & Transactions

A view of financial activity pulled from your connected business API:

- Transaction list with type, amount, user, and timestamp
- Filterable by transaction type
- Summary KPIs: total revenue, transaction count, average transaction value

{% hint style="info" %}
Revenue data comes from your own application's API, not from the relayer. You connect it by configuring your business API endpoint in the admin settings.
{% endhint %}

### Alerts & Monitoring

An automatic feed of anomalies detected across all apps:

| Alert type | Triggered when |
|-----------|----------------|
| `error` | Spike in `_failed` events above baseline |
| `warn` | Metric deviates significantly from 7-day average |
| `info` | Notable positive events (new record, new app connected) |

Each alert shows the affected app, a description, and a timestamp. Alerts can be individually dismissed or cleared all at once.

### Manage App IDs

Register new apps or manage existing ones:

**Register a new app:**
- App Name
- App ID (must be unique)
- Network (Base, BSC, Arbitrum, Polygon)

**Manage existing apps:**
- Suspend an app (stops accepting events)
- Delete an app (permanent, removes from Registry)

---

## Connecting your business API

The Revenue view and user-level stats in the Admin Panel require a connection to your application's own backend. Configure this in Admin Settings → API Configuration.

The Admin Panel makes authenticated requests to your API using the same session token the user logged in with. Your API must implement the following endpoints:

| Endpoint | Returns |
|----------|---------|
| `POST /api` `{ action: 'admin-dashboard' }` | Global KPIs |
| `POST /api` `{ action: 'admin-transactions', limit, offset, type }` | Transaction list |
| `POST /api` `{ action: 'admin-users', limit, offset, search }` | User list |
| `POST /api` `{ action: 'online-count' }` | `{ count: number }` |
