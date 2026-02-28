# Kairos Analytics

**The first Web3 analytics protocol with cryptographic proof of data integrity.**

Every event your dApp generates is hashed with keccak256, batched into Merkle trees, and anchored on Base mainnet. Your analytics data isn't just collected — it's mathematically proven and publicly verifiable.

## Why Kairos?

Traditional analytics tools like Google Analytics weren't designed for Web3. They can't track wallet interactions, don't understand on-chain behavior, and offer zero proof that the data hasn't been tampered with. Block explorers show on-chain activity but miss everything that happens before and after a transaction.

Kairos fills the gap: **full-funnel analytics from first page view to confirmed transaction, with every event cryptographically anchored on-chain.**

## Core Features

**2-Line Integration** — Drop a script tag into your dApp. 15+ event types are auto-tracked with zero configuration. Wallet detection, page views, clicks, scroll depth, transactions — all captured automatically.

**Cryptographic Proof** — Every event is hashed. Batches are organized into Merkle trees. The Merkle root is anchored on Base mainnet via the KPPEAnchor contract. Batch-level integrity is public and verifiable on Basescan. Event-level verification is available to app owners via the API — keeping user data private by design.

**21 Dashboard Widgets** — Real-time analytics purpose-built for Web3. Wallet analytics, transaction funnels, conversion tracking, session timelines, error monitoring, geographic distribution, and more.

**Web3 Native** — Auto-detects MetaMask, Coinbase Wallet, Rabby, Brave Wallet, Trust Wallet, and Phantom. Tracks wallet connects, disconnects, chain switches, and transaction lifecycles without any additional code.

## Quick Example

```html
<script>window.KAIROS_APP_ID = "your-app-id";</script>
<script src="https://kairosanalytics.org/snippet.js" async></script>
```

That's it. Your dApp is now tracked with on-chain proof.

## Architecture at a Glance

```
Browser (snippet.js)
    ↓ POST /track
Relayer (Node.js on Railway)
    ↓ keccak256 hash → Merkle tree
Base Mainnet (KPPEAnchor contract)
    ↓ anchor()
Verifiable on Basescan
```

## Plans

| Plan | Price | Events/month | Widgets | Proof |
|------|-------|-------------|---------|-------|
| Free | $0 | 5M | 7 | Base Sepolia |
| Builder | $29/mo | 50M | 14 | Base mainnet (daily) |
| Protocol | $99/mo | 500M | 21 | Base mainnet (hourly) |

[Get started for free →](https://kairosanalytics.org/register.html)

## Links

* **Website**: [kairosanalytics.org](https://kairosanalytics.org)
* **Dashboard**: [kairosanalytics.org/dashboard](https://kairosanalytics.org/dashboard)
* **KPPEAnchor Contract**: Verified on Base mainnet — [View on Basescan](https://basescan.org)
* **Status**: [kairosanalytics.org/status](https://kairosanalytics.org/status)
