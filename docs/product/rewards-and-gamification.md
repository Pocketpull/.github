# Rewards & gamification

Pocket Points (PP) drive long-term progression: earn from spend and activities, spend in the **Point Shop** on shipping discounts, marketplace fee reductions, and buyback boosts. Tiers unlock shop access and discounts — they do **not** auto-apply perks without redemption.

## Authoritative sources

| Document | Contents |
| -------- | -------- |
| [rewards.md](../reference/rewards.md) | Client earn rates, shop catalog, tier thresholds |
| [rewards-system-spec-and-tasks.md](rewards-system-spec-and-tasks.md) | Spec vs code gaps, file paths, tasks |
| [onboarding-points-system.md](../operations/onboarding-points-system.md) | Onboarding quest IDs and triggers |

## Earn rules (summary)

| Source | Rate |
| ------ | ---- |
| Purchases | **4 PP per $1** spent |
| Daily login | 10 PP; 7-day streak +75; 30-day +300 |
| Pack milestones | 10 / 25 / 50 / 100 / 250 opens → 250–5,000 PP |
| Referral | **1,000 PP** when referee opens first **paid** pack or slab |
| Share reveal | +25 PP, max **3**/day |
| Profile / marketplace firsts | 50–300 PP (see rewards.md) |
| Binder / type collection | Modest PP (see rewards.md) |

**Rule:** No PP from **free** packs.

## Tier ladder (lifetime cumulative PP)

| Tier | PP required |
| ---- | ----------- |
| Ruby | (base) |
| Emerald | 4,000 |
| Sapphire | 9,000 |
| Pearl | 20,000 |
| Diamond | 45,000 |
| Platinum | 90,000 |

Point Shop **tier discount** on redemptions: 0% Ruby → 15% Platinum (see rewards.md table).

## Point Shop categories

| Category | Examples | PP cost range |
| -------- | -------- | ------------- |
| Shipping | 10% / 25% off, free, double free | 1,500 – 14,000 |
| Marketplace fee | 0.25% – 1.00% off | 2,000 – 10,000 |
| Buyback boost | +1% – +5% | 5,000 – 35,000 |

**Rule:** Shipping benefits = **one shipment** per redemption.

## API surface

| Method | Path | Purpose |
| ------ | ---- | ------- |
| GET | `/gamification/overview` | Quests, badges, PP, tier progress |
| GET | `/rewards/summary` | Rewards summary |
| GET | `/me/store-benefits` | Active redemptions / benefits |
| POST | `/gamification/store/redeem` | Redeem shop item |
| POST | `/gamification/daily-login` | Daily login PP |
| POST | `/gamification/reveal-share` | Share reward |
| POST | `/gamification/quests/:id/claim` | Claim quest |
| GET | `/meta/quests` | Quest definitions (public meta) |

Admin: `/admin/rewards/*` — economy, quests, store items, milestones, achievements.

## Code map

| Area | Path |
| ---- | ---- |
| Tier ladder | `backend/src/lib/tiers.ts`, `rewards-display.ts` |
| Redeem | `backend/src/lib/gamification.ts` (and related) |
| Seed catalog | `backend/src/scripts/seed.ts` |
| UI | `frontend/src/pages/Rewards.tsx` |

## Implementation gaps (engineering)

From [rewards-system-spec-and-tasks.md](rewards-system-spec-and-tasks.md):

| Gap | Detail |
| --- | ------ |
| Tier thresholds | Code may still use older ladder; align to client table above |
| Point Shop | Seed has ~2 items; client lists **15** benefits |
| Redemption fulfillment | PP deducted; shipping/fee/buyback not always applied at use time |
| Rewards UI | Missing shop grouping, locked/unlocked filter |

## Referrals

- Canonical link + cookie/local persistence: `POST /referrals/persist`, `GET /referrals/attribution`
- Claim: `POST /referrals/claim` — blocked after first pack open
- **1,000 PP** on referee first paid open — `referral_first_pack:*` ledger

See Masterdoc §24 and [implementation-status.md](../changelog/implementation-status.md).

## Masterdoc cross-reference

- §8 Pocket Points  
- §21 Rewards  
- §22 Activities / Quests  
- §23 Milestones / Achievements  
- §25 Collector Level & Score  
