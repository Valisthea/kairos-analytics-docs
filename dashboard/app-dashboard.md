# App Dashboard

Your dashboard at [kairosanalytics.org/dashboard](https://kairosanalytics.org/dashboard) is the central hub for all your dApp analytics.

## Overview

The dashboard displays real-time data from your dApp across 21 widgets, organized in a responsive grid layout. Widgets are unlocked based on your plan tier (see [Plans & Pricing](../reference/plans.md)).

## Navigation

**Header** — Shows your App ID, current plan badge, relayer status indicator, and settings gear icon.

**Sidebar** — Sections for Dashboard (main view), Audit & Security, and Settings. The sidebar can be collapsed for more screen space.

**Mode Toggle** — Switch between **Live** (real data from your dApp) and **Demo** (sample data to explore all features). Demo mode shows all widgets regardless of plan.

## Data Modes

### Live Mode

Displays real data from events tracked by your snippet. If your dApp hasn't sent any events yet, the dashboard shows an empty state with setup instructions.

Data refreshes automatically every 60 seconds. The relayer status indicator in the header shows whether the connection is healthy.

### Demo Mode

Pre-loaded with sample data so you can explore all 21 widgets and see how the dashboard looks with real traffic. Useful before integrating the snippet, or to preview features available in higher plan tiers.

Access demo mode by appending `?demo=true` to the dashboard URL.

## Time Range

Use the time range selector in the header to filter data:

* Last 24 hours (default)
* Last 7 days
* Last 30 days
* Custom range

The selected range applies to all widgets simultaneously.

## Widget Grid

Widgets can be rearranged by dragging. Your layout is saved automatically. See the [Widgets Reference](widgets.md) for detailed documentation on each widget.

## Settings

Click the gear icon to access:

* **Profile** — Your email and account info
* **App ID** — Your registered App ID and snippet code
* **Billing** — Current plan, next billing date, invoices, and upgrade/cancel options
* **Preferences** — Dashboard preferences (theme, refresh interval, notification settings)

## Empty State

If your dApp hasn't sent any events yet, the dashboard shows:

1. Your App ID and snippet code to copy
2. A polling indicator that checks for incoming events every 10 seconds
3. A "Use Demo Mode" button to explore the dashboard with sample data

Once the first event arrives, the dashboard automatically switches to live mode.
