# Dashboards Overview

Kairos Analytics provides two dashboards with different scopes and access levels.

---

## App Dashboard

A per-application analytics view. Each registered app gets its own dashboard showing behavioral data tracked by the SDK.

**What it shows:**
- Live event feed
- Active user sessions
- Page-level statistics
- Conversion funnels
- Wallet interaction patterns
- Hourly activity heatmap
- Error and failure events

**Who can access it:** Any authenticated user who owns the app (verified via the Registry contract). Optionally public for read-only views.

→ [App Dashboard details](app-dashboard.md)

---

## Admin Panel

A cross-application overview for platform operators. Combines behavioral data from the relayer with business data from your application's own API.

**What it shows:**
- Stats across all registered apps
- Revenue and transaction history
- Alerts and anomaly detection
- App ID management (register, suspend, delete)
- Drill-down into any individual app

**Who can access it:** Users with `is_admin` privileges only.

→ [Admin Panel details](admin.md)

---

## Data sources

Both dashboards pull from two independent sources:

| Source | Contains | Used in |
|--------|----------|---------|
| **Relayer** | Behavioral events (clicks, page views, funnels) | App Dashboard |
| **Your app's API** | Business data (users, revenue, transactions) | Admin Panel |

This separation means Kairos Analytics never touches your financial data — you connect it yourself if you want the combined view in the Admin Panel.

---

## Accessing the dashboards

Both dashboards require authentication. See [Authentication](authentication.md) for the full connection flow.

At a high level:
- **App Dashboard** → authenticate with your app account credentials
- **Admin Panel** → same credentials, but your account must have admin privileges
