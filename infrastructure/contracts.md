# Smart Contracts

The contracts that form the on-chain backbone of Kairos Analytics.

---

## Networks

Kairos Analytics is deployed on two networks:

| Network | Chain ID | Use case |
|---|---|---|
| **Base Mainnet** | 8453 | Production apps |
| **BSC Testnet** | 97 | Testing, early integrations |

---

## Base Mainnet Contracts

### Registry Contract
**Address:** `0xcbf3e71fb2E09929682b8448442d184f8E1E37B8`
[View on Basescan ↗](https://basescan.org/address/0xcbf3e71fb2E09929682b8448442d184f8E1E37B8)

### Relayer Contract
**Address:** `0x90991Ae4F708D62a9BB8B09734d3411987d1A6FE`
[View on Basescan ↗](https://basescan.org/address/0x90991Ae4F708D62a9BB8B09734d3411987d1A6FE)

---

## BSC Testnet Contracts

### Registry Contract
**Address:** `0x6C7986a3d79E1AFECfE242f92f2A0DFeC3397133`
[View on BscScan Testnet ↗](https://testnet.bscscan.com/address/0x6C7986a3d79E1AFECfE242f92f2A0DFeC3397133)

---

## Contract roles

### Registry
Maps App IDs to their owners. The relayer checks the Registry before accepting events for any app.

```solidity
// Register a new app
function registerApp(string calldata appId, string calldata name) external

// Check if an app is registered
function isRegistered(string calldata appId) external view returns (bool)

// Get the owner of an app
function getOwner(string calldata appId) external view returns (address)
```

### Relayer Contract
Stores batched event hashes on-chain. Each batch is a hash of N events, committed in a single transaction.

```solidity
// Submit a batch (called by the relayer service only)
function submitBatch(
  string calldata appId,
  bytes32 batchHash,
  uint256 eventCount,
  uint256 fromTimestamp,
  uint256 toTimestamp
) external onlyRelayer

// Read a batch — callable by anyone
function getBatch(uint256 batchId) external view returns (Batch memory)
```

---

## Registering your app

Your App ID must be registered in the Registry before the relayer accepts your events.

**Via the dashboard (recommended)**
Register during onboarding at [kairosanalytics.org](https://kairosanalytics.org). The registration wizard handles the contract call automatically.

**Via Basescan (manual)**
1. Go to the Registry contract on Basescan
2. Connect your wallet → Write Contract tab
3. Call `registerApp(appId, name)`

---

## Verifying data integrity

Anyone can verify that a batch of events has not been modified:

1. Note the TX hash shown in the dashboard's on-chain proof bar
2. Go to Basescan or BscScan Testnet
3. Find the transaction → read the `batchHash` in the input data
4. Compare with the hash recomputed from the raw events

Or simply click **"Verify integrity"** in the dashboard — it does this automatically.

> ℹ️ On-chain proof is entirely server-side. Your users do not need to connect a wallet for verification to work.
