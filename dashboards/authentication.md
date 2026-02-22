# Authentication

How to connect your account and activate live data in both dashboards.

---

## App Dashboard

### Step 1 — Open the dashboard

Navigate to your App Dashboard URL. In the top right corner, you'll see a **Connect** button.

### Step 2 — Enter your credentials

A login modal will appear with two options:

**Access Code**
Enter your account access code. This is the code you received when your account was provisioned, or the code you use to log into the platform.

**Wallet Address**
Enter your wallet address (0x...). The system will look up your account by wallet ownership.

### Step 3 — Live data activates

Once authenticated:
- The Connect button turns green and shows your username
- The **MOCK DATA** badge changes to **LIVE DATA**
- All widgets reload with real data from the relayer
- Data auto-refreshes every 30 seconds

---

## Admin Panel

The Admin Panel uses the same login flow but additionally requires your account to have **admin privileges**.

### Setting up admin access

If you have direct access to your application's database, set `is_admin = true` on your user record in the `profiles` table.

If you don't have direct database access, contact your platform administrator to grant admin access.

### What happens without admin access

If you authenticate with valid credentials but your account doesn't have admin privileges, you'll see:

```
Admin role required. Contact your platform administrator.
```

Your credentials are valid — you just need the admin flag enabled on your account.

### Continue without authentication

Both dashboards offer a **"Continue with mock data"** option in the login modal. This lets you explore the full interface with realistic sample data without authenticating. Useful for demos or evaluating the dashboard before committing.

---

## How authentication works

```
User enters access code or wallet address
  ↓
POST to your application API { action: 'auth-connect', code: '...' }
  ↓
API returns { session_token, user }
  ↓
Token stored in browser localStorage
  ↓
All dashboard API calls use Bearer token
  ↓
Admin Panel additionally checks user.is_admin === true
```

The `kairos-analytics-api.js` client handles this flow automatically.

---

## Session persistence

Once authenticated, your session is stored in `localStorage`. You won't need to log in again when you return to the dashboard, until the session expires.

To manually log out, click your username in the top navigation bar and select **Disconnect**.

---

## Security notes

- The dashboard only makes read requests to your API — it never writes or modifies data
- Session tokens are stored in `localStorage`, not cookies — no CSRF risk
- No passwords are ever stored in the dashboard client
