# Architecture Overview

Kairos Analytics consists of four main components: the client-side snippet, the relayer server, the Supabase database, and the on-chain smart contracts.

## System Diagram

```
┌──────────────────────┐
│  Your dApp (Browser)  │
│  + snippet.js         │
└──────────┬───────────┘
           │ HTTPS POST /track
           ▼
┌──────────────────────┐     ┌──────────────────────┐
│  Relayer              │────▶│  Supabase             │
│  Node.js / TypeScript │     │  PostgreSQL + Auth    │
│  Hosted on Railway    │     │  Tables: apps,        │
│  54 API endpoints     │     │  merkle_batches,      │
│  3 auth levels        │     │  daily_stats,         │
└──────────┬───────────┘     │  profiles             │
           │ anchor()        └──────────────────────┘
           ▼
┌──────────────────────┐     ┌──────────────────────┐
│  Base Mainnet         │     │  Stripe               │
│  KPPEAnchor contract  │     │  Subscriptions +      │
│  Registry contract    │     │  Webhooks             │
│  Verified on Basescan │     │  Payment processing   │
└──────────────────────┘     └──────────────────────┘
```

## Components

### Snippet (Client)

A lightweight JavaScript file served from `kairosanalytics.org/snippet.js`. Runs in the user's browser, captures events, and sends them to the relayer. No dependencies, no build step, loads asynchronously.

### Relayer (Backend)

A Node.js/TypeScript server hosted on Railway. Responsibilities:

* Receives events via POST /track
* Hashes events with keccak256
* Manages Merkle tree batching
* Anchors Merkle roots on Base mainnet
* Serves the dashboard API (stats, widgets, admin)
* Handles Stripe webhooks for billing
* Manages authentication via Supabase JWT verification

The relayer exposes 54 endpoints grouped by auth level. See [Relayer API](relayer.md) for the full reference.

### Supabase (Database + Auth)

PostgreSQL database managed by Supabase. Stores:

| Table | Purpose |
|-------|---------|
| `apps` | Registered dApps: app_id, owner email, plan, events_limit, dapp_type |
| `profiles` | User profiles linked to Supabase Auth |
| `merkle_batches` | Anchored proof batches: merkle_root, event_count, tx_hash, block_number |
| `daily_stats` | Aggregated daily metrics per app |

Supabase Auth handles user authentication (Google OAuth, GitHub OAuth, email/password) with JWT tokens.

### Smart Contracts (On-Chain)

Two contracts deployed and verified on Base mainnet:

* **KPPEAnchor** — Stores Merkle roots with `anchor(root, eventCount, appId)`. Immutable once written.
* **Registry** — Registers app IDs on-chain with owner verification via `registerApp(appId)`.

Both contracts are verified on Basescan, meaning anyone can read the source code and verify the data.

## Authentication Model

The relayer uses a 3-level authentication system:

| Level | Verification | Example Endpoints |
|-------|-------------|-------------------|
| **Public** | None | `/health`, `/verify/*`, `/status` |
| **User** | Supabase JWT + appId ownership | `/track`, `/stats`, `/plan`, `/billing` |
| **Admin** | Supabase JWT + ADMIN_EMAILS whitelist | `/admin/*`, `/update-plan`, `/cockpit/*` |

## Data Retention

| Plan | Retention |
|------|-----------|
| Free | 30 days |
| Builder | 12 months |
| Protocol | Unlimited |

On-chain Merkle roots are permanent (blockchain is immutable). Dashboard data follows the retention policy above.
