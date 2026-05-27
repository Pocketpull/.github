# Pocketpull — Internal documentation

Central index for engineering, product, and operations. These docs describe the **production stack** (Fastify on Railway, SPA on Cloudflare Pages, Google OAuth, Privy wallets). Where they differ from [reference/Masterdoc.md](reference/Masterdoc.md), trust this tree and the service READMEs.

## Getting started

| Doc | Description |
| --- | ----------- |
| [local-development.md](getting-started/local-development.md) | Clone, env, Postgres, run frontend + backend |
| [staging.md](getting-started/staging.md) | `staging` branch, CF Preview, Railway `Staging-pp`, OAuth allowlists |
| [environment-variables.md](getting-started/environment-variables.md) | Full env matrix (local, staging, production) |

## Architecture

| Doc | Description |
| --- | ----------- |
| [system-overview.md](architecture/system-overview.md) | Request path, deploy topology, integrations |
| [authentication.md](architecture/authentication.md) | Google OAuth, Bearer sessions, `#pp_token=` |
| [privy-and-wallets.md](architecture/privy-and-wallets.md) | Embedded Solana, server provision, manual link |
| [backend-api-reference.md](architecture/backend-api-reference.md) | Fastify route catalog by domain |
| [frontend-routes.md](architecture/frontend-routes.md) | SPA routes, providers, auth gates |

## Product

| Doc | Description |
| --- | ----------- |
| [product-overview.md](product/product-overview.md) | Vision, principles, production vs prototype |
| [information-architecture.md](product/information-architecture.md) | Hub, Market, Collections, Rewards, Leaderboards |
| [rewards-and-gamification.md](product/rewards-and-gamification.md) | Pocket Points, tiers, shop, quests — spec + implementation |
| [badges-and-achievements.md](product/badges-and-achievements.md) | Badge catalogue and milestone families |
| [binder-and-collections.md](product/binder-and-collections.md) | Vault, binder model, collection quests |

## Integrations

| Doc | Description |
| --- | ----------- |
| [collector-crypt-gacha.md](integrations/collector-crypt-gacha.md) | Slab roll, pool sync, CC proxy |
| [collector-crypt-marketplace.md](integrations/collector-crypt-marketplace.md) | Marketplace API, rollout, env |

## Operations

| Doc | Description |
| --- | ----------- |
| [gacha-pool-sync.md](operations/gacha-pool-sync.md) | R2 snapshots, cron, live pool cache |
| [onboarding-quests.md](operations/onboarding-quests.md) | Quest IDs, triggers, gamification API |
| [point-shop-runbook.md](operations/point-shop-runbook.md) | Redemptions, SQL, CC partner fees |

## Changelog & reference

| Doc | Description |
| --- | ----------- |
| [implementation-status.md](changelog/implementation-status.md) | Shipped vs backlog summary |
| [client-notes-round-3.md](changelog/client-notes-round-3.md) | Full round-3 implementation log |
| [source-documents.md](reference/source-documents.md) | Complete file catalogue |
| [Masterdoc.md](reference/Masterdoc.md) | V3 product UX spec |
| [rewards.md](reference/rewards.md) · [badges.md](reference/badges.md) | PP + badge numbers |

## Deep-dive specs (moved from repo root)

| Doc | Description |
| --- | ----------- |
| [rewards-system-spec-and-tasks.md](product/rewards-system-spec-and-tasks.md) | Rewards engineering gaps + tasks |
| [binder-implementation-strategy.md](product/binder-implementation-strategy.md) | Binder / quests plan |
| [onboarding-points-system.md](operations/onboarding-points-system.md) | Onboarding quest reference |
| [store-benefits-runbook.md](operations/store-benefits-runbook.md) | Point Shop SQL + CC env |
| [collector-crypt-integration-plan.md](integrations/collector-crypt-integration-plan.md) | CC gacha integration blueprint |
| [marketplace-api-notes.md](reference/marketplace-api-notes.md) | CC marketplace API notes |
| [marketplace-rollout-checklist.md](reference/marketplace-rollout-checklist.md) | Marketplace rollout checklist |

## External READMEs

- [Frontend README](https://github.com/Pocketpull/Frontend/blob/main/README.md)
- [Backend README](https://github.com/Pocketpull/Backend/blob/main/README.md)
- [Org profile](https://github.com/Pocketpull)

## Product bible

**[reference/Masterdoc.md](reference/Masterdoc.md)** — full V3 UX/product specification (~1,000 lines). Sections 1–28 are product truth; §29 (mock Cloudflare Functions) and parts of §30 are **outdated** relative to the Fastify backend. See [product-overview.md](product/product-overview.md) for a production-aligned summary.

Authoritative number specs: [reference/rewards.md](reference/rewards.md), [reference/badges.md](reference/badges.md).
