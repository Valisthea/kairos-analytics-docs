# Roadmap

Current status of every feature.

---

## Stable ✅

| Feature | Description |
|---|---|
| SDK snippet | One `<script>` tag, zero dependencies, auto page/click tracking |
| `trackSwap()` / `trackTransaction()` / `trackWalletConnect()` | Core Web3 tracking functions |
| `trackEvent()` / `trackPageView()` | Generic event and page tracking |
| Railway Relayer v4.0 | Multi-chain, batch processing, health endpoint |
| Base Mainnet contracts | Registry + Relayer contract live |
| BSC Testnet contracts | Registry deployed — `0x6C7986a3...` |
| Multi-chain routing | Events routed to correct chain per appId |
| App Dashboard | 11 widgets — live feed, wallet analytics, geo, funnels, TX feed |
| On-chain proof bar | Last anchored block, TX hash, Basescan / BscScan link |
| Admin Panel | Cross-app overview, alerts, app management |
| Supabase Auth | Google OAuth + GitHub OAuth + Email/Password |
| Register.html wizard | 4-step onboarding — account → appId → interests → plan |
| Mandatory snippet popup | Blocks dashboard until snippet is copied and confirmed |
| Plan gating | Widgets locked/unlocked based on active plan |
| Stripe payments | Builder ($29/mo) and Protocol ($99/mo) via Stripe Checkout |
| Plan badge in navbar | FREE / BUILDER / PROTOCOL shown in dashboard header |

---

## In Progress 🔶

| Feature | Status |
|---|---|
| PostgreSQL persistence | Variables configured — sprint pending |
| Stripe webhook handler | Endpoint defined — activation flow pending |
| Plan auto-activation after payment | Pending webhook handler |
| Email confirmation flow | Supabase default — UX polish pending |

---

## Planned 📋

| Feature | Description |
|---|---|
| **React SDK** | `useAnalytics()` hook, `usePageView()`, `<Track>` component |
| **Next.js plugin** | Auto page view on route change, middleware support |
| **Webhook alerts** | Push to Slack or Discord on alert triggers |
| **Custom dashboard layouts** | Drag-and-drop widget builder |
| **Funnel builder** | Define funnels from the UI, no code changes |
| **CSV export** | Export any event dataset as CSV |
| **Polygon / Arbitrum** | Additional chain support |
| **Team access** | Multiple users per app with role-based permissions |
| **Asterchain migration** | Native analytics layer on Asterchain mainnet at launch |

---

## Chain deployment status

| Chain | Registry | Relayer | Status |
|---|---|---|---|
| Base Mainnet (8453) | `0xcbf3e71f...` | `0x90991Ae4...` | ✅ Production |
| BSC Testnet (97) | `0x6C7986a3...` | — | ✅ Active |
| BSC Mainnet | — | — | 📋 Planned |
| Asterchain | — | — | 📋 On mainnet launch |
