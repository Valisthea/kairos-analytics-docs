# Verification API

Kairos provides cryptographic verification of analytics data at two levels: public batch integrity (anyone) and private event inclusion (authorized app owners only).

## Two Levels of Verification

### Level 1 — Batch Integrity (Public)

Anyone can verify that a batch was anchored on-chain by reading the KPPEAnchor contract on Basescan. This proves that a specific dataset existed at a specific time and has not been tampered with — without revealing any individual event data.

### Level 2 — Event Inclusion (Authorized)

App owners can verify that a specific event is included in a batch by requesting its Merkle proof via the API. This requires authentication because events contain sensitive user data (geolocation, wallet addresses, navigation behavior).

This separation is intentional: **batch-level integrity is public, event-level data stays private.**

## Verify via API

### Get Batch Info

```
GET /verify/batch?batchId={batchId}
```

Returns the Merkle root, event count, timestamp, and on-chain transaction hash for a specific batch.

```json
{
  "ok": true,
  "batch": {
    "batchId": "b_20260228_001",
    "merkleRoot": "0xabc123...",
    "eventCount": 1024,
    "timestamp": 1709142000,
    "txHash": "0xdef456...",
    "blockNumber": 12345678,
    "appId": "your-app-id"
  }
}
```

### Get Merkle Proof for an Event

```
GET /verify/proof?eventHash={hash}&batchId={batchId}
```

Returns the Merkle proof (array of sibling hashes) needed to verify an event's inclusion in the batch.

```json
{
  "ok": true,
  "proof": {
    "eventHash": "0x789...",
    "siblings": ["0xaaa...", "0xbbb...", "0xccc..."],
    "root": "0xabc123...",
    "index": 42,
    "verified": true
  }
}
```

### Verify On-Chain

The Merkle root can be verified directly on Basescan by reading the KPPEAnchor contract's stored roots:

1. Go to the [KPPEAnchor contract on Basescan](https://basescan.org)
2. Click "Read Contract"
3. Call `getAnchor(batchId)` — returns the stored Merkle root and timestamp
4. Compare with the root from the API response

If they match, the batch is provably anchored on-chain.

## Verify Programmatically

```javascript
import { ethers } from 'ethers';

const provider = new ethers.JsonRpcProvider('https://mainnet.base.org');
const anchor = new ethers.Contract(KPPE_ANCHOR_ADDRESS, KPPE_ABI, provider);

// Get the on-chain root
const onChainRoot = await anchor.getAnchor(batchId);

// Compare with the root from Kairos API
const apiResponse = await fetch(`${RELAYER}/verify/batch?batchId=${batchId}`);
const { batch } = await apiResponse.json();

console.log('Match:', onChainRoot === batch.merkleRoot); // true = verified
```

## What This Proves

**Batch-level (public, on-chain):**
* A specific dataset of N events existed at a specific timestamp
* The dataset has not been modified since it was anchored
* No events have been added to or removed from the batch

**Event-level (authorized, via API):**
* A specific event existed at the time the batch was created
* The event data has not been modified since it was hashed
* The event is provably part of the anchored batch

## Privacy Considerations

Events contain sensitive user data: approximate geolocation (derived from IP), wallet addresses, page URLs visited, click behavior, device information, and timestamps. Publishing Merkle proofs publicly would allow anyone to reconstruct and verify individual user behavior — which is a privacy violation.

By keeping event-level proofs behind app owner authentication, Kairos achieves **tamper-proof integrity without compromising user privacy**. The on-chain Merkle root proves the dataset is intact; the private API lets you verify individual events when needed.

## Future: K-PPC and Zero-Knowledge Proofs

K-PPC (planned for Asterchain) will add zero-knowledge proofs: the ability to verify properties about the dataset ("this dApp had 1,000+ unique wallets this week") without revealing any raw event data. This extends verification beyond app owners to anyone, while keeping all user data private.
