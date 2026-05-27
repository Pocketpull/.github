# Source documents index

Every markdown file in the monorepo and how it maps into [platform docs](../README.md).

## Platform docs (this tree)

| Path | Role |
| ---- | ---- |
| [docs/README.md](../README.md) | Central index |
| `getting-started/*` | Local dev, staging, env |
| `architecture/*` | System, auth, API, routes |
| `product/*` | Product summaries + Masterdoc pointers |
| `integrations/*` | Collector Crypt |
| `operations/*` | Runbooks |
| `changelog/*` | Shipped vs backlog |

## Product bible

| File | Lines | Use |
| ---- | ----- | --- |
| [Masterdoc.md](Masterdoc.md) | ~1,034 | Full V3 UX spec (§1–28 product; §29–30 partly outdated) |

## Client & spec (repo root)

| File | Use |
| ---- | --- |
| [rewards.md](rewards.md) | PP earn/spend numbers, shop, tiers |
| [badges.md](badges.md) | 49-badge catalogue + UI rules |
| [Client_notes_3.md](../../Client_notes_3.md) | Round-3 implementation log |
| [marketplace_api_notes.md](../../marketplace_api_notes.md) | CC marketplace API discovery |
| [marketplace_rollout_checklist.md](../../marketplace_rollout_checklist.md) | Marketplace rollout gates |

## docs/ (monorepo)

| File | Platform doc |
| ---- | ------------- |
| [staging.md](../../docs/staging.md) | → [getting-started/staging.md](../getting-started/staging.md) |
| [rewards-system-spec-and-tasks.md](../../docs/rewards-system-spec-and-tasks.md) | → [product/rewards-and-gamification.md](../product/rewards-and-gamification.md) |
| [onboarding-points-system.md](../../docs/onboarding-points-system.md) | → [operations/onboarding-quests.md](../operations/onboarding-quests.md) |
| [ops-store-benefits-runbook.md](../../docs/ops-store-benefits-runbook.md) | → [operations/point-shop-runbook.md](../operations/point-shop-runbook.md) |
| [binder-client-implementation-strategy.md](../../docs/binder-client-implementation-strategy.md) | → [product/binder-and-collections.md](../product/binder-and-collections.md) |

## frontend/

| File | Use |
| ---- | --- |
| [frontend/README.md](../../frontend/README.md) | SPA deploy + env |
| [frontend/docs/COLLECTOR_CRYPT_INTEGRATION_PLAN.md](../../frontend/docs/COLLECTOR_CRYPT_INTEGRATION_PLAN.md) | → [integrations/collector-crypt-gacha.md](../integrations/collector-crypt-gacha.md) |
| [frontend/src/components/home/_archive/README.md](../../frontend/src/components/home/_archive/README.md) | Archived home components |

## backend/

| File | Use |
| ---- | --- |
| [backend/README.md](../../backend/README.md) | API deploy + auth + gacha sync |

## GitHub

| Repo | Content |
| ---- | ------- |
| [Pocketpull/Pocketpull](https://github.com/Pocketpull/Pocketpull) | This docs tree |
| [Pocketpull/.github](https://github.com/Pocketpull/.github) | Org profile README |

## Maintenance

When adding a new root-level `.md` spec:

1. Add a row here  
2. Link from the relevant platform doc section  
3. If it supersedes Masterdoc content, note the section in [product-overview.md](../product/product-overview.md)
