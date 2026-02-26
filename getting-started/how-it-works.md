# How It Works

Understanding the full K-PPE data flow before you write a single line of code.

---

## The five components

Kairos Analytics is built from five components working together in a trustless pipeline:

### 1. The SDK (`@kairoslab/analytics-sdk`)

A lightweight JavaScript/TypeScript library you add to your frontend via npm or a `<script>` tag. It tracks page views, custom events, swaps, and transactions. Critically, **it hashes every event client-side with keccak256 before sending anything to the server**. This is the foundation of the trustless model -- the hash exists before any third party sees the data.

### 2. Supabase (Data Storage)

All event data and hashes are stored in Supabase (PostgreSQL). Three tables power the system:
- `analytics_events` -- raw event data plus the client-computed `client_hash`
- `merkle_batches` -- batch metadata, Merkle roots, TX hashes, status
- `event_receipts` -- per-event Merkle proofs for verification

### 3. K-PPE Engine (Merkle Batching)

A TypeScript cron job running on Railway. Every 60 seconds it collects unprocessed events from Supabase, builds a binary Merkle tree from their `client_hash` values (it never re-hashes them), and anchors the root on-chain. Proofs for each event are written back to `event_receipts`.

### 4. KPPEAnchor Smart Contract (On-Chain)

A Solidity contract deployed on Base mainnet at [`0x3B53F7044E47766769156bF210c2661F03Df45dd`](https://basescan.org/address/0x3B53F7044E47766769156bF210c2661F03Df45dd). It stores Merkle roots, emits `KPPEMerkleAnchored` events, and provides `verifyProof` / `verifyProofView` functions anyone can call to verify inclusion. Only opaque 32-byte roots are stored -- no event data ever touches the chain.

### 5. The Dashboard

A real-time web interface deployed on Vercel. Shows behavioral analytics, wallet data, geographic distribution, conversion funnels, and K-PPE proof status -- all filterable by App ID and time range.

---

## Full data flow

```
Your dApp (browser)
       |
       |  kairos.track('swap_attempted', { pair: 'ETH/USDC', amount: 1.5 })
       |
       v
  Kairos SDK (client-side)
       |
       |  1. Canonicalize event: JSON.stringify(event, sortedKeys)
       |  2. Hash: keccak256(canonicalJSON) --> client_hash (0x7a3f...)
       |  3. Send event + client_hash to Supabase
       |  4. Return EventReceipt { eventId, clientHash, timestamp }
       |
       v
  Supabase (analytics_events table)
       |
       |  Stores: eventId, appId, eventType, data, client_hash, sessionId, timestamp
       |  Status: "pending" (waiting for Merkle batching)
       |
       v
  K-PPE Engine (Railway cron, every 60s)
       |
       |  1. SELECT events WHERE batch_id IS NULL (max 256)
       |  2. Read client_hash from each event (NEVER re-hash)
       |  3. Build binary Merkle tree (sorted-pair hashing, padded to power of 2)
       |  4. Anchor root on-chain: KPPEAnchor.anchorMerkleRoot(batchId, root, count, depth, ts)
       |  5. Store Merkle proofs in event_receipts for each event
       |  6. Update merkle_batches with TX hash and status
       |
       v
  Base Mainnet (KPPEAnchor contract)
       |
       |  On-chain record:
       |    - merkleRoot (bytes32)
       |    - eventCount (uint32)
       |    - treeDepth (uint8)
       |    - prevRoot (bytes32) -- chain of trust
       |    - anchoredAt (uint64)
       |
       |  Emits: KPPEMerkleAnchored(protocol, batchId, merkleRoot, eventCount, treeDepth, prevRoot)
       |
       v
  Verification (anyone, anytime)
       |
       |  1. Take your EventReceipt (clientHash + merkleProof + batchId)
       |  2. Call KPPEAnchor.verifyProofView(batchId, eventHash, proof, leafIndex)
       |  3. Returns true/false -- no permission needed, fully trustless
       |
       v
  Kairos Dashboard
       |
       |  Displays behavioral widgets + K-PPE proof status
       |  Batch info, on-chain TX links, Merkle tree visualization
       |  Live verification of any event against its on-chain root
```

---

## Why this is trustless

The key insight is the **order of operations**:

1. The hash is computed **in your user's browser** before any server sees the data
2. The K-PPE Engine uses the `client_hash` directly as the Merkle leaf -- it never re-hashes
3. The Merkle root is anchored on-chain where it is immutable
4. Anyone can recompute the hash from the original data and verify it against the on-chain root

This means:
- **Kairos cannot modify your data** -- changing even one byte would produce a different hash that would not match the on-chain Merkle root
- **Kairos cannot fabricate events** -- a fabricated event would not have a valid client-side hash
- **Anyone can verify** -- the `verifyProofView` function on KPPEAnchor is a public view function requiring zero permissions
- **Data stays private** -- only opaque 32-byte hashes are on-chain; the underlying event data stays in Supabase

---

## On-chain proof -- step by step

Here is exactly what happens when a batch is anchored:

1. The K-PPE Engine collects up to 256 pending events from Supabase
2. It reads the `client_hash` field from each event (a keccak256 hash computed by the SDK)
3. It pads the leaf count to the next power of 2 using zero hashes
4. It builds a binary Merkle tree using sorted-pair hashing: `keccak256(min(a,b) || max(a,b))`
5. It calls `KPPEAnchor.anchorMerkleRoot(batchId, root, eventCount, treeDepth, timestamp)` on Base mainnet
6. The contract stores the root, links it to the previous root (`prevRoot`), and emits `KPPEMerkleAnchored`
7. For each event, the engine computes its Merkle proof (the sibling hashes needed to reconstruct the root) and stores it in `event_receipts`

To verify any event:

1. Take the event's original data
2. Canonicalize it: `JSON.stringify(event, Object.keys(event).sort())`
3. Hash it: `keccak256(canonicalJSON)`
4. Retrieve the Merkle proof from `event_receipts`
5. Call `KPPEAnchor.verifyProofView(batchId, eventHash, proof, leafIndex)` -- returns `true` if the event is in the batch

If the hash does not match, it means the data was modified after collection. This is fraud detection built into the protocol.

---

## Session management

The SDK automatically manages sessions:
- A session ID (random UUID) is created on the first event
- Persisted in `sessionStorage` for the browser session
- Attached to every event automatically
- Expires when the browser tab closes or after 30 minutes of inactivity

You never manage sessions manually. Session DNA (behavioral fingerprints derived from actual user event patterns) is computed from the session's event sequence for advanced analytics.

---

## What data is collected

**Collected automatically:**
- Session ID (random UUID, not tied to identity)
- Timestamp, App ID, event name
- Page path, referrer, device type
- Country and city (via IP geolocation, IP is not stored)

**Collected when you call tracking functions:**
- Event type and custom properties you pass to `track()`
- Swap details (token pair, amount)
- Transaction details (TX hash, amount, success/failure)
- Wallet connection events (address, wallet type)

**Never collected:**
- IP addresses (used only for geo lookup, then discarded immediately)
- Browser fingerprints or cookies
- Email addresses or real names
- Wallet private keys or signatures
- Any PII unless you explicitly pass it in event properties (which you should not)

---

## Privacy model

Kairos Analytics is designed for privacy by default:

| Aspect | How it works |
|---|---|
| **No cookies** | Sessions use `sessionStorage`, not cookies |
| **No fingerprinting** | No canvas, WebGL, or font fingerprinting |
| **No PII storage** | IPs discarded after geo lookup |
| **Client-side hashing** | Event data is hashed before leaving the browser |
| **On-chain opacity** | Only 32-byte Merkle roots are on-chain, not data |
| **User wallet not required** | On-chain proof is handled by the K-PPE Engine, not users |
| **GDPR-friendly** | No cross-site tracking, no third-party data sharing |
