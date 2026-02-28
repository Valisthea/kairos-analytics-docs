# Roadmap

Kairos Analytics is actively developed. Here's what's live, what we're building, and where we're headed.

## 🟢 Live

These features are in production and available today.

* **Snippet v2.0** — 2-line integration, 15+ auto-tracked events, Web3 wallet auto-detection
* **Dashboard** — 21 widgets across 3 plan tiers with live and demo modes
* **K-PPE Proof Anchoring** — Merkle tree batching and anchoring on Base mainnet
* **Supabase Auth** — Google, GitHub, and email/password authentication
* **3-Step Onboarding** — App ID registration, interest selection, plan and payment
* **Stripe Billing** — Subscription management with Payment Element integration
* **Audit & Security** — 8-tab unified audit dashboard with dApp Health Score
* **Admin Cockpit** — App management, billing overview, server logs
* **Self-Tracking** — Kairos tracks itself across 7 pages (transparency)
* **Smart Contracts** — KPPEAnchor and Registry verified on Base mainnet

## 🟡 Building Now

Features currently in development.

* **Client-side hashing** — Adding keccak256 hashing directly in the snippet so event hashes originate in the user's browser (currently done server-side as fallback)
* **Dashboard auth hardening** — Supabase session verification on all dashboard routes
* **Data persistence** — Migrating in-memory stores to Supabase tables for durability across relayer restarts
* **Session DNA** — Behavioral fingerprinting for bot detection without cookies
* **dApp Health Score improvements** — Adding financial impact quantification and cross-audit linking
* **SEO and discoverability** — Meta tags, sitemap, structured data, Google Search Console

## 🔵 2026 Vision

Planned features for 2026.

* **Asterchain Integration** — Proof anchoring on Asterchain as the default proof chain
* **Aster Native Plan** — $49/month with permanent rate lock for early adopters
* **K-PPC (Zero-Knowledge Proofs)** — Verify analytics properties without revealing raw data
* **Wallet Reputation Scoring** — Anti-Sybil analysis with wallet behavior patterns
* **Predictive Analytics** — DAU forecasting and churn risk detection
* **Session Replay** — Visual replay of user sessions (Q2 2026)
* **Cross-dApp Journey Mapping** — Track user journeys across multiple dApps (opt-in)
* **Public Proof Dashboard** — Anyone can verify proofs for any appId without an account
* **Mobile SDK** — Native iOS and Android tracking
* **Self-Hosted Option** — Run the full Kairos stack on your own infrastructure

## Feature Requests

We build based on user feedback. If there's a feature you'd like to see, reach out via GitHub issues or email us at contact@kairosanalytics.org.
