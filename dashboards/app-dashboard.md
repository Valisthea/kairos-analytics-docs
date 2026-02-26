# App Dashboard

A real-time behavioral analytics view for a single registered application, with K-PPE proof status and Merkle verification.

---

## Accessing your dashboard

Each registered app gets a dedicated dashboard URL:

```
https://kairos-dashboard-seven.vercel.app/dashboard-app.html?appId=your-app-id
```

Or navigate to it from the Admin Panel by clicking **View Dashboard** on any app.

---

## KPI Cards

Six summary cards at the top of the dashboard update in real time:

| Card | What it measures |
|---|---|
| **Total Events** | All events received for this app |
| **Active Sessions** | Users currently active on your site |
| **Unique Wallets** | Distinct wallet addresses seen |
| **Conversions** | Your defined conversion event count |
| **Key Actions** | Your primary tracked action count |
| **Error Rate** | Ratio of `_failed` events to total |

---

## K-PPE Merkle Anchor Status

The K-PPE section at the top of the dashboard displays the current state of on-chain proof anchoring:

### Proof Bar

A status bar showing:
- **Latest Batch ID** -- the most recent batch number anchored on-chain
- **Total Events Anchored** -- cumulative count of events with on-chain Merkle proof
- **Latest Merkle Root** -- the most recent 32-byte root stored in KPPEAnchor
- **Last Anchored** -- timestamp of the most recent on-chain transaction
- **Basescan Link** -- direct link to the KPPEAnchor contract events on Basescan

### Batch Info

Clicking on a batch reveals:
- **Merkle Root** -- the `bytes32` root hash stored on-chain
- **Event Count** -- number of events in this batch
- **Tree Depth** -- depth of the binary Merkle tree
- **Previous Root** -- link in the chain of trust (`prevRoot`)
- **TX Hash** -- the Base mainnet transaction hash, linked to Basescan
- **Anchored At** -- block timestamp of the on-chain transaction

### On-Chain Proof Links

Every batch includes a direct link to its transaction on Basescan:
```
https://basescan.org/tx/0x...
```

And a link to the KPPEAnchor contract for manual verification:
```
https://basescan.org/address/0x3B53F7044E47766769156bF210c2661F03Df45dd#readContract
```

### Merkle Tree Visualization

The dashboard includes an interactive Merkle tree visualizer. Select any confirmed batch to:
- See the full tree structure rendered visually
- Click any leaf node to generate its Merkle proof
- View the proof path highlighted from leaf to root
- See the proof detail panel with each step's sibling hash and position
- Verify the proof result (valid or invalid) in real time

---

## Widgets

Click the gear icon in the navigation bar to show or hide any widget. Your preferences are saved automatically.

### Live Event Feed
A scrolling stream of events as they arrive. Each row shows:
- Event name and type
- Page path
- Client hash (truncated)
- Time elapsed

### Active Sessions
Cards for each currently active session showing the page they are on, how many events they have fired, and session duration.

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

When you first open the dashboard, it displays **mock data** -- realistic-looking sample data to demonstrate the interface. The badge in the top bar reads **MOCK DATA**.

To switch to your real data, authenticate your account. The badge changes to **LIVE DATA** and all widgets reload with actual events from Supabase and on-chain data from KPPEAnchor.

See [Authentication](authentication.md) for how to connect.

---

## Auto-refresh

When connected, the dashboard automatically refreshes all data every 30 seconds. The "last synced" timestamp at the bottom of the page shows when data was last fetched. K-PPE batch status is refreshed alongside event data.
