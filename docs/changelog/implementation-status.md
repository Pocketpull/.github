# Implementation status

Living tracker for product ↔ engineering alignment. Sourced from [Client_notes_3.md](../../Client_notes_3.md) and production verification.

**Last reviewed:** May 2026

## Shipped

| Area | Item |
| ---- | ---- |
| **Auth** | Google OAuth, Bearer `#pp_token=`, Privy server wallet on callback |
| **Deploy** | Cloudflare Pages + Railway, staging branch + `Staging-pp` |
| **Home** | Recent pulls strip (images, marquee, New/Top $, 150 cap) |
| **Home** | Removed global `SitePullsTicker`; hero copy tightened |
| **Slab roll** | Pool from live CC `getNfts`; pool grid alignment |
| **Referrals** | Attribution, persist, claim; block claim after first pack |
| **Referrals** | 1,000 PP on referee first paid pack/slab |
| **Onboarding quests** | DB seed + gamification overview (see [onboarding-quests.md](../operations/onboarding-quests.md)) |

## In progress / partial

| Area | Item | Notes |
| ---- | ---- | ----- |
| **Rewards** | Full Point Shop catalog (15 items) | Seed has ~2; see [rewards-system-spec](../../docs/rewards-system-spec-and-tasks.md) |
| **Rewards** | Tier thresholds (Emerald 4k → Platinum 90k) | Align `tiers.ts` with client spec |
| **Rewards** | Redemption fulfillment at ship/fee/buyback | PP deducts; benefits not always applied |
| **Binder** | Server-backed binder + quests | P0 partial; P1 backlog |
| **Marketplace** | CC marketplace rollout | Checklist in [marketplace_rollout_checklist.md](../../marketplace_rollout_checklist.md) |

## Not done / deferred

| Area | Item |
| ---- | ---- |
| Slab roll | CC inventory UI parity, hard refresh model (reduce polling) |
| Binder | Backend integration, slot data source rules |
| Masterdoc gaps | CNFT mint, burn-to-redeem shipping, production randomness audit |
| Apple Sign-In | Planned |

## Slab pool decision (accepted)

**Live `getNfts`** is canonical for `GET /gacha/machines/:id/pool`. R2 sync is auxiliary.

## Referrals (detail)

- `POST /referrals/persist`, `GET /referrals/attribution`  
- `POST /referrals/claim` — 403 when `packsOpened > 0`  
- Profile hides claim UI when ineligible  

## How to update this doc

After each client round, append rows to [Client_notes_3.md](../../Client_notes_3.md) and mirror summary here.
