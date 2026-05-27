# Collector Crypt — Gacha & slab roll

All CC credentials stay **server-side**. The SPA calls Pocketpull API routes that proxy to Collector Crypt with the correct devnet/mainnet host based on `X-PocketPull-Solana-Cluster`.

## Blueprint

**[frontend/docs/COLLECTOR_CRYPT_INTEGRATION_PLAN.md](../../frontend/docs/COLLECTOR_CRYPT_INTEGRATION_PLAN.md)** — proxy architecture, endpoint mapping, wallet/sign flows, security constraints.

## Slab pool (`GET /gacha/machines/:id/pool`)

**Canonical source (production decision):** live CC **`getNfts`** with optional in-process TTL (`GACHA_POOL_LIVE_CACHE_TTL_MS`; default **0** = always fresh).

- R2 scheduled sync is **auxiliary** (snapshots, CDN, ops) — not primary read for “what’s in the pack right now.”
- Manual refresh: `POST /gacha/machines/:id/pool/refresh` with `GACHA_POOL_REFRESH_SECRET`

See [gacha-pool-sync.md](../operations/gacha-pool-sync.md).

## Machine catalog

- `backend/src/gacha-machines-catalog.ts`
- Seed: `npm run db:seed-machines`
- List: `GET /gacha/machines`, `GET /gacha/machines/:id`

## CC proxy routes

Module: `backend/src/routes/collector-crypt.ts`

| Area | Endpoints |
| ---- | --------- |
| Status / stock / NFTs | `/integrations/collector-crypt/status`, `stock`, `nfts` |
| Packs | `packs/generate`, `yolo`, `open` |
| Transactions | `transactions/submit` |
| Buyback | `buyback`, `buyback/check` |
| Gifts / purchased packs | `gifts/generate`, `purchased-packs/*` |

Opens from Pocketpull DB path: `POST /opens/slab` in `register.ts`.

## Environment

Dual devnet/mainnet variable pairs in `backend/.env.example`:

- `COLLECTORCRYPT_API_BASE_URL` (+ devnet variant)
- API keys, bearer tokens per cluster

## Scheduler

- `gacha-pool-sync-scheduler.ts` — cron every 30 min + ~90s after boot
- `npm run gacha:pool-sync` — manual with Postgres advisory lock

## Client notes

- Home live pulls: merged feed, cap 150 — `GET /activity/recent-pulls`
- Slab Roll pool UI: `slab-pool-grid.tsx`
- Inventory vs CC wallet UI parity — still open (polling / hard refresh)
