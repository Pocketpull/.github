# Collector Crypt — Marketplace

Marketplace uses a **different base URL** than gacha in many setups (`COLLECTORCRYPT_MARKETPLACE_API_BASE_URL`).

## Discovery notes

**[marketplace-api-notes.md](../reference/marketplace-api-notes.md)** — staging hosts, `/marketplace` feed, `/v2` RPC methods, filter params (2026-04-09).

## Rollout checklist

**[marketplace-rollout-checklist.md](../reference/marketplace-rollout-checklist.md)** — health, list/buy smoke tests, metrics, gates.

## API routes

Module: `backend/src/routes/collector-crypt-marketplace.ts`

| Method | Path |
| ------ | ---- |
| GET | `/integrations/collector-crypt/marketplace/rollout-health` |
| GET | `/integrations/collector-crypt/marketplace` |
| GET | `/integrations/collector-crypt/marketplace/gemrate-options` |
| POST | `/integrations/collector-crypt/marketplace/v2` |
| POST | `/integrations/collector-crypt/marketplace/broadcast` |
| POST | `/integrations/collector-crypt/marketplace/volume-event` |

## Watches (Pocketpull)

`backend/src/routes/marketplace-watches.ts` — user watchlists:

- `GET/POST /marketplace/watches`
- `PATCH/DELETE /marketplace/watches/:id`

## Frontend

- `/marketplace` — P2P marketplace UI (`P2pPage`)
- Legacy `/marketplace/p2p` redirects

## Point Shop tie-in

Marketplace fee discounts from PP redemptions — fulfillment gaps documented in [rewards-and-gamification.md](../product/rewards-and-gamification.md) and [point-shop-runbook.md](../operations/point-shop-runbook.md).

## Cluster header

SPA sends `X-PocketPull-Solana-Cluster` — API selects matching CC marketplace host and credentials.
