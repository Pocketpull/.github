# Client notes — round 3

Working spec from product feedback. Implement against **`frontend/`** + **`backend/`** only (Vite SPA + Fastify API).

---

## Implementation log (status)

| Area | Item | Status |
|------|------|--------|
| Home | **Card images** + `imageUrl` API | **Done** |
| Home | **Smaller strip** (~half prior width), **no heavy boxes** — image + title + **$** only | **Done** (`home-recent-hits.tsx`) |
| Home | **Auto carousel** (infinite CSS marquee), **no drag**; **New** / **Top $** toggle (sorts same feed) | **Done** |
| Home | **Demo note** — “Demo feed · live ships real-time only” + `sr-only` line | **Done** |
| Home | **Remove top banner** (`SitePullsTicker`) | **Done** — removed from `app-shell.tsx`; main `md:pt-16` only |
| Home | **Feed size** — up to **150** merged pulls, inventory/CC limits **100** each | **Done** (`register.ts`) |
| Home | **Hero / copy** — tighter dopamine lines on `Home.tsx` | **Done** |
| Slab Roll | Pool card price + title alignment / row uniformity | **Done** (earlier — `slab-pool-grid.tsx`) |
| Slab Roll | **Pool data vs CC** — live `getNfts` as source of truth (optional short TTL) | **Done / accepted** — best fit for identical packs; R2 sync is auxiliary |
| Slab Roll | CC inventory UI parity + hard refresh / polling (separate from pool API) | **Not done** (skipped per direction) |
| Binder | Backend + data sources + quests | **Not done** |
| Referrals | Attribution (code, link, signup attach, claim) | **Done** — see **Referrals** section below |
| Referrals | **1,000 PP** to referrer on referee’s **first** credit pack or slab open (`referral_first_pack:*` ledger) | **Done** (`gamification.ts` + `register.ts`) |
| Referrals | **Block** attaching a code **after** first pack purchase | **Done** (`referrals.ts`, `/referrals/claim` 403, `Profile.tsx` hides claim when `packsOpened > 0`) |

### Done — home strip (technical)

- **`backend`** `GET /activity/recent-pulls`: `imageUrl` sources unchanged; response cap **150** rows; larger inventory + platform-event pulls.
- **`frontend`** `HomeRecentHitsStrip`: “On the board”, **New** / **Top $**, duplicated marquee track (`home-pulls-marquee` in `index.css`), `prefers-reduced-motion` disables animation.
- **`SitePullsTicker`**: not mounted globally (file kept for reuse).

---

## Slab Roll page

### Pool preview data (`GET /gacha/machines/:id/pool`)

- **Decision (accepted):** **Live Collector Crypt `getNfts`** (optional in-process TTL via `GACHA_POOL_LIVE_CACHE_TTL_MS`; default **0** = always fresh). **No R2 snapshot as primary read** — best way to keep slab pool tiles aligned with CC. Scheduled **R2 sync** remains useful for snapshots/CDN/ops, not as the canonical “what’s in the pack right now” source.

### Card inventory vs Collector Crypt (wallet / vault UI)

- **Issue:** Card inventory shown in-app may not match CC (separate from pool `getNfts`).
- **Direction:** Still open if you want fewer polls / hard refresh on **inventory** only.

### Performance — hard refresh + API churn

- **Issue:** Card inventory area constantly pings APIs and updates, causing heavy lag.
- **Direction:** Implement a **hard refresh** model for card inventory (explicit user-triggered or controlled refresh) instead of continuous polling / constant updates. Reduce background update frequency so the UI stays responsive.

---

## Home page

### Packs Pulled / Top pack pulls (recent pulls strip)

- [x] **Card images** — wired end-to-end.
- [x] **Smaller cards**, minimal chrome — image, **title**, **$** (no actor row on the strip).
- [x] **Infinite auto marquee** (no manual drag); **New** vs **Top $** sort on the same pooled data.
- [x] **Demo** label — live product will restrict to real-time pulls only.

### Top banner

- [x] **Removed** — `SitePullsTicker` no longer in `AppShell`.

---

## Binder

### Backend integration

- Wire Binder to the **backend** so it behaves as previously discussed (persistent progress, server-backed state where appropriate).

### Data source for binder slots

- **Clarify and implement** where data comes from to **fill binder cards** (e.g. which API/DB fields map to “owned” vs “empty”).
- **Example rule (product):** e.g. *Pokémon 1–100* filled by cards obtained from **Slab Roll** pulls — define exact mapping (set, dex range, or card IDs).

### Reference list

- A full list of Pokémon (e.g. 1–151 or “original 100” as product defines) may be used to **seed or validate** binder rows; organize data so **quests** can target:
  - e.g. “Collect all 100 original Pokémon”
  - e.g. “Collect 50 Water-type”  
  (Quest system builds on consistent binder + type metadata.)

---

## Referral system

### What’s already shipped (attribution + purchase rewards)

| Piece | Status |
|-------|--------|
| **DB** — `users.referral_code`, `referred_by_user_id`, `referrals` table (referrer → referred) | Done (`schema.ts`; comment: attribution only, no rewards in table) |
| **Codes** — generated on email signup; `ensureUserReferralCodeTx` on `/auth/me` if missing | Done (`referrals.ts`, `register.ts`) |
| **Signup attach** — `POST /auth/register` body may include `referralCode`; `tryAttachReferrerTx` | Done |
| **Login link** — `/login?ref=` or `?invite=` stores pending code (`referral-pending.ts`) and sends on **Register** | Done |
| **Signup form** — optional referral **text field** on `Login.tsx` (overrides / complements URL + pending) | Done |
| **Public validate** — `GET /referrals/preview?code=` | Done |
| **Session** — `GET /referrals/me` returns `referralCode`, `referralLink` (`WEB_ORIGIN/login?ref=…`), `referralCount`, `referredBy` | Done |
| **Post-signup claim** — `POST /referrals/claim` with `{ referralCode }` | Done |
| **Profile UI** — shows code, link, copy, count, “Referred by …” | Done (`Profile.tsx`) |
| **Referrer PP** — **1,000 PP** when referee completes **first** pack/slab open (`packsOpened` was 0) | Done |

### Purchase PP (rewards.md §3) — Slab-roll + credit packs

- **10 PP per $1** on spend was already applied via `grantPackOpenGamification` when `spendCents` is set.
- **Slab-roll** opens now log **`slab_roll_purchase`**; credit `/packs` opens still log **`pack_open`** (unchanged reason string for analytics continuity).

### Deferred / optional (not MVP)

| Piece | Status |
|-------|--------|
| **Anti-abuse** — chargeback hold, delayed **1,000 PP** release | **Not done** (see Open questions) |

### User-facing behavior (original spec)

- Users can obtain a **referral code** and/or **referral link**. — **Done**
- **Signup:** Enter code in a field **or** link pre-fill. — **Done** (`Login.tsx` field + `?ref=` / `?invite=` + Profile claim for Google / missed link)
- **Reward:** Referrer earns **1,000 PP** on referee’s **first** credit pack or slab open. — **Done** (MVP immediate grant; no fraud hold yet)
- **Constraint:** No code **after** first pack (`packsOpened > 0`). — **Done** (`tryAttachReferrerTx` + **403** on claim with clear `message`; Profile hides claim UI when locked)

---

## Open questions (resolve during implementation)

1. ~~Slab Roll pool~~: live `getNfts` — **resolved** (see Slab section).
2. Home “Top Packs” vs “Recent Hits”: definitions (time window, sort key, same feed filtered differently vs two APIs).
3. Binder: confirm dex range (“original 100” vs 151), and whether Slab-only vs all pull sources count toward slots.
4. Referrals: anti-abuse / chargeback hold window; optional delay before releasing **1,000 PP**.

---

*Last updated: referral signup field on Login, post–first-pack lock (`packsOpened`), claim UX + API message aligned.*
