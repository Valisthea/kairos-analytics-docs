# Kairos Analytics

**Complete Technical Documentation -- Version 6.0 -- February 2026**

> Every click. Hashed. Proved on-chain.

Kairos Analytics is the first Web3 analytics protocol with cryptographic proof of data integrity. Every event is hashed, batched into Merkle trees, and anchored on-chain on Base mainnet via the KPPEAnchor contract.

---

## Why Kairos Analytics?

Traditional analytics tools were built for Web2 -- they own your data, track via cookies, and offer zero cryptographic guarantees. Blockchain-native tools like Dune focus on on-chain data, not frontend behavior.

Kairos gives you:

- **Behavioral analytics** -- page views, clicks, funnels, drop-offs, wallet events
- **Cryptographic integrity** -- every event hashed with keccak256
- **On-chain proof** -- Merkle roots anchored on Base mainnet, tamper-proof
- **Data privacy** -- only a 32-byte root goes on-chain; event data stays private
- **Data ownership** -- your data lives in Supabase, not ours
- **2-line integration** -- one script tag, live in 30 seconds

---

## Tech Stack

| Layer | Technology |
|-------|-----------|
| Frontend | Next.js 14 (App Router), React 18, TypeScript |
| Styling | Inline CSSProperties, CSS variables, Instrument Sans + JetBrains Mono |
| Auth | Supabase Auth (Google OAuth, GitHub OAuth, Email/Password) |
| Backend | Node.js relayer on Railway |
| Database | Supabase (PostgreSQL) |
| Blockchain | Base mainnet (chain ID 8453) |
| Contracts | KPPEAnchor (Merkle anchoring), App Registry |
| Payments | Stripe (Subscriptions + Payment Element) |
| Icons | KI Icon System (70+ custom SVG icons) |

---

## Smart Contracts (Base Mainnet)

| Contract | Address | Purpose |
|----------|---------|---------|
| **KPPEAnchor** | [`0x3B53F7044E47766769156bF210c2661F03Df45dd`](https://basescan.org/address/0x3B53F7044E47766769156bF210c2661F03Df45dd) | Merkle root anchoring and proof verification |
| **App Registry** | [`0xcbf3e71fb2E09929682b8448442d184f8E1E37B8`](https://basescan.org/address/0xcbf3e71fb2E09929682b8448442d184f8E1E37B8) | App registration and ownership |

Both contracts are verified on Basescan.

---

## Comparison

| | Google Analytics | Mixpanel | Dune Analytics | **Kairos Analytics** |
|---|---|---|---|---|
| Behavioral tracking | Yes | Yes | No | **Yes** |
| Web3 / wallet events | No | No | On-chain only | **Yes (frontend + wallet)** |
| Client-side hashing | No | No | No | **Yes (keccak256)** |
| On-chain Merkle proof | No | No | No | **Yes (Base mainnet)** |
| Data ownership | Google owns it | Mixpanel owns it | Public blockchain | **You own it** |
| Ad-blocker resistant | No | Partially | Yes | **Yes** |
| Integration time | 5 min | 30 min | N/A | **30 seconds** |
| Free tier | Limited | Limited | Free queries | **5M events/mo** |

---

## How to use these docs

| If you want to... | Go to |
|---|---|
| Understand the system | [How It Works](getting-started/how-it-works.md) |
| Integrate in 3 minutes | [Quickstart](getting-started/quickstart.md) |
| Track custom events | [Event Tracking](sdk/tracking.md) |
| Verify data on-chain | [Proof Verification](sdk/verification.md) |
| Understand Stripe billing | [Stripe Architecture](stripe/architecture.md) |
| See the full architecture | [System Architecture](infrastructure/architecture.md) |
| Check the roadmap | [Roadmap](reference/roadmap.md) |
| Fix issues | [Troubleshooting](reference/troubleshooting.md) |

---

**Domain:** kairosanalytics.org
**Primary chain:** Base mainnet
**Relayer:** kairos-relayer-production.up.railway.app
