# System Architecture

## Overview



---

## Components

### Snippet (Browser)
- File: public/snippet.js (v2.1)
- Automatic capture of 15+ Web2 and Web3 event types
- Auto-detects MetaMask, Coinbase Wallet, Rabby, Brave Wallet, Trust Wallet, Phantom
- Sends events via POST /track

### Relayer (Server)
- Runtime: Node.js on Railway
- URL: https://kairos-relayer-production.up.railway.app
- 54 endpoints, 3 auth levels: public, user (Supabase JWT), admin (JWT + ADMIN_EMAILS)
- Hashes events, manages Merkle batches, anchors on-chain

### Database (Supabase)
- PostgreSQL with Row Level Security
- Tables: apps, events, sessions, daily_stats, wallet_registry, session_journeys, merkle_batches, api_keys
- Auth: Google OAuth, GitHub OAuth, Email/Password

### Blockchain (Base Mainnet)
- Chain ID: 8453
- KPPEAnchor: Merkle root anchoring and proof verification
- App Registry: appId registration and ownership
- Both contracts verified on Basescan

### Frontend (Next.js)
- Next.js 14 App Router with (dashboard) and (admin) route groups
- React 18 client components with inline CSSProperties
- KI Icon System: 70+ custom SVG icons
- Fonts: Instrument Sans (body) + JetBrains Mono (mono/data)

---

## Auth Levels

| Level | Verification | Endpoints |
|-------|-------------|-----------|
| **Public** | None | /health, /verify, /track |
| **User** | Supabase JWT + appId ownership | /events, /stats, /daily-stats, /sessions, etc. |
| **Admin** | Supabase JWT + ADMIN_EMAILS whitelist | /admin/* endpoints |
