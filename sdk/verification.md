# Proof Verification

Verify that analytics events have not been modified -- without trusting anyone.

---

## How Verification Works

1. Each event produces a deterministic keccak256 hash
2. Events are batched into Merkle trees
3. The Merkle root is anchored on Base mainnet
4. Anyone can verify an event's inclusion using its Merkle proof

## Verify via API

    GET /verify?appId=your-app-id&eventHash=0x...&batchId=123

Returns: proof (Merkle siblings), root, and on-chain verification status.

## Verify on Basescan

1. Go to the KPPEAnchor contract on Basescan
2. Call verifyProof(root, leaf, proof)
3. Returns true if the event is included in the anchored batch

Contract: [0x3B53F7044E47766769156bF210c2661F03Df45dd](https://basescan.org/address/0x3B53F7044E47766769156bF210c2661F03Df45dd)
