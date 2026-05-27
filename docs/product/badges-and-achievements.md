# Badges & achievements

## Badge catalogue

**[badges.md](../reference/badges.md)** — complete reference for **49** ship-ready badges:

- 44 quests + 3 legacy milestones + 2 achievements  
- IDs match database for 1:1 asset wiring  
- Display rules: locked (grey), complete (category color), recurring reset  
- Asset naming: `badges/{id}-locked.svg`, `badges/{id}-unlocked.svg`

### Category families (accent colors)

| Family | Accent | Examples |
| ------ | ------ | -------- |
| Onboarding | Emerald `#34d399` | First login, link wallet |
| Daily | Sky `#5b9fff` | Daily login streaks |
| Weekly / Monthly | Purple / gold variants | Period quests |
| Slab / Pack | Brand blues | Open counts, big hits |
| Social | Pink / twitter tones | Share, referral |
| Collection | Green | Binder, type milestones |

## Achievements (Masterdoc §23)

Milestone families separate from recurring quests:

- **Pokemon type** — collect N Grass/Water/Fire/Electric  
- **Rarity** — pull counts by rarity tier  
- **Volume** — spend, opens, marketplace actions  

Rewards grant PP and/or badge unlocks. Meta endpoints:

- `GET /meta/achievements`
- `GET /meta/milestones`

## UI routes

- `/badges` — badge grid, progress, claim  
- `/rewards` — overlaps quests + shop + tier  

## Future catalogue

Masterdoc references **200+** future badge tiers not yet in DB — track in a dedicated `badges-future.md` when product defines them (see note in badges.md).

## Engineering

- Badge progress: gamification overview + quest claim endpoints  
- Admin CRUD: `/admin/rewards/achievements`, `/admin/rewards/milestones`
