# Stripe Architecture

The Stripe integration uses the Subscriptions + Payment Element model.

---

## Flow

1. User selects a paid plan in the registration wizard
2. Frontend calls POST /create-payment-intent on the relayer
3. Relayer creates a Stripe Customer + Subscription
4. Returns a clientSecret
5. Payment Element renders in the browser
6. User pays (supports 3D Secure)
7. Stripe sends a webhook to the relayer
8. Relayer activates the plan and persists to Supabase

## Three Keys Required

| Key | Location | Purpose |
|-----|----------|---------|
| pk_live_ (public) | register.html frontend | Renders Payment Element. Not sensitive. |
| sk_live_ (secret) | Railway env STRIPE_SECRET_KEY | Creates subscriptions, manages customers. NEVER in code. |
| whsec_ (webhook) | Railway env STRIPE_WEBHOOK_SECRET | Verifies webhook authenticity. NEVER in code. |
