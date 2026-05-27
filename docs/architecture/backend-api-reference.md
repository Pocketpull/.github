# Backend API reference

Fastify API on Railway. Most routes require `Authorization: Bearer <session>` unless noted.

**Source of truth:** `backend/src/routes/*.ts` — this doc is a curated index (May 2026).

## Health & public

| Method | Path | Auth |
| ------ | ---- | ---- |
| GET | `/health` | — |
| GET | `/public/landing-stats` | — |
| GET | `/s/:token` | — | Share / OG landing |
| GET | `/meta/seasons` | — |
| GET | `/meta/milestones` | — |
| GET | `/meta/achievements` | — |
| GET | `/meta/quests` | — |

## Authentication

| Method | Path |
| ------ | ---- |
| GET | `/auth/google` |
| GET | `/auth/google/callback` |
| GET | `/auth/me` |
| POST | `/auth/logout` |
| GET | `/auth/privy/status` |
| GET | `/auth/privy/jwt` |
| POST | `/auth/privy/ensure` |
| POST | `/auth/privy/link` |
| POST | `/auth/privy/exchange` |
| POST | `/auth/discord/link/start` |
| GET | `/auth/discord/callback` |
| POST | `/auth/discord/unlink` |

## Profile & wallet

| Method | Path |
| ------ | ---- |
| PATCH | `/profile` |
| POST | `/profile/avatar/presign` |
| POST | `/profile/avatar/upload` |
| GET | `/wallet` |
| POST | `/deposits/stripe` |
| GET | `/wallets/solana` |
| POST | `/wallets/solana/link` |
| POST | `/wallets/solana/challenge` |
| POST | `/wallets/solana/verify` |

## Packs & opens

| Method | Path |
| ------ | ---- |
| GET | `/packs` |
| GET | `/packs/:slug` |
| POST | `/opens/pack` |
| POST | `/opens/slab` |

## Gacha / slab machines

| Method | Path |
| ------ | ---- |
| GET | `/gacha/machines` |
| GET | `/gacha/machines/:id` |
| GET | `/gacha/machines/:id/pool` |
| POST | `/gacha/machines/:id/pool/refresh` | Secret header |

## Inventory

| Method | Path |
| ------ | ---- |
| GET | `/inventory` |
| PATCH | `/inventory/items/:id` |

## Gamification & rewards

| Method | Path |
| ------ | ---- |
| POST | `/gamification/daily-login` |
| POST | `/gamification/reveal-share` |
| POST | `/gamification/quests/:id/claim` |
| GET | `/gamification/overview` |
| GET | `/rewards/summary` |
| GET | `/me/store-benefits` |
| POST | `/gamification/store/redeem` |

## Activity & leaderboard

| Method | Path |
| ------ | ---- |
| GET | `/activity/recent-pulls` |
| GET | `/activity/my-pulls` |
| GET | `/me/platform-events` |
| GET | `/leaderboard` |

## Referrals & share

| Method | Path |
| ------ | ---- |
| POST | `/referrals/persist` |
| GET | `/referrals/attribution` |
| GET | `/referrals/preview` |
| GET | `/referrals/me` |
| POST | `/referrals/claim` |
| POST | `/share/pull` |

## X (Twitter) verification

| Method | Path |
| ------ | ---- |
| POST | `/integrations/x/verification/start` |
| POST | `/integrations/x/verification/confirm` |

## Collector Crypt — gacha proxy

`backend/src/routes/collector-crypt.ts`

| Method | Path |
| ------ | ---- |
| GET | `/integrations/collector-crypt/status` |
| GET | `/integrations/collector-crypt/stock` |
| GET | `/integrations/collector-crypt/nfts` |
| GET | `/integrations/collector-crypt/winners` |
| GET | `/integrations/collector-crypt/buyback/check` |
| GET | `/integrations/collector-crypt/gifted` |
| GET | `/integrations/collector-crypt/purchased-packs` |
| POST | `/integrations/collector-crypt/packs/generate` |
| POST | `/integrations/collector-crypt/packs/yolo` |
| POST | `/integrations/collector-crypt/packs/open` |
| POST | `/integrations/collector-crypt/transactions/submit` |
| POST | `/integrations/collector-crypt/buyback` |
| POST | `/integrations/collector-crypt/gifts/generate` |
| POST | `/integrations/collector-crypt/purchased-packs/challenge` |
| POST | `/integrations/collector-crypt/purchased-packs/use` |

## Collector Crypt — marketplace

`backend/src/routes/collector-crypt-marketplace.ts`

| Method | Path |
| ------ | ---- |
| GET | `/integrations/collector-crypt/marketplace/rollout-health` |
| GET | `/integrations/collector-crypt/marketplace` |
| GET | `/integrations/collector-crypt/marketplace/gemrate-options` |
| POST | `/integrations/collector-crypt/marketplace/v2` |
| POST | `/integrations/collector-crypt/marketplace/broadcast` |
| POST | `/integrations/collector-crypt/marketplace/volume-event` |

## Marketplace watches

| Method | Path |
| ------ | ---- |
| GET | `/marketplace/watches` |
| POST | `/marketplace/watches` |
| PATCH | `/marketplace/watches/:id` |
| DELETE | `/marketplace/watches/:id` |

## Binder & collection quests

| Method | Path |
| ------ | ---- |
| GET | `/binder/pokemon/summary` |
| GET | `/collection-quests` |
| GET | `/collection-quests/:slug` |

## Pokémon catalogue & market data

| Method | Path |
| ------ | ---- |
| GET | `/catalogue/pokemon/*` | Sets, cards, manifest |
| GET | `/market-data/pokemon/card` |
| POST | `/market-data/pokemon/cards` |

## Devnet utilities

| Method | Path |
| ------ | ---- |
| GET | `/integrations/devnet/claim-usdc/config` |
| POST | `/integrations/devnet/claim-usdc` |

## Webhooks & partners

| Method | Path |
| ------ | ---- |
| POST | `/webhooks/stripe` | Raw body |
| POST | `/partners/applications` |

## Admin (requires `users.is_admin`)

| Prefix | Module |
| ------ | ------ |
| `/admin/users` | `admin-users.ts` |
| `/admin/rewards/*` | `admin-rewards.ts` — economy, quests, store, milestones, achievements |
| `/admin/collection-quests/*` | `collection-quests-admin.ts` |

## Obsolete (Masterdoc §29)

The prototype described mock Cloudflare Functions (`GET /api/packs`, etc.). **All production traffic** uses the Fastify routes above.
