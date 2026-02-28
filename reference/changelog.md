# Changelog

Production deployment history for Kairos Analytics.

## February 2026

### v6.0 — Documentation & Stripe (Feb 28, 2026)

* **Stripe Integration**: Payment Element, subscriptions, webhooks connected. Plans activate after payment confirmation via webhook.
* **Register Page Redesign**: Netlify-style 50/50 split layout with value propositions panel.
* **Landing Page Refactor**: 12 sections, all factual claims verified. Removed references to Waku P2P (dead code), wallet-gated dashboard (not implemented), permanent on-chain storage (legacy model).
* **SEO Foundation**: robots.txt, sitemap.xml, meta descriptions, Open Graph tags, Twitter Cards, JSON-LD structured data, canonical URLs added across all pages.
* **Documentation v6**: Complete GitBook documentation rewrite covering all features, APIs, and architecture.

### v5.0 — Audit & Admin (Feb 2026)

* **Audit & Security Dashboard**: Unified 8-tab section (Overview, Conversion, Integrity, Reliability, Growth, UX, K-PPE, Sentinel) with composite dApp Health Score.
* **Admin Cockpit**: Full admin interface with app management, billing overview, server logs, and global event feed.
* **Plan Gating**: Widget locking by plan tier (7 Free / 14 Builder / 21 Protocol) with upgrade prompts.

### v4.0 — Authentication Overhaul (Feb 2026)

* **Supabase Auth**: Migrated from custom JWT to Supabase Auth with Google, GitHub, and email/password support.
* **3-Step Onboarding Wizard**: Replaced simple registration with guided wizard (App ID → Interests → Plan).
* **OAuth Popup Flow**: Google and GitHub OAuth via popup window instead of page redirect.

## January 2026

### v3.0 — Dashboard & K-PPE (Jan 2026)

* **21 Dashboard Widgets**: Full widget set including Wallet Analytics, TX Analytics, Funnel, Heatmap, Alerts, and Audit.
* **K-PPE Engine**: Production Merkle tree batching and anchoring on Base mainnet.
* **Demo Mode**: Pre-loaded sample data for exploring all widgets without integration.
* **Smart Contracts**: KPPEAnchor and Registry deployed and verified on Base mainnet.

### v2.0 — Snippet & Web3 (Jan 2026)

* **Snippet v2.0**: Universal snippet with 15+ auto-tracked events and Web3 wallet auto-detection.
* **Supported Wallets**: MetaMask, Coinbase Wallet, Rabby, Brave Wallet, Trust Wallet, Phantom.
* **Transaction Lifecycle**: Full tracking from tx_submitted through tx_confirmed/tx_failed.
* **data-track Attribute**: HTML attribute for zero-JS custom event tracking.

## December 2025

### v1.0 — Initial Launch (Dec 2025)

* **Core Platform**: Relayer, basic dashboard, event tracking.
* **Base Mainnet**: Initial contract deployment.
* **npm SDK**: `@kairoslab/analytics-sdk` published.
* **Basic Auth**: JWT-based authentication with email/password.
