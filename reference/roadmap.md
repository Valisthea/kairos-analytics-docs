# Roadmap

Current status of every feature.

---

## Stable

| Feature | Description |
|---|---|
| **K-PPE (Kairos Proof Protocol Engine)** | Trustless Merkle-based event anchoring on Base mainnet |
| **KPPEAnchor.sol** | Smart contract deployed and verified on Base mainnet (`0x3B53...45dd`) |
| **Client-side keccak256 hashing** | SDK hashes events in the browser before any server sees the data |
| **Merkle tree batching** | K-PPE Engine builds trees every 60s, max 256 events/batch |
| **Trustless verification** | `verifyProofView` on-chain, SDK `verify()` method, Basescan manual verification |
| **Chain of trust (prevRoot)** | Each batch links to the previous root for tamper-evident ordering |
| **Event receipts** | Every event gets an `EventReceipt` with `clientHash` for verification |
| **SDK (`@kairoslab/analytics-sdk`)** | npm package with client-side hashing, Supabase integration, receipt storage |
| **App Registry contract** | Maps App IDs to owners on Base mainnet (`0xcbf3...37B8`) |
| **Railway Relayer v4.0** | Multi-chain aware, K-PPE headers, `/kpe/health` endpoint |
| **App Dashboard** | 11 widgets -- live feed, wallet analytics, geo, funnels, TX feed |
| **K-PPE Proof Bar** | Dashboard displays latest batch, Merkle root, Basescan TX link |
| **Merkle Tree Visualizer** | Interactive tree rendering with proof path highlighting |
| **KA-HAP 7-Layer Security** | Dynamic security verification from K-PPE crypto self-tests |
| **Session DNA** | Behavioral fingerprints derived from real user event patterns |
| **Admin Panel** | Cross-app overview, alerts, app management |
| **Supabase Auth** | Google OAuth + GitHub OAuth + Email/Password |
| **Registration wizard** | 4-step onboarding -- account, appId, interests, plan |
| **Plan gating** | Widgets locked/unlocked based on active plan |
| **Stripe payments** | Builder ($29/mo) and Protocol ($99/mo) via Stripe Checkout |
| **Plan badge in navbar** | FREE / BUILDER / PROTOCOL shown in dashboard header |

---

## In Progress

| Feature | Status |
|---|---|
| PostgreSQL persistence | Variables configured -- sprint pending |
| Stripe webhook handler | Endpoint defined -- activation flow pending |
| Plan auto-activation after payment | Pending webhook handler |
| Email confirmation flow | Supabase default -- UX polish pending |
| Multi-app K-PPE batching | Separate Merkle trees per App ID |

---

## Planned

| Feature | Description |
|---|---|
| **React SDK** | `useKairos()` hook, `usePageView()`, `<Track>` component |
| **Next.js plugin** | Auto page view on route change, middleware support |
| **Webhook alerts** | Push to Slack or Discord on alert triggers |
| **Custom dashboard layouts** | Drag-and-drop widget builder |
| **Funnel builder** | Define funnels from the UI, no code changes |
| **CSV export** | Export any event dataset as CSV |
| **Batch explorer** | Full batch history with search and filtering |
| **Proof export** | Export Merkle proofs as JSON for external audits |
| **Team access** | Multiple users per app with role-based permissions |
| **Additional chains** | Polygon, Arbitrum, and other L2 support for anchoring |
| **Zero-knowledge proofs** | ZK-SNARKs for privacy-preserving verification |

---

## Chain deployment status

| Chain | Contract | Address | Status |
|---|---|---|---|
| Base Mainnet (8453) | KPPEAnchor | `0x3B53F7044E47766769156bF210c2661F03Df45dd` | Production |
| Base Mainnet (8453) | App Registry | `0xcbf3e71fb2E09929682b8448442d184f8E1E37B8` | Production |
| Polygon | -- | -- | Planned |
| Arbitrum | -- | -- | Planned |

---

## K-PPE Milestones

| Date | Milestone |
|---|---|
| 2025 Q1 | K-PPE architecture designed, sorted-pair hashing specification |
| 2025 Q2 | KPPEAnchor.sol deployed on Base mainnet, verified on Basescan |
| 2025 Q2 | K-PPE Engine operational on Railway (60s cron) |
| 2025 Q2 | SDK updated with client-side keccak256 hashing and receipt system |
| 2025 Q2 | Trustless verification live: `verify()` method and Basescan manual verification |
| 2025 Q3 | Merkle tree visualizer added to dashboard |
| 2025 Q3 | KA-HAP 7-layer security with dynamic crypto self-tests |
| 2025 Q3 | Session DNA behavioral fingerprinting |
