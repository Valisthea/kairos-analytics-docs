# Architecture

A complete technical overview of all Kairos Analytics components.

---

## Components

### SDK (`@valisthea/analytics`)

Published on npm. Imported via CDN (esm.sh) or installed locally. Runs entirely in the browser.

**Responsibilities:**
- Session management (generates and persists session IDs)
- Event batching before sending
- HTTP transport to the relayer
- Optional Waku P2P transport (future)
- Fallback handling when the relayer is unreachable

### Relayer (Railway)

A Node.js/Express service deployed on Railway. Acts as the trusted intermediary between browsers and the blockchain.

**Responsibilities:**
- Receive events from SDK clients
- Validate app IDs against the Registry contract
- Batch events for gas efficiency
- Sign and submit batches to the Relayer contract on Base mainnet
- Optionally maintain a queryable event cache for the dashboard

### Registry Contract (Base mainnet)

`0xcbf3e71fb2E09929682b8448442d184f8E1E37B8`

Stores the mapping of app IDs to their owner addresses. Before the relayer accepts events for an app, it verifies the app is registered here.

### Relayer Contract (Base mainnet)

`0x90991Ae4F708D62a9BB8B09734d3411987d1A6FE`

Receives batched event hashes from the relayer service and stores them immutably on Base mainnet. Anyone can verify that a given event was recorded at a given time by checking this contract.

### Dashboard (Vercel)

A static HTML/JS frontend deployed on Vercel. Pulls data from:
- The relayer service (behavioral event data)
- Your application's own API (business data, optional)

---

## Data flow diagram

```
┌─────────────────────────────────────────────────────┐
│                   User's Browser                    │
│                                                     │
│  Your web app + Kairos Analytics SDK                │
│  ─────────────────────────────────                  │
│  init({ appId, mode: 'offchain' })                  │
│  page('/path')                                      │
│  track('event_name', 'category', { ...props })      │
└─────────────────────────┬───────────────────────────┘
                          │ HTTPS POST /track
                          ▼
┌─────────────────────────────────────────────────────┐
│                Railway Relayer                      │
│                                                     │
│  1. Receive event payload                           │
│  2. Validate appId in Registry contract             │
│  3. Store event in local cache                      │
│  4. Every N seconds: batch pending events           │
│  5. Sign batch with relayer wallet                  │
│  6. Submit to Relayer contract on Base              │
└─────────────┬───────────────────────┬───────────────┘
              │                       │
              │ On-chain TX           │ Query API
              ▼                       ▼
┌────────────────────┐  ┌────────────────────────────┐
│   Base Mainnet     │  │   Kairos Dashboard         │
│                    │  │                            │
│  Registry contract │  │  GET /events?appId=X       │
│  Relayer contract  │  │  GET /sessions?appId=X     │
│                    │  │  GET /health               │
│  Immutable record  │  │                            │
│  of all events     │  │  + Your app's API          │
└────────────────────┘  │  (revenue, users, KPIs)    │
                        └────────────────────────────┘
```

---

## Network topology

| Component | Hosted on | Access |
|-----------|-----------|--------|
| SDK | CDN (esm.sh) or npm | Browser import |
| Relayer | Railway | Public HTTPS |
| Registry contract | Base mainnet | Public read, owner write |
| Relayer contract | Base mainnet | Public read, relayer write |
| Dashboard | Vercel | Public HTTPS |

---

## Security model

**App registration:** Only the relayer can register events. Apps must be pre-registered in the Registry contract, preventing unauthorized data submission.

**Event integrity:** Batches are signed by the relayer's private key before submission. The signature can be verified on-chain.

**Immutability:** Once committed to Base mainnet, event hashes cannot be modified or deleted.

**No PII:** The SDK never collects IP addresses, cookies, or identifiable user data. Only what you explicitly pass in `properties` is recorded.
