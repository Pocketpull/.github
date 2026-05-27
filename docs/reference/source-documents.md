# Source documents index

All documentation lives under `pocketpull-platform/docs/`. This file is the complete catalogue.

## Platform docs tree

| Path | Role |
| ---- | ---- |
| [README.md](../README.md) | Central index |
| `getting-started/*` | Local dev, staging, env |
| `architecture/*` | System, auth, API, routes |
| `product/*` | Product summaries + engineering specs |
| `integrations/*` | Collector Crypt |
| `operations/*` | Runbooks |
| `changelog/*` | Implementation logs |
| `reference/*` | Masterdoc, rewards, badges, API notes |

## Reference (authoritative specs)

| File | Contents |
| ---- | -------- |
| [Masterdoc.md](Masterdoc.md) | Full V3 UX/product spec (~1,034 lines) |
| [rewards.md](rewards.md) | PP earn/spend, shop, tiers |
| [badges.md](badges.md) | 49-badge catalogue + UI rules |
| [marketplace-api-notes.md](marketplace-api-notes.md) | CC marketplace API discovery |
| [marketplace-rollout-checklist.md](marketplace-rollout-checklist.md) | Marketplace rollout gates |

## Product engineering specs

| File | Contents |
| ---- | -------- |
| [rewards-system-spec-and-tasks.md](../product/rewards-system-spec-and-tasks.md) | Rewards spec vs code gaps |
| [binder-implementation-strategy.md](../product/binder-implementation-strategy.md) | Binder / collection quests plan |

## Operations (full runbooks)

| File | Summary doc |
| ---- | ----------- |
| [onboarding-points-system.md](../operations/onboarding-points-system.md) | [onboarding-quests.md](../operations/onboarding-quests.md) |
| [store-benefits-runbook.md](../operations/store-benefits-runbook.md) | [point-shop-runbook.md](../operations/point-shop-runbook.md) |

## Integrations (full plans)

| File | Summary doc |
| ---- | ----------- |
| [collector-crypt-integration-plan.md](../integrations/collector-crypt-integration-plan.md) | [collector-crypt-gacha.md](../integrations/collector-crypt-gacha.md) |

## Changelog

| File | Contents |
| ---- | -------- |
| [client-notes-round-3.md](../changelog/client-notes-round-3.md) | Round-3 product/engineering log |
| [implementation-status.md](../changelog/implementation-status.md) | Shipped vs backlog summary |

## Service READMEs

| File | Contents |
| ---- | -------- |
| [frontend/README.md](../../../frontend/README.md) | SPA deploy + env |
| [backend/README.md](../../../backend/README.md) | API deploy + auth |

## GitHub

| Repo | Content |
| ---- | ------- |
| [Pocketpull/Pocketpull](https://github.com/Pocketpull/Pocketpull) | This docs tree |
| [Pocketpull/.github](https://github.com/Pocketpull/.github) | Org profile |

## Maintenance

When adding documentation:

1. Place it under the correct `docs/` subdirectory (not repo root).
2. Add a row here and link from [README.md](../README.md).
3. Update [implementation-status.md](../changelog/implementation-status.md) if it changes shipped/backlog status.
