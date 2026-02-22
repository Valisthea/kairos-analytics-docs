# Smart Contracts

The two contracts that form the on-chain backbone of Kairos Analytics.

---

## Network

All contracts are deployed on **Base mainnet** (chain ID: 8453).

Base is an Ethereum L2 built by Coinbase. It offers low transaction costs, fast finality, and is EVM-compatible.

---

## Registry Contract

**Address:** `0xcbf3e71fb2E09929682b8448442d184f8E1E37B8`

**View on Basescan:** [basescan.org/address/0xcbf3e71fb2E09929682b8448442d184f8E1E37B8](https://basescan.org/address/0xcbf3e71fb2E09929682b8448442d184f8E1E37B8)

### Purpose

The Registry maps app IDs to their owner addresses. Before the relayer accepts and stores events for an app, it checks the Registry to confirm the app is registered.

### Key functions

```solidity
// Register a new app
function registerApp(string calldata appId, string calldata name) external

// Check if an app is registered
function isRegistered(string calldata appId) external view returns (bool)

// Get the owner of an app
function getOwner(string calldata appId) external view returns (address)
```

### Who calls it

- **Relayer** — reads `isRegistered()` before processing incoming events
- **Dashboard** — reads `getOwner()` to verify that the connected wallet owns the app it's trying to view
- **App developers** — call `registerApp()` to register a new app

---

## Relayer Contract

**Address:** `0x90991Ae4F708D62a9BB8B09734d3411987d1A6FE`

**View on Basescan:** [basescan.org/address/0x90991Ae4F708D62a9BB8B09734d3411987d1A6FE](https://basescan.org/address/0x90991Ae4F708D62a9BB8B09734d3411987d1A6FE)

### Purpose

The Relayer contract stores batched event hashes on-chain. Each batch is a Merkle root or hash of multiple events, committed in a single transaction for gas efficiency.

### Key functions

```solidity
// Submit a batch of events (called by the relayer service)
function submitBatch(
  string calldata appId,
  bytes32 batchHash,
  uint256 eventCount,
  uint256 fromTimestamp,
  uint256 toTimestamp
) external onlyRelayer

// Verify a batch exists on-chain
function getBatch(uint256 batchId) external view returns (Batch memory)
```

### Who calls it

- **Relayer service** — calls `submitBatch()` on each commit cycle
- **Anyone** — can read `getBatch()` to verify that events were recorded

---

## Registering your app

To start receiving events, your app ID must be registered in the Registry contract.

**Option 1 — Via the Admin Panel**
Go to Admin Panel → Manage App IDs → Register New App. Fill in your App Name, App ID, and network. The Admin Panel calls the Registry contract on your behalf.

**Option 2 — Directly via Basescan**
1. Go to the Registry contract on Basescan
2. Connect your wallet
3. Call `registerApp(appId, name)` from the **Write Contract** tab

**Option 3 — Via code (ethers.js)**
```javascript
import { ethers } from 'ethers'

const provider = new ethers.BrowserProvider(window.ethereum)
const signer = await provider.getSigner()
const registry = new ethers.Contract(REGISTRY_ADDRESS, REGISTRY_ABI, signer)

await registry.registerApp('your-app-id', 'Your App Name')
```
