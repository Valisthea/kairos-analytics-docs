# K-PPE Engine

K-PPE (Kairos Proof of Process Events) is the cryptographic engine that transforms raw analytics events into verifiable on-chain proofs.

## What K-PPE Does

1. **Hashes events** — Each event is hashed with keccak256, creating a unique fingerprint
2. **Builds Merkle trees** — Event hashes are organized into binary Merkle trees
3. **Computes roots** — The tree root is a single hash representing all events in the batch
4. **Anchors on-chain** — The root is stored permanently on Base mainnet

## Why It Matters

Without K-PPE, analytics data requires trust: you trust that the analytics provider hasn't modified the data. With K-PPE, trust is replaced by math: anyone can verify that a specific event exists in a batch by recomputing the hash and checking the Merkle proof against the on-chain root.

This is particularly important for Web3, where transparency and verifiability are core values.

## How Batching Works

Events are accumulated per app ID in an in-memory buffer. A batch is finalized when either condition is met:

* **Size threshold** — The buffer reaches 1,024 events
* **Time threshold** — 1 hour has passed since the last batch (Builder plan) or 1 day (Free plan)

Protocol plans use a 1-hour batching interval for more frequent proof anchoring.

When a batch is finalized:

```
1. All event hashes in the buffer are collected
2. A Merkle tree is constructed from the hashes
3. The Merkle root is computed
4. anchor(root, count, appId) is called on the KPPEAnchor contract
5. The transaction receipt is stored in Supabase (merkle_batches table)
6. The buffer is cleared
```

## Merkle Tree Structure

```
          Root
         /    \
       H(AB)  H(CD)
       /  \    /  \
     H(A) H(B) H(C) H(D)
      |    |    |    |
     ev1  ev2  ev3  ev4
```

Each leaf is the keccak256 hash of an event. Each parent node is the keccak256 hash of its two children concatenated. The root represents the entire batch.

## Proof Verification

Verification operates at two levels:

### Batch Integrity (Public)

The Merkle root stored on-chain proves that the entire batch of events is intact. Anyone can read the KPPEAnchor contract on Basescan and confirm that a batch was anchored at a specific time with a specific number of events. No event data is revealed — only the root hash.

This guarantees **tamper-proof integrity**: if any event in the batch had been added, removed, or modified after anchoring, the Merkle root would be different.

### Event Inclusion (Authorized)

To verify that a specific event exists in a batch:

1. Compute `H(B) = keccak256(ev2)`
2. Request the Merkle proof from the relayer: `[H(A), H(CD)]` (requires app owner authentication)
3. Compute: `H(AB) = keccak256(H(A) + H(B))`
4. Compute: `Root = keccak256(H(AB) + H(CD))`
5. Compare the computed root with the on-chain root

If they match, `ev2` is provably in the batch.

Event-level proofs are available only to authenticated app owners via `GET /verify/proof`. This is by design: events contain sensitive user data (geolocation, wallet addresses, navigation patterns, device information). Publishing individual Merkle proofs would expose this data publicly, which would violate user privacy.

### Privacy by Design

The architecture deliberately separates what's public from what's private:

| Data | Visibility | Purpose |
|------|-----------|---------|
| Merkle root | Public (on-chain) | Proves batch integrity |
| Event count + timestamp | Public (on-chain) | Proves batch metadata |
| Individual events | Private (API, auth required) | User analytics data |
| Merkle proofs | Private (API, auth required) | Event-level verification |

This means the system is **tamper-proof without being transparent about user data** — the optimal balance for analytics.

## K-PPE Modes

The relayer supports three K-PPE modes controlled by the `KPPE_MODE` environment variable:

| Mode | Behavior |
|------|----------|
| `full` | Events are hashed, batched, and anchored on-chain. Default for production. |
| `hash-only` | Events are hashed and batched but not anchored. Useful for testing. |
| `off` | K-PPE is disabled. Events are stored but not hashed. Used in development. |

## Anchoring Schedule by Plan

| Plan | Anchoring Frequency | Chain |
|------|-------------------|-------|
| Free | Daily | Base Sepolia (testnet) |
| Builder | Daily | Base Mainnet |
| Protocol | Hourly | Base Mainnet |

## Monitoring

The K-PPE engine status is available via:
* `GET /kppe/status` — Queue length, last anchor timestamp, next scheduled anchor
* `GET /kppe/batches?appId=xxx` — List of anchored batches with tx hashes
* Dashboard → Audit & Security → K-PPE tab — Visual overview

## Future: K-PPC

K-PPC (Kairos Proof Protocol Chain) is the next evolution, planned for deployment on Asterchain. K-PPC adds zero-knowledge proofs, allowing verification of analytics properties (e.g., "this dApp had more than 1,000 unique wallets this week") without revealing the raw event data.

K-PPC is documented separately and is not yet in production.
