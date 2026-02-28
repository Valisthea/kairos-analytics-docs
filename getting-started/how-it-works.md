# How It Works

Kairos Analytics follows a linear 4-step flow from browser to blockchain.

```
Browser (snippet.js) --> Relayer (Node.js) --> Merkle Tree --> Base Mainnet (KPPEAnchor)
```

---

## Step 1: Event Capture (Browser)

The snippet (snippet.js) runs in the user s browser and captures 15+ event types:

**Web2 events:** page_view, click, scroll_depth, form_submit, outbound_click, tab_hidden, performance, error

**Web3 events:** wallet_detected, wallet_connected, wallet_disconnected, chain_changed, tx_submitted, tx_confirmed, tx_failed, swap_executed

Auto-detects: MetaMask, Coinbase Wallet, Rabby, Brave Wallet, Trust Wallet, Phantom.

Events are sent to the relayer via POST /track.

## Step 2: Hashing and Accumulation (Relayer)

The relayer hashes each event with keccak256 and accumulates them per appId.

## Step 3: Merkle Tree Building

When a batch is ready (size threshold or timer), the relayer builds a binary Merkle tree and computes the root.

## Step 4: On-chain Anchoring

The Merkle root is anchored on Base mainnet via KPPEAnchor.anchor(). The receipt is stored in Supabase.

---

## Verification

Every step is verifiable:

1. An event hash can be recomputed from its raw data
2. Its inclusion in the Merkle tree can be proven with a Merkle proof
3. The anchoring can be verified on [Basescan](https://basescan.org/address/0x3B53F7044E47766769156bF210c2661F03Df45dd)

---

## Innovations

6 innovations that require the full pipeline from browser to blockchain:

| Innovation | Description |
|-----------|-------------|
| **Session DNA** | Behavioral fingerprinting. Bot detection without cookies. |
| **K-PPE Merkle Proofs** | Trustless event verification against on-chain Merkle roots. |
| **dApp Health Score** | Composite score: conversion, integrity, reliability, growth, UX, engagement, retention, volume. |
| **Wallet Reputation** | Anti-Sybil scoring for bots, farms, and Sybil attacks. |
| **Anomaly Radar** | Real-time statistical anomaly detection. |
| **Predictive Analytics** | DAU prediction and churn risk detection. |
