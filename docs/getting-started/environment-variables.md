# Environment variables

Reference for local, staging, and production. Secrets stay in Railway / Cloudflare dashboards — never commit `.env`.

## Frontend (Vite — `VITE_*` only)

Set in Cloudflare Pages (**Production** vs **Preview**) or `frontend/.env`.

| Variable | Required | Description |
| -------- | -------- | ----------- |
| `VITE_API_URL` | Yes | API base, no trailing slash |
| `VITE_PRIVY_APP_ID` | For wallets | Matches backend Privy app |
| `VITE_ENABLE_PRIVY_AUTH` | Optional | `false` disables Privy in browser |
| `VITE_SOLANA_NETWORK` | Optional | Default `devnet` or `mainnet` |
| `VITE_SOLANA_RPC_URL` | Optional | RPC override |
| `VITE_SOLANA_RPC_URL_DEVNET` / `_MAINNET` | Optional | Per-cluster RPC |
| `VITE_USDC_MINT_*` | Optional | USDC mint overrides |

| Environment | `VITE_API_URL` |
| ----------- | -------------- |
| Local | `http://localhost:8080` |
| Production | `https://pocketpull-production.up.railway.app` |
| Staging (Preview) | `https://staging-pp-production.up.railway.app` |

## Backend (Railway / local `.env`)

See `backend/.env.example` and `backend/.staging.env` for full lists.

### Core

| Variable | Purpose |
| -------- | ------- |
| `DATABASE_URL` | Postgres |
| `SESSION_SECRET` | Session signing (unique per env) |
| `WEB_ORIGIN` | Comma-separated SPA origins (CORS + OAuth return) |
| `API_PUBLIC_URL` | Public API URL for OAuth redirects |

**Production `WEB_ORIGIN` example:**  
`http://localhost:8008,https://frontend-9a5.pages.dev,https://pocketpull.io`

### Auth

| Variable | Purpose |
| -------- | ------- |
| `GOOGLE_CLIENT_ID` / `GOOGLE_CLIENT_SECRET` | Required on Railway |
| `ENABLE_PRIVY_AUTH` | `true` to enable Privy |
| `PRIVY_APP_ID` / `PRIVY_APP_SECRET` | Privy dashboard |
| `PRIVY_CUSTOM_AUTH_JWT_SECRET` | Optional client JWT path |

### Collector Crypt

| Variable | Purpose |
| -------- | ------- |
| `COLLECTORCRYPT_*` | Gacha API credentials (devnet/mainnet pairs) |
| `COLLECTORCRYPT_MARKETPLACE_*` | Marketplace hosts / tokens |
| `GACHA_POOL_LIVE_CACHE_TTL_MS` | In-process pool TTL (0 = fresh `getNfts`) |
| `GACHA_POOL_REFRESH_SECRET` | Protect `POST .../pool/refresh` |

### Storage & payments

| Variable | Purpose |
| -------- | ------- |
| `R2_*` | Cloudflare R2 for pool snapshots, assets |
| `STRIPE_*` | Deposits + webhooks |

### Solana

| Variable | Purpose |
| -------- | ------- |
| `SOLANA_RPC_URL` | Default RPC |
| Treasury / fee payer vars | Deposits, devnet USDC claim (see `.env.example`) |

## Google OAuth redirect URIs

| Environment | Redirect URI |
| ----------- | ------------ |
| Local | `http://localhost:8080/auth/google/callback` |
| Production | `https://pocketpull-production.up.railway.app/auth/google/callback` |
| Staging | `https://staging-pp-production.up.railway.app/auth/google/callback` |

## Privy dashboard

- Embedded **Solana** wallets on
- **Allowed origins:** every `WEB_ORIGIN` entry + localhost
- Optional: Third-party auth plugin for browser JWT sync
