# Privy & Solana wallets

Privy provides **embedded Solana wallets** for new users. It is **not** a separate login method.

## Primary path (server-side)

On successful Google OAuth callback, the API:

1. Calls Privy REST with `custom_auth` identity (Pocketpull user id / email).
2. Creates or links the Privy user with an embedded Solana wallet.
3. Persists rows in `oauth_accounts` and `wallets`.

| File | Role |
| ---- | ---- |
| `backend/src/lib/ensure-privy-wallet.ts` | `provisionPrivyWalletForPocketPullUser` |
| `backend/src/lib/privy-user-sync.ts` | DB linkage |
| `backend/src/routes/register.ts` | Invoked from Google callback |

## Fallback (client)

After login, `privy-wallet-bootstrap.tsx` calls `POST /auth/privy/ensure` if `/auth/me` shows no linked wallet. Client Privy JWT sync is **disabled by default** (`useSubscribeToJwtAuthWithFlag` with `enabled: false`).

## API endpoints

| Method | Path | Purpose |
| ------ | ---- | ------- |
| GET | `/auth/privy/status` | Is Privy configured? |
| GET | `/auth/privy/jwt` | Short-lived JWT for optional browser sync |
| POST | `/auth/privy/ensure` | Idempotent server provision |
| POST | `/auth/privy/link` | Link Privy access token to session |
| POST | `/auth/privy/exchange` | Token exchange helper |

## Manual wallet link (Phantom / Solflare)

| Method | Path |
| ------ | ---- |
| GET | `/wallets/solana` |
| POST | `/wallets/solana/challenge` |
| POST | `/wallets/solana/verify` |
| POST | `/wallets/solana/link` |

Frontend: `solana-wallet-provider.tsx` (optional alongside Privy).

## Privy dashboard

- **Embedded Solana** wallets enabled
- **Allowed origins:** all `WEB_ORIGIN` values + localhost
- **App ID** on frontend (`VITE_PRIVY_APP_ID`) and backend (`PRIVY_APP_ID`, `PRIVY_APP_SECRET`)

Optional: **Third-party auth** plugin + `PRIVY_CUSTOM_AUTH_JWT_SECRET` if enabling browser JWT sync.

## Environment

| Variable | Where |
| -------- | ----- |
| `ENABLE_PRIVY_AUTH` | Backend |
| `PRIVY_APP_ID` / `PRIVY_APP_SECRET` | Backend |
| `PRIVY_CUSTOM_AUTH_JWT_SECRET` | Backend (optional) |
| `VITE_PRIVY_APP_ID` | Frontend |

## Troubleshooting

| Issue | Check |
| ----- | ----- |
| “External auth providers not enabled” | Client JWT path; rely on server `ensure` instead |
| No wallet after login | Railway secrets; logs on callback; `POST /auth/privy/ensure` |
| Wrong cluster | `VITE_SOLANA_NETWORK` + `X-PocketPull-Solana-Cluster` header |
