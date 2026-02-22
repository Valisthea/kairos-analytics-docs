# Prerequisites

Everything you need before integrating the SDK.

---

## What you need

### Required

| Requirement | Description |
|-------------|-------------|
| **App ID** | A unique identifier for your application (e.g. `my-dapp`). You choose this when registering. |
| **Registered app** | Your app ID must be registered in the Registry contract on Base mainnet. Contact the Kairos Analytics team or use the admin panel to register. |
| **A web frontend** | Any HTML page, React app, Next.js app, or static site. The SDK works anywhere JavaScript runs in the browser. |

### Optional (for dashboard access)

| Requirement | Description |
|-------------|-------------|
| **Dashboard account** | An account on the Kairos Analytics dashboard to view your data. |
| **Admin access** | Required only if you need to manage multiple apps or view cross-app data. |

---

## No backend required

The SDK works entirely on the frontend. You do not need to:

- Set up a server
- Configure webhooks
- Store API keys in your backend
- Handle authentication for tracking

All you need is your **App ID** and the snippet.

---

## Supported environments

| Environment | Supported |
|-------------|-----------|
| Static HTML | ✅ |
| React / Next.js | ✅ |
| Vue / Nuxt | ✅ |
| Svelte / SvelteKit | ✅ |
| Any framework with a `<script>` tag | ✅ |
| Node.js (server-side) | ❌ Browser only |
| React Native / Mobile | ❌ Not yet |

---

## Getting your App ID

Your App ID is a short, lowercase string you choose when registering your application with Kairos Analytics. It should:

- Be unique across all registered apps
- Be descriptive of your application
- Use only lowercase letters, numbers, and hyphens
- Be between 3 and 32 characters

**Examples of valid App IDs:**
```
my-dapp
trading-platform-v2
nft-marketplace
defi-dashboard
```

Once chosen, your App ID cannot be changed. Choose carefully.

---

## Infrastructure checklist (for self-hosters)

If you are running your own Kairos Analytics infrastructure:

- [ ] Relayer service deployed and reachable
- [ ] `PRIVATE_KEY` environment variable set on the relayer
- [ ] Registry contract address configured
- [ ] Relayer contract address configured
- [ ] RPC URL pointing to Base mainnet
- [ ] App ID registered in the Registry contract
- [ ] CORS headers enabled on the relayer
