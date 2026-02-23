# Authentication

How to sign in to Kairos Analytics.

---

## Sign up

Go to [kairosanalytics.org](https://kairosanalytics.org) and click **Start Free**, **Start Builder** or **Start Protocol** depending on the plan you want.

You'll be taken to the registration wizard with three sign-in options:

### Google
Click **Continue with Google** → authorize the Kairos app in your Google account → you're in.

### GitHub
Click **Continue with GitHub** → authorize the Kairos OAuth app → you're in.

### Email / Password
Enter your email and a password → click **Create account** → check your email for the confirmation link → click it → you're in.

---

## Plans and access

| Plan | Price | Access |
|---|---|---|
| **Free Beta** | $0/month | Core widgets, 5M events/month, 30 days history |
| **Builder** | $29/month | Full behavioral + Web3 widgets, 50M events, 12 months history |
| **Protocol** | $99/month | Everything + API access, unlimited dApps, team seats |

Free plan users see locked widgets for Builder and Protocol features. Click any locked widget to see the upgrade prompt.

---

## After signing in — onboarding

On your first login, a mandatory setup screen appears:

1. **Choose your App ID** — unique identifier for your dApp (`my-dex`, `nft-market`, etc.)
2. **Copy your snippet** — the script tag to paste in your dApp's HTML
3. **Confirm** — click "Open my dashboard" once you've copied the snippet

This screen can't be skipped. Once completed, it won't appear again.

---

## Signing out

Click your account avatar in the top-right navbar → **Sign out**.

Your session ends immediately. All localStorage keys (`ka_appId`, `ka_onboarding_done`) are cleared.

---

## Switching apps

If you have multiple dApps registered, use the App ID selector in the navbar to switch between them. Each App ID has its own event stream and dashboard state.

---

## Password reset

On the login screen → **Forgot password** → enter your email → click the reset link in your inbox → set a new password.
