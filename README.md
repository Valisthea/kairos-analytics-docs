# Kairos Analytics

**Trustless behavioral analytics for Web3 — with on-chain Merkle proof of integrity.**

Kairos Analytics tracks what happens on your frontend before the transaction: where users hesitate, which flows convert, where they drop off. Every event is hashed client-side with keccak256, batched into Merkle trees, and the root is anchored immutably on Base mainnet. No one — not even Kairos — can modify your data after collection.

---

## Why Kairos Analytics?

If you run a dApp, you need to understand your users. But traditional analytics tools (Google Analytics, Mixpanel) were built for Web2 -- they own your data, track via cookies, and offer zero cryptographic guarantees. Blockchain-native tools like Dune focus on on-chain data, not frontend behavior.

Kairos Analytics is the first analytics solution purpose-built for dApps that gives you:

- **Behavioral analytics** -- page views, clicks, funnels, drop-offs, wallet events
- **Cryptographic integrity** -- every event is hashed in your user's browser with keccak256
- **On-chain proof** -- Merkle roots anchored on Base mainnet; your data is tamper-proof
- **Data privacy** -- only a 32-byte root goes on-chain; your event data stays private
- **Data ownership** -- your data lives in your Supabase instance, not ours
- **3-minute integration** -- one npm install or one script tag

---

## The K-PPE Trustless Model

Kairos Proof Protocol Engine (K-PPE) replaces the traditional trusted-server model with cryptographic proof at every layer:

1. **Client-side hashing** -- The SDK hashes each event with keccak256 before it leaves the browser. The hash is born in your user's browser and is never modified by any server.
2. **Local Merkle tree building** -- The relayer includes a built-in Merkle tree builder (`src/kpe/merkle.ts`). No external service is needed for core anchoring. Events are grouped into binary Merkle trees (max 256 per batch) every 60 seconds.
3. **On-chain anchoring** -- Only the opaque 32-byte Merkle root is stored on Base mainnet via `KPPEAnchor.anchorBatch()`. No event data is ever written on-chain. Your analytics data stays **private**.
4. **Trustless verification** -- Anyone can verify that a specific event is included in a batch using its Merkle proof and the on-chain root. No permission needed.

---

## Comparison with other solutions

| | Google Analytics | Mixpanel | Dune Analytics | **Kairos Analytics** |
|---|---|---|---|---|
| Behavioral tracking | Yes | Yes | No | **Yes** |
| Web3 / wallet events | No | No | Yes (on-chain only) | **Yes (frontend + wallet)** |
| Client-side hashing | No | No | No | **Yes (keccak256)** |
| On-chain Merkle proof | No | No | No | **Yes (Base mainnet)** |
| Trustless verification | No | No | No | **Yes** |
| Data privacy | Google owns it | Mixpanel owns it | Public blockchain | **You own it (Supabase + on-chain proof)** |
| Event data on-chain | N/A | N/A | Yes (public) | **No (only Merkle root -- data stays private)** |
| Works without cookies | No | No | Yes | **Yes** |
| Ad-blocker resistant | No | Partially | Yes | **Yes** |
| Integration time | 5 min | 30 min | N/A (query tool) | **3 min** |
| Free tier | Yes (limited) | Yes (limited) | Free for queries | **Yes (5M events/mo)** |

### Google Analytics vs. Kairos

Google Analytics gives you page views and sessions, but Google owns your data, tracks users with cookies, and offers no cryptographic guarantees. GA is frequently blocked by ad blockers. Kairos gives you the same behavioral analytics but with on-chain proof and full data ownership.

### Mixpanel vs. Kairos

Mixpanel offers powerful funnel and retention analytics, but your data lives on Mixpanel's servers. There is no way to prove your data has not been tampered with. Kairos provides similar event tracking with the added guarantee that every event is hashed client-side and anchored on-chain.

### Dune Analytics vs. Kairos

Dune is excellent for querying on-chain data, but it cannot track frontend behavior -- clicks, page views, wallet connection flows, UI drop-offs. Kairos tracks what happens **before** the transaction hits the chain.

---

## Smart Contracts (Base Mainnet)

| Contract | Address | Purpose |
|---|---|---|
| **KPPEAnchor** | [`0x3B53F7044E47766769156bF210c2661F03Df45dd`](https://basescan.org/address/0x3B53F7044E47766769156bF210c2661F03Df45dd) | Merkle root anchoring and proof verification |
| **App Registry** | [`0xcbf3e71fb2E09929682b8448442d184f8E1E37B8`](https://basescan.org/address/0xcbf3e71fb2E09929682b8448442d184f8E1E37B8) | App registration and ownership |

Both contracts are verified on Basescan and deployed on Base mainnet (chain ID 8453).

---

## K-PPE Mode (Relayer Configuration)

The relayer supports three on-chain submission modes via the `KPPE_MODE` environment variable:

| Mode | Description | Data on-chain? |
|---|---|---|
| **`kppe`** (default) | Trustless Merkle anchor -- only the 32-byte root goes on-chain | **No** (private) |
| `legacy` | Old `trackBatch` -- full event data published on-chain | Yes (public) -- **DEPRECATED** |
| `disabled` | No on-chain submission | N/A |

**Production deployments should always use `kppe` mode.** The legacy mode is deprecated and will be removed in a future release. It exists only for backward compatibility.

---

## Plans and Pricing

| Plan | Price | Events/month | Retention | Features |
|---|---|---|---|---|
| **Free Beta** | $0/mo | 5M events | 30 days | Dashboard, K-PPE proof, verification |
| **Builder** | $29/mo | 50M events | 12 months | Everything in Free + extended retention |
| **Protocol** | $99/mo | Unlimited | Unlimited | API access, team seats, priority support |

All plans include on-chain Merkle proof anchoring via K-PPE. No credit card required for the Free plan.

---

## How to use these docs

New to Kairos? Start with [How It Works](getting-started/how-it-works.md) then follow the [Quick Start](getting-started/quickstart.md).

Already integrated? Go to [Tracking Events](sdk/tracking.md) or [Verification](sdk/verification.md).

Want to understand the cryptography? Read [K-PPE Engine](infrastructure/kppe-engine.md) and [Smart Contracts](infrastructure/contracts.md).

Something broken? Check [Troubleshooting](reference/troubleshooting.md).

---

## Quick links

| | |
|---|---|
| [Quick Start](getting-started/quickstart.md) | Live in 3 minutes |
| [Installation](sdk/installation.md) | npm or script tag |
| [Tracking Events](sdk/tracking.md) | Track events, get receipts |
| [Trustless Verification](sdk/verification.md) | Verify your data on-chain |
| [Architecture](infrastructure/architecture.md) | Full system overview |
| [Smart Contracts](infrastructure/contracts.md) | KPPEAnchor ABI and addresses |
| [K-PPE Engine](infrastructure/kppe-engine.md) | Merkle batching engine |
| [Authentication](dashboards/authentication.md) | Sign in to your dashboard |
| [Troubleshooting](reference/troubleshooting.md) | Fix common issues |
| [Roadmap](reference/roadmap.md) | What is live and what is next |
