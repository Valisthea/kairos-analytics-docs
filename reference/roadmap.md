# Roadmap

What's done, what's in progress, and what's coming next.

---

## Stable

| Feature | Description |
|---------|-------------|
| SDK `@valisthea/analytics` | Published on npm, importable via CDN |
| `offchain` mode | HTTP transport to Railway relayer |
| Session management | Automatic session ID generation and persistence |
| `init()` / `page()` / `track()` | Core SDK API |
| Relayer service | Deployed on Railway, accepts events via `POST /track` |
| Registry contract | App registration on Base mainnet |
| Relayer contract | On-chain event batch storage |
| App Dashboard | 11 toggleable widgets, mock + live data modes |
| Admin Panel | Cross-app overview, revenue, alerts, app management |
| Auth modal | Login via access code or wallet address |
| `kairos-analytics-api.js` | Shared client for dashboard data fetching |

---

## In Progress

| Feature | Description | Status |
|---------|-------------|--------|
| Relayer `/health` endpoint | Health check for dashboard status indicator | 🔶 Verify |
| Relayer `/events` endpoint | Query stored events by appId | 🔶 Add to relayer |
| Relayer `/sessions` endpoint | Query active sessions by appId | 🔶 Add to relayer |
| AppId registration flow | Register apps from the Admin Panel UI | 🔶 Contract call |

---

## Planned

| Feature | Description |
|---------|-------------|
| **React SDK** | First-class hooks: `useAnalytics()`, `usePageView()`, `<Track>` component |
| **Next.js plugin** | Auto page view tracking, route-aware session management |
| **Onchain mode** | User-signed events via MetaMask — maximum decentralization |
| **Webhook alerts** | Push notifications to Slack or Discord on alert triggers |
| **Custom dashboards** | Drag-and-drop widget layout builder |
| **Funnel builder** | Define custom funnels from the UI without code changes |
| **Data export** | CSV export of any event dataset |
| **Multi-chain support** | Polygon, Arbitrum, BSC in addition to Base |
| **Team access** | Multiple users per app with role-based permissions |

---

## Contributing

Kairos Analytics is designed to be extended. If you want to contribute:

- **New SDK events** — open a PR with the event definition and tracking call
- **Dashboard widgets** — widgets are self-contained components, easy to add
- **Relayer endpoints** — Express routes, follow existing patterns
- **Bug reports** — open an issue with console logs and reproduction steps
