# Gacha pool sync (R2)

Scheduled and manual sync from Collector Crypt into Cloudflare R2 for snapshots and ops. **Live slab pool UI** reads CC `getNfts` directly — see [collector-crypt-gacha.md](../integrations/collector-crypt-gacha.md).

## Automatic sync

| Trigger | Behavior |
| ------- | -------- |
| Cron | Every **30 minutes** when CC credentials set |
| Boot | ~**90s** after API start |

Module: `backend/src/gacha-pool-sync-scheduler.ts`

## Manual sync

```bash
cd backend
npm run gacha:pool-sync
```

Uses Postgres **advisory lock** when `DATABASE_URL` is set (safe for parallel Railway instances).

## Catalog

- Machine list: `backend/src/gacha-machines-catalog.ts`
- Seed machines: `npm run db:seed-machines`

## Live API cache

| Endpoint | Cache |
| -------- | ----- |
| `GET /gacha/machines/:id/pool` | In-process TTL via `GACHA_POOL_LIVE_CACHE_TTL_MS` (0 = fresh) |
| `POST /gacha/machines/:id/pool/refresh` | Protected by `GACHA_POOL_REFRESH_SECRET` |

Public cache header on pool responses: ~90s where applicable.

## R2 variables

Same as API asset storage — see `backend/.env.example` (`R2_*`).

## Related scripts

| Script | Purpose |
| ------ | ------- |
| `npm run gacha:pool-sync` | CC → R2 snapshots |
| `npm run pokemon:sync` | Pokémon archive → R2 (resume) |
| `npm run pokemon:sync:full` | Full wipe + sync |
