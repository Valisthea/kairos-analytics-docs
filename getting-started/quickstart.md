# Quick Start

From zero to your first tracked event in under 5 minutes.

---

## Step 1 — Add the snippet to your page

Paste this block before `</body>` in any HTML page. Replace `your-app-id` with your registered App ID.

```html
<!-- Kairos Analytics -->
<script type="module">
  import { init, page, track } from 'https://esm.sh/@valisthea/analytics'
  ;(async () => {
    try {
      await init({ appId: 'your-app-id', mode: 'offchain' })
      page(window.location.pathname)
      window._k = { track }
    } catch (e) {
      window._k = { track: () => {} }
    }
  })()
</script>
```

That's it. The SDK is now running.

---

## Step 2 — Track your first event

After the snippet, call `_k.track()` anywhere in your page:

```html
<button onclick="_k.track('signup_clicked', 'ui', { location: 'hero' })">
  Sign Up
</button>
```

Or in a separate JS file:

```javascript
document.getElementById('connectBtn').addEventListener('click', () => {
  _k.track('wallet_connect_clicked', 'wallet', {
    page: window.location.pathname
  })
})
```

---

## Step 3 — Verify in the browser console

Open DevTools → Console. You should see:

```
[KairosAnalytics] SDK initialized | appId: your-app-id | session: a3f9b2
[KairosAnalytics] page_view → /
[KairosAnalytics] Event sent: wallet_connect_clicked
```

If you see this, events are flowing to the relayer.

{% hint style="info" %}
If you see `Waku not yet available` — that's expected. In `offchain` mode the SDK sends events via HTTP to the relayer, not through Waku. This message does not mean events are failing.
{% endhint %}

---

## Step 4 — View your data

Open the Kairos Analytics dashboard, connect with your account, and select your app from the list. Events appear in the Live Feed within seconds.

```
https://your-dashboard-url/dashboard-app.html
```

---

## What happens next

- Every `page()` call tracks a page view with the current path
- Every `track()` call sends an event to the relayer
- The relayer batches events and commits them to Base mainnet
- Your dashboard shows live behavioral data

You're done. From here, start adding `_k.track()` calls wherever user actions matter. See [Tracking Events](../sdk/tracking.md) for the full guide.
