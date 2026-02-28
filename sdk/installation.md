# Installation

Add Kairos Analytics with a script tag or npm.

---

## Script Tag (recommended)

Add before closing body tag:

    <script>window.KAIROS_APP_ID = "your-app-id";</script>
    <script src="https://kairosanalytics.org/snippet.js" async></script>

## npm Package

    npm install kairos-analytics

    import { KairosAnalytics } from "kairos-analytics"
    const kairos = new KairosAnalytics({ appId: "your-app-id" })

## Configuration

The only required configuration is your App ID. Get it from the registration wizard at kairosanalytics.org/register.

The snippet automatically tracks 15+ event types and auto-detects Web3 wallets.
