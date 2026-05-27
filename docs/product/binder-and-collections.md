# Binder & collections

**Collections** is the parent concept: **Inventory** (owned cards) + **Binder** (set completion with missing slots visible but locked).

## Routes

| Route | Purpose |
| ----- | ------- |
| `/vault` | Inventory (canonical) |
| `/binder` | Pokémon binder home |
| `/bindernew` | Binder UI experiments |

## Binder model (Masterdoc §19)

- **Set binders** — primary focus (e.g. original 151 / product-defined ranges)
- **Spread view** — collected vs empty slots
- **Cover cards** — user-selected showcase
- **Future categories** — holo sets, type sets, character sets

### Example product rule

Cards from **Slab Roll** pulls fill dex ranges (e.g. Pokémon 1–100) — exact mapping TBD with catalogue + inventory APIs.

## Backend

| Endpoint | Module |
| -------- | ------ |
| `GET /binder/pokemon/summary` | `binder-pokemon.ts` |
| `GET /collection-quests` | `collection-quests.ts` |
| `GET /collection-quests/:slug` | |
| Admin `/admin/collection-quests/*` | `collection-quests-admin.ts` |

## Pokémon catalogue

Binder slots reference catalogue data:

- `GET /catalogue/pokemon/sets`
- `GET /catalogue/pokemon/sets/:setSlug/cards`
- `GET /catalogue/pokemon/card/:cardId`

Archive sync: `npm run pokemon:sync` (backend).

## Engineering plan

**[binder-implementation-strategy.md](binder-implementation-strategy.md)**

| Phase | Status |
| ----- | ------ |
| P0 — terminology, routes, summary API | Shipped / partial |
| P1 — rule mode, overview merge, quest wiring | Backlog |

## Implementation status

From [client-notes-round-3.md](../changelog/client-notes-round-3.md):

| Item | Status |
| ---- | ------ |
| Binder backend + data sources + quests | **Not done** |
| Clarify owned vs empty slot data source | Open |

## Rewards tie-in

Binder completion milestones grant PP (see [rewards.md](../reference/rewards.md) — binder pages, Pokédex set, variety). Collection quests may grant badges — see [badges-and-achievements.md](badges-and-achievements.md).

## Inventory API

- `GET /inventory` — owned items  
- `PATCH /inventory/items/:id` — metadata / flags  

CC wallet inventory parity is a separate concern from binder slots — see [collector-crypt-gacha.md](../integrations/collector-crypt-gacha.md).
