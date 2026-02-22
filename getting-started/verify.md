# Verify Your Setup

Use this checklist to confirm each part of your integration is working correctly.

---

## 1. SDK is initializing

Open DevTools → Console on any page where you've added the snippet. You should see:

```
[KairosAnalytics] SDK initialized | appId: your-app-id | session: ...
```

**If you don't see this:**
- Check that the `<script type="module">` tag is present before `</body>`
- Check that `appId` is not empty or undefined
- Check for any JavaScript errors above this line in the console

---

## 2. Events are being sent

After calling `_k.track()`, you should see in the console:

```
[KairosAnalytics] Event sent: your_event_name
```

**If events are not sending:**
- Make sure `init()` completed before calling `track()` — it's async, so use `await`
- Make sure `window._k` is defined before your other scripts run
- Check the Network tab in DevTools for POST requests to the relayer URL

---

## 3. Relayer is reachable

Test the relayer health endpoint directly:

```bash
curl https://your-relayer-url/health
```

Expected response:
```json
{ "status": "ok", "version": "1.0" }
```

**If this returns 404 or fails:**
- The `/health` endpoint may not be implemented yet — see [Relayer](../infrastructure/relayer.md)
- The relayer service may be down — check your Railway deployment logs

---

## 4. Events reach the relayer

Check your relayer logs (Railway → your service → Logs tab). You should see lines like:

```
Event received: page_view from your-app-id
Event received: button_clicked from your-app-id
Batch of 5 events submitted to Base
```

**If no events appear in the logs:**
- The relayer URL in the SDK config may be wrong
- There may be a CORS issue — see [Troubleshooting](../reference/troubleshooting.md)

---

## 5. Dashboard shows data

Go to your dashboard, connect your account, and open your app. You should see:

- Events appearing in the Live Event Feed
- Page view count increasing
- Session count updating

**If the dashboard shows "MOCK DATA":**
- You are not authenticated — click Connect and enter your credentials
- See [Authentication](../dashboards/authentication.md)

---

## Full checklist

| Check | How to verify | Status |
|-------|--------------|--------|
| SDK initializes | Console log on page load | — |
| Events sent | Console log after `_k.track()` | — |
| Relayer reachable | `GET /health` returns 200 | — |
| Events in relayer logs | Railway Logs tab | — |
| Dashboard shows live data | Authenticated + Live badge visible | — |
| On-chain proof | Search tx hash on basescan.org | — |
