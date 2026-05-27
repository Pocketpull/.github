# Local development

Run the Pocketpull API and SPA on your machine. Production uses Cloudflare Pages + Railway; locally you mirror the same split with different URLs.

## Prerequisites

- **Node.js 20+**
- **PostgreSQL 16** (Docker or native)
- **Google OAuth** web client (for sign-in)
- **Privy** app (embedded Solana wallets) — optional but recommended

## 1. Postgres

```bash
docker run --name pp-pg \
  -e POSTGRES_PASSWORD=postgres \
  -e POSTGRES_USER=postgres \
  -e POSTGRES_DB=pocketpull \
  -p 5432:5432 -d postgres:16
```

## 2. Backend

```bash
cd backend
cp .env.example .env
```

| Variable | Local value |
| -------- | ----------- |
| `DATABASE_URL` | `postgresql://postgres:postgres@localhost:5432/pocketpull` |
| `WEB_ORIGIN` | `http://localhost:8008` |
| `API_PUBLIC_URL` | `http://localhost:8080` |
| `SESSION_SECRET` | Any 32+ char string |
| `GOOGLE_CLIENT_ID` / `GOOGLE_CLIENT_SECRET` | From Google Cloud Console |
| `ENABLE_PRIVY_AUTH` | `true` |
| `PRIVY_APP_ID` / `PRIVY_APP_SECRET` | From Privy dashboard |

**Google OAuth redirect URI:** `http://localhost:8080/auth/google/callback`  
**JavaScript origin:** `http://localhost:8008`

```bash
npm install
npm run db:migrate
npm run db:seed
npm run dev
# → http://localhost:8080
```

## 3. Frontend

```bash
cd frontend
cp .env.example .env
```

| Variable | Local value |
| -------- | ----------- |
| `VITE_API_URL` | `http://localhost:8080` |
| `VITE_PRIVY_APP_ID` | Same as backend `PRIVY_APP_ID` |
| `VITE_ENABLE_PRIVY_AUTH` | `true` (or omit if app ID set) |

```bash
npm install
npm run dev
# → http://localhost:8008 (strictPort in vite.config.ts)
```

## 4. Sign in flow (verify)

1. Open `http://localhost:8008/login` → **Continue with Google**.
2. Browser hits `http://localhost:8080/auth/google`, then Google, then callback on **8080**.
3. Redirect to `http://localhost:8008/login#pp_token=…` — token stored before React mounts.
4. `GET /auth/me` with Bearer token; Privy wallet provisioned via callback or `POST /auth/privy/ensure`.

## 5. Optional tooling

| Tool | Use |
| ---- | --- |
| `stripe listen --forward-to localhost:8080/webhooks/stripe` | Stripe webhooks |
| `npm run gacha:pool-sync` (backend) | Manual CC → R2 pool snapshot |
| `npm run test` (frontend) | Unit tests |

## Troubleshooting

| Symptom | Fix |
| ------- | --- |
| CORS / OAuth redirect wrong | `WEB_ORIGIN` must include exact frontend origin (no trailing slash). |
| Google `redirect_uri_mismatch` | Redirect URI must be on **API** host (`:8080`), not Vite `:8008`. |
| 401 on all API calls | Check `#pp_token=` was consumed; `Authorization: Bearer` in Network tab. |
| Privy wallet missing | `ENABLE_PRIVY_AUTH`, secrets on API; check `/auth/me` for `privyLinked`. |

See [authentication.md](../architecture/authentication.md) and [environment-variables.md](environment-variables.md).
