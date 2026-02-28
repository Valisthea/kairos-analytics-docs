# Audit and Issues

---

## Critical Issues (HANDOFF February 2026)

1. Client-side hashing not yet implemented in snippet.js (events hashed server-side)
2. Dashboard auth needs hardening (Supabase session verification)
3. In-memory Maps need full Supabase persistence
4. Stripe webhook persistence fix deployed (plans.set + Supabase UPDATE)

## Issues Resolved

- Stripe public key installed in register.html
- Webhook handler now persists plan changes to Supabase
- Price IDs verified (Builder and Protocol)
- Live Feed font weight and time unit display fixed
- Audit security text readability improved
- Dashboard text contrast upgraded
- Kairos snippet installed globally via next/script
- Sidebar: Health, Wallet Rep, Anomalies set to ALPHA (disabled)
- Content overflow on right side of audit-security page fixed

## Remaining Action Plan

**Immediate**: Configure Stripe (sk_live, whsec, Products, Webhook) in Railway + Stripe Dashboard.

**This week**: Test end-to-end payment flow. Deploy register.html + landing.

**Next week**: Client-side hashing in snippet.js, dashboard auth, in-memory persistence.

**Week 3**: Audit and Security integration, dead code cleanup.
