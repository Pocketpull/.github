# PocketPull Badge Catalogue

Complete reference for all **49** ship-ready badges (44 quests + 3 legacy milestones + 2 achievements).

Use this doc for art review, UI implementation, and asset naming. IDs match the database so engineering can wire images 1:1.

---

## Display rules (UI)

| State | Badge | Label underneath |
|-------|-------|------------------|
| **Locked / in progress** | Grey medallion, no category color | `progress / target` (e.g. `3/10`) |
| **Complete** | Full-color medallion per category | `target / target` or checkmark |
| **Hover** | Tooltip / popover | Name, description, PP reward, claim button if claimable |

**Recurring badges** (daily / weekly / monthly) reset each period — grey again until re-earned.

---

## Shared badge shape (v1 — basic)

One reusable template for all badges. Swap icon + accent color per item.

```
        ___
       /   \
      |  ◆  |   ← simple hex or shield medallion (~64×64 display)
       \___/
        3/10     ← progress text below (not inside badge)
```

| Layer | Locked | Unlocked |
|-------|--------|----------|
| Fill | `#64748b` (slate-500) flat | Category accent (see families below) |
| Border | `#475569` 1px | Lighter tint of accent + subtle glow |
| Icon | White @ 40% opacity | White or dark @ full opacity |
| Ribbon (optional) | Hidden | Small bottom banner with target number for count badges |

**Asset naming:** `badges/{id}-locked.svg` and `badges/{id}-unlocked.svg`  
**Fallback (no art yet):** CSS medallion + Lucide icon from design column.

---

## Category color palette

| Family | Accent (unlocked) | Icon motif |
|--------|-------------------|------------|
| Onboarding | Emerald `#34d399` | Compass / check / link |
| Daily | Sky `#5b9fff` | Sun / calendar dot |
| Weekly | Violet `#a78bfa` | Flag / 7-day arc |
| Monthly | Amber `#E8C77A` | Moon / star ring |
| Pack opens | Gold `#f59e0b` | Pack box + number |
| Login streak | Orange `#fb923c` | Flame + day count |
| Shipping | Teal `#2dd4bf` | Package / truck |
| Marketplace | Purple `#c084fc` | Tag / handshake |
| Binder — variety | Blue `#60a5fa` | Grid of cards |
| Binder — type | Type color (see each) | Type symbol leaf/drop/flame/bolt |
| Binder — Kanto | Red `#ef4444` + blue rim | Pokéball silhouette |
| Binder — pages | Tan `#d97706` | 3×3 binder grid |
| Legacy milestone | Neutral gold `#E8C77A` | Trophy |
| Achievement | Legendary gold `#E8C77A` | Star / shield |

---

## 1. Onboarding (10)

One-time badges. Emerald family. Shape: shield with a small “START” notch at top for the first badge only.

| ID | Name | Description | Target | PP | Design idea |
|----|------|-------------|--------|-----|-------------|
| `onb-account` | **Start** | Sign up for PocketPull (email/password or Google). | 1 | 50 | Shield with open door or keyhole. Label ribbon: “START”. Simplest badge — flat emerald fill when unlocked. |
| `onb-referral-code` | **Link** | Your unique referral code was issued — share it from Profile or Rewards. | 1 | 25 | Chain link icon centered. Tiny copy icon in corner. |
| `onb-display-name` | **Name** | Choose how you appear on leaderboards and activity (update Profile). | 1 | 40 | Stylized “Aa” or name tag silhouette. |
| `onb-avatar` | **Photo** | Upload an avatar in Profile settings. | 1 | 40 | Circle frame with generic portrait silhouette (not user photo). |
| `onb-first-pack` | **Open** | Complete any paid credit pack open or slab roll. | 1 | 100 | Pack box with burst lines — first-open energy. Slightly larger icon than other onboarding badges. |
| `onb-buyback` | **Buyback** | Cash out a vaulted item through buyback once. | 1 | 75 | Circular arrows (cash-out) around a card outline. |
| `onb-referral-signup` | **Refer** | Someone signed up with your referral code. | 1 | 150 | Two person silhouettes + plus sign. |
| `onb-referral-converted` | **Convert** | A referred user completed their first paid pack or slab open. | 1 | 200 | Refer icon + small pack box with checkmark overlay. |
| `onb-discord` | **Discord** | Connect your Discord account from Profile. | 1 | 35 | Simplified game-controller / chat bubble (no Discord trademark — abstract “community” bubble). |
| `onb-x-verified` | **X** | Link and verify X (Twitter) from Profile. | 1 | 50 | Abstract “X” cross or bird silhouette (generic, not brand logo). |

---

## 2. Daily activities (2)

Reset every UTC day. Sky blue family. Small sun arc at top of medallion.

| ID | Name | Description | Target | PP | Design idea |
|----|------|-------------|--------|-----|-------------|
| `act-daily-login` | **Daily** | Log in once today. | 1 | 25 | Sun rising over horizon line. Single ray accent. |
| `act-daily-share` | **Share** | Share an earned card or reveal moment today. | 1 | 25 | Share arrow / megaphone + tiny card corner peeking out. |

---

## 3. Weekly activities (2)

Reset each UTC week (Monday start). Violet family. Small flag pennant on rim.

| ID | Name | Description | Target | PP | Design idea |
|----|------|-------------|--------|-----|-------------|
| `act-weekly-open-3` | **3 Opens** | Open three paid pack or slab-rolls during the weekly tournament window. | 3 | 250 | Three stacked pack silhouettes. Ribbon: “3”. |
| `act-weekly-streak-3` | **3-Day** | Log in on three consecutive UTC days this week. | 3 | 150 | Three flame dots in a row (mini streak). |

---

## 4. Monthly activities (3)

Reset each UTC month. Amber/gold family. Ring of 12 dots around rim (calendar metaphor).

| ID | Name | Description | Target | PP | Design idea |
|----|------|-------------|--------|-----|-------------|
| `act-monthly-open-25` | **25 Opens** | Open twenty-five paid pack or slab-rolls during the monthly race. | 25 | 2000 | Pack stack with bold “25” ribbon. Heavier gold border when unlocked. |
| `act-monthly-referral-1` | **Referral** | Refer one user who opens their first paid pack this month. | 1 | 1000 | Handshake + star. Premium feel — double ring border. |
| `act-monthly-share-5` | **5 Shares** | Share five earned card or reveal moments this month. | 5 | 250 | Five small share arrows arranged in fan. Ribbon: “5”. |

---

## 5. Pack open milestones (5)

Permanent. Gold family. Same base pack icon; number ribbon scales up.

| ID | Name | Description | Target | PP | Design idea |
|----|------|-------------|--------|-----|-------------|
| `q-pack-opens-10` | **10-Pack** | Ten paid credit or slab opens (free opens do not count). | 10 | 250 | Bronze-tier pack badge. Single pack + “10”. |
| `q-pack-opens-25` | **25-Pack** | Twenty-five paid opens — slabs count. | 25 | 500 | Silver-tier pack badge. Slight shine line on shield. |
| `q-pack-opens-50` | **50-Pack** | Fifty paid opens. | 50 | 1000 | Gold pack badge. Two pack silhouettes offset. |
| `q-pack-opens-100` | **100-Pack** | One hundred paid opens. | 100 | 2000 | Platinum pack badge. Star behind pack icon. |
| `q-pack-opens-250` | **250-Pack** | Two hundred fifty paid opens. | 250 | 5000 | Diamond pack badge. Crown notch at top + “250” banner. Highest pack tier visually. |

---

## 6. Login streak milestones (2)

Permanent. Orange flame family.

| ID | Name | Description | Target | PP | Design idea |
|----|------|-------------|--------|-----|-------------|
| `q-login-streak-7` | **Week Warrior** | Log in on seven consecutive UTC calendar days. | 7 | 75 | Single flame with “7” inside. Warm orange gradient fill. |
| `q-login-streak-30` | **Month Master** | Thirty consecutive UTC calendar days with a login. | 30 | 300 | Triple flame stack + “30”. Deeper orange-to-red gradient. |

---

## 7. Shipping & marketplace (3)

Permanent first-action badges.

| ID | Name | Description | Target | PP | Design idea |
|----|------|-------------|--------|-----|-------------|
| `q-first-shipment` | **First Class** | Mark a credit inventory item as shipped. | 1 | 300 | Teal package with wings (mail speed). Shipping family. |
| `q-first-marketplace-list` | **Seller** | List your first marketplace item. | 1 | 150 | Purple price tag hanging from card corner. |
| `q-first-marketplace-sell` | **Sold** | Complete your first marketplace sale (settled, amount > 0). | 1 | 300 | Sold stamp circle overlay on card icon. Green check in stamp. |

---

## 8. Binder — variety (3)

Permanent. Blue family. Scattered card backs motif.

| ID | Name | Description | Target | PP | Design idea |
|----|------|-------------|--------|-----|-------------|
| `q-binder-unique-25` | **Variety 25** | Distinct Pokémon TCG cards in your vault (any type). | 25 | 250 | 2×2 mini card grid (4 dots) + “25” ribbon. |
| `q-binder-unique-50` | **Variety 50** | Fifty distinct cards in your vault. | 50 | 750 | 3×3 mini card grid + “50”. |
| `q-binder-unique-100` | **Variety 100** | One hundred distinct cards in your vault. | 100 | 2000 | Full binder page grid (9 squares) + “100”. Strongest variety badge. |

---

## 9. Binder — type collection (8)

Permanent. Each type uses Pokémon TCG type color when unlocked.

| ID | Name | Description | Target | PP | Design idea |
|----|------|-------------|--------|-----|-------------|
| `q-binder-grass-10` | **Grass 10** | Owned cards whose types include Grass (TCG types). | 10 | 100 | Green leaf icon. Locked: grey leaf outline. |
| `q-binder-grass-20` | **Grass 20** | Twenty Grass-type cards in your vault. | 20 | 250 | Same leaf + double leaves + “20” ribbon. Richer green. |
| `q-binder-water-10` | **Water 10** | Owned cards whose types include Water. | 10 | 100 | Blue water drop. |
| `q-binder-water-20` | **Water 20** | Twenty Water-type cards in your vault. | 20 | 250 | Double drops / splash + “20”. |
| `q-binder-fire-10` | **Fire 10** | Owned cards whose types include Fire. | 10 | 100 | Red flame teardrop. |
| `q-binder-fire-20` | **Fire 20** | Twenty Fire-type cards in your vault. | 20 | 250 | Larger flame + “20”. |
| `q-binder-electric-10` | **Electric 10** | Includes Lightning-type TCG cards. | 10 | 100 | Yellow lightning bolt. |
| `q-binder-electric-20` | **Electric 20** | Twenty Electric / Lightning cards in your vault. | 20 | 250 | Double bolt + “20”. |

**Type color hex (unlocked):** Grass `#78C850` · Water `#6890F0` · Fire `#F08030` · Electric `#F8D030`

---

## 10. Binder — Kanto dex (3)

Permanent. Red badge with blue outer ring (classic palette). Silhouette of Kanto map or Pokéball, no copyrighted art.

| ID | Name | Description | Target | PP | Design idea |
|----|------|-------------|--------|-----|-------------|
| `q-binder-kanto-25` | **Kanto 25** | 25 distinct species #1–151 by national dex on the card. | 25 | 150 | Pokéball half-filled (¼ progress metaphor) + “25”. |
| `q-binder-kanto-50` | **Kanto 50** | 50 distinct Kanto dex species in your vault. | 50 | 350 | Pokéball half-filled + “50”. |
| `q-binder-kanto-100` | **Kanto 100** | 100 distinct Kanto dex species in your vault. | 100 | 1000 | Full Pokéball + star burst. “100” banner. |

---

## 11. Binder — page completion (3)

Permanent. Tan/brown binder family. 3×3 grid icon.

| ID | Name | Description | Target | PP | Design idea |
|----|------|-------------|--------|-----|-------------|
| `q-binder-page-1` | **Page 1** | Nine unique Pokémon TCG cards in your vault (one 3×3 page). | 1 | 250 | Single binder ring + one filled 3×3 page. |
| `q-binder-page-5` | **Page 5** | Five full pages (45 unique cards). | 5 | 1000 | Binder spine with 5 page tabs + “5”. |
| `q-binder-page-10` | **Page 10** | Ten full pages (90 unique cards). | 10 | 2500 | Thick binder + “10”. Gold page edges when unlocked. |

---

## 12. Legacy milestones (3)

Separate `milestone_definitions` table. Neutral gold family. **Note:** overlap with pack quests — confirm whether these stay or merge into quest badges at implementation time.

| ID | Name | Description | Target | PP | Design idea |
|----|------|-------------|--------|-----|-------------|
| `m-first-pull` | **First Pull** | Opened your first pack or slab pull. | 1 | — | Minimal trophy with single card popping out. Simplest gold medallion. |
| `m-ten-pulls` | **Ten Pulls** | Opened ten total pulls. | 10 | — | Trophy + “10” (distinct from `q-pack-opens-10` by using trophy not pack icon — or merge visually). |
| `m-vault-25` | **Vault Depth 25** | Vault holds 25 items or unique cards (confirm rule at wiring). | 25 | — | Vault/safe door icon + “25”. Cool grey-blue tint when unlocked. |

*PP rewards TBD — legacy table has no reward column today. Likely map to existing quest PP or Collector Score only.*

---

## 13. Achievements (2)

Separate `achievement_definitions` table. Profile-facing status badges. Legendary gold + star.

| ID | Name | Description | Target | PP | Design idea |
|----|------|-------------|--------|-----|-------------|
| `a-collector` | **Collector** | Opened your first premium pack. | 1 | — | Star inside shield. Tier ribbon: “Collector”. Shown on profile when unlocked. |
| `a-marketplace` | **Marketplace Mover** | Completed a P2P trade. | 1 | — | Two arrows exchanging cards. Purple-gold split accent. |

*Also referenced in code but not in DB yet:* `a-sharp-eye`, `a-seasonal` — skip until seeded.

---

## Summary counts

| Group | Count |
|-------|-------|
| Onboarding | 10 |
| Daily | 2 |
| Weekly | 2 |
| Monthly | 3 |
| Pack opens | 5 |
| Login streaks | 2 |
| Shipping & marketplace | 3 |
| Binder variety | 3 |
| Binder types | 8 |
| Binder Kanto | 3 |
| Binder pages | 3 |
| Legacy milestones | 3 |
| Achievements | 2 |
| **Total** | **49** |

---

## Art production checklist

- [ ] Approve shared hex/shield template
- [ ] Approve category color palette
- [ ] Export 49 × 2 SVGs (locked + unlocked) — or 1 template + 49 icon overlays
- [ ] Confirm legacy milestone / achievement overlap with quests
- [ ] Wire `badgeImageUrl` or static `/badges/{id}.svg` path in frontend
- [ ] Hover popover copy review with client

---

## Out of scope (this doc)

- **Collection quests** (`collection_quests` table) — admin-created, per-set, dynamic count. Add a section when sets are finalized.
- **Masterdoc future badges** — 200+ type/rarity/roll tiers not yet in DB. Track in a separate `badges-future.md` when ready.
