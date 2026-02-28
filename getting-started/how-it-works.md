# How It Works

Kairos Analytics operates as a 4-step pipeline that takes raw browser events and turns them into cryptographically verifiable on-chain proofs.

## The Pipeline

### Step 1 — Snippet Captures Events

The Kairos snippet runs in your users' browsers. It automatically captures 15+ event types without any configuration:

* **Web2 events**: page views, clicks, scroll depth, form submissions, outbound clicks, tab visibility, performance metrics, JavaScript errors
* **Web3 events**: wallet detection, wallet connect/disconnect, chain changes, transaction submission/confirmation/failure, token swaps

The snippet also auto-detects which wallet provider the user has installed (MetaMask, Coinbase Wallet, Rabby, Brave Wallet, Trust Wallet, Phantom) and fires a `wallet_detected` event.

Each event is sent to the Kairos relayer via `POST /track` with the app ID, event type, timestamp, and metadata.

### Step 2 — Relayer Hashes Events

The relayer receives events and hashes each one using **keccak256** (the same hash function used by Ethereum). This creates a unique, deterministic fingerprint for every event.

The hash includes the event type, timestamp, app ID, and payload — making it impossible to alter any detail without changing the hash.

### Step 3 — Merkle Tree Batching

Events are accumulated per app ID. When a batch reaches the configured threshold (size or time-based), the relayer constructs a **Merkle tree** from all the event hashes in the batch.

A Merkle tree is a binary hash tree where every leaf is an event hash, and each parent node is the hash of its two children. The single hash at the top — the **Merkle root** — represents every event in the batch.

This structure allows **individual event verification**: you can prove any single event is included in a batch without revealing the other events, by providing the Merkle proof (the sibling hashes along the path from the leaf to the root).

### Step 4 — On-Chain Anchoring

The Merkle root is submitted to the **KPPEAnchor** smart contract on Base mainnet via `anchor()`. The transaction receipt is stored in Supabase alongside the batch metadata (event count, timestamp, app ID).

Once anchored, the Merkle root is **permanent, public, and tamper-proof**. Anyone can:

1. Recompute the hash of any event
2. Verify its inclusion in the Merkle tree using the proof
3. Confirm the Merkle root matches what's stored on-chain via Basescan

## Verification

Kairos operates on two verification levels, designed to balance data integrity with user privacy.

### Batch-Level Integrity (Public)

Anyone can verify that a batch of events was anchored on-chain:

1. Go to the KPPEAnchor contract on Basescan
2. Call `getAnchor(batchId)` — returns the Merkle root, event count, timestamp, and app ID
3. This proves that a specific dataset existed at a specific time and has not been modified since

The Merkle root on-chain proves that **no event was added, removed, or modified after anchoring** — without exposing any individual user data. Batch-level integrity is public and permanent.

### Event-Level Verification (Authorized Only)

To verify that a specific event is included in a batch, you need the Merkle proof — the sibling hashes along the path from the event leaf to the root. This proof is available only to authorized app owners via the Kairos API (`GET /verify/proof`).

```
Event data → keccak256(event) → Merkle proof → Merkle root → On-chain anchor
```

This is intentional: events contain sensitive user data (approximate geolocation, wallet addresses, navigation behavior, device info). Publishing individual proofs would expose this data. By keeping event-level verification behind authentication, Kairos ensures **privacy by design** while maintaining **tamper-proof integrity**.

### What This Means in Practice

You don't need to trust Kairos to know your dataset hasn't been altered — the on-chain Merkle root proves it mathematically. But individual event data stays private, visible only to you as the dApp owner.

## Data Flow Diagram

```
┌─────────────────────────────────────────────────┐
│  Browser                                         │
│  snippet.js auto-tracks events                   │
│  → page_view, click, wallet_connected, tx_sent  │
└──────────────────┬──────────────────────────────┘
                   │ POST /track
                   ▼
┌─────────────────────────────────────────────────┐
│  Relayer (Node.js / Railway)                     │
│  1. Receives event                               │
│  2. keccak256(event) → hash                      │
│  3. Accumulates in batch by appId                │
│  4. Batch ready → builds Merkle tree             │
│  5. anchor(merkleRoot) on Base mainnet           │
│  6. Stores receipt in Supabase                   │
└──────────────────┬──────────────────────────────┘
                   │ anchor()
                   ▼
┌─────────────────────────────────────────────────┐
│  Base Mainnet                                    │
│  KPPEAnchor contract stores Merkle root          │
│  Verifiable on Basescan by anyone                │
└─────────────────────────────────────────────────┘
```

## What Makes This Different

| Feature | Traditional Analytics | Block Explorers | Kairos |
|---------|----------------------|-----------------|--------|
| Page views & clicks | ✅ | ❌ | ✅ |
| Wallet tracking | ❌ | Partial | ✅ |
| Transaction funnels | ❌ | ❌ | ✅ |
| Proof of data integrity | ❌ | N/A | ✅ On-chain |
| Self-hosted option | ❌ | ❌ | Planned |
| Privacy-first | ❌ | ❌ | ✅ No cookies |
