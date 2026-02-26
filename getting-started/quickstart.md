# Quick Start

From zero to your first tracked and cryptographically anchored event in under 3 minutes. No wallet connection from your users is required -- K-PPE handles all on-chain anchoring automatically.

---

## What you will get

After following this guide, your dApp will:
- Track every user interaction (page views, clicks, wallet events, custom events)
- Hash each event client-side with keccak256 -- the hash is born in the user's browser
- Anchor Merkle roots on Base mainnet via K-PPE (only 32 bytes on-chain; your data stays **private**)
- Give you verifiable receipts for every event

---

## Step 1 -- Create your account

Go to [kairosanalytics.org](https://kairosanalytics.org) and click **Start Free**. Sign up with Google, GitHub, or email.

No credit card required for the Free Beta plan (5M events/month, 30-day retention).

---

## Step 2 -- Choose your App ID

During registration, you will pick an App ID -- a unique identifier for your dApp:

```
my-dex         valid
nft-market     valid
kairos-lab     valid
My App         invalid (no spaces)
my_app!        invalid (no special chars)
```

Rules: lowercase letters, numbers, and hyphens only. 3 to 32 characters.

Your App ID links your frontend events to your dashboard and appears in your SDK configuration.

---

## Step 3 -- Install the SDK

The SDK is published as `@kairoslab/analytics-sdk` on npm. Pick the option that matches your setup.

### Option A: npm (recommended for React, Next.js, Vue, Vite, Webpack)

```bash
npm install @kairoslab/analytics-sdk
```

Then in your app entry point (e.g., `main.ts`, `App.tsx`, `app.js`):

```javascript
import { KairosAnalytics } from '@kairoslab/analytics-sdk'

const kairos = new KairosAnalytics({
  appId: 'your-app-id',
  supabaseUrl: 'https://your-project.supabase.co',
  supabaseAnonKey: 'your-supabase-anon-key',
})

// Track a page view
kairos.track('page_view', { path: window.location.pathname })
```

The SDK uses ethers v6 internally for keccak256 hashing. It is bundled with the package -- you do not need to install ethers separately.

### Option B: Script tag (zero build step, works on any HTML page)

Copy and paste this snippet before `</body>` on any HTML page. No build tools, no bundler, no npm required:

```html
<!-- Kairos Analytics -->
<script src="https://cdn.jsdelivr.net/npm/@kairoslab/analytics-sdk/dist/browser.min.js"></script>
<script>
  const kairos = new window.KairosAnalytics({
    appId: 'your-app-id',
    supabaseUrl: 'https://your-project.supabase.co',
    supabaseAnonKey: 'your-supabase-anon-key',
  });
  kairos.track('page_view', { path: window.location.pathname });
</script>
```

When loaded via script tag, the SDK is available as `window.KairosAnalytics`.

### Option C: yarn or pnpm

```bash
yarn add @kairoslab/analytics-sdk
# or
pnpm add @kairoslab/analytics-sdk
```

Then use the same `import` syntax as Option A.

---

## Step 4 -- Track your first custom event

After initialization, call `track()` anywhere in your code:

```javascript
// Track a swap attempt
kairos.track('swap_attempted', {
  fromToken: 'ETH',
  toToken: 'USDC',
  amount: 1.5,
})

// Track a wallet connection
kairos.track('wallet_connected', {
  address: '0xabc...',
  walletType: 'MetaMask',
})

// Track any custom event
kairos.track('button_clicked', { label: 'connect_wallet' })
```

Every `track()` call returns an `EventReceipt` containing the event's `clientHash` (keccak256). Keep this receipt -- you will use it to verify your event later.

```javascript
const receipt = await kairos.track('trade_opened', {
  pair: 'BTC/USDT',
  amount: 1000,
})

console.log(receipt.clientHash) // 0x7a3f...
console.log(receipt.eventId)    // evt-abc123
```

---

## Step 5 -- Verify in the browser console

Open DevTools and check the Console. You should see:

```
[Kairos] SDK initialized | appId: your-app-id
[Kairos] Event hashed client-side: 0x7a3f... (page_view)
[Kairos] Event stored in Supabase | eventId: evt-abc123
[Kairos] Receipt saved | clientHash: 0x7a3f...
```

---

## Step 6 -- Verify your event on-chain

Once the K-PPE Engine processes your event (within 60 seconds), you can verify it:

```javascript
const result = await kairos.verify(receipt)

console.log(result.hashMatch)  // true -- your client hash matches
console.log(result.proofValid) // true -- Merkle proof is valid
console.log(result.anchored)   // true -- root is on Base mainnet
console.log(result.batchId)    // 42
console.log(result.txHash)     // 0xdef... (Basescan link)
```

Or verify manually on Basescan:

1. Go to the [KPPEAnchor contract on Basescan](https://basescan.org/address/0x3B53F7044E47766769156bF210c2661F03Df45dd#readContract)
2. Call `verifyProofView` with your `batchId`, `eventHash`, `proof`, and `leafIndex`
3. Returns `true` if your event is in the batch

See [Trustless Verification](../sdk/verification.md) for full details.

---

## Step 7 -- Open your dashboard

Navigate to your dashboard and select your App ID from the navbar. Events appear in the Live Feed within seconds, and K-PPE batch status shows at the top.

```
https://kairos-dashboard-seven.vercel.app/dashboard-app.html?appId=your-app-id
```

---

## What is tracked automatically

Once the SDK is initialized, these events fire without any extra code:

| Event | Trigger |
|---|---|
| `page_view` | Every page load |
| `click` | Every button/link click |
| `session_start` | New browser session |
| `session_end` | Tab close or 30 min inactivity |

Everything else -- swaps, transactions, wallet connections, custom events -- you add with one-line `track()` calls.

---

## What happens behind the scenes

```
Your track() call
      |
      v
SDK hashes event client-side (keccak256)
  → The hash is born in the user's browser, never modified by any server
      |
      v
Event + client_hash sent to Supabase
      |
      v
Relayer's local Merkle tree builder (every 60s):
  - Reads client_hash values directly (never re-hashes)
  - Builds sorted-pair Merkle tree (src/kpe/merkle.ts)
  - Calls KPPEAnchor.anchorBatch() on Base mainnet
  - Only the 32-byte Merkle root goes on-chain (data stays PRIVATE)
  - Stores proofs in event_receipts
      |
      v
Your event is now verifiable on-chain forever
```

No wallet connection from your users is required. The relayer handles all on-chain anchoring using its own authorized wallet. No external K-PPE service is needed -- the Merkle tree builder runs locally inside the relayer.
