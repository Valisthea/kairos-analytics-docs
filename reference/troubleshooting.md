# Troubleshooting

Solutions to the most common issues.

---

## K-PPE verification issues

### `verify()` returns `anchored: false`

Your event has not been batched yet. The K-PPE Engine runs every 60 seconds, so there may be up to a 60-second delay between tracking an event and it being anchored on-chain.

**Solutions:**
1. Wait 60 seconds and call `verify()` again
2. Check that the K-PPE Engine is running: query the relayer at `/kpe/health`
3. Verify the event exists in Supabase by checking the `analytics_events` table

---

### `verify()` returns `hashMatch: false`

The recomputed hash does not match the stored `client_hash`. This means the event data was modified after the SDK hashed it.

**Possible causes:**
1. The event data was modified in Supabase after insertion
2. The event object you are verifying is different from what was originally tracked (different key order does not matter, but different values do)
3. A bug in the canonicalization -- ensure you are using `JSON.stringify(event, Object.keys(event).sort())`

**How to debug:**
```javascript
import { ethers } from 'ethers'

// Reconstruct the exact event object
const event = {
  appId: 'your-app-id',
  data: { ... },           // same properties you passed to track()
  eventType: 'swap_attempted',
  sessionId: '...',
  timestamp: 1700000000000,
}

const canonical = JSON.stringify(event, Object.keys(event).sort())
const hash = ethers.keccak256(ethers.toUtf8Bytes(canonical))

console.log('Recomputed:', hash)
console.log('Stored:    ', receipt.clientHash)
console.log('Match:     ', hash === receipt.clientHash)
```

---

### `verify()` returns `proofValid: false`

The Merkle proof does not reconstruct the on-chain root. This is a more serious issue than a hash mismatch.

**Possible causes:**
1. The proof data in `event_receipts` is corrupted
2. The event was never actually included in the batch
3. The on-chain root does not match what the K-PPE Engine submitted (would indicate a contract issue)

**How to debug:**
1. Query `event_receipts` for the event and check that `proof`, `leaf_index`, and `root` are populated
2. Call `getBatch(batchId)` on [Basescan](https://basescan.org/address/0x3B53F7044E47766769156bF210c2661F03Df45dd#readContract) and compare the on-chain `merkleRoot` with the `root` in `event_receipts`
3. Call `verifyProofView(batchId, eventHash, proof, leafIndex)` directly on Basescan to see if the proof validates on-chain

---

### Receipt not found

The SDK could not find a receipt for the event.

**Possible causes:**
1. `receiptStorage` is set to `'none'` and you did not capture the receipt from the `track()` return value or `onReceipt` callback
2. `receiptStorage` is set to `'sessionStorage'` and the tab was closed (receipts are cleared)
3. The user cleared their browser storage (`localStorage.clear()`)

**Solutions:**
1. Always capture the receipt from `track()`:
   ```javascript
   const receipt = await kairos.track('event', { ... })
   // Save this receipt somewhere durable
   ```
2. Use the `onReceipt` callback to send receipts to your backend
3. If you have the `eventId`, you can query `event_receipts` in Supabase directly to reconstruct the proof

---

### Event not batched yet

When querying `event_receipts` for a recently tracked event and finding no row, the event has not been processed by the K-PPE Engine yet.

**Timeline:**
- Event tracked: immediate
- K-PPE Engine picks it up: up to 60 seconds
- TX submitted: a few seconds
- TX confirmed: about 4 seconds on Base
- Proof stored: immediately after confirmation

Total: approximately 70 seconds worst case.

**Solutions:**
1. Wait at least 90 seconds after tracking before attempting verification
2. Query the `analytics_events` table and check if `batch_id` is populated (NULL means not yet batched)

---

### `verifyProofView` returns false on Basescan

You are calling the contract's `verifyProofView` directly on Basescan and it returns `false`.

**Checklist:**
1. **Correct batchId** -- is this the batch your event belongs to? Check `event_receipts`
2. **Correct eventHash** -- did you include the `0x` prefix? Is this the exact `client_hash` from the receipt?
3. **Correct proof format** -- the proof must be an array of `bytes32` values. On Basescan, enter them as a JSON array: `["0xabc...", "0xdef...", "0x789..."]`
4. **Correct leafIndex** -- this is the position of your event in the Merkle tree, not the event ID
5. **Sorted-pair hashing** -- the proof was generated with sorted-pair hashing. If you are verifying with custom code, ensure you sort pairs before hashing

---

## SDK issues

### Dashboard shows "MOCK DATA" and does not update

You are not authenticated. Click the **Connect** button in the top right corner of the dashboard and enter your credentials. See [Authentication](../dashboards/authentication.md).

---

### No console logs from the SDK

The SDK is not initializing. Check:

1. Is the npm package installed and imported correctly?
2. If using the script tag, is the `<script>` tag present before `</body>`?
3. Is `appId` set to a non-empty string?
4. Is `supabaseUrl` a valid URL?
5. Is `supabaseAnonKey` a valid Supabase key?
6. Are there any JavaScript errors higher up in the console?

---

### `KairosAnalytics is not defined` error

The SDK script has not loaded yet when your code runs. Either:

- Place your tracking code after the SDK script tag
- Wrap tracking calls in `window.addEventListener('load', ...)` to ensure they run after the SDK loads
- If using npm, make sure the import is at the top of your file and the SDK is initialized before calling `track()`

---

### Events appear in the console but not in the dashboard

The SDK is sending events to Supabase but the dashboard cannot see them. Likely causes:

1. The `appId` in your SDK configuration does not match the App ID selected in the dashboard
2. The Supabase project in your SDK configuration is different from the one the dashboard reads
3. Row Level Security (RLS) policies in Supabase are blocking reads

---

## Relayer issues

### Relayer status indicator is red

The relayer's `/health` endpoint is not responding. Check:

1. **Railway > your service > Deployments** -- is the latest deployment green?
2. **Railway > your service > Logs** -- is the server printing `Running on port ...`?
3. Does the `/health` route exist in your relayer code?
4. Is the `RPC_URL` environment variable correct and reachable?

---

### CORS error when the dashboard fetches the relayer

The relayer is missing CORS headers or is not exposing the K-PPE headers. Ensure your CORS middleware includes:

```javascript
app.use((req, res, next) => {
  res.header('Access-Control-Allow-Origin', '*')
  res.header('Access-Control-Allow-Headers', 'Content-Type, x-api-key')
  res.header('Access-Control-Allow-Methods', 'GET, POST, OPTIONS')
  res.header('Access-Control-Expose-Headers',
    'X-KA-HAP-Status, X-KA-HAP-Layers, X-KA-HAP-Crypto, X-KA-HAP-Anchor, X-KA-HAP-Timestamp')
  if (req.method === 'OPTIONS') return res.sendStatus(200)
  next()
})
```

---

### Events are received but not anchored on-chain

The K-PPE Engine is not submitting batches. Check the Railway logs for errors:

**Common causes:**
- `PRIVATE_KEY` environment variable not set in Railway
- The K-PPE Engine wallet has insufficient ETH on Base mainnet for gas
- The K-PPE Engine wallet is not authorized (`setAuthorized` not called by the contract owner)
- `KPPE_CONTRACT_ADDRESS` is incorrect
- `RPC_URL` is unreachable or rate-limited
- `SUPABASE_SERVICE_ROLE_KEY` is incorrect (engine cannot read events)

---

### Batch ID conflict error

The K-PPE Engine logs `Batch ID conflict -- latestBatchId mismatch`.

This happens when the engine's local batch counter is out of sync with the on-chain `latestBatchId`. Usually caused by a restart or crash during batch submission.

**Solution:** The engine should read `latestBatchId` from the contract on startup and set its counter to `latestBatchId + 1`.

---

## Dashboard issues

### "Admin role required" when logging into the Admin Panel

Your account does not have admin privileges. You need to set `is_admin = true` on your user record in your database. Contact your database administrator or update it directly via your database management tool.

---

### App not appearing in the Admin Panel app list

The app is not registered in the App Registry contract. Register it via the Admin Panel or directly on Basescan at the [Registry contract](https://basescan.org/address/0xcbf3e71fb2E09929682b8448442d184f8E1E37B8#writeContract).

---

### K-PPE proof bar shows "No batches"

No K-PPE batches have been anchored for your app yet. This could mean:

1. No events have been tracked yet (the K-PPE Engine only batches events that exist)
2. The K-PPE Engine is not running (check Railway logs)
3. The engine wallet has no gas (check ETH balance on Base mainnet)

---

### Revenue data not showing in Admin Panel

The Admin Panel's Revenue view requires a connection to your application's own backend API. Make sure:

1. Your API is running and accessible
2. The Admin Panel is configured with your API base URL
3. Your API implements the required endpoints
4. Your authenticated token has permission to call these endpoints

---

## Getting more help

If none of the above resolves your issue:

1. Check the browser console for the full error message
2. Check the Railway logs for server errors
3. Check the Supabase dashboard for failed queries or RLS policy issues
4. Verify the K-PPE Engine status at `/kpe/health`
5. Check the KPPEAnchor contract state on [Basescan](https://basescan.org/address/0x3B53F7044E47766769156bF210c2661F03Df45dd#readContract)
6. Open an issue on the Kairos Analytics GitHub repository with the error message and steps to reproduce
