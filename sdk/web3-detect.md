# Web3 Auto-Detection

The Kairos snippet automatically detects and tracks Web3 wallet interactions without any additional configuration.

## Supported Wallets

| Wallet | Detection Method | Events |
|--------|-----------------|--------|
| MetaMask | `window.ethereum.isMetaMask` | ✅ All |
| Coinbase Wallet | `window.ethereum.isCoinbaseWallet` | ✅ All |
| Rabby | `window.ethereum.isRabby` | ✅ All |
| Brave Wallet | `window.ethereum.isBraveWallet` | ✅ All |
| Trust Wallet | `window.ethereum.isTrust` | ✅ All |
| Phantom (EVM) | `window.phantom?.ethereum` | ✅ All |
| Phantom (Solana) | `window.phantom?.solana` | Detection only |

## How Detection Works

On page load, the snippet checks for wallet provider injection into `window.ethereum`. Since some wallets inject after the page loads, the snippet retries detection every 500ms for up to 5 seconds.

When a wallet is detected, a `wallet_detected` event fires:

```json
{
  "type": "wallet_detected",
  "meta": {
    "provider": "metamask",
    "providerType": "extension",
    "multipleProviders": false
  }
}
```

## Tracked Events

### wallet_connected

Fires when `eth_requestAccounts` succeeds (user approves connection):

```json
{
  "type": "wallet_connected",
  "meta": {
    "address": "0x1234...abcd",
    "chainId": "0x2105",
    "chainName": "base",
    "provider": "metamask"
  }
}
```

### wallet_disconnected

Fires when the wallet emits a disconnect event:

```json
{
  "type": "wallet_disconnected",
  "meta": {
    "previousAddress": "0x1234...abcd",
    "provider": "metamask"
  }
}
```

### chain_changed

Fires when the user switches chains:

```json
{
  "type": "chain_changed",
  "meta": {
    "previousChainId": "0x1",
    "newChainId": "0x2105",
    "chainName": "base",
    "provider": "metamask"
  }
}
```

### tx_submitted / tx_confirmed / tx_failed

The snippet intercepts `eth_sendTransaction` calls to track the full transaction lifecycle:

```json
{
  "type": "tx_submitted",
  "meta": {
    "hash": "0xabc...",
    "from": "0x1234...abcd",
    "to": "0x5678...efgh",
    "value": "1000000000000000000",
    "chain": "base",
    "method": "swap"
  }
}
```

After submission, the snippet polls for the transaction receipt to fire `tx_confirmed` (with block number and gas used) or `tx_failed` (with error message).

## Chain ID Reference

| Chain ID | Name | Network |
|----------|------|---------|
| `0x1` | Ethereum | Mainnet |
| `0x2105` | Base | Mainnet |
| `0x14a34` | Base | Sepolia |
| `0x38` | BNB Chain | Mainnet |
| `0x89` | Polygon | Mainnet |
| `0xa4b1` | Arbitrum | One |
| `0xa` | Optimism | Mainnet |

## Disabling Web3 Tracking

To disable wallet auto-detection:

```javascript
window.KAIROS_DISABLE_WEB3 = true;
```

This is useful if your dApp doesn't interact with wallets and you want to avoid unnecessary detection attempts.
