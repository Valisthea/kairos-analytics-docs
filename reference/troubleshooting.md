# Troubleshooting

Common issues and how to resolve them.

## Snippet Issues

### Events not appearing in the dashboard

1. **Check your App ID** — Open DevTools Console and verify `window.KAIROS_APP_ID` matches the ID you registered. It's case-sensitive.

2. **Check network requests** — In DevTools → Network tab, filter by `track`. You should see POST requests. If you see 401 errors, your app ID may not be registered.

3. **Enable debug mode** — Add `window.KAIROS_DEBUG = true` before the snippet. Events will be logged to the console as they fire.

4. **Check for ad blockers** — Some ad blockers may block requests to analytics endpoints. If you see blocked requests in the Network tab, ask users to whitelist your domain or consider proxying through your own backend.

5. **Verify the snippet URL** — Make sure you're loading from `https://kairosanalytics.org/snippet.js` (not `http://`, and not a cached/local copy).

### Wallet events not firing

The snippet detects wallets by checking `window.ethereum` on page load. Some wallets inject after the DOM is ready. The snippet retries for 5 seconds, but if your wallet injects later, try:

```javascript
// Force re-detection after wallet injects
window.kairos?.detectWallets();
```

### Duplicate events

If you see duplicate `page_view` events, check that you haven't loaded the snippet twice (e.g., once in `<head>` and once in a component). The snippet includes deduplication for rapid-fire events, but loading it twice will generate true duplicates.

## Dashboard Issues

### Dashboard shows "No data" even though snippet is installed

The dashboard polls for data on load. If you just installed the snippet, wait 30–60 seconds and refresh. New events appear in the Live Feed widget first.

If you've waited and still see no data:
1. Check that the App ID in your snippet matches the one shown in Settings
2. Verify events are being sent (DevTools → Network tab)
3. Try loading the dashboard in a different browser to rule out cache issues

### Plan badge shows wrong plan

The dashboard fetches your plan from the server on each load. If you see an incorrect plan:
1. Clear your browser's localStorage (`localStorage.removeItem('ka_plan')`)
2. Refresh the dashboard — it will fetch the correct plan from the server

### Widgets are locked despite having a paid plan

After upgrading, it may take a few seconds for the webhook to process and update your plan. Refresh the dashboard. If widgets are still locked after a minute, check Settings → Billing to confirm your subscription is active.

## Authentication Issues

### "Check your email" after signup but no email received

1. Check your spam/junk folder
2. If using Gmail, check the Promotions tab
3. Try signing up with Google or GitHub OAuth instead (no email confirmation required)
4. Contact support if the issue persists

### OAuth redirects to a blank page

This can happen if cookies are blocked for third-party domains. Ensure that your browser allows cookies from `supabase.co`. Some privacy-focused browsers (Brave, Firefox Strict) may block the OAuth redirect.

### Can't sign in after clearing browser data

Clearing localStorage removes your session. Simply sign in again at [kairosanalytics.org/register](https://kairosanalytics.org/register.html).

## Billing Issues

### Payment failed

If your card is declined:
1. Verify the card details and try again
2. Check with your bank — international transactions or recurring charges may be flagged
3. Try a different payment method

### Can't find my invoice

Go to Dashboard → Settings → Billing. Invoices are listed under your subscription details. Stripe also sends email receipts to the email associated with your account.

## Relayer Issues

### Relayer status shows "Offline" in the dashboard

The relayer indicator checks connectivity every 60 seconds. A temporary offline status may indicate:
* A Railway deployment in progress (usually < 30 seconds)
* A network issue between your browser and Railway

Events are queued in the snippet and resent when the connection is restored. No data is lost during brief outages.

## Still Stuck?

If none of the above resolves your issue:

1. Check the [System Status](https://kairosanalytics.org/status.html) page for known outages
2. Open an issue on GitHub with your App ID, browser, and steps to reproduce
3. Email support at contact@kairosanalytics.org
