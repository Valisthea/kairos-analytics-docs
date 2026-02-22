# How It Works

Understanding the full data flow before you write a single line of code.

---

## The three components

Kairos Analytics is made of three parts that work together:

### 1. The SDK

A lightweight TypeScript library you add to your frontend. It exposes three functions:

- `init()` — initialize the SDK with your app ID
- `page()` — track a page view
- `track()` — track any custom event

The SDK runs entirely in the browser, collects no personally identifiable information, and sends events to the relayer over HTTPS.

### 2. The Relayer

A Node.js service hosted on Railway that acts as the bridge between your frontend and the blockchain. It:

- Receives events from the SDK via HTTP POST
- Batches them for efficiency
- Signs and submits them to Base mainnet
- Optionally stores a queryable cache for the dashboard

### 3. The Smart Contracts

Two contracts deployed on Base mainnet:

- **Registry** — maps app IDs to their owners. Only registered apps can submit events.
- **Relayer contract** — receives batched event hashes from the relayer service and stores them immutably on-chain.

---

## Full data flow

```
Your web app (browser)
        │
        │  init({ appId: 'your-app-id', mode: 'offchain' })
        │  page('/current-path')
        │  track('event_name', 'category', { ...properties })
        │
        ▼
  Kairos Analytics SDK
        │
        │  HTTP POST /track
        │  { appId, event, category, sessionId, timestamp, properties }
        │
        ▼
  Railway Relayer
        │
        │  Validates appId against Registry
        │  Batches events every N seconds
        │  Signs batch with relayer wallet
        │
        ▼
  Base Mainnet
        │
        │  Batch hash stored in Relayer contract
        │  Events permanently verifiable on-chain
        │
        ▼
  Kairos Dashboard
        │
        │  Queries relayer for events by appId
        │  Queries your business API for KPIs
        │
        └──→ App Dashboard (behavioral data)
        └──→ Admin Panel (cross-app overview)
```

---

## Offchain vs Onchain mode

The SDK supports two modes:

| Mode | What it does | When to use |
|------|-------------|-------------|
| `offchain` | Sends events to the relayer via HTTP. Relayer handles blockchain storage. | **Recommended.** Fastest, no user wallet needed. |
| `onchain` | Sends events directly from the user's wallet as signed transactions. | When you want maximum decentralization and user-signed proofs. |

For most applications, start with `offchain`. You can migrate to `onchain` later without changing your tracking code.

---

## Session management

The SDK automatically assigns a session ID to each user visit. A session is:

- Created on `init()`
- Persisted in `sessionStorage` for the duration of the browser session
- Attached to every event automatically

You never need to manage sessions manually.

---

## What data is collected

Kairos Analytics deliberately avoids collecting personally identifiable information.

**Collected automatically:**
- Session ID (random UUID, not tied to identity)
- Timestamp
- App ID
- Event name and category
- Custom properties you explicitly pass

**Never collected:**
- IP addresses
- Browser fingerprints
- Cookies
- Email addresses
- Real names
