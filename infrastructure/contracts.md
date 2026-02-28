# Smart Contracts

Deployed and verified on Base mainnet (chain ID 8453).

---

## Contracts

| Contract | Address | Purpose |
|----------|---------|---------|
| **KPPEAnchor** | [0x3B53F7044E47766769156bF210c2661F03Df45dd](https://basescan.org/address/0x3B53F7044E47766769156bF210c2661F03Df45dd) | Merkle root anchoring and proof verification |
| **App Registry** | [0xcbf3e71fb2E09929682b8448442d184f8E1E37B8](https://basescan.org/address/0xcbf3e71fb2E09929682b8448442d184f8E1E37B8) | App registration and ownership |

## KPPEAnchor

- anchorBatch(bytes32 root, uint256 eventCount, string appId) -- Anchors a Merkle root on-chain
- verifyProof(bytes32 root, bytes32 leaf, bytes32[] proof) -- Verifies a Merkle inclusion proof
- Each batch stores: Merkle root, event count, timestamp, appId

## App Registry

- registerApp(string appId) -- Registers an appId on-chain
- Ownership tracked per wallet address
