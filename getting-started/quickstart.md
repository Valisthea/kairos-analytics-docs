# Quick Start

From zero to your first tracked event in under 3 minutes.

---

## Step 1 — Create your account

Go to [kairosanalytics.org](https://kairosanalytics.org) → click **Start Free** → sign up with Google, GitHub or email.

No credit card required for the Free plan.

---

## Step 2 — Choose your App ID

During registration, you'll pick an App ID — a unique identifier for your dApp:

```
my-dex         ✅ valid
nft-market     ✅ valid
My App         ❌ no spaces
my_app!        ❌ no special chars
```

Rules: lowercase letters, numbers and hyphens only. 3–32 characters.

Your App ID appears in your snippet and links your frontend events to your dashboard.

---

## Step 3 — Add the snippet

Paste this before `</body>` on every page of your dApp. Replace `your-app-id` with your registered App ID.

```html
<!-- Kairos Analytics -->
<script>
  (function() {
    window.KAIROS_APP_ID = "your-app-id";
    var s = document.createElement('script');
    s.src = 'https://kairos-dashboard-seven.vercel.app/snippet.js';
    s.async = true;
    document.head.appendChild(s);
  })();
</script>
```

That's it. Page views and clicks are now tracked automatically.

---

## Step 4 — Track your first custom event

After the snippet loads, call tracking functions anywhere in your JS:

```javascript
// Track a swap attempt
KairosAnalytics.trackSwap('ETH', 'USDC', 1.5);

// Track a transaction confirmation
KairosAnalytics.trackTransaction('0xabc...', 500, true);

// Track any custom event
KairosAnalytics.trackEvent('button_clicked', { label: 'connect_wallet' });
```

---

## Step 5 — Verify in the browser console

Open DevTools → Console. You should see:

```
[Kairos] SDK initialized | appId: your-app-id
[Kairos] page_view → /
[Kairos] Event queued: swap_attempted
[Kairos] Batch sent → relayer ✓
```

---

## Step 6 — Open your dashboard

Go to your dashboard and select your App ID from the navbar. Events appear in the Live Feed within seconds.

```
https://kairos-dashboard-seven.vercel.app/dashboard-app.html
```

---

## What's tracked automatically

Once the snippet is on your page, these fire without any extra code:

| Event | Trigger |
|---|---|
| `page_view` | Every page load |
| `click` | Every button/link click |
| `session_start` | New browser session |
| `session_end` | Tab close or 30min inactivity |

Everything else — swaps, transactions, wallet connections — you add with one-line function calls.
