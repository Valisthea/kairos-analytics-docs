# Troubleshooting

Solutions to the most common issues.

---

## SDK issues

### Dashboard shows "MOCK DATA" and doesn't update

You are not authenticated. Click the **Connect** button in the top right corner of the dashboard and enter your credentials. See [Authentication](../dashboards/authentication.md).

---

### No console logs from the SDK

The SDK is not initializing. Check:

1. Is the `<script type="module">` tag present before `</body>`?
2. Is `appId` set to a non-empty string?
3. Are there any JavaScript errors higher up in the console?
4. Is `type="module"` included on the script tag? (Required for ES module imports)

---

### `_k is not defined` error

The `_k.track()` call is running before the SDK snippet has initialized. Either:

- Your script runs before the SDK snippet on the page — move the SDK snippet earlier, or move your script later
- Wrap your tracking calls in `document.addEventListener('DOMContentLoaded', ...)` to ensure they run after the snippet

---

### `Waku not yet available, events queued locally`

This is **not an error**. It is expected behavior in `offchain` mode. The SDK uses HTTP to send events to the relayer, bypassing Waku entirely. Events are being sent correctly.

---

### Events appear in the console but not in the dashboard

The relayer is receiving events but the dashboard cannot query them. Likely cause: the `/events` endpoint is not yet implemented on the relayer. See [Relayer — Adding missing endpoints](../infrastructure/relayer.md).

---

## Relayer issues

### Relayer status indicator is red (✕)

The relayer's `/health` endpoint is not responding. Check:

1. **Railway → your service → Deployments** — is the latest deployment green?
2. **Railway → your service → Logs** — is the server printing `Running on port ...`?
3. Does the `/health` route exist in your relayer code? If not, add it — see [Relayer](../infrastructure/relayer.md).

---

### `CORS error` when the dashboard fetches the relayer

The relayer is missing CORS headers. Add this to your Express app:

```javascript
app.use((req, res, next) => {
  res.header('Access-Control-Allow-Origin', '*')
  res.header('Access-Control-Allow-Headers', 'Content-Type')
  res.header('Access-Control-Allow-Methods', 'GET, POST, OPTIONS')
  if (req.method === 'OPTIONS') return res.sendStatus(200)
  next()
})
```

---

### Events are received but not committed on-chain

Check the relayer logs for errors during batch submission. Common causes:

- `PRIVATE_KEY` environment variable not set in Railway
- Wallet has insufficient ETH on Base mainnet to pay for gas
- `REGISTRY_ADDRESS` or `RELAYER_ADDRESS` is incorrect
- `RPC_URL` is unreachable or rate-limited

---

## Dashboard issues

### "Admin role required" when logging into the Admin Panel

Your account is valid but doesn't have admin privileges. You need to set `is_admin = true` on your user record in your application's database. Contact your database administrator or do it directly via your database management tool.

---

### App not appearing in the Admin Panel app list

The app is not registered in the Registry contract. Go to Admin Panel → Manage App IDs → Register New App, or register it directly via the Registry contract on Basescan.

---

### Revenue data not showing in Admin Panel

The Admin Panel's Revenue view requires a connection to your application's own backend API. Make sure:

1. Your API is running and accessible
2. The Admin Panel is configured with your API base URL
3. Your API implements the required endpoints (`admin-dashboard`, `admin-transactions`)
4. Your authenticated token has permission to call these endpoints

---

## Getting more help

If none of the above resolves your issue:

1. Check the browser console for the full error message
2. Check the Railway logs for the full server error
3. Open an issue on the Kairos Analytics GitHub repository with the error message and steps to reproduce
