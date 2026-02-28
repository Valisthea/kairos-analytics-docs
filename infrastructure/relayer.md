# Relayer

The Node.js server that bridges the SDK, Supabase, and the blockchain.

---

## Deployment

- Runtime: Node.js
- Host: Railway
- URL: https://kairos-relayer-production.up.railway.app

## Environment Variables

| Variable | Description |
|----------|-------------|
| SUPABASE_URL | Supabase project URL |
| SUPABASE_SERVICE_KEY | Supabase service role key |
| ADMIN_EMAILS | Comma-separated admin email whitelist |
| STRIPE_SECRET_KEY | Stripe secret key (sk_live_...) |
| STRIPE_WEBHOOK_SECRET | Stripe webhook signing secret (whsec_...) |
| KPPE_MODE | On-chain mode: kppe (default), legacy, disabled |

## Key Endpoints

| Method | Path | Auth | Description |
|--------|------|------|-------------|
| POST | /track | Public | Ingest events from snippet |
| GET | /events | User | Fetch events for appId |
| GET | /stats | User | Aggregated statistics |
| GET | /daily-stats | User | Daily time series |
| GET | /sessions | User | Session list |
| GET | /batches | User | Merkle batch list |
| GET | /flow-tree | User | User flow graph |
| GET | /journeys | User | Session journeys |
| GET | /kpe/health | User | K-PPE engine status |
| GET | /audit-data | User | Audit data |
| POST | /create-payment-intent | Public | Stripe subscription creation |
| POST | /stripe/webhook | Public | Stripe webhook handler |
| GET | /verify | Public | Verify Merkle proof |
