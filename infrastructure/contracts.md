# Smart Contracts

Kairos uses two smart contracts deployed and verified on Base mainnet.

## KPPEAnchor

The KPPEAnchor contract stores Merkle roots representing batches of analytics events.

### Purpose

Every batch of tracked events is organized into a Merkle tree. The root of that tree — a single 32-byte hash representing all events in the batch — is stored permanently on-chain via this contract.

### Key Functions

**`anchor(bytes32 root, uint256 eventCount, string appId)`**

Stores a Merkle root on-chain. Called by the relayer when a batch is ready. Only callable by the authorized relayer address.

Parameters:
* `root` — The Merkle root hash of the event batch
* `eventCount` — Number of events in the batch
* `appId` — The app ID this batch belongs to

**`getAnchor(uint256 batchId) → (bytes32 root, uint256 eventCount, uint256 timestamp, string appId)`**

Read function. Returns the stored data for a given batch ID. Anyone can call this to verify a proof.

**`totalEventsAnchored() → uint256`**

Returns the total number of events anchored across all batches and all apps.

### Verification

The contract is **verified on Basescan**, meaning:
* The Solidity source code is publicly visible
* Anyone can read the contract's stored data
* The bytecode matches the source (no hidden logic)

To verify a batch:
1. Go to the contract on Basescan
2. Click "Read Contract"
3. Call `getAnchor(batchId)` with the batch ID
4. Compare the returned Merkle root with the one from the Kairos API

## Registry

The Registry contract manages on-chain registration of app IDs.

### Purpose

When a developer registers a new dApp on Kairos, the app ID can optionally be registered on-chain. This creates an immutable record linking the app ID to the owner's wallet address.

### Key Functions

**`registerApp(string appId)`**

Registers an app ID to the caller's wallet address. Each app ID can only be registered once.

**`getOwner(string appId) → address`**

Returns the wallet address that registered a given app ID.

**`verifyAppOwnership(string appId, address owner) → bool`**

Returns true if the given address is the registered owner of the app ID.

## Network Details

| Property | Value |
|----------|-------|
| Network | Base Mainnet |
| Chain ID | 8453 |
| RPC | https://mainnet.base.org |
| Explorer | https://basescan.org |
| Gas token | ETH (bridged) |

Both contracts are deployed and verified. Gas costs for anchoring are minimal on Base (typically < $0.01 per batch due to Base's L2 fee structure).

## Multi-Chain (Future)

Builder plans currently anchor on Base mainnet only. Protocol plans include future support for additional chains. The relayer's `OnChainSender` module is designed to support multiple chain targets.

Asterchain support is planned for when the chain is operational.
