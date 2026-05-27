# Pocket Pull — Rewards System Specification & Implementation Tasks

This document captures the **client’s updated rewards specification**, a **review of the current codebase**, and a **task list** for engineering to make behavior and UI match the spec.

---

## 1. Executive summary


| Area                            | Current state                                                                                                  | Gap                                                                                                        |
| ------------------------------- | -------------------------------------------------------------------------------------------------------------- | ---------------------------------------------------------------------------------------------------------- |
| **Lifetime PP tier thresholds** | Fixed ladder in `backend/src/lib/tiers.ts` and `backend/src/lib/rewards-display.ts` (e.g. Emerald at 5,000 PP) | Client requires **new cumulative thresholds** (Emerald 4,000 PP through Platinum 90,000 PP)                |
| **Point Shop catalog**          | Only **2** seeded items in `backend/src/scripts/seed.ts`                                                       | Client lists **15** purchasable benefits with exact PP costs                                               |
| **Redemption fulfillment**      | `redeemStoreItem` deducts PP and inserts `store_redemptions` only                                              | **No** application of shipping discounts, marketplace fee discounts, or buyback boosts at fulfillment time |
| **Free packs**                  | PP from opens scales with `spendCents`; $0 → 0 PP mathematically                                               | Must **explicitly** skip PP (and clarify quest rules); quests may still increment today                    |
| **Shipping rewards**            | Not modeled as single-use                                                                                      | Client: **one shipment only** per applicable redemption — needs consumption tracking                       |
| **Rewards UI**                  | Strong baseline (PP, tier progress, shop, quests)                                                              | Missing **shop item grouping**, **locked/unlocked filter toggle**, and alignment with new catalog          |


---

## 2. Client specification (authoritative)

### 2.1 Point Shop — Shipping


| Benefit               | Cost (PP) |
| --------------------- | --------- |
| 10% Shipping Discount | 1,500     |
| 25% Shipping Discount | 3,500     |
| Free Shipping         | 7,000     |
| Double Free Shipping  | 14,000    |


**Rule:** Shipping rewards apply for **one shipment only** (per redemption / benefit — confirm “Double Free Shipping” = two uses vs one shipment with two items).

### 2.2 Point Shop — Marketplace fee discount


| Benefit                        | Cost (PP) |
| ------------------------------ | --------- |
| 0.25% Marketplace Fee Discount | 2,000     |
| 0.50% Marketplace Fee Discount | 4,000     |
| 0.75% Marketplace Fee Discount | 7,000     |
| 1.00% Marketplace Fee Discount | 10,000    |


### 2.3 Point Shop — Buyback boost


| Benefit           | Cost (PP) |
| ----------------- | --------- |
| +1% Buyback Boost | 5,000     |
| +2% Buyback Boost | 10,000    |
| +3% Buyback Boost | 16,000    |
| +4% Buyback Boost | 24,000    |
| +5% Buyback Boost | 35,000    |


### 2.4 Tier unlock amounts (lifetime Pocket Points, cumulative)

Tiers are unlocked by **lifetime PP earned** (cumulative), not by spend alone.


| Tier     | Lifetime PP required |
| -------- | -------------------- |
| Emerald  | 4,000                |
| Sapphire | 9,000                |
| Pearl    | 20,000               |
| Diamond  | 45,000               |
| Platinum | 90,000               |


**Note:** In code, display names map to existing tier IDs roughly as: Ruby = base, Emerald = collector, Sapphire = rare, Pearl = holo, Diamond = elite, Platinum = black-label. Confirm naming in UI matches this mapping everywhere.

### 2.5 Global rules

- **No Pocket Points** for **free** pack opens (see §4.3).
- **Shipping** shop benefits: **single shipment** consumption per product rules above.
- **Quests:** can be added **manually** in the database; **logic** that applies shop benefits and tier rules must still be implemented in code.

### 2.6 Rewards page (client expectations)

The Rewards experience should show:

1. **Current Pocket Points** (spendable balance).
2. **Current tier** with **progress to next tier** (same spirit as today — bar / “PP to go”).
3. **Point Shop discount %** from the **current tier** (tier-based discount on shop PP prices — already conceptually present as `pointShopDiscountPct` / `discountedPointCost`).
4. **Available** shop items (user can afford + meets tier gate if any).
5. **Locked** higher-tier or not-yet-affordable items, with a **toggle**: show **unlocked only** vs **all** (including locked).

---

## 3. Current implementation map

### 3.1 Backend — key files


| File                                 | Role                                                                                                                                                                                                                         |
| ------------------------------------ | ---------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| `backend/src/lib/tiers.ts`           | `tierForLifetimePocketPoints`, `PP_DESC` thresholds, `tierProgressFromLifetimePocketPoints`, `effectiveMembershipTier` (max of **spend** tier and **PP** tier), `buybackRateForTier` (numeric rate for pack/slab simulation) |
| `backend/src/lib/rewards-display.ts` | `REWARD_TIER_LADDER`, `tierBenefitsForApi` (buyback **display** %, marketplace fee label, shipping label, **point shop discount %**), `PP_UNLOCK_BY_TIER`, `discountedPointCost`                                             |
| `backend/src/lib/gamification.ts`    | `applyPocketPointsEarnTx`, `grantPackOpenGamification` (10 PP per $1 when `spendCents` set), quests, `redeemStoreItem`, daily login PP                                                                                       |
| `backend/src/routes/register.ts`     | `GET /gamification/overview`, `GET /rewards/summary`, `POST /gamification/store/redeem`, pack/slab open → `grantPackOpenGamification`                                                                                        |
| `backend/src/db/schema.ts`           | `store_items`, `store_redemptions`, `quest_definitions`, `points_ledger`, `users` (`lifetimePocketPoints`, `currentTierId`, …)                                                                                               |
| `backend/src/scripts/seed.ts`        | Seeds **2** `store_items` and several `quest_definitions`                                                                                                                                                                    |


### 3.2 Frontend — key files


| File                                                          | Role                                                            |
| ------------------------------------------------------------- | --------------------------------------------------------------- |
| `frontend/src/pages/Rewards.tsx`                              | Loads `/gamification/overview`, redeem mutation                 |
| `frontend/src/components/rewards/pocket-points-dashboard.tsx` | Rewards + Point Shop tabs, tier ladder table, quests, shop list |


### 3.3 Tier threshold comparison (must change)


| Display tier | Client lifetime PP | Current `PP_DESC` (`tiers.ts`) | Current ladder display `PP_UNLOCK_BY_TIER` |
| ------------ | ------------------ | ------------------------------ | ------------------------------------------ |
| Ruby (entry) | 0                  | 0                              | 0                                          |
| Emerald      | **4,000**          | 5,000                          | 5,000                                      |
| Sapphire     | **9,000**          | 15,000                         | 15,000                                     |
| Pearl        | **20,000**         | 35,000                         | 35,000                                     |
| Diamond      | **45,000**         | 75,000                         | 75,000                                     |
| Platinum     | **90,000**         | 150,000                        | 150,000                                    |


---

## 4. Technical notes & risks

### 4.1 Two “tier” concepts

- `**effectiveMembershipTier(spend, lifetimePp)`** upgrades tier from **either** lifetime spend **or** lifetime PP (whichever rank is higher).
- **Progress bar** in API uses `**tierProgressFromLifetimePocketPoints`** (PP-only).

**Risk:** UI shows a single `tierId` from `users.currentTierId` (combined). If spend pushes tier above PP ladder, the “PP to next tier” bar can disagree with the badge. **Product decision:** PP-only ladder on Rewards page vs combined tier everywhere — document and implement consistently.

### 4.2 Buyback: display vs simulation

- `rewards-display.ts` exposes **display** buyback percentages (e.g. 85–90%).
- `tiers.ts` `**buybackRateForTier`** uses a **different** numeric model (`0.78` + bonuses) for `simulatePullFromPack`.

**Risk:** User-visible “base buyback” may not match engine. Align or rename fields so the Rewards page reflects real economics.

### 4.3 Free packs and PP

`grantPackOpenGamification` today:

- If `spendCents` is passed: `ppEarn = round((spendCents / 100) * 10)` → **$0 → 0 PP**.
- Still calls `**advanceQuestsByKindTx(..., kind: "pack_opens", ...)`** regardless.

**Client rule:** no PP for free packs — implement **explicit** guard and confirm whether **quests** should also not advance for $0 opens.

### 4.4 Store redemptions today

`redeemStoreItem` only:

1. Validates tier (`min_tier_id`).
2. Deducts discounted PP cost.
3. Inserts `points_ledger` and `store_redemptions`.

There is **no** link to shipments, marketplace fees, or buyback calculations.

---

## 5. Task list (implementation checklist)

Use this as the master backlog. Check items off in PRs as you complete them.

### A. Specification & product

- **A1** Confirm with client: **“Double Free Shipping”** = two separate single-use free shipments vs one shipment.
- **A2** Confirm: **tier for Rewards UI** = PP-only ladder vs **combined** spend+PP (`effectiveMembershipTier`).
- **A3** Confirm stacking: shop **buyback boosts** vs **tier buyback** — additive cap? max +5% from shop only?
- **A4** Confirm whether **marketplace fee discounts** can be enforced in-app or are **manual/ops** until Collector Crypt API supports it.
- **A5** Confirm **free pack** definition ($0 credit open, promo code, separate CC-only flow, etc.).

### B. Data model & migrations

- **B1** Add columns (or a child table) to support **typed benefits** and **consumption**, e.g.:
  - `benefit_category`: `shipping` | `marketplace_fee` | `buyback_boost` (enum or varchar)
  - `benefit_value`: numeric (percent or basis points — pick one convention)
  - `max_uses` / `remaining_uses` (default **1** for single-shipment shipping perks)
  - Optional: `consumed_at`, `metadata` jsonb for extensibility
- **B2** Create Drizzle migration + update `backend/src/db/schema.ts`.
- **B3** Backfill or migrate existing `store_redemptions` rows if schema changes.

### C. Tier thresholds (PP ladder)

- **C1** Update `**PP_DESC`** in `backend/src/lib/tiers.ts` to: 4k / 9k / 20k / 45k / 90k (mapped to correct `MembershipTierId` rows in **ascending** order for progress math).
- **C2** Update `**PP_UNLOCK_BY_TIER`** in `backend/src/lib/rewards-display.ts` (and `tierLadderRowsForApi`) to match **exactly**.
- **C3** Grep for hardcoded old numbers (tests, docs, `docs/reference/rewards.md`) and update.

### D. Point Shop catalog

- **D1** Insert **15** `store_items` matching §2.1–§2.3 (titles, descriptions, `cost_points`, `benefit_category`, `benefit_value`, `max_uses` / defaults).
- **D2** Use `**onConflictDoUpdate`** in seed (or a dedicated migration SQL) so staging/prod can refresh catalog without duplicate errors.
- **D3** Set `**min_tier_id`** if product adds tier gates; otherwise `null` where only PP cost applies.

### E. Free packs — no PP (and optional quest skip)

- **E1** In `grantPackOpenGamification`, if `spendCents === 0` (or explicit `isFreeOpen` after A5), **skip** `applyPocketPointsEarnTx`.
- **E2** If client confirms: **skip** `advanceQuestsByKindTx` for `pack_opens` when open is free.
- **E3** Update `RewardPackSourcesPanel` / any UI copy that claims “10 PP per $1” unconditionally.

### F. Fulfillment — shipping (single use)

- **F1** Identify the **canonical event** for “shipment” (e.g. inventory status → `shipped`, label purchase, 3PL webhook).
- **F2** On that event, resolve **active** shipping redemption(s) with `remaining_uses > 0`, apply discount/free logic, then **decrement** or **close** redemption.
- **F3** Ensure **idempotency** (retries don’t double-consume).
- **F4** Add API or internal function to **list active benefits** for profile/support if needed.

### G. Fulfillment — buyback boost

- **G1** When computing buyback for **credit** inventory (and any other buyback surfaces), load active `**buyback_boost`** redemptions and apply per A3.
- **G2** Mark boost as consumed if one-shot, or decay if time-limited (per future spec).

### H. Fulfillment — marketplace fee discount

- **H1** Trace where CC marketplace **listing/buy** fees are set; inject **active fee discount** from redemptions if API allows.
- **H2** If not automatable: add **admin/ops workflow** + UI note “pending manual application” until H1 exists.

### I. API responses for Rewards UI

- **I1** Ensure `GET /gamification/overview` returns enough to drive **toggle** (all items + `userMeetsMinTier` / `canAfford` booleans, or raw `minTierId` + costs for client-side filter).
- **I2** Optionally return `**tierFromPpOnly`** and `**tierFromSpend**` if product chooses split display (depends on A2).

### J. Frontend — Rewards / Point Shop

- **J1** Add **toggle**: “Unlocked only” ↔ “Show all (locked grayed out)”.
- **J2** Group shop items by **Shipping / Marketplace / Buyback** (from API field).
- **J3** Disable **Redeem** for locked rows; show **PP cost**, **tier lock**, and **insufficient PP** states clearly.
- **J4** Keep **current PP**, **tier + progress**, **tier shop discount %** prominent; verify copy matches new thresholds after C1–C2.
- **J5** Align **tier ladder** table with server after C2.

### K. Tests & quality

- **K1** Unit tests: tier boundary at **4k / 9k / 20k / 45k / 90k** PP (`tierProgressFromLifetimePocketPoints`, `tierForLifetimePocketPoints`).
- **K2** Unit/integration: **$0** open → **0** PP; quest behavior per E2.
- **K3** Tests: redeem → **consume** shipping benefit on mocked shipment event.
- **K4** Tests: buyback math with **+n%** active redemption.
- **K5** Turn on **lint/typecheck** in CI for touched packages (currently `lint` may be stubbed).

### L. Documentation & ops

- **L1** Update [rewards.md](../reference/rewards.md) to match this spec once implemented.
- **L2** Runbook: how support verifies a user’s **active redemptions** and **consumption** state.

---

## 6. Quests (manual)

- **Q1** Client / ops: insert or update rows in `**quest_definitions`** as desired (titles, `kind`, `target_count`, `reward_points`, `category`).
- **Q2** Engineering: for any **new** `kind` value, add matching `**advanceQuestsByKindTx`** calls at the appropriate domain events (grep existing kinds: `pack_opens`, `inventory_shipped`, `buyback_complete`, `marketplace_list`, `marketplace_buy`, `login_streak_7`, `login_streak_30`, etc.).

---

## 7. Document history


| Date       | Author               | Change                                                      |
| ---------- | -------------------- | ----------------------------------------------------------- |
| 2026-04-27 | Engineering (Cursor) | Initial spec consolidation + task list from codebase review |
| 2026-04-27 | Engineering (Cursor) | Implemented v2: migration `0010_store_point_shop_benefits`, tier PP 4k–90k, 15 shop SKUs, fulfillment hooks, API + Rewards UI |


---

*End of document.*