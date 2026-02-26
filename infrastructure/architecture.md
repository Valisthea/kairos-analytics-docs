# Architecture

A complete technical overview of all Kairos Analytics components and the K-PPE trustless pipeline.

---

## Components

### SDK (`@kairoslab/analytics-sdk`)

Published on npm. Can be imported via npm or loaded as a browser script. Runs entirely in the browser.

**Responsibilities:**
- Session management (generates and persists session IDs in `sessionStorage`)
- **Client-side keccak256 hashing** of every event before transmission (uses ethers v6)
- Canonical JSON serialization for deterministic hashing (`JSON.stringify(event, sortedKeys)`)
- Direct writes to Supabase (event data + `client_hash`)
- Receipt generation and local storage
- Optional `onReceipt` callback for real-time receipt handling

### Supabase (Data Storage)

PostgreSQL database hosted on Supabase. Stores all event data and K-PPE metadata.

**Tables:**

| Table | Purpose |
|---|---|
| `analytics_events` | Raw event data, session info, `client_hash`, `batch_id` |
| `merkle_batches` | Batch metadata: Merkle root, TX hash, status, event count |
| `event_receipts` | Per-event Merkle proofs: leaf hash, proof path, batch reference |

### K-PPE Engine (Railway)

A TypeScript cron job deployed on Railway. Runs every 60 seconds.

**Responsibilities:**
- Query Supabase for unprocessed events (max 256 per batch)
- Read `client_hash` from each event (never re-hashes)
- Build a binary Merkle tree with sorted-pair hashing
- Pad leaves to the next power of 2 with zero hashes
- Call `KPPEAnchor.anchorMerkleRoot()` on Base mainnet
- Write Merkle proofs for each event back to `event_receipts`
- Update batch status in `merkle_batches`

### KPPEAnchor Contract (Base Mainnet)

`0x3B53F7044E47766769156bF210c2661F03Df45dd`

A Solidity contract that stores Merkle roots and provides trustless verification.

**Responsibilities:**
- Store batch Merkle roots immutably on-chain
- Maintain chain-of-trust via `prevRoot` linking
- Provide `verifyProof` and `verifyProofView` for anyone to verify event inclusion
- Emit `KPPEMerkleAnchored` and `KPPEProofVerified` events
- Track `totalEventsAnchored` and `latestBatchId`

### App Registry Contract (Base Mainnet)

`0xcbf3e71fb2E09929682b8448442d184f8E1E37B8`

Stores the mapping of App IDs to their owner addresses. Used for authentication and authorization.

### Relayer (Railway)

A Node.js/Express service deployed on Railway. Acts as the API layer between the frontend and the data.

**Responsibilities:**
- Receive events via `POST /track` (legacy compatibility)
- Session management and validation
- Expose K-PPE security headers (`X-KA-HAP-*`)
- Provide `/kpe/health` endpoint for K-PPE status
- Serve dashboard data via query endpoints

### Dashboard (Vercel)

A Next.js application deployed on Vercel. Provides real-time analytics, K-PPE proof visualization, and Merkle tree inspection.

---

## Data flow diagram

```
                       CLIENT SIDE                              SERVER SIDE
  +---------------------------------------------+    +---------------------------+
  |                                             |    |                           |
  |  Your dApp                                  |    |  Supabase (PostgreSQL)    |
  |    |                                        |    |                           |
  |    | kairos.track('event', { data })        |    |  analytics_events         |
  |    v                                        |    |  merkle_batches           |
  |  @kairoslab/analytics-sdk                   |    |  event_receipts           |
  |    |                                        |    |                           |
  |    | 1. Canonicalize: JSON.stringify(sorted) |    +------------|-------------+
  |    | 2. Hash: keccak256(canonical)          |                  |
  |    | 3. Send event + client_hash            |                  |
  |    |                                        |                  v
  |    +---------------------------------------->    +---------------------------+
  |    |                                        |    |                           |
  |    | 4. Return EventReceipt                 |    |  K-PPE Engine (Railway)   |
  |    |    { eventId, clientHash, timestamp }  |    |                           |
  |    |                                        |    |  Cron: every 60 seconds   |
  +---------------------------------------------+    |  1. Fetch pending events  |
                                                     |  2. Read client_hash      |
                                                     |  3. Build Merkle tree     |
                                                     |  4. Anchor root on-chain  |
                                                     |  5. Store proofs          |
                                                     |                           |
                                                     +------------|-------------+
                                                                  |
                                                                  v
                                                     +---------------------------+
                                                     |                           |
                                                     |  Base Mainnet             |
                                                     |  KPPEAnchor.sol           |
                                                     |                           |
                                                     |  Stores:                  |
                                                     |  - merkleRoot (bytes32)   |
                                                     |  - eventCount (uint32)    |
                                                     |  - treeDepth (uint8)      |
                                                     |  - prevRoot (bytes32)     |
                                                     |  - anchoredAt (uint64)    |
                                                     |                           |
                                                     |  Emits:                   |
                                                     |  KPPEMerkleAnchored(...)  |
                                                     |                           |
                                                     +---------------------------+
                                                                  |
                        VERIFICATION                              |
  +---------------------------------------------+                |
  |                                             |                |
  |  Anyone (SDK, Basescan, direct RPC)         |                |
  |                                             |                |
  |  1. Recompute hash from original data       |<---------------+
  |  2. Retrieve Merkle proof from Supabase     |
  |  3. Call verifyProofView(batchId,           |
  |     eventHash, proof, leafIndex)            |
  |  4. Returns true/false                      |
  |                                             |
  +---------------------------------------------+
```

---

## Network topology

| Component | Hosted on | Technology | Access |
|---|---|---|---|
| SDK | npm / CDN | TypeScript, ethers v6 | Browser import |
| Supabase | Supabase Cloud | PostgreSQL | Authenticated client |
| K-PPE Engine | Railway | TypeScript cron | Internal (Supabase + RPC) |
| KPPEAnchor | Base mainnet | Solidity 0.8.24 | Public read, authorized write |
| App Registry | Base mainnet | Solidity | Public read, owner write |
| Relayer | Railway | Node.js / Express | Public HTTPS |
| Dashboard | Vercel | Next.js | Public HTTPS |

---

## Security model

### Client-side hashing (trustless foundation)

The SDK computes `keccak256(canonicalJSON)` for every event **in the user's browser** before any server sees the data. This means:

- The hash exists before Kairos infrastructure is involved
- The server cannot produce a hash that differs from the client's hash
- Any modification to the data after collection will produce a different hash

### Sorted-pair Merkle trees

The K-PPE Engine builds Merkle trees using sorted-pair hashing: `keccak256(min(a,b) || max(a,b))`. This ensures that the tree is deterministic regardless of the order in which sibling nodes are combined, eliminating a class of implementation bugs.

### Chain of trust (prevRoot)

Each batch stores a `prevRoot` field linking it to the previous batch's Merkle root. This creates a hash chain similar to a blockchain, making it impossible to insert, delete, or reorder batches without detection.

### On-chain immutability

Once a Merkle root is anchored on Base mainnet, it cannot be modified or deleted. The `KPPEMerkleAnchored` event is permanently recorded in the blockchain's event log.

### Authorization model

Only wallets explicitly authorized via `setAuthorized()` (called by the contract owner) can anchor Merkle roots. This prevents unauthorized parties from anchoring fake batches. However, **verification is fully permissionless** -- anyone can call `verifyProofView`.

### No PII on-chain

Only 32-byte opaque Merkle roots are stored on-chain. No event data, session IDs, user agents, or any other data is written to the blockchain. The underlying data stays in Supabase.

### KA-HAP 7-Layer Security Architecture

The Kairos Analytics Hardened Application Protocol (KA-HAP) provides defense-in-depth across seven security layers, with each layer dynamically verified using K-PPE cryptographic self-tests. The security status is exposed via `X-KA-HAP-*` headers from the relayer for real-time monitoring.
