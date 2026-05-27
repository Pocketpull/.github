# Onboarding quests

One-time (`timeframe: permanent`) quests that advance from real product events. Users **claim** PP the same way as daily/weekly/monthly quests.

**Full reference:** [onboarding-points-system.md](onboarding-points-system.md)

## Quest inventory

| Quest ID | Reward PP | Trigger (summary) |
| -------- | --------- | ----------------- |
| `onb-account` | 50 | New user on **Google** signup |
| `onb-referral-code` | 25 | Referral code assigned at signup or backfill |
| `onb-display-name` | 40 | First `PATCH /profile` display name (≥2 chars) |
| `onb-avatar` | 40 | First avatar via profile patch or upload |
| `onb-first-pack` | 100 | First **paid** pack or slab open |
| `onb-buyback` | 75 | First buyback / cash-out |
| `onb-referral-signup` | 150 | Referred user signs up with your code |
| `onb-referral-converted` | 200 | Referred user’s first paid open |
| `onb-discord` | 35 | Discord OAuth link success |
| `onb-x-verified` | 50 | X tweet-and-code verification confirm |

Seeded in `backend/drizzle/0019_onboarding_quests.sql`.

> **Note:** The legacy onboarding doc mentions email/password signup; production auth is **Google OAuth only** — `onb-account` fires on Google new-user insert.

## API

| Method | Path |
| ------ | ---- |
| GET | `/gamification/overview` | Includes onboarding category |
| GET | `/meta/quests` | Public catalogue (onboarding sorts first) |
| POST | `/gamification/quests/:id/claim` | Claim PP |

Profile endpoints may return `taskCompletions` on avatar/name update for UI toasts.

## Ops checklist

1. Migration `0019_onboarding_quests` on each environment  
2. Smoke: signup → overview; profile → name/avatar; paid open → `onb_first_pack`  
3. Watch duplicate PP with `profile_complete` awards (see full doc)

## Code

- `backend/src/lib/gamification.ts` — `advanceQuestsByKindTx`, claim handlers  
- Frontend: Rewards page, notification bell ordering
