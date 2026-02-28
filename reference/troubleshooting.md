# Troubleshooting

---

## Events not appearing

1. Check that KAIROS_APP_ID is set correctly in your HTML
2. Verify the snippet is loaded (check Network tab for snippet.js)
3. Check browser console for errors
4. Ensure the relayer is running: https://kairos-relayer-production.up.railway.app/health

## Authentication issues

1. Clear browser cookies and try again
2. Check Supabase project status
3. For OAuth: ensure redirect URLs are configured in Supabase

## Stripe payment not working

1. Verify pk_live_ key is correct in register.html
2. Check Railway variables for STRIPE_SECRET_KEY and STRIPE_WEBHOOK_SECRET
3. Verify webhook endpoint in Stripe Dashboard
4. Check relayer logs on Railway for webhook errors

## Proof verification failing

1. Ensure the batch has been confirmed (status = confirmed or kppe-anchored)
2. Verify the event hash matches the original event data
3. Check that the KPPEAnchor contract is accessible on Basescan

## Dashboard shows no data

1. Verify your App ID matches what is in the snippet
2. Check that events are being sent (POST /track in Network tab)
3. Wait 30 seconds for events to propagate
4. Try refreshing with Ctrl+Shift+R

## Relayer returning 401

1. Your Supabase JWT may have expired. Sign out and sign in again.
2. Verify the appId belongs to your account
3. For admin endpoints: check that your email is in ADMIN_EMAILS
