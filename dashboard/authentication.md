# Authentication & Onboarding

## Creating an Account

Go to [kairosanalytics.org/register](https://kairosanalytics.org/register.html) to create your account.

### Sign-Up Options

* **Google OAuth** — one-click sign-up with your Google account
* **GitHub OAuth** — sign up with your GitHub account
* **Email & Password** — traditional sign-up with email confirmation

### Onboarding Wizard

After authentication, a 3-step wizard guides you through setup:

**Step 1 — App ID & dApp Type**

Choose a unique App ID for your dApp (3+ characters, lowercase, no spaces). This identifier is used in the snippet and cannot be changed later. Select your dApp type (DeFi, NFT, Gaming, Social, DAO, Infrastructure, or Other).

**Step 2 — Interests**

Select the analytics features most relevant to you. This helps personalize your dashboard experience. You can change these later in Settings.

**Step 3 — Plan Selection**

Choose your plan:

* **Free** ($0) — 5M events/month, 7 widgets. No credit card required.
* **Builder** ($29/mo) — 50M events/month, 14 widgets. Enter payment details via Stripe.
* **Protocol** ($99/mo) — 500M events/month, 21 widgets, API access. Enter payment details via Stripe.

For paid plans, the Stripe Payment Element appears inline. Your card is charged immediately and billed monthly. Cancel anytime from your dashboard settings.

### Setup Modal

After completing the wizard, you'll see your App ID and the snippet code to copy:

```html
<script>window.KAIROS_APP_ID = "your-app-id";</script>
<script src="https://kairosanalytics.org/snippet.js" async></script>
```

Copy this, add it to your dApp, and click "Go to Dashboard."

## Signing In

Returning users can sign in at the same page. If you're already authenticated, you'll be redirected to your dashboard automatically.

## Session Management

Sessions use Supabase Auth with 7-day JWT expiry. You stay signed in across browser sessions unless you explicitly log out.

## Upgrading Your Plan

1. Click "Upgrade" in the dashboard header or on any locked widget
2. Select your new plan and enter payment details
3. Your plan upgrades immediately after payment confirmation

## Canceling Your Subscription

1. Go to Dashboard → Settings → Billing
2. Click "Cancel subscription"
3. Your plan reverts to Free at the end of the current billing period
