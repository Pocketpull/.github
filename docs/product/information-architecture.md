# Information architecture

Summary of [Masterdoc §3–28](../reference/Masterdoc.md) with route mapping to the current SPA.

## Primary sections

| Section | Purpose | Routes |
| ------- | ------- | ------ |
| **Hub** | Landing: featured drop, live pulls, quick actions | `/` |
| **Market** | Slab-roll machines, purchase, pool preview | `/slab-roll`, `/gacha`, `/packs` |
| **Collections** | Inventory + binder progress | `/vault`, `/binder` |
| **Rewards** | Quests, milestones, PP, referrals | `/rewards`, `/badges` |
| **Leaderboards** | Weekly / monthly / all-time competition | `/leaderboard` |

## Global chrome

| Control | Masterdoc | Implementation |
| ------- | --------- | -------------- |
| Guide | Onboarding wizard | Product surface TBD |
| Collector Level | Long-term status | Gamification / profile |
| Pocket Points | Points balance | `/rewards`, header |
| Balance | Spendable USD | Wallet / deposits |
| Deposit | Funding | Stripe `POST /deposits/stripe` |
| Account | Profile, shipping, audio, referrals | `/profile` |

## Hub (§4)

- Hero: “Rip packs. Run the room.”
- Actions: Open Packs, Browse Market
- Trust chips: odds, physical-backed, delivery, burn-to-redeem
- Featured drop, live pulls module, rewards panel
- Live feed filters: All, Pokémon, Sports, Anime, Big Hits

**Implementation:** `Home.tsx`, `home-recent-hits.tsx`, `GET /activity/recent-pulls` (up to 150 rows, image URLs).

## Market / slab roll (§9–12)

- Machine filters: category, price, featured
- Rare pull tape, roll purchase modal, balance rules
- Available card pool preview before purchase
- Rarity odds disclosure (product requirement)

**Implementation:** `/slab-roll/:machineId`, `GET /gacha/machines/:id/pool` (live CC `getNfts` — see [collector-crypt-gacha.md](../integrations/collector-crypt-gacha.md)).

## Pack reveal (§13–14)

- Single and multi-roll reveal, flip interaction, sound

**Routes:** `/reveal/:slug`, `/easy|medium|hard/:slug`, `POST /opens/pack`, `POST /opens/slab`.

## Collections (§16–19)

- **Inventory:** owned cards, list/sell/redeem actions
- **Binder:** set grids, missing slots locked, cover cards, spreads

**Routes:** `/vault`, `/binder` — binder backend integration in progress.

## Rewards (§21–24)

- Activities / quests (daily, weekly, monthly)
- Milestones / achievements (type, rarity families)
- Referral program with durable attribution
- Share rewards on reveal

See [rewards-and-gamification.md](rewards-and-gamification.md).

## Leaderboards (§26)

- Periods: all-time, weekly, monthly
- Metrics: Collector Score, pull value, etc.
- “My rank” and history

**Route:** `/leaderboard`, `GET /leaderboard`.

## Sharing (§15)

- Shareable pull moments, OG/Twitter cards, referral codes on share links
- `POST /share/pull`, `GET /s/:token`

## Account (§6–7, §28)

- Linked accounts (Google, Discord, X verification, wallets)
- Balance, deposit, withdraw (product)
- Audio preferences (collapsed in account)

**Route:** `/profile`.

## Themes (§27)

Visual themes for arcade / collector aesthetic — see frontend design tokens and `index.css`.
