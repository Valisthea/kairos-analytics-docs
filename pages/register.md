# Register Page

50/50 split layout at kairosanalytics.org/register.

---

## Layout

- **Left panel (50%)**: Authentication form centered vertically. Kairos logo, Log in title, OAuth buttons (Google, GitHub), email/password fields, CTA, legal line. Max-width 420px.
- **Right panel (50%)**: 3 value propositions with colored icon badges. Subtle grid + green glow background. Rotating tagline at bottom.
- **Mobile (< 900px)**: Right panel hidden, form takes full width.

## Value Propositions

1. **Every event proved on-chain** -- Events are hashed, batched into Merkle trees and anchored on Base mainnet.
2. **Real-time analytics for Web3** -- 21 dashboard widgets from wallet connects to conversion funnels.
3. **2 lines of code. Live in 30 seconds.** -- One script tag, 15+ event types auto-tracked.

## Auth Flow

Supabase Auth: Google OAuth, GitHub OAuth, Email/Password signup + signin.

3-step onboarding wizard:
1. App ID + dApp type
2. Interests
3. Plan + Stripe Payment (if paid)

Final setup modal: dashboard name + snippet copy.
