# Authentication

How to sign in to Kairos Analytics (Kairos Lab).

---

## Sign up

Go to [kairosanalytics.org](https://kairosanalytics.org) and click **Start Free**, **Start Builder**, or **Start Protocol** depending on the plan you want.

You will be taken to the registration wizard with three sign-in options:

### Google
Click **Continue with Google** and authorize the Kairos app in your Google account.

### GitHub
Click **Continue with GitHub** and authorize the Kairos OAuth app.

### Email / Password
Enter your email and a password, click **Create account**, check your email for the confirmation link, and click it.

---

## Plans and access

| Plan | Price | Access |
|---|---|---|
| **Free Beta** | $0/month | Core widgets, 5M events/month, 30 days history |
| **Builder** | $29/month | Full behavioral + Web3 widgets, 50M events, 12 months history |
| **Protocol** | $99/month | Everything + API access, unlimited dApps, team seats |

Free plan users see locked widgets for Builder and Protocol features. Click any locked widget to see the upgrade prompt.

All plans include K-PPE trustless verification. On-chain Merkle proof is not a premium feature -- every event is anchored regardless of plan.

---

## After signing in -- onboarding

On your first login, a mandatory setup screen appears:

1. **Choose your App ID** -- unique identifier for your dApp (`my-dex`, `nft-market`, etc.)
2. **Configure your SDK** -- copy the `appId`, `supabaseUrl`, and `supabaseAnonKey` for your SDK configuration
3. **Confirm** -- click "Open my dashboard" once you have configured the SDK

This screen cannot be skipped. Once completed, it will not appear again.

---

## Signing out

Click your account avatar in the top-right navbar and select **Sign out**.

Your session ends immediately. All localStorage keys (`ka_appId`, `ka_onboarding_done`) are cleared.

---

## Switching apps

If you have multiple dApps registered, use the App ID selector in the navbar to switch between them. Each App ID has its own event stream, K-PPE batch history, and dashboard state.

---

## Password reset

On the login screen, click **Forgot password**, enter your email, click the reset link in your inbox, and set a new password.

---

## API authentication

For programmatic access (Protocol plan), the relayer accepts an API key via the `x-api-key` header:

```javascript
fetch('https://kairos-relayer-production.up.railway.app/events?appId=your-app-id', {
  headers: {
    'x-api-key': 'your-api-key',
  },
})
```

API keys are generated from the dashboard under Account Settings. Keep your API key secret -- do not expose it in client-side code.
