# Widgets Reference

The Kairos dashboard includes 21 widgets across three plan tiers. Each widget is designed for Web3-specific analytics.

## Free Tier — 7 Widgets

### Live Feed (`w-feed`)
Real-time stream of events as they arrive. Shows event type, timestamp, URL, and metadata. Color-coded by event category (green for Web3, blue for navigation, amber for errors).

### Sessions (`w-sessions`)
Active and historical session data. Shows session count, average duration, pages per session, and bounce rate. Highlights sessions with wallet interactions.

### Pages (`w-pages`)
Top pages by view count. Shows URL, view count, average time on page, and exit rate. Useful for identifying your most-visited dApp routes.

### Who's Here (`w-whoshere`)
Currently active users on your dApp. Shows count, geographic distribution, and which pages they're on. Updates in real time.

### Timeline (`w-timeline`)
Event volume over time as a line chart. Toggle between hourly, daily, and weekly views. Overlay multiple event types to compare trends.

### Top Events (`w-topevents`)
Most frequent event types ranked by count. Shows event name, count, and percentage of total. Helps identify what users do most in your dApp.

### Device (`w-device`)
User device breakdown: browser, operating system, screen resolution, and mobile vs desktop ratio.

## Builder Tier — +7 Widgets (14 total)

### Funnel (`w-funnel`)
Conversion funnel visualization. Configure up to 8 steps (e.g., page_view → wallet_connected → tx_submitted → tx_confirmed). Shows drop-off at each step with percentages.

### Breakdown (`w-breakdown`)
Segment any metric by a dimension. Examples: events by wallet provider, page views by country, transactions by chain. Configurable via dropdown.

### Heatmap (`w-heatmap`)
Click heatmap overlaid on your dApp pages. Shows where users click most. Requires the snippet to capture click coordinates (enabled by default).

### Errors (`w-errors`)
JavaScript error tracking. Shows error message, file, line number, occurrence count, and affected sessions. Grouped by error signature for deduplication.

### Button Map (`w-btnmap`)
Interactive button click analysis. Shows which buttons get the most clicks, click-through rates, and rage clicks (rapid repeated clicks indicating frustration).

### Geo (`w-geo`)
Geographic distribution map. Shows user locations by country and city, with traffic volume indicated by color intensity. Based on IP geolocation (approximate).

### Volume (`w-volume`)
Event volume metrics: total events today, this week, this month. Shows usage against your plan limit with a progress bar. Alerts when approaching the limit.

## Protocol Tier — +7 Widgets (21 total)

### Wallet Analytics (`w-wallet`)
Deep wallet analysis: unique wallets, new vs returning, wallet provider distribution, average session per wallet, and wallet-to-transaction conversion rate.

### Wallet Deep Dive (`w-walletanalytics`)
Individual wallet behavior analysis. Search by wallet address to see: sessions, pages visited, transactions, time to first transaction, and behavioral patterns.

### TX Analytics (`w-txanalytics`)
Transaction analytics: volume, success rate, average gas used, popular contract methods, transaction value distribution, and peak transaction times.

### Wallet Retention (`w-walletretention`)
Cohort retention analysis for wallet addresses. Shows how many wallets return after their first visit, broken down by day/week/month cohorts.

### Alerts (`w-alerts`)
Configurable alerts for anomalous metrics. Set thresholds for: traffic drops, error spikes, transaction failures, unusual wallet activity. Alerts display in the widget and optionally via email.

### Audit (`w-audit`)
Security and integrity audit score. Composite score based on data integrity, proof anchoring frequency, error rates, and anomaly detection. See [Audit & Security](audit-security.md) for details.

## Plan Gating

When you encounter a locked widget, it displays:
* The widget name and a brief description
* Which plan tier is required to unlock it
* An "Upgrade" button linking to the plan selection page

In Demo mode, all widgets are unlocked with sample data so you can evaluate features before upgrading.
