# Data Pipeline

Detailed flow from snippet capture to on-chain proof.

---

## Snippet (Browser Layer)

Automatic capture of:
- **Web2 events:** page_view, click, scroll_depth, form_submit, outbound_click, tab_hidden, performance, error
- **Web3 events:** wallet_detected, wallet_connected, wallet_disconnected, chain_changed, tx_submitted, tx_confirmed, tx_failed, swap_executed

Auto-detects wallets: MetaMask, Coinbase Wallet, Rabby, Brave Wallet, Trust Wallet, Phantom.

## Relayer (Server Layer)

54 endpoints grouped into 3 auth levels:
- **Public:** health, verify, track
- **User:** Supabase JWT + appId ownership
- **Admin:** Supabase JWT + ADMIN_EMAILS whitelist

The relayer hashes events, manages Merkle batches, anchors on-chain, and serves the dashboard API.

## On-chain (Blockchain Layer)

- KPPEAnchor: Merkle root anchoring and proof verification
- App Registry: appId registration and ownership
- Deployed and verified on Base mainnet (chain ID 8453)
- Each batch contains: Merkle root, event count, timestamp, appId
