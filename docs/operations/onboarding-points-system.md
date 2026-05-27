# Onboarding Pocket Points

This document describes the **onboarding** quest category: one-time (`timeframe: permanent`) activities that advance from real product events. Users **claim** rewards the same way as daily/weekly/monthly activity quests (completed → **claimable** in the UI).

## Quest inventory

| Quest ID | Kind | Title (summary) | Reward PP | When progress advances |
|----------|------|-----------------|----------:|------------------------|
| `onb-account` | `onb_account` | Create account | 50 | Successful **Google** signup (new user insert). |
| `onb-referral-code` | `onb_referral_code` | Referral link ready | 25 | At signup when a code is assigned, **or** later when `ensureUserReferralCodeTx` first assigns a code (legacy users). |
| `onb-display-name` | `onb_display_name` | Display name | 40 | `PATCH /profile` with a **new** display name (length ≥ 2, not equal to previous). |
| `onb-avatar` | `onb_avatar` | Profile photo | 40 | **`PATCH /profile`** (first non-empty avatar) or **`POST /profile/avatar/upload`** (first photo). |
| `onb-first-pack` | `onb_first_pack` | First paid pack/slab | 100 | First **paid** pack open or slab roll: `packsOpenedBefore === 0` and `totalCents > 0`. |
| `onb-buyback` | `onb_buyback` | Buyback | 75 | Inventory item moves to **buyback / cashed_out** flow (same moment as `buyback_complete` advance). |
| `onb-referral-signup` | `onb_referral_signup` | Refer a signup | 150 | Another user completes signup with your referral attached (`tryAttachReferrerTx`). |
| `onb-referral-converted` | `onb_referral_converted` | Referral’s first paid open | 200 | Referred user’s first paid pack/slab (runs with referrer payout logic). |
| `onb-discord` | `onb_discord` | Link Discord | 35 | Discord OAuth callback succeeds and link is stored. |
| `onb-x-verified` | `onb_x_verified` | Verify X | 50 | X verify **confirm** path succeeds. |

Definitions are seeded in `backend/drizzle/0019_onboarding_quests.sql` (`category = 'onboarding'`).

## Product mapping notes

- **“Username”** in product copy maps to **referral code + shareable link** (`onb_referral_code`) plus **display name** (`onb_display_name`), not a separate username column.
- **First pack** is intentionally **paid spend only** (credits/money path). Free or zero-cost opens do not complete this step—confirm with product if wallet-only promos should count.
- **Existing users** are not backfilled: they earn onboarding progress when the relevant events fire **after** deploy + migration.

## API / UI surfacing

- **`GET /gamification/overview`** — includes quests where `timeframe` is daily/weekly/monthly **or** `category === 'onboarding'`. Each quest row includes `category`, `claimable`, etc.
- **`GET /meta/quests`** — public catalogue with the same filter; **onboarding** sorts first.
- **Claim** — same activity-claim handler as rolling quests; `claimActivityQuestRewardTx` supports `permanent` timeframe with period key `all-time` (see `backend/src/lib/gamification.ts`).

### Profile responses & “task complete” UX

- **`POST /profile/avatar/upload`** and **`PATCH /profile`** may include **`taskCompletions`**: an array of quests that **just finished** (progress reached target — PP is still claimed separately unless you add auto-claim). Each item has `id`, `title`, `rewardPoints`, `kind`, `category`. The SPA uses this for a **completion chime + toast** (distinct from the shorter **claim** sound on `POST /gamification/quests/:id/claim`).
- **`advanceQuestsByKindTx`** returns these hints whenever progress crosses from incomplete → complete (including recurring quests, though Profile only surfaces the payload today).

Frontend surfaces onboarding first in the notification bell and Pocket Points dashboard (`questGroups` / section ordering).

## Economy / duplicates

- **`tryAwardProfileCompletionTx`** may still award separate **profile completion** PP when name + avatar rules pass. That can **stack** with `onb_display_name` and `onb_avatar` claims. Align copy with finance: e.g. zero `profile_complete_pp`, or treat profile completion as narrative only.

## Operational checklist

1. Run migration **`0019_onboarding_quests`** on each environment.
2. Smoke-test: signup → overview shows `onb_account` / `onb_referral_code` progress; profile patch → name/avatar; paid open → `onb_first_pack`; referral flows on staging.
3. Optional: dedicated **onboarding checklist** page (same data as overview quests).

## Extending

Add a row to `quest_definitions` with `category = 'onboarding'`, `timeframe = 'permanent'`, a unique `kind`, then call **`advanceQuestsByKindTx`** from the appropriate route or `lib` transaction when the event occurs. Avoid circular imports (`referrals.ts` → `gamification.ts`; keep `gamification` from importing `referrals`).
