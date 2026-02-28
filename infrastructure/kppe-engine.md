# K-PPE Engine

Kairos Proof Protocol Engine -- the system that bridges client-side event hashes to on-chain Merkle roots.

---

## How K-PPE Works

1. **Client-side hashing** -- The SDK hashes each event with keccak256 before it leaves the browser
2. **Local Merkle tree building** -- Events are grouped into binary Merkle trees (max 256 per batch) every 60 seconds
3. **On-chain anchoring** -- Only the 32-byte Merkle root is stored on Base mainnet via KPPEAnchor.anchorBatch()
4. **Trustless verification** -- Anyone can verify that a specific event is included in a batch using its Merkle proof

## K-PPE Mode

The relayer supports three modes via the KPPE_MODE environment variable:

| Mode | Description | Data on-chain? |
|------|-------------|----------------|
| **kppe** (default) | Trustless Merkle anchor -- only the 32-byte root goes on-chain | No (private) |
| legacy | Old trackBatch -- full event data on-chain | Yes (public) -- DEPRECATED |
| disabled | No on-chain submission | N/A |

Production deployments should always use kppe mode.

## Batch Lifecycle

1. Events accumulate in memory per appId
2. Timer or size threshold triggers batch creation
3. Merkle tree is built from event hashes
4. Root is anchored on Base mainnet
5. Receipt (txHash, block, root, eventCount) is stored in Supabase merkle_batches table
