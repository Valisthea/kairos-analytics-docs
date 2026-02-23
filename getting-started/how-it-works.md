# How It Works

Understanding the full data flow before you write a single line of code.

---

## The four components

Kairos Analytics is made of four parts working together:

### 1. The SDK
A lightweight JavaScript library you add to your frontend via a `<script>` tag or npm. It exposes functions to track page views, custom events, swaps, and transactions. No wallet required.

### 2. The Relayer
A Node.js service deployed on Railway. It receives events from the SDK, batches them, and commits cryptographic hashes to the blockchain. It also exposes REST endpoints for the dashboard to query.

### 3. The Smart Contracts
Two contracts deployed on **Base mainnet** and **BSC Testnet**:
- **Registry** — maps App IDs to their owners. Only registered apps can submit events.
- **Relayer contract** — stores batched event hashes on-chain, permanently and immutably.

### 4. The Dashboard
A real-time web interface showing behavioral analytics, wallet data, geographic distribution, conversion funnels and on-chain proof status — all filterable by App ID and time range.

---

## Full data flow

```
Your dApp (browser)
       │
       │  KairosAnalytics.trackSwap(...)
       │  KairosAnalytics.trackPageView()
       │
       ▼
  Kairos SDK (snippet.js)
       │
       │  HTTP POST /track
       │  { appId, event, sessionId, timestamp, properties }
       │
       ▼
  Railway Relayer
       │
       │  Validates appId against Registry contract
       │  Batches events every N seconds
       │  Computes cryptographic hash of batch
       │  Submits hash to Base mainnet or BSC Testnet
       │
       ▼
  Base Mainnet / BSC Testnet
       │
       │  Batch hash stored immutably on-chain
       │  Anyone can verify data integrity
       │
       ▼
  Kairos Dashboard
       │
       │  Queries relayer for events by appId
       │  Fetches on-chain proof status
       │
       └──→ Behavioral widgets (clicks, sessions, funnels)
       └──→ Web3 widgets (wallets, swaps, transactions)
       └──→ On-chain proof bar (last anchor, TX hash)
```

---

## On-chain proof — how it actually works

> ⚠️ **Important clarification:** On-chain proof does **not** require your users to connect a wallet. It is handled entirely server-side by the relayer.

Here is what happens:

1. The relayer collects N events into a batch
2. It computes a cryptographic hash (SHA-256) of the batch contents
3. It writes that hash to the blockchain in a single transaction
4. The dashboard shows the last anchored block, TX hash, and event count
5. Anyone can click "Verify integrity" to recompute the hash and compare it with what's on-chain

If the hashes match → data is untouched. If they don't → data was modified after collection. No other analytics platform offers this.

---

## Multi-chain support

Kairos supports two networks simultaneously:

| Network | Chain ID | Status | Use case |
|---|---|---|---|
| Base Mainnet | 8453 | ✅ Live | Production apps |
| BSC Testnet | 97 | ✅ Live | Testing, Asterdex integration |

Each App ID is assigned to a chain at registration. The relayer routes proofs to the correct network automatically.

---

## Session management

The SDK automatically manages sessions:
- Created on first event
- Persisted in `sessionStorage` for the browser session
- Attached to every event automatically
- Expires when the browser tab closes

You never manage sessions manually.

---

## What data is collected

**Collected automatically:**
- Session ID (random UUID, not tied to identity)
- Timestamp, App ID, event name
- Page path, referrer, device type
- Country and city (via IP, not stored raw)

**Never collected:**
- IP addresses (used only for geo lookup, then discarded)
- Browser fingerprints or cookies
- Email addresses or real names
- Wallet private keys or signatures
