# Trustless Verification

How to verify that your analytics events have not been modified -- without trusting anyone.

---

## Why verification matters

Traditional analytics platforms require you to trust the provider. If they modify, delete, or fabricate data, you have no way to detect it.

With K-PPE, every event is hashed client-side and anchored in a Merkle tree on Base mainnet. You can independently verify that:

1. **The data has not been modified** -- recompute the hash and compare with the on-chain root
2. **The event was included in a batch** -- verify the Merkle proof against the on-chain root
3. **The batch was anchored on-chain** -- check the transaction on Basescan
4. **The batch order is intact** -- verify the `prevRoot` chain of trust

No permission, API key, or account is needed. Verification is fully permissionless.

---

## The `verify()` method

The SDK provides a `verify()` method that checks an event's integrity in one call:

```javascript
const receipt = await kairos.track('swap_attempted', {
  fromToken: 'ETH',
  toToken: 'USDC',
  amount: 1.5,
})

// Wait for K-PPE Engine to batch the event (up to 60 seconds)
// Then verify:

const result = await kairos.verify(receipt)
```

### Verification result

```typescript
interface VerificationResult {
  hashMatch: boolean    // Does the recomputed hash match the stored client_hash?
  proofValid: boolean   // Is the Merkle proof valid against the on-chain root?
  anchored: boolean     // Is the batch confirmed on Base mainnet?
  batchId: number       // The batch this event belongs to
  txHash: string        // Base mainnet transaction hash
  merkleRoot: string    // The on-chain Merkle root
  leafIndex: number     // Position in the Merkle tree
  proof: string[]       // The Merkle proof (sibling hashes)
}
```

### What each field means

| Field | Meaning | What it proves |
|---|---|---|
| `hashMatch` | `true` if `keccak256(event)` equals the stored `client_hash` | Data has not been modified since the SDK hashed it |
| `proofValid` | `true` if the Merkle proof reconstructs the on-chain root | The event is included in the batch -- not fabricated |
| `anchored` | `true` if the batch TX is confirmed on Base mainnet | The batch is permanently recorded on-chain |
| `batchId` | Batch number | Which batch contains this event |
| `txHash` | Transaction hash | Verifiable on Basescan |
| `merkleRoot` | Root hash from KPPEAnchor | The immutable on-chain commitment |
| `leafIndex` | Index in the tree | Position used for proof verification |
| `proof` | Sibling hash array | The cryptographic path from leaf to root |

### Interpreting results

```javascript
const result = await kairos.verify(receipt)

if (result.hashMatch && result.proofValid && result.anchored) {
  // Everything checks out. This event is provably unmodified
  // and permanently anchored on Base mainnet.
  console.log('Verified on-chain:', result.txHash)
}

if (!result.hashMatch) {
  // FRAUD DETECTED: The event data was modified after the SDK hashed it.
  // The stored data does not match the original client-side hash.
  console.error('Data integrity violation: hash mismatch')
}

if (!result.proofValid) {
  // The Merkle proof does not reconstruct the on-chain root.
  // Either the proof is corrupted or the event was not in this batch.
  console.error('Merkle proof verification failed')
}

if (!result.anchored) {
  // The batch has not been confirmed on-chain yet.
  // Wait and try again -- the K-PPE Engine runs every 60 seconds.
  console.log('Event not yet anchored, try again in 60s')
}
```

---

## Verify yourself on Basescan

You do not need to trust the SDK's `verify()` method. You can perform every step yourself using only public tools:

### Step 1: Recompute the hash

Take the original event data and recompute the keccak256 hash:

```javascript
import { ethers } from 'ethers'

const event = {
  appId: 'your-app-id',
  data: { fromToken: 'ETH', toToken: 'USDC', amount: 1.5 },
  eventType: 'swap_attempted',
  sessionId: 'a3f9b2c1-...',
  timestamp: 1700000000000,
}

const canonical = JSON.stringify(event, Object.keys(event).sort())
const hash = ethers.keccak256(ethers.toUtf8Bytes(canonical))

// Compare with receipt.clientHash -- they must match
console.log(hash === receipt.clientHash) // true
```

### Step 2: Get the Merkle proof

Query the `event_receipts` table in Supabase for your event ID:

```javascript
const { data } = await supabase
  .from('event_receipts')
  .select('*')
  .eq('event_id', receipt.eventId)
  .single()

const proof = data.proof       // Array of sibling hashes
const leafIndex = data.leaf_index
const batchId = data.batch_id
```

### Step 3: Verify on Basescan

1. Go to the [KPPEAnchor Read Contract tab on Basescan](https://basescan.org/address/0x3B53F7044E47766769156bF210c2661F03Df45dd#readContract)
2. Find `verifyProofView`
3. Enter:
   - `batchId`: the batch number from Step 2
   - `eventHash`: the hash from Step 1 (must include `0x` prefix)
   - `proof`: the array of sibling hashes from Step 2
   - `leafIndex`: the leaf index from Step 2
4. Click **Query**
5. If it returns `true`, your event is provably included in the on-chain Merkle root

### Step 4: Verify the batch data

Still on the Read Contract tab, call `getBatch(batchId)` to see:
- `merkleRoot` -- the committed root
- `eventCount` -- number of events in the batch
- `treeDepth` -- depth of the Merkle tree
- `anchoredAt` -- when it was anchored (block timestamp)
- `prevRoot` -- link to the previous batch (chain of trust)

### Step 5: Verify the chain of trust

Call `getBatch` for consecutive batch IDs and check that each `prevRoot` matches the previous batch's `merkleRoot`. An unbroken chain means no batches were inserted, deleted, or reordered.

---

## Fraud detection

The K-PPE system is designed to make data tampering detectable. Here are the scenarios:

### Hash mismatch (data was modified)

```
Original event: { type: 'swap', amount: 1.5 }
Client hash:    0x7a3f... (computed in browser)

Tampered event: { type: 'swap', amount: 15.0 }  <-- modified!
Recomputed:     0xb2c8... (different hash)

0x7a3f... != 0xb2c8... --> DATA WAS MODIFIED
```

If someone modifies the event data in Supabase after collection, the recomputed hash will not match the `client_hash`. The Merkle proof will also fail because the leaf hash is different.

### Proof verification failure (event was fabricated)

If someone inserts a fake event into Supabase and tries to claim it was in a batch, the `verifyProofView` call on-chain will return `false` because the event's hash was never a leaf in the Merkle tree.

### Chain of trust break (batches were reordered)

If someone tries to delete or reorder batches, the `prevRoot` chain will break. Batch N+1's `prevRoot` will not match Batch N's `merkleRoot`, revealing the tampering.

---

## Verifying with ethers.js directly

For maximum trustlessness, you can verify against the contract using your own RPC connection:

```javascript
import { ethers } from 'ethers'

const provider = new ethers.JsonRpcProvider('https://mainnet.base.org')
const contract = new ethers.Contract(
  '0x3B53F7044E47766769156bF210c2661F03Df45dd',
  [
    'function verifyProofView(uint256 batchId, bytes32 eventHash, bytes32[] calldata proof, uint256 leafIndex) external view returns (bool valid)',
    'function getBatch(uint256 batchId) external view returns (tuple(bytes32 merkleRoot, uint32 eventCount, uint8 treeDepth, uint64 timestamp, uint64 anchoredAt, bytes32 prevRoot, bool exists))',
  ],
  provider
)

// Verify an event
const valid = await contract.verifyProofView(
  42,                    // batchId
  '0x7a3f...',          // eventHash (your client_hash)
  ['0xabc...', '0xdef...', '0x789...'],  // proof array
  0                      // leafIndex
)

console.log('Valid:', valid) // true or false
```

This uses only a public RPC endpoint and the contract's view function. No API keys, no Kairos servers, no trust required.

---

## Timing considerations

The K-PPE Engine runs on a 60-second cron interval. After you call `track()`, the event goes through this pipeline:

| Step | Timing | Status |
|---|---|---|
| `track()` called | Immediate | Event hashed and stored in Supabase |
| K-PPE Engine picks up event | Up to 60 seconds | Event included in Merkle tree |
| TX submitted to Base mainnet | A few seconds | Batch pending on-chain |
| TX confirmed (2 blocks) | ~4 seconds on Base | Batch confirmed |
| Proof stored in `event_receipts` | Immediately after confirmation | Receipt available |

If you call `verify()` before the event has been batched, `anchored` will be `false`. Wait and try again.

```javascript
const result = await kairos.verify(receipt)

if (!result.anchored) {
  // Event not yet batched -- wait 60 seconds and retry
  setTimeout(async () => {
    const retryResult = await kairos.verify(receipt)
    console.log('Anchored:', retryResult.anchored)
  }, 60000)
}
```
