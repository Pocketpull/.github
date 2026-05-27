# Binder (client-controlled) — implementation strategy

## Implementation status (repo)

**Shipped (P0):** Drizzle tables `collection_quests`, `collection_quest_requirements`, `user_collection_quest_progress` (migration `0007_collection_quests.sql`); shared ownership helper [`backend/src/lib/binder-inventory.ts`](backend/src/lib/binder-inventory.ts); catalogue [`GET /catalogue/pokemon/names`](backend/src/routes/pokemon-catalogue.ts) and [`GET /catalogue/pokemon/cards-by-name`](backend/src/routes/pokemon-catalogue.ts); evaluation [`backend/src/lib/collection-quests-eval.ts`](backend/src/lib/collection-quests-eval.ts); sync + PP grant [`backend/src/lib/collection-quests-sync.ts`](backend/src/lib/collection-quests-sync.ts) (hooked after credit pack + slab inventory inserts); player [`GET /collection-quests`](backend/src/routes/collection-quests.ts) and [`GET /collection-quests/:slug`](backend/src/routes/collection-quests.ts); admin CRUD under `/admin/collection-quests/*`; **platform admins** via `users.is_admin` (migration `0008_is_admin_users.sql`, seed UPDATE for initial team emails); [`GET /admin/users`](backend/src/routes/admin-users.ts) + [`PATCH /admin/users/:userId`](backend/src/routes/admin-users.ts) `{ isAdmin }`; [`isAdmin` on `/auth/me`](backend/src/routes/register.ts) reads DB; frontend Binder **Collection quests** link, [`/binder/pokemon/quests`](frontend/src/pages/PokemonBinderQuestsPage.tsx), detail page, and [`/admin/collection-quests`](frontend/src/pages/AdminCollectionQuestsPage.tsx) (includes admin user list).

**Not yet (P1+):** `rule` mode (type/set/dex JSON), optional `indexes/cards/by-name.json` in R2, merging collection quests into `/gamification/overview`, versioning.

---

This document turns the agreed product direction into an engineering plan: **clients author collection-style quests and themed sets** (TCG-inspired: starters, type themes, curated packs) **without shipping new code for every quest**, while **Binder stays the catalog surface** and **inventory stays the source of “owned cards.”**

---

## 1. Terminology (lock these in)

| Term | Meaning in Pocketpull |
|------|------------------------|
| **Inventory** | The user’s vault: concrete items they own (`inventoryItems`, duplicates, provenance, etc.). Operational source of truth for ownership. |
| **Binder** | The **catalog / collection UX**: browsing by set, checklists, progress. Not a second copy of “all Pokémon”; it’s organization + presentation over the same card IDs the app already indexes. |
| **Card index** | Server-side lookup (e.g. `pokemonTcgCardId` → set, name, type, dex, …) used today by binder summary routes. Quests must resolve against **these IDs**, not free-text names. |
| **Quest** | A client-authored challenge: “collect this roster / satisfy these rules / reach a threshold,” usually with rewards (e.g. PP per `rewards.md`). |
| **Quest set / theme** | A named grouping (e.g. “Starter Set 1”, “Forest friends”) defined in admin as explicit card lists and/or rule blocks—not a duplicate master catalog of species. |

**Important:** Binder progress today is **derived from inventory** (e.g. scanning owned `pokemonTcgCardId`s). Quest completion should use the **same resolution path** (inventory → card IDs → rules), optionally with **filters** (e.g. only items from Slab opens) as a product flag—not a parallel “quest inventory.”

---

## 2. Goals

1. **Client-operable:** Non-engineers can create, edit, publish, and retire quests and themed lists.
2. **TCG-style variety:** Support fixed rosters (e.g. Bulbasaur, Charmander, Squirtle, Pikachu), rule-based buckets (e.g. Bug-type, “forest” tag), hybrids, and “collect any N matching” thresholds.
3. **Consistent with Binder:** Quests reference the **same** `pokemonTcgCardId` universe the Binder uses; no drifting second catalog.
4. **Rewardable:** Completion ties into gamification (ledger reasons, idempotent grants).
5. **Safe:** Admin-only APIs; audit-friendly changes; optional draft → publish.

---

## 3. Design principles

1. **Author sets, don’t re-enter Pokémon.** Admin UX is “search catalog → add to quest” and “add rule block,” not “type Bulbasaur from scratch.”
2. **Explicit IDs win for curated quests.** Themed starter packs should be **lists of `pokemonTcgCardId`** (or stable internal IDs that map 1:1 to index entries). Rule-based quests use metadata on the index (type, set, dex range, **custom tags** if the base API lacks “habitat”).
3. **Single evaluation pipeline.** One function conceptually: `ownedCardIdsForUser(userId, options)` × `questDefinition` → progress / complete. Binder summary and quest worker both benefit from shared helpers.
4. **Version carefully.** Changing a live quest’s required cards mid-flight is unfair; prefer **immutable published versions** or **new quest revision** with clear cutover.

---

## 4. Suggested data model (high level)

Exact naming can follow your Drizzle conventions; shape matters more than table names.

### 4.1 Quest definition (admin-authored)

- **`quests`** (or `binder_quests`): `id`, `slug`, `title`, `description`, `status` (`draft` | `published` | `archived`), `rewardPocketPoints` (nullable or FK to reward table), `startsAt` / `endsAt` (nullable), `sortOrder`, `createdAt`, `updatedAt`, `publishedAt`, `createdByUserId` (optional).
- **`quest_criteria`** (ordered steps OR parallel requirements—pick one product model early):
  - **Type A — Explicit cards:** `quest_id`, `kind = card_list`, ordered list of `pokemonTcgCard_id`, optional `min_count` per card (usually 1 for unique).
  - **Type B — Rule:** `quest_id`, `kind = filter`, JSON payload, e.g. `{ "types": ["Grass"], "setSlugs": ["sv1"], "dexMin": 1, "dexMax": 151, "tags": ["forest"] }`, plus `targetUniqueCount` (e.g. collect any 10 matching).
  - **Type C — Hybrid:** multiple rows; completion = all rows satisfied (AND) or any (OR)—**define in product**.

### 4.2 User progress

- **`user_quest_progress`:** `user_id`, `quest_id`, `status` (`in_progress` | `completed`), `completed_at`, `last_evaluated_at`, unique `(user_id, quest_id)`.
- Optional detail table **`user_quest_criterion_progress`** if you need per-step UI (cached counts, which cards matched).

### 4.3 Catalog enrichment (only if rule-based quests need it)

If “forest” or “insect collector” can’t be expressed with existing index fields:

- **`card_tags`** or JSON on index build: `pokemonTcgCardId` → `tags[]` maintained via **bulk import** or a thin “tag editor” in admin (still keyed by ID, not by retyping species).

Avoid storing full card metadata in quest tables; **reference the index**.

---

## 5. Admin experience (MVP → full)

### MVP

- List quests (draft/published).
- Create/edit quest: title, copy, reward, schedule.
- **Add cards:** search against existing card index API (by name, set, ID); multi-select → saves criterion row(s).
- **Add simple rule:** type + optional set + dex range + “need N unique matches.”
- Publish / unpublish.
- **Preview completion** for a given `user_id` (internal tool) or % formula explanation.

### Later

- Duplicate quest, revision history, scheduling campaigns, localization, art assets, featured ordering on Binder home.
- Bulk import: CSV of `pokemonTcgCardId` per quest slug.

---

## 6. Backend API shape (illustrative)

Split **public/player** vs **admin** (admin guarded by role or separate secret + allowlist).

**Player (authenticated)**

- `GET /quests` — list published quests + user progress summary.
- `GET /quests/:slug` — detail + progress for current user.
- `POST /quests/:slug/claim` (optional) — if completion is not auto-granted; or use **event-driven evaluation** after inventory changes.

**Admin**

- `GET/POST/PATCH /admin/quests`, `.../criteria`, publish endpoints.
- `GET /admin/catalog/search?q=` — search card index for authoring (reuse same source as binder).

**Evaluation**

- On **read** (lazy): computing progress when user opens Binder/Quests tab.
- On **write** (eager): after pack open / inventory mutation, enqueue “reevaluate active quests for user” (optional optimization for large quest catalogs).

Idempotent reward grant on first transition to `completed` (same pattern as other gamification awards).

---

## 7. Frontend surfaces

- **Binder:** Dedicated “Quests” or “Challenges” area that feels continuous with set browsing (shared card art, shared IDs).
- **Admin:** New route group, e.g. `/admin/binder-quests` (protected). Keep UI boring-crisp: tables + searchable card picker + rule builder.

---

## 8. Security and operations

- **Authorization:** `isAdmin` (or role claim) on all `/admin/*` quest routes; never expose draft quests publicly.
- **Validation:** Reject criteria that reference unknown `pokemonTcgCardId` at publish time (or warn + block).
- **Audit:** Log publish/unpublish and reward grants.

---

## 9. Phased rollout

| Phase | Outcome |
|-------|---------|
| **P0** | Schema + admin CRUD for quests with **explicit card lists** only; player list + detail; progress = intersection of inventory IDs with list; PP grant on complete. |
| **P1** | Rule-based criteria + index fields; tag support if needed. |
| **P2** | Eager re-evaluation hooks, analytics, versioning, bulk import, richer Binder UX. |

---

## 10. Product decisions to confirm before build

1. **Completion scope:** Any inventory card counts, or only cards from **Slab / in-app packs** (provenance filter)?
2. **Duplicates:** Quests count **unique** `pokemonTcgCardId` only, or stacks?
3. **Retroactivity:** If a user already owns all cards when a quest publishes, auto-complete or require “claim” action?
4. **Rewards:** Fixed PP per quest vs scaling; alignment with `rewards.md` quest table.

---

## 11. Relation to current codebase

- **Today:** `GET /binder/pokemon/summary` aggregates owned IDs from `inventoryItems` against `loadPokemonCardIndexById()`.
- **Direction:** Extract shared helpers (owned TCG IDs, optional provenance filter) and use them for **both** binder set progress and **quest evaluation**, so Binder and quests never disagree on what the user “has.”

---

*This strategy assumes the Binder remains the player-facing catalog and inventory remains ownership truth; the client controls **quest content** via admin, not a second Pokémon database.*
