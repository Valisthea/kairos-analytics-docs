# K-PPE Engine

The Kairos Proof Protocol Engine -- the system that bridges client-side event hashes to on-chain Merkle roots.

---

## Architecture Update: Local Merkle Builder

**Important:** The relayer now includes a **local Merkle tree builder** (`src/kpe/merkle.ts`). For most deployments, no separate K-PPE service is needed. The `BatchManager` inside the relayer handles Merkle tree construction and on-chain anchoring directly.

The standalone K-PPE Engine (described below) remains available for deployments that need a separate anchoring process, but the local builder is the recommended approach for new setups.

### How the local builder works

The relayer's `BatchManager` uses `flushKPPE()` on each batch cycle:

1. Collects pending events (events where `batched = false`)
2. Calls `buildMerkleBatch()` from `src/kpe/merkle.ts` -- reads `client_hash` values directly (never re-hashes)
3. Calls `OnChainSender.anchorMerkleRoot()` which invokes `KPPEAnchor.anchorBatch()` on Base mainnet
4. Stores Merkle proofs in the database
5. Marks events as batched

The deprecated path (`flushLegacy()`) calls `OnChainSender.sendBatch()` which publishes full event data on-chain. This is controlled by the `KPPE_MODE` environment variable (see [Relayer](relayer.md)).

---

## Standalone K-PPE Engine (Railway)

For deployments that prefer a separate anchoring service, the K-PPE Engine is a TypeScript cron job deployed on Railway that executes on a 60-second interval. Each cycle it:

1. **Fetches** unprocessed events from the `analytics_events` table in Supabase (events where `batch_id IS NULL`)
2. **Reads** the `client_hash` field from each event -- it **never re-hashes** the event data
3. **Builds** a binary Merkle tree from the client hashes
4. **Anchors** the Merkle root on Base mainnet via `KPPEAnchor.anchorBatch()`
5. **Stores** the Merkle proof for each event in the `event_receipts` table
6. **Updates** the `merkle_batches` table with the TX hash, status, and batch metadata

---

## Why it never re-hashes

This is the critical design decision that makes K-PPE trustless.

The SDK computes `keccak256(canonicalJSON)` in the user's browser and sends both the raw event data and the resulting `client_hash` to Supabase. The K-PPE Engine takes those `client_hash` values and uses them directly as Merkle tree leaves.

If the engine re-hashed the event data server-side, a compromised server could modify the data and produce a new hash that matches. By using the client-computed hash as-is, the engine commits to the exact hash the client produced. Any discrepancy between the stored data and the client hash is detectable by anyone who recomputes the hash from the original data.

```
CLIENT (browser)                    SERVER (K-PPE Engine)

event = { type: 'swap', ... }
canonical = JSON.stringify(sorted)
client_hash = keccak256(canonical)

  -----> event + client_hash ----->
                                    reads client_hash directly
                                    builds Merkle tree from client_hash values
                                    anchors root on-chain
```

---

## Merkle tree construction

### Leaf preparation

Each event's `client_hash` (a 32-byte keccak256 hash) becomes a leaf node. If the number of leaves is not a power of 2, zero hashes (`0x0000...0000`) are appended to pad the tree.

### Tree building

The tree is built bottom-up using sorted-pair hashing:

```typescript
function sortedPairHash(a: string, b: string): string {
  if (a.toLowerCase() <= b.toLowerCase()) {
    return keccak256(solidityPacked(['bytes32', 'bytes32'], [a, b]))
  } else {
    return keccak256(solidityPacked(['bytes32', 'bytes32'], [b, a]))
  }
}
```

Sorted-pair hashing ensures that the same result is produced regardless of which node is "left" and which is "right". This matches the verification logic in the `KPPEAnchor.sol` contract exactly.

### Example with 5 events

```
Input: 5 client hashes --> padded to 8 leaves (next power of 2)

Level 0 (leaves):   H0  H1  H2  H3  H4  00  00  00
Level 1:            H01     H23     H4_     0_
Level 2:            H0123           H4___
Level 3 (root):     ROOT

Depth: 3
Leaf count: 5 (padded to 8)
Root: 0x... (anchored on-chain)
```

---

## Batching rules

| Parameter | Value | Description |
|---|---|---|
| **Cron interval** | 60 seconds | How often the engine runs |
| **Max events per batch** | 256 | Maximum leaves in one Merkle tree |
| **Min events per batch** | 1 | A batch is created if any pending events exist |
| **Batch ID** | Strictly increasing | Enforced by the contract (`batchId > latestBatchId`) |

If more than 256 events are pending, the engine processes the oldest 256 first. The remaining events will be batched in the next cycle.

---

## On-chain anchoring

After building the Merkle tree, the relayer (or standalone engine) calls `OnChainSender.anchorMerkleRoot()`, which invokes the contract method:

```solidity
KPPEAnchor.anchorBatch(
    batchId,        // uint256 -- next batch ID
    merkleRoot,     // bytes32 -- root of the Merkle tree
    eventCount,     // uint32  -- number of real events (not padding)
    treeDepth,      // uint8   -- depth of the tree
    timestamp       // uint64  -- unix timestamp of batch creation
)
```

**Only the 32-byte Merkle root goes on-chain.** No event data is written to the blockchain. Your analytics data stays private.

The contract automatically sets `prevRoot` to the current `latestRoot` before updating, creating the chain of trust.

The system waits for the transaction to be confirmed (typically 2 block confirmations on Base) before updating the batch status in Supabase.

### Deprecated: `sendBatch()` (legacy mode)

The old `OnChainSender.sendBatch()` method published full event data on-chain as calldata. This is **deprecated** and should not be used for new deployments. It exists only for backward compatibility when `KPPE_MODE=legacy`. In legacy mode, all event data is **public** on the blockchain.

---

## Proof storage

After a batch is confirmed on-chain, the engine computes the Merkle proof for each event and writes it to `event_receipts`:

```
event_receipts
--------------
id            -- UUID
event_id      -- Reference to analytics_events
batch_id      -- Reference to merkle_batches
client_hash   -- The keccak256 hash (leaf)
leaf_index    -- Position in the Merkle tree
proof         -- JSON array of sibling hashes
root          -- The Merkle root (for quick validation)
tx_hash       -- Base mainnet transaction hash
verified      -- Boolean (proof validated at write time)
created_at    -- Timestamp
```

The proof is an array of sibling hashes that, combined with the leaf hash using sorted-pair hashing, reconstructs the root. This proof is what the SDK's `verify()` method or Basescan's `verifyProofView` uses to confirm event inclusion.

---

## Supabase tables

### analytics_events

| Column | Type | Description |
|---|---|---|
| `id` | UUID | Primary key |
| `app_id` | text | Application identifier |
| `event_type` | text | Event name (e.g., `page_view`, `swap_attempted`) |
| `data` | jsonb | Event properties |
| `client_hash` | text | keccak256 hash computed by the SDK (born client-side) |
| `client_event_id` | text | UUID generated by the SDK for event correlation |
| `session_id` | text | Session identifier |
| `batched` | boolean | Whether this event has been included in a Merkle batch |
| `batch_id` | UUID | Reference to merkle_batches (NULL if pending) |
| `created_at` | timestamptz | Insertion timestamp |

### merkle_batches

| Column | Type | Description |
|---|---|---|
| `id` | UUID | Primary key |
| `batch_number` | integer | Strictly increasing batch identifier |
| `merkle_root` | text | Root hash of the Merkle tree |
| `event_count` | integer | Number of events in this batch |
| `tree_depth` | integer | Depth of the Merkle tree |
| `tx_hash` | text | Base mainnet transaction hash |
| `status` | text | `pending`, `submitted`, `confirmed`, `failed` |
| `prev_root` | text | Previous batch's Merkle root |
| `created_at` | timestamptz | Batch creation timestamp |
| `confirmed_at` | timestamptz | On-chain confirmation timestamp |

### event_receipts

| Column | Type | Description |
|---|---|---|
| `id` | UUID | Primary key |
| `event_id` | UUID | Reference to analytics_events |
| `batch_id` | UUID | Reference to merkle_batches |
| `client_hash` | text | The keccak256 hash (leaf) |
| `leaf_index` | integer | Position in the Merkle tree |
| `proof` | jsonb | Array of sibling hashes |
| `root` | text | Merkle root |
| `tx_hash` | text | Base mainnet TX hash |
| `verified` | boolean | Proof validated at write time |
| `created_at` | timestamptz | Timestamp |

### kppe_verifications

| Column | Type | Description |
|---|---|---|
| `id` | UUID | Primary key |
| `event_id` | UUID | Reference to analytics_events |
| `batch_id` | UUID | Reference to merkle_batches |
| `verified_by` | text | Who initiated the verification |
| `result` | boolean | Verification result |
| `created_at` | timestamptz | Timestamp |

### KA-HAP tables

The following tables support the KA-HAP (Kairos Analytics Hardened Application Protocol) 7-layer security architecture:

| Table | Purpose |
|---|---|
| `kpe_proof_batches` | K-PPE proof batch records for KA-HAP verification |
| `kpe_event_proofs` | Individual event proofs with KA-HAP security metadata |
| `kpe_signing_keys` | Cryptographic signing key management |
| `kpe_verifications` | KA-HAP verification audit trail |
| `kpe_status` | K-PPE system status snapshots |
| `kpe_nonces` | Replay-attack prevention nonces |

---

## Environment variables

| Variable | Description | Required |
|---|---|---|
| `SUPABASE_URL` | Supabase project URL | Yes |
| `SUPABASE_SERVICE_ROLE_KEY` | Supabase service role key (not anon key) | Yes |
| `PRIVATE_KEY` | Private key of the authorized K-PPE Engine wallet | Yes |
| `KPPE_CONTRACT_ADDRESS` | KPPEAnchor contract address on Base mainnet | Yes |
| `RPC_URL` | Base mainnet RPC endpoint (e.g., `https://mainnet.base.org`) | Yes |
| `BATCH_INTERVAL_MS` | Cron interval in milliseconds (default: 60000) | No |
| `MAX_EVENTS_PER_BATCH` | Maximum events per Merkle tree (default: 256) | No |
| `CONFIRMATIONS` | Block confirmations to wait (default: 2) | No |

---

## Monitoring

### Railway logs

Access real-time logs from your Railway project: **Railway > your service > Logs tab**

Key log lines to look for:

```
[K-PPE] Engine started | interval: 60s
[K-PPE] Cycle: 12 pending events found
[K-PPE] Merkle tree built | leaves: 12, depth: 4, root: 0x7a3f...
[K-PPE] TX submitted: 0xdef... | waiting for confirmation...
[K-PPE] Batch #42 confirmed in block 12345678 | 12 events anchored
[K-PPE] Proofs stored for 12 events in event_receipts
[K-PPE] Cycle complete | totalEventsAnchored: 1,234
```

Warning and error patterns:

```
[K-PPE] No pending events -- skipping cycle
[K-PPE] WARNING: RPC rate limited, retrying in 5s...
[K-PPE] ERROR: TX reverted -- insufficient gas
[K-PPE] ERROR: Batch ID conflict -- latestBatchId mismatch
```

### Health checks

The K-PPE Engine exposes its status through the relayer's `/kpe/health` endpoint (see [Relayer](relayer.md)). The dashboard checks this endpoint to display K-PPE status.

### On-chain monitoring

You can monitor K-PPE anchoring activity directly on Basescan:
- [KPPEAnchor events](https://basescan.org/address/0x3B53F7044E47766769156bF210c2661F03Df45dd#events) -- see all `KPPEMerkleAnchored` events
- Call `totalEventsAnchored()` and `latestBatchId()` on the Read Contract tab for live counts
