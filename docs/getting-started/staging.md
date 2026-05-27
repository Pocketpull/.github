# Staging environment

One codebase, two deploy tracks: **`main`** → production, **`staging`** → preview. No duplicate repositories.

## URL map

| Layer | Production | Staging |
| ----- | ---------- | ------- |
| **Frontend** | `main` → [frontend-9a5.pages.dev](https://frontend-9a5.pages.dev), [pocketpull.io](https://pocketpull.io) | `staging` branch → Cloudflare **Preview** URL |
| **API** | Railway `pocketpull` | Railway `Staging-pp` |
| **API URL** | `https://pocketpull-production.up.railway.app` | `https://staging-pp-production.up.railway.app` |
| **Postgres** | `pocketpull-db` | `Staging-db` |

After the first `staging` deploy, copy the preview URL from Cloudflare **Deployments** (e.g. `https://staging.frontend-9a5.pages.dev`) and add it to Railway `Staging-pp` → `WEB_ORIGIN`.

## Checklist

### Git

- [ ] Branch `staging` exists on [Frontend](https://github.com/Pocketpull/Frontend) and tracks `main` when ready.
- [ ] Backend deploys from same branch strategy on Railway (or manual `railway up` on staging service).

### Cloudflare Pages

- [ ] **Production branch:** `main`
- [ ] **Preview** enabled for `staging` (and PRs)
- [ ] **Production** env: `VITE_API_URL=https://pocketpull-production.up.railway.app`
- [ ] **Preview** env: `VITE_API_URL=https://staging-pp-production.up.railway.app` (see `frontend/.staging.env`)
- [ ] Prefer `VITE_SOLANA_NETWORK=devnet` on Preview
- [ ] Retry deployment after env changes (vars baked at build time)

### Railway (`Staging-pp`)

- [ ] Separate `SESSION_SECRET` and `PRIVY_CUSTOM_AUTH_JWT_SECRET` (never production values)
- [ ] `DATABASE_URL` from `Staging-db` plugin
- [ ] `WEB_ORIGIN` includes staging CF preview URL + `http://localhost:8008`
- [ ] `API_PUBLIC_URL=https://staging-pp-production.up.railway.app`
- [ ] `npm run db:migrate` on empty staging DB once

```bash
cd backend
railway link -p pocketpull -e production -s Staging-pp
railway variables
railway up
railway run npm run db:migrate
```

### Google OAuth

Add to the same (or staging) web client:

- **Redirect:** `https://staging-pp-production.up.railway.app/auth/google/callback`
- **JavaScript origins:** staging CF preview URL, `http://localhost:8008`

### Privy

- Add staging frontend origin to **Allowed origins**
- Same dev app ID is fine until you split staging/production Privy apps

## Isolation rules

- Staging frontend → **only** staging API (`VITE_API_URL`).
- Staging API `WEB_ORIGIN` → **only** staging + local origins (not production domains).
- Never point staging at production Postgres.

Templates: `frontend/.staging.env`, `backend/.staging.env`.
