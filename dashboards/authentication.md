# Authentication

Sign in to Kairos Analytics.

---

## Methods

- **Google OAuth** -- One-click sign in
- **GitHub OAuth** -- One-click sign in
- **Email/Password** -- Sign up with email confirmation

## Auth Provider

Supabase Auth handles all authentication. Sessions are stored as JWTs.

## Registration Flow

1. Sign up at kairosanalytics.org/register
2. Complete the 3-step onboarding wizard:
   - Step 1: App ID + dApp type
   - Step 2: Interests
   - Step 3: Plan selection (Free / Builder / Protocol) + Stripe payment if paid
3. Setup modal: dashboard name + snippet copy
4. Redirected to dashboard

## Returning Users

Returning users are automatically detected and redirected to their dashboard.
