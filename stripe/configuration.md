# Stripe Configuration

Manual setup steps (not in code).

---

## Steps

1. **Stripe Dashboard > Products**: Create Builder (29 USD/mo recurring) and Protocol (99 USD/mo recurring). Note the Price IDs.

2. **Stripe Dashboard > Developers > API keys**: Copy sk_live_...

3. **Railway > Variables**: Add STRIPE_SECRET_KEY = sk_live_...

4. **Stripe Dashboard > Developers > Webhooks > Add endpoint**:
   - URL: https://kairos-relayer-production.up.railway.app/stripe/webhook
   - Events: invoice.payment_succeeded, customer.subscription.deleted, checkout.session.completed

5. **Railway > Variables**: Add STRIPE_WEBHOOK_SECRET = whsec_...

## Public Key

Already in code (register.html):

    pk_live_51T41ttQ1p2FdLWK601GnV8EkjL6jIqth0fWQonNExoJkhxHe3MqkAQ1KoVwvgae6PRNqV3PYUZfsKWAOkW0Cd25x00Svxqjxse

## Price IDs

| Plan | Price ID |
|------|----------|
| Builder (29 USD/mo) | price_1T420PLBUqyjloiYAQIAS52V |
| Protocol (99 USD/mo) | price_1T420sLBUqyjloiYLNRXkLL4 |
