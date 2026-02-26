# Smart Contracts

The on-chain backbone of Kairos Analytics -- trustless Merkle root anchoring and proof verification on Base mainnet.

---

## Contracts Overview

| Contract | Address | Chain | Purpose |
|---|---|---|---|
| **KPPEAnchor** | [`0x3B53F7044E47766769156bF210c2661F03Df45dd`](https://basescan.org/address/0x3B53F7044E47766769156bF210c2661F03Df45dd) | Base mainnet (8453) | Merkle root anchoring and proof verification |
| **App Registry** | [`0xcbf3e71fb2E09929682b8448442d184f8E1E37B8`](https://basescan.org/address/0xcbf3e71fb2E09929682b8448442d184f8E1E37B8) | Base mainnet (8453) | App registration and ownership |

Both contracts are verified on Basescan.

---

## KPPEAnchor

The primary contract of the K-PPE (Kairos Proof Protocol Engine) system. It receives Merkle roots from the K-PPE Engine, stores them immutably, and provides trustless verification functions anyone can call.

**Protocol identifier:** `K-PPE-v1`

### Storage

```solidity
struct MerkleBatch {
    bytes32 merkleRoot;    // Root hash of the event Merkle tree
    uint32  eventCount;    // Number of events in this batch
    uint8   treeDepth;     // Depth of the Merkle tree
    uint64  timestamp;     // Off-chain creation time (unix seconds)
    uint64  anchoredAt;    // block.timestamp when anchored
    bytes32 prevRoot;      // Chain-of-trust link to previous batch
    bool    exists;        // Whether this batch exists
}

mapping(uint256 => MerkleBatch) public batches;
uint256 public latestBatchId;
bytes32 public latestRoot;
uint256 public totalEventsAnchored;
address public owner;
mapping(address => bool) public authorizedAnchors;
```

### Functions

#### `anchorMerkleRoot` -- Anchor a batch (authorized only)

```solidity
function anchorMerkleRoot(
    uint256 batchId,       // Strictly increasing batch identifier
    bytes32 merkleRoot,    // Root hash of the event Merkle tree
    uint32  eventCount,    // Number of events in this batch
    uint8   treeDepth,     // Depth of the Merkle tree
    uint64  timestamp      // Off-chain creation timestamp (unix seconds)
) external onlyAuthorized
```

Called by the K-PPE Engine to anchor a batch. The contract automatically sets `prevRoot` to the current `latestRoot` before updating, creating the chain of trust. Emits `KPPEMerkleAnchored`.

**Validation:**
- `batchId` must be strictly greater than `latestBatchId` (or first batch)
- `merkleRoot` cannot be zero
- `eventCount` must be greater than 0
- Batch ID must not already exist

#### `verifyProof` -- Verify inclusion (emits event)

```solidity
function verifyProof(
    uint256 batchId,              // The batch to verify against
    bytes32 eventHash,            // keccak256 hash of the event
    bytes32[] calldata proof,     // Merkle proof (sibling hashes)
    uint256 leafIndex             // Index of the leaf in the tree
) external returns (bool valid)
```

Verifies that an event hash is included in a batch's Merkle tree. Uses sorted-pair hashing for deterministic verification. Emits `KPPEProofVerified(batchId, eventHash, valid)`.

#### `verifyProofView` -- Verify inclusion (read-only, no gas)

```solidity
function verifyProofView(
    uint256 batchId,
    bytes32 eventHash,
    bytes32[] calldata proof,
    uint256 leafIndex
) external view returns (bool valid)
```

Same logic as `verifyProof` but as a `view` function -- no gas cost, no event emitted. Use this for read-only verification from frontends or scripts.

#### `getBatch` -- Read batch data

```solidity
function getBatch(uint256 batchId) external view returns (MerkleBatch memory)
```

Returns the full `MerkleBatch` struct for a given batch ID. Reverts with `BatchNotFound` if the batch does not exist.

#### `getLatestBatch` -- Read the most recent batch

```solidity
function getLatestBatch() external view returns (
    uint256 batchId,
    bytes32 root,
    uint32  eventCount,
    uint64  anchoredAt
)
```

Returns summary data for the most recently anchored batch.

#### `setAuthorized` -- Manage authorized anchors (owner only)

```solidity
function setAuthorized(address anchor, bool status) external onlyOwner
```

Authorizes or deauthorizes a wallet address to call `anchorMerkleRoot`. Only the contract owner can call this. The K-PPE Engine wallet must be authorized before it can anchor batches.

#### `transferOwnership` -- Transfer contract ownership

```solidity
function transferOwnership(address newOwner) external onlyOwner
```

Transfers ownership to a new address.

### Events

```solidity
event KPPEMerkleAnchored(
    string  protocol,            // "K-PPE-v1"
    uint256 indexed batchId,     // Batch identifier
    bytes32 merkleRoot,          // Merkle root hash
    uint32  eventCount,          // Number of events
    uint8   treeDepth,           // Tree depth
    bytes32 prevRoot             // Previous batch root (chain of trust)
);

event KPPEProofVerified(
    uint256 indexed batchId,     // Batch verified against
    bytes32 eventHash,           // Hash that was verified
    bool    valid                // Verification result
);

event AnchorAuthorized(address indexed anchor, bool authorized);
event OwnershipTransferred(address indexed previousOwner, address indexed newOwner);
```

### Custom Errors

```solidity
error Unauthorized();          // Caller is not authorized to anchor
error OnlyOwner();             // Caller is not the owner
error InvalidBatchId();        // batchId not strictly increasing
error InvalidMerkleRoot();     // merkleRoot is zero
error InvalidEventCount();     // eventCount is zero
error BatchAlreadyExists();    // batchId already used
error InvalidPrevRoot();       // prevRoot mismatch
error BatchNotFound();         // No batch at this ID
```

---

## Sorted-Pair Hashing

KPPEAnchor uses sorted-pair hashing for Merkle tree construction and verification. When combining two sibling nodes, the smaller value is always placed first:

```solidity
if (computedHash <= sibling) {
    computedHash = keccak256(abi.encodePacked(computedHash, sibling));
} else {
    computedHash = keccak256(abi.encodePacked(sibling, computedHash));
}
```

This ensures that verification is deterministic regardless of whether a node is a left or right child. The same approach is used in the K-PPE Engine (TypeScript) and the SDK:

```typescript
function hashPair(left: string, right: string): string {
  const [a, b] = left < right ? [left, right] : [right, left]
  return ethers.keccak256(ethers.concat([a, b]))
}
```

---

## Chain of Trust (prevRoot)

Each batch stores the `prevRoot` -- the Merkle root of the previous batch. This creates a linked chain:

```
Batch #1: root=0xabc..., prevRoot=0x000...  (first batch, prev is zero)
Batch #2: root=0xdef..., prevRoot=0xabc...  (links to batch #1)
Batch #3: root=0x789..., prevRoot=0xdef...  (links to batch #2)
```

This makes it impossible to:
- **Insert a batch** between existing batches (the prevRoot chain would break)
- **Delete a batch** (subsequent batches would reference a non-existent root)
- **Reorder batches** (prevRoot values would not match)

---

## App Registry

The App Registry maps App IDs to their owner addresses. Before the relayer accepts events for an app, it verifies the app is registered here.

**Address:** [`0xcbf3e71fb2E09929682b8448442d184f8E1E37B8`](https://basescan.org/address/0xcbf3e71fb2E09929682b8448442d184f8E1E37B8)

```solidity
event AppRegistered(
    bytes32 indexed appId,
    address indexed owner,
    string  name,
    uint256 registeredAt
);
```

### Registering your app

**Via the dashboard (recommended):**
Register during onboarding at [kairosanalytics.org](https://kairosanalytics.org). The registration wizard handles the contract call automatically.

**Via Basescan (manual):**
1. Go to the [Registry contract on Basescan](https://basescan.org/address/0xcbf3e71fb2E09929682b8448442d184f8E1E37B8#writeContract)
2. Connect your wallet via the Write Contract tab
3. Call `registerApp(appId, name)`

---

## Verifying on Basescan

Anyone can verify that a specific event was included in a batch directly on Basescan:

### Step 1: Find the batch transaction

Go to the [KPPEAnchor contract events on Basescan](https://basescan.org/address/0x3B53F7044E47766769156bF210c2661F03Df45dd#events) and find the `KPPEMerkleAnchored` event for your batch ID.

### Step 2: Read the batch data

Go to the [Read Contract tab](https://basescan.org/address/0x3B53F7044E47766769156bF210c2661F03Df45dd#readContract) and call `getBatch(batchId)` to see the Merkle root, event count, tree depth, and timestamps.

### Step 3: Verify a specific event

Still on the Read Contract tab, call `verifyProofView` with:
- `batchId` -- the batch the event belongs to
- `eventHash` -- the keccak256 hash of your event (from your receipt)
- `proof` -- the array of sibling hashes (from your receipt)
- `leafIndex` -- the position of your event in the tree (from your receipt)

If it returns `true`, your event is provably included in the on-chain Merkle root and has not been modified.

### Step 4: Cross-check the chain of trust

Call `getBatch` for consecutive batch IDs and verify that each batch's `prevRoot` matches the previous batch's `merkleRoot`. An unbroken chain confirms no batches have been inserted, deleted, or reordered.
