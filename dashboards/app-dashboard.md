# App Dashboard

A real-time behavioral analytics view for a single registered application.

---

## Accessing your dashboard

Each registered app gets a dedicated dashboard URL:

```
https://your-dashboard-url/dashboard-app.html?appId=your-app-id
```

Or navigate to it from the Admin Panel by clicking **View Dashboard** on any app.

---

## KPI Cards

Six summary cards at the top of the dashboard update in real time:

| Card | What it measures |
|------|-----------------|
| **Total Events** | All events received for this app |
| **Active Sessions** | Users currently active on your site |
| **Unique Wallets** | Distinct wallet addresses seen |
| **Conversions** | Your defined conversion event count |
| **Key Actions** | Your primary tracked action count |
| **Error Rate** | Ratio of `_failed` events to total |

---

## Widgets

Click **⚙ Widgets** in the navigation bar to show or hide any widget. Your preferences are saved automatically.

### Live Event Feed
A scrolling stream of events as they arrive from the relayer. Each row shows:
- Event name
- Category (color-coded)
- Page path
- Time elapsed

### Active Sessions
Cards for each currently active session showing the page they're on, how many events they've fired, and session duration.

### Page Statistics
Horizontal bar chart showing event counts per page path. Useful for identifying your most and least visited pages.

### Event Timeline
A 24-hour line chart showing event volume, session count, and transaction count over time.

### Wallet Events
A filtered feed showing only wallet-category events: connections, disconnections, and signature requests. Useful for measuring onboarding friction.

### Conversion Funnel
A step-by-step funnel showing drop-off between key events you define. Percentage rates are calculated automatically.

### Event Breakdown
All event names ranked by volume, with a colored bar indicating their category.

### Hourly Heatmap
A 24-cell grid showing which hours of the day have the highest activity.

### Top Events
A ranked list of your 8 highest-volume events with category labels.

### Device Breakdown
Split between Desktop, Mobile, and Tablet based on user agent detection.

### Error Feed
A list of all `_failed` events with their error context, useful for monitoring transaction failures.

---

## Live vs Mock data

When you first open the dashboard, it displays **mock data** — realistic-looking sample data to demonstrate the interface. The badge in the top bar reads **MOCK DATA**.

To switch to your real data, authenticate your account. The badge changes to **LIVE DATA** and all widgets reload with actual events from the relayer.

See [Authentication](authentication.md) for how to connect.

---

## Auto-refresh

When connected, the dashboard automatically refreshes all data every 30 seconds. The "last synced" timestamp at the bottom of the page shows when data was last fetched.
