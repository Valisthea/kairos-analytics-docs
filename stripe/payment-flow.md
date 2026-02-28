# Payment Flow

How Stripe integrates into the registration wizard.

---

## In register.html

Payment is part of Step 3 (plan selection) of the onboarding wizard:

- **Free plan**: No payment required. Goes directly to setup modal.
- **Builder/Protocol**: Stripe Payment Element appears. User enters card. confirmPayment() is called. On success, setup modal opens.
- **3D Secure**: If required, redirects to Stripe then back to register.html?payment=success.

## Payment Element Appearance

Themed to match Kairos UI:
- Dark theme (bg: #141418)
- Green accent colors (#10b981)
- JetBrains Mono font
- 3px border-radius

## Webhook Events

| Event | Action |
|-------|--------|
| invoice.payment_succeeded | Activate plan, persist to Supabase |
| customer.subscription.deleted | Downgrade to free, persist to Supabase |
| checkout.session.completed | Activate plan (legacy), persist to Supabase |

## Persistence Fix

Webhooks update both the in-memory Map (plans.set()) AND Supabase (UPDATE apps SET plan = ... WHERE app_id = ...). This prevents plan loss on relayer restart.
