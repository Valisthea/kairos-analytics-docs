# Kairos Analytics

**Trustless behavioral analytics for Web3 — with on-chain Merkle proof of integrity.**

Kairos Analytics (Kairos Lab) tracks what happens on your frontend before the transaction: where users hesitate, which flows convert, where they drop off. Every event is hashed client-side with keccak256, batched into Merkle trees, and the root is anchored immutably on Base mainnet. No one — not even Kairos — can modify your data after collection.

---

## The K-PPE Trustless Model

Kairos Proof Protocol Engine (K-PPE) replaces the traditional trusted-server model with cryptographic proof at every layer:

1. **Client-side hashing** -- The SDK hashes each event with keccak256 before it leaves the browser. The server never sees raw data before the hash exists.
2. **Merkle tree batching** -- The K-PPE Engine groups hashed events into binary Merkle trees (max 256 events per batch) every 60 seconds.
3. **On-chain anchoring** -- Only the opaque 32-byte Merkle root is stored on Base mainnet. No event data is ever written on-chain.
4. **Trustless verification** -- Anyone can verify that a specific event is included in a batch using its Merkle proof and the on-chain root. No permission needed.

---

## Comparison

| | Google Analytics | Mixpanel | Dune Analytics | **Kairos Analytics** |
|---|---|---|---|---|
| Behavioral tracking | Yes | Yes | No | Yes |
| Web3 / wallet events | No | No | Yes | Yes |
| Client-side hashing | No | No | No | **Yes (keccak256)** |
| On-chain Merkle proof | No | No | No | **Yes (Base mainnet)** |
| Trustless verification | No | No | No | **Yes** |
| Data you own | No | No | No | Yes |
| Works without cookies | No | No | Yes | Yes |
| Integration time | 5 min | 30 min | N/A | **3 min** |

---

## Smart Contracts (Base Mainnet)

| Contract | Address | Purpose |
|---|---|---|
| **KPPEAnchor** | [`0x3B53F7044E47766769156bF210c2661F03Df45dd`](https://basescan.org/address/0x3B53F7044E47766769156bF210c2661F03Df45dd) | Merkle root anchoring and proof verification |
| **App Registry** | [`0xcbf3e71fb2E09929682b8448442d184f8E1E37B8`](https://basescan.org/address/0xcbf3e71fb2E09929682b8448442d184f8E1E37B8) | App registration and ownership |

Both contracts are verified on Basescan and deployed on Base mainnet (chain ID 8453).

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
