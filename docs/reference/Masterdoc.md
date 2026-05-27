# PocketPull V3 Masterdoc

Last updated: May 2026

> **Documentation hub:** Production engineering docs live in **[pocketpull-platform/docs/README.md](pocketpull-platform/docs/README.md)**. This file is the full **UX/product specification**. Sections **29–30** describe the old prototype API and gap list — cross-check [implementation-status.md](pocketpull-platform/docs/changelog/implementation-status.md) and [product-overview.md](pocketpull-platform/docs/product/product-overview.md) before treating gaps as still open.

| Topic | Where |
| ----- | ----- |
| Auth (Google + Privy) | [authentication.md](pocketpull-platform/docs/architecture/authentication.md) |
| API routes | [backend-api-reference.md](pocketpull-platform/docs/architecture/backend-api-reference.md) |
| Rewards / PP numbers | [rewards.md](rewards.md) + [rewards-and-gamification.md](pocketpull-platform/docs/product/rewards-and-gamification.md) |
| Badges | [badges.md](badges.md) |
| Staging | [staging.md](pocketpull-platform/docs/getting-started/staging.md) |

## 1. Product Summary

PocketPull is a digital pack-opening and collector marketplace experience for physical-backed card collectibles. Users can fund an account, open slab-roll style packs, receive digital collectibles in their account or wallet, and later sell back, trade, or burn/redeem eligible cards for shipment of the physical card.

The V3 prototype focuses on the "Arcade Rip Room" direction: a high-energy collector hub with live pulls, slab-roll marketplace, inventory, set binders, rewards, referrals, leaderboards, and shareable reveal moments.

The prototype is intentionally UX-first. It does not perform real payments, real wallet transactions, real CNFT minting, real burning, production randomness, or physical fulfillment.

## 2. Core Product Principles

- Physical-backed collectibles: every card shown in the product is intended to correspond to a real physical card held or fulfilled by the platform.
- Solana CNFT ownership: earned cards are planned to be represented as compressed NFTs on Solana.
- Burn to redeem: a user can burn the digital collectible to claim the physical version, then pay shipping.
- Balance-first purchase flow: users can roll if they have sufficient account balance, even before connecting a wallet.
- Wallet optional at first, important later: Gmail login can create an account, and wallet linking can happen afterward.
- Social proof and hype: live pulls, card values, rarity treatments, leaderboards, and sharing make the experience feel active.
- Collector progression: Pocket Points, quests, achievements, Collector Score, and Collector Level add long-term goals beyond opening packs.
- Binder nostalgia: collections should feel like real collecting, with locked slots, set completion, and themed binders.

## 3. Information Architecture

The V3 site uses five primary sections:

- Hub: central landing view with featured drop, live pulls, quick actions, and product orientation.
- Market: slab-roll machine selection and purchase flow.
- Collections: inventory and binder views for owned and missing cards.
- Rewards: activities, milestones, referrals, Pocket Points, and Collector Score progression.
- Leaderboards: all-time, weekly, and monthly competition views.

Global controls include:

- Guide: opens the onboarding wizard.
- Collector Level: shows current long-term level.
- Pocket Points: shows current points balance.
- Balance: shows spendable account balance.
- Deposit: opens the funding flow.
- Account: opens profile, linked accounts, shipping, audio, and referral settings.

## 4. Hub

The Hub is the main landing page after opening the prototype.

### Purpose

The Hub keeps the homepage clean while still giving users a sense that the room is alive. It is designed to answer: "What is happening right now, and where should I go next?"

### Features

- Hero message: "Rip packs. Run the room."
- Primary actions: Open Packs and Browse Market.
- Trust/value chips: public odds, physical-backed, account delivery, burn to redeem.
- Featured drop card: shows a highlighted slab-roll machine and bonus progress.
- Live pulls module: shows recent cards entering users' wallets.
- Rewards panel: shows progress toward a bonus pack or reward milestone.
- Product map section: explains what lives in each tab.
- Live feed filters: All, Pokemon, Sports, Anime, Big Hits.

### Live Pull Cards

Live pull cards show:

- Pulling user name.
- Card name.
- Rarity.
- Collection/category.
- Estimated value.
- Redeemable/listed status.
- Time since pull.

Rarity is visually differentiated through borders and labels:

- Common: grey.
- Uncommon: green.
- Rare: blue.
- Epic: purple.
- Legendary: yellow/gold.

Clicking a live pull opens the card detail modal.

## 5. Guide / Onboarding Wizard

The Guide is a first-visit tutorial and can also be opened manually from the top bar.

### Purpose

The homepage should not carry too much explanatory text. The Guide explains the product in a lightweight, dismissible wizard.

### Current Steps

1. Start in the Hub: featured drops, live pulls, reward progress, and monthly collector race.
2. Choose a machine: preview pools, use Pocket Points, and open one or more rolls.
3. Flip the card: reveal the one-card pull, with future support for multi-card packs.
4. Build collections: inventory and binder progress.
5. Compete and claim: quests, achievements, referrals, leaderboards, and Pocket Points.

### Behavior

- Appears automatically on first visit.
- Can be skipped.
- Tracks progress with dots.
- Can be reopened with the Guide button.

## 6. Login And Account

PocketPull supports a flexible account model in the prototype.

### Login Entry Point

The top-level login concept is consolidated into a single Account/Login flow rather than separate Gmail and wallet buttons.

### Login Methods

- Gmail login: user can create an account using email identity.
- Wallet login: user can connect a Solana wallet directly.

### Linked Accounts

Users need a way to link missing identities:

- If they login with Gmail, they can later connect a wallet.
- If they login with wallet, they can later add Gmail.
- **X (Twitter)** is linked via a **tweet-and-code verification** flow (see below). Optional future: “Sign in with X” OAuth for login is a separate product decision; the reference design here assumes **paid X API access** for verification and metadata, without requiring X as the primary identity provider.

### X (Twitter) account verification (username + tweet + code)

**Purpose:** Confirm that the PocketPull user controls a specific X account so the product can show a verified handle on profile, personalize share copy, and gate or credit X-oriented quests. This uses the **X API** (paid tier is expected in production).

**Flow**

1. **Collect handle:** User enters their X username (with or without `@`). Normalize for display and API calls (trim, strip `@`; treat handle comparison as case-insensitive for lookup per X behavior).
2. **Persist pending state:** Store the claimed handle on the account (or in a `pending_x_verification` record) **before** issuing a code so the server knows which identity is being proved.
3. **Issue code:** Server generates a **high-entropy, single-use** code (e.g. cryptographically random, sufficiently long) and a **short expiry** (recommended **24–48 hours**). Show the plaintext code **once** in the UI. Store a **hash** of the code server-side (or a single-row challenge with `userId`, `expires_at`, `consumed_at`), never log plaintext codes in production logs.
4. **User action:** User posts a **public** tweet whose body contains the exact verification string. Product copy must define allowed shapes (e.g. main tweet only vs replies/quotes) to reduce ambiguity.
5. **Verify (server):**
   - Resolve handle → **`x_user_id`** via X API (stable id; use this for persistence even if the handle changes later).
   - **Primary check:** Fetch **recent tweets** for that user id; find a tweet containing the code, with `created_at` after the code was issued and before expiry.
   - **Optional fallback:** User submits the **tweet URL** (status id). Server fetches that tweet, confirms **author id** matches resolved `x_user_id`, and confirms the text contains the code—useful if the timeline is noisy or rate limits require fewer list calls.
6. **Success:** Mark `x_verified_at`, store `x_user_id` and a snapshot of `username` at verification time, consume the challenge, clear pending. **Failure:** Return a **generic** message (“We couldn’t verify yet—check that the tweet is public and includes the full code”) without revealing whether another user consumed the code.

**Operational rules**

- **Protected or restricted accounts:** Verification fails until content is visible to the API; surface clear UX.
- **Abuse:** Per-user and global rate limits on verification attempts; captcha or cooldown if needed.
- **Single active challenge:** Rotating or re-issuing a code should **invalidate** the prior code.
- **Rename:** After success, anchor on **`x_user_id`**; optionally refresh display handle from the API on login or on demand.

**Relation to sharing:** Verifying X does **not** auto-post tweets. The share UX may still open X’s composer via **web intent** or a future server-post integration; verification only establishes identity.

### Account Modal Features

- Display name with edit button.
- Default dummy name such as Collector #1.
- Collector Level and Collector Score summary.
- Balance summary.
- Pocket Points summary.
- Referral code and copy link action near the top.
- Linked accounts: Gmail, wallet, **X/Twitter (verified via tweet-and-code)**.
- Withdraw funds.
- Collapsible Shipping section.
- Collapsible Audio section.
- Notifications preferences.
- Theme control.
- Save and sign out actions.

## 7. Balance, Deposits, And Withdrawals

### Balance

Balance represents spendable funds in the user's PocketPull account. In production, this may come from credit card deposits, Stripe payments, wallet USDC deposits, or other supported payment rails.

The top bar displays the balance directly next to Deposit because depositing increases the account balance.

### Deposit

The Deposit flow lets users add funds. The prototype uses mocked deposit amounts and updates local account balance.

Production intent:

- Credit card or Stripe deposit.
- Backend conversion to USDC or equivalent account ledger.
- Wallet-based USDC deposit.
- Clear ledger history for deposits, purchases, refunds, sell backs, and withdrawals.

### Withdraw

Users need a way to withdraw unused balance.

Production intent:

- Withdraw to a connected wallet.
- Potentially refund to original card or bank rails where legally and operationally possible.
- Apply compliance, fraud, chargeback, and withdrawal limits.

## 8. Pocket Points

Pocket Points are a reward currency separate from account balance.

### Product Direction

The Pocket Points system should stay simple. Users should not need to convert Pocket Points into gems or another intermediary item before receiving value.

The simplified flow is:

1. Earn Pocket Points from buying packs and/or completing activities.
2. Use Pocket Points directly for specific single-use discounts.

The primary Pocket Points use cases are:

- Reduce pack costs, potentially up to a free pack if the economics support it.
- Reduce shipping costs, potentially up to free shipping if the economics support it.

Shipping costs may vary by location, so fixed-dollar shipping discounts are safer than percentage-based shipping discounts in production.

### How Users Earn Pocket Points

- Purchasing packs.
- Completing quests.
- Completing achievements.
- Referral progress.
- Winning leaderboard rewards.

### Example Pack Earning Rates

Dummy earning rates used for planning:

- Each Starter Pack Opened = +100 Points.
- Each Elite Pack Opened = +250 Points.
- Each Themed Pack Opened = +500 Points.
- Each Premium Pack Opened = +1,000 Points.
- Each Grail Pack Opened = +5,000 Points.

### Example Pack Discount Costs

Dummy discount costs used for planning:

- Reduced price of next Starter Pack = 1,000 points for 10% discount.
- Reduced price of next Elite Pack = 2,500 points for 10% discount.
- Reduced price of next Themed Pack = 5,000 points for 10% discount.
- Reduced price of next Premium Pack = 10,000 points for 10% discount.
- Reduced price of next Grail Pack = 50,000 points for 10% discount.

### Example Free Pack Costs

Dummy free pack costs used for planning:

- Free Starter Pack = 10,000 points.
- Free Elite Pack = 25,000 points.
- Free Themed Pack = 50,000 points.
- Free Premium Pack = 100,000 points.
- Free Grail Pack = 500,000 points.

### Example Shipping Discount Costs

Dummy shipping discount costs used for planning:

- Reduced price of shipping by $1 = 1,000 points.
- Reduced price of shipping by $5 = 2,500 points.
- Reduced price of shipping by $10 = 5,000 points.
- Reduced price of shipping by $20 = 10,000 points.
- Reduced price of shipping by $50 = 50,000 points.

### Current Prototype Behavior

The roll modal includes a collapsible Use Pocket Points section. Users can choose:

- No discount.
- 5% discount.
- 10% discount.

Point costs scale by machine tier. Example:

- Starter: 5,000 points for 5%, 10,000 points for 10%.
- Elite: 10,000 points for 5%, 20,000 points for 10%.
- Higher tiers should scale upward.

## 9. Market

The Market is currently focused on slab-roll machines.

### Market Positioning

Slab rolls are the primary marketplace experience first. Peer-to-peer listings, trades, offers, raw singles, and sealed products can be added later once collectors have more inventory.

### Market Features

- Hero heading: Choose your rip.
- Slab Rolls tab.
- P2P Later tab placeholder.
- Rare Pull Tape conveyor.
- Machine category filters.
- Large machine cards.
- Machine image, tier labels, description, price, and remaining hits.
- View pool and roll button on each machine card.

### Machine Filters

- All.
- Starter.
- Elite.
- Themed.
- Premium.
- Grail.

### Current Example Machines

- PKMN 25: starter Pokemon slab roll.
- PKMN 50: broader Pokemon slab pool.
- FIREGRASS 100: fire and grass character chase roll.
- MEW 250: premium psychic/Mew-themed chase roll.
- PKMN 250: higher-stakes Pokemon roll.
- PKMN 1000: grail Pokemon roll.

## 10. Rare Pull Tape

The Rare Pull Tape is a horizontal conveyor feed on the Market page.

### Purpose

It gives market visitors a quick sense of recent valuable hits and creates excitement before purchasing.

### Features

- Moving strip of recent high-value pulls.
- Card thumbnail.
- User who hit the card.
- Card name.
- Rarity.
- Estimated USD value.
- Dynamic value styling:
  - Under $100: blended and subtle.
  - Over $100: slightly more visible.
  - Over $250: stronger highlight.
  - Over $1,000: highest emphasis.

Clicking a pull opens the card detail modal.

## 11. Roll Purchase Modal

The roll purchase modal appears after selecting a machine.

### Purpose

It lets the user review the machine, choose roll quantity, apply points, inspect odds, preview the pool, and start the roll.

### Features

- Machine image.
- Machine tier and name.
- Description.
- Roll quantity: 1, 3, 5, or 10.
- Total due.
- Available balance.
- Pocket Points discount section.
- Rarity odds.
- One-card-per-roll callout.
- Available cards preview.
- Start roll button.
- Cancel button.

### Balance Rules

The user can start a roll if they have enough balance. A wallet is not required for the prototype purchase flow because a user may have deposited funds by credit card.

### Rarity Odds

Current visible odds:

- 54% Common.
- 26% Uncommon.
- 14% Rare.
- 5% Epic.
- 1% Legendary.

## 12. Available Card Pool

The Available Cards modal shows what can be pulled from a machine.

### Features

- Pool heading with machine name.
- Search by card name, subtype, grade, or value.
- Refresh list action.
- Card grid.
- Estimated value.
- Rarity label.
- Category/subtype labels.
- Click card to inspect details.

### Product Role

The pool gives users confidence and lets them browse potential hits before spending. In production, this should be backed by real inventory and availability rules.

## 13. Pack Reveal Flow

The reveal flow is one of the most important emotional moments in the product.

### Flow

1. User clicks Start roll.
2. Pack opening animation begins.
3. Card back appears.
4. User taps the card.
5. Card flips.
6. Rarity-specific reveal effect appears.
7. Revealed card is shown.
8. User can inspect metadata, share, view inventory, or roll again.

### Single Roll

The single roll experience focuses screen real estate on one large card.

### Multi Roll

For 3, 5, and 10 rolls:

- The modal shows multiple card backs.
- User can tap each card one at a time.
- User can reveal all.
- Revealed cards scale up enough for desktop and mobile.
- Final summary shows total cards added, best pull, rarity breakdown, and estimated total value.

### Current Pack Rule

Each current roll opens one physical-backed card. Future pack formats may support multiple cards per pack.

## 14. Reveal Interactions And Sound

The prototype includes interaction polish:

- Hover tilt on reveal cards.
- Flip animation on click.
- Impact effect during reveal.
- Detail modal click sound.
- Interface sound controls in Account.

Audio can be muted or adjusted from Account.

## 15. Sharing

Sharing supports the social loop around collecting.

### Shareable Moments

- Revealed cards.
- Owned inventory cards.
- Tournament wins.
- Finished tournaments.
- Completed achievements and milestones.
- Future collection completion moments.

### Channels

- X/Twitter is the primary channel.
- Native share (Web Share API where available).
- Reddit, Facebook, Discord paste.
- Copy link/caption.

### Public share links and rich embeds (Twitter, Discord, messaging)

For high-impact pulls, the product should support **dedicated public share URLs** (per share event), not only raw image files or client-only composer text.

**Share page content**

- Each generated link resolves to an **HTML page** tailored to one share payload: **card (or slab) image**, title, value/rarity line, short brand line, and optional **referrer context** encoded in the URL or server-side lookup token.
- The page sets **Open Graph** and **Twitter Card** meta tags (`og:title`, `og:description`, `og:image`, `og:url`, `twitter:card` with a large image when appropriate) so that pasting the link in **Discord**, **X**, **iMessage**, etc. shows a **rich preview** (image + key details).
- **`og:image`** must be an **absolute HTTPS URL** to a **stable** asset for that share id (direct image or CDN URL). Embeds are **cached** by consumers; treat each “big pull” share as **immutable** for preview purposes (new pull → **new** share id / new URL) to avoid stale or wrong previews.

**Referral inside the share URL**

- Share links should carry a **referral payload** that survives copy-paste: either the existing **`ref` code** in query (e.g. `?ref=CODE`) or a short **signed opaque token** (path or query) that expands server-side to `(referrerUserId | referralCode, shareId, optional expiry)`. Signing prevents tampering with referrer identity without logging in.

**Landing and attribution (see §24)**

- First request to the share URL should **set first-party attribution state** (cookie / localStorage per §24) and may **302** to a canonical marketing or signup route while preserving referral in the cookie.
- Goal: a visitor who **opens the preview**, **closes the tab**, and **returns days later** can still be attributed to the sharer when they register—without relying on **raw IP** as the primary key.

### Share flow vs gamification

- **Quest / Pocket Points** rules may still treat “shared” as a client or server event (e.g. user tapped Share and confirmed). A share link does not prove someone posted; ops can combine **link generation** with existing honor-based or future X verification rules as needed.

### Account tie-in

- Once X is **verified** (§6), share copy and profile can show **`@handle`** and optionally sync avatar from X. Users without verified X still get generic share text and links.

## 16. Collections

Collections combines Inventory and Binder in one top-level tab.

### Why Collections Combines Both

Inventory is operational: what the user owns and can act on.

Binder is emotional: what the user is collecting, missing, and completing.

Both describe the user's cards, so they live together.

## 17. Inventory

Inventory shows all owned cards.

### Purpose

Inventory should let users admire their cards, filter them, inspect metadata, and take actions.

### Features

- Card-first vertical layout.
- Owned cards sorted by highest rarity by default.
- Search by card, set, subtype, or pull tier.
- Filters:
  - All.
  - Pokemon.
  - Vintage.
  - Modern Chase.
  - Character.
  - Promos.
  - Gem Mint 10.
  - Sealed.
  - Redeemed.
- Sort options:
  - Highest rarity.
  - Highest value.
  - Highest quantity.
  - Newest pull.
- Summary cards:
  - Cards owned.
  - Redeemable.
  - Ready to ship.
  - Estimated value.
  - Collector Level.
  - Collector Score.
  - Top set.

### Inventory Card Details

Clicking an owned card opens metadata and actions.

Actions:

- Sell back: instant platform buyback, currently conceptualized as 85% of value.
- Share card.
- Burn to redeem.

Trade is intentionally removed for now until P2P is designed.

## 18. Card Detail Modal

The card detail modal appears from live feeds, reveal results, inventory, binder, and pool preview.

### Metadata Fields

- Card name.
- Rarity and category.
- Card image.
- Set.
- Card number.
- Grade.
- Estimated value.
- Last sale.
- Quantity owned.
- Vault status.
- Chain status.

### Contextual Actions

Actions change by context:

- Inventory: Sell back, Share card, Burn to redeem.
- Binder: Share card and Burn to redeem for owned cards; no Sell back.
- Pool preview: view-only inspection.
- Reveal: share, view inventory, roll again.
- Live feed: inspect details.

## 19. Binder

Binder is the nostalgic collection view.

### Binder Model

Binders are saved views over card metadata, not exclusive folders. One card can appear in multiple future binders.

Example: a Charizard PSA 10 could appear in:

- Brilliant Stars Set Binder.
- Sword & Shield Era Binder.
- Charizard Binder.
- Modern Hits Binder.
- PSA 10 Slabs Binder.
- Fire Type Binder.

### Current Focus

The prototype focuses on Set Binders first.

### Future Binder Categories

- Sets.
- Characters.
- Hits.
- Slabs.
- Eras.
- Types.
- Holo sets.
- Power/type sets, such as fire, water, electric, psychic, and other card types.
- Character sets, such as a dedicated Charizard binder.

### Pokemon Set Binder Order

Pokemon sets are shown in this chronological order:

1. Base Set.
2. Jungle.
3. Fossil.
4. Team Rocket.
5. Neo Genesis.
6. Evolving Skies.
7. Crown Zenith.
8. Scarlet & Violet 151.
9. Prismatic Evolutions.

### Binder Cover Cards

Binder cards show:

- Collection label.
- Era label.
- Set name.
- Completion percentage.
- Owned count.
- Total card count.
- Progress bar.
- Open Binder button.

### Binder Spread

Opening a binder shows:

- Two-page spread.
- 12 cards per spread.
- Official set order.
- Owned cards in full color.
- Missing cards greyed out/locked.
- Previous and next spread controls.
- Owned count and progress.

### Non-Pokemon Binders

The prototype also introduces non-Pokemon binder examples for future expansion:

- Magic.
- Sports.
- Anime.

## 20. Redeemable And Shipping

Every card is intended to be redeemable for its physical version.

### Burn To Redeem

To receive a physical card, the user burns the digital collectible on-chain. The platform verifies the burn and moves the physical card into a shipping-eligible state.

### Shipping Section

The Account modal includes a Shipping section where users should be able to:

- Enter shipping details.
- See redeemed cards eligible to ship.
- Track shipment status.
- Potentially apply Pocket Points to reduce shipping cost.

Inventory also needs to surface redeemed or ready-to-ship cards so users do not lose track of them.

## 21. Rewards

Rewards are split into three clearer groups:

- Activities: recurring quests.
- Milestones: permanent achievements.
- Referral Program: invite tracking and rewards.

The Rewards structure exists to create repeatable reasons to come back, medium-term goals, and long-term completionist goals.

## 22. Activities / Quests

Quests are recurring activities that reset by timeframe.

### Timeframes

- Daily.
- Weekly.
- Monthly.

### Quest Types

- Login.
- Pack purchases.
- Multi-day purchase streak.
- Referred users.
- Shared content, such as sharing an earned card.

### Behavior

- Completed quests must be claimed.
- Claiming grants Pocket Points.
- Completed quests do not reset until their timeframe resets.

## 23. Milestones / Achievements

Achievements are permanent accomplishment tracks. Once completed, they cannot be completed again.

The goal of achievements is to create short, medium, and long-term goals for completionist users. The current achievement examples primarily focus on Pokemon, but the system can expand to One Piece, Sports, Anime, Magic, and other categories.

### Achievement Families

- Total rolls opened: 25, 50, 100, 250, 1,000, 5,000, 10,000.
- Set binders completed: 1st, 2nd, 3rd, 4th, 5th.
- High-value pulls: $50, $100, $150, $250, $500, $1,000, $2,000, $5,000.
- Pokemon type collection milestones.
- Rarity pull milestones.

### Pokemon Type Milestones

Each type supports collection counts of 10, 20, 30, 40, 50, 100, 250, 500, and 1,000:

- Fire.
- Water.
- Normal.
- Electric.
- Grass.
- Ice.
- Fighting.
- Poison.
- Ground.
- Flying.
- Psychic.
- Bug.
- Rock.
- Ghost.
- Dragon.
- Dark.
- Steel.
- Fairy.

### Rarity Milestones

For common, uncommon, rare, epic, and legendary cards:

- Pull 1.
- Pull 5.
- Pull 10.
- Pull 20.
- Pull 50.
- Pull 100.
- Pull 200.
- Pull 500.
- Pull 1,000.
- Pull 2,000.

### Rewards

Achievements award:

- Pocket Points.
- Collector Score XP.

## 24. Referral Program

The referral program has a dedicated section in Rewards and a quick code card in Account. **Share links** (§15) extend the same program: anyone who lands from a friend’s pull share should be able to register with referral attribution even if they **do not sign up immediately**.

### Referral features (product surface)

- Persistent **referral code** per user (or per campaign where applicable).
- **Copy referral link** (e.g. `/login?ref=CODE` or equivalent).
- **Total referrals** and **referral status** in Rewards / Account.
- **Rewards** tied to milestone rules (e.g. referred user opens first pack)—specific amounts live in rewards policy, not this doc.

### Canonical signup link

- Default pattern: send new users to signup/login with an explicit **`ref` query parameter** (or path-based code) that the server validates against `users.referral_code` (or signed token mapping).
- Server-side registration and “claim referral after signup” flows must **read the same attribution state** as the browser (below), not only the query string on the current page.

### Robust attribution when the visitor delays signup

**Problem:** A visitor taps a share preview, views the pull page, leaves, and returns later without `?ref=` in the URL. **IP-only** attribution is **not** acceptable as the primary signal (NAT, mobile, VPNs, corporate Wi‑Fi, privacy expectations).

**Primary mechanism: first-party persistence**

1. **HTTP cookie (recommended backbone):** On first visit to a **referral-bearing** URL (share page, `/login?ref=`, etc.), set a **first-party** cookie such as `pp_ref` (or a namespaced variant) containing:
   - the **referral code** or a **signed, opaque** blob the server can resolve to a referrer user id + metadata, and
   - optional **issued-at** / **exp** inside the signed blob.
2. **Attributes:** `Secure`, `HttpOnly` if only the server needs it (registration endpoint reads cookie); if the client must pass it to an SPA bootstrap, use `SameSite=Lax` (or `Strict` if embed/redirect flows allow), document the tradeoff with iframes.
3. **TTL:** **30–90 days** (product choice); align with “attribution window” in rewards policy.
4. **Consumption:** On successful **referral attachment** (first referral bound for that new user, subject to rules), **clear or invalidate** the cookie so it cannot double-apply.

**Secondary mechanism: localStorage mirror (e.g. `pp_ref` mirror)**

- On the same first visit, mirror the code or signed token in **localStorage** as a fallback when cookies are blocked or cleared selectively.
- Registration must check **cookie first**, then localStorage, with **consistent validation** (same server-side validation as query param).

**Tertiary mechanism: re-hydrate from URL**

- Any return visit that includes `?ref=` (e.g. from a fresh share) **refreshes** cookie + localStorage **only if** the new ref passes validation and optional conflict rules (see below).

**Layered “strength” signals (not standalone identity)**

Use these to **reduce fraud and accidental collisions**, **not** to replace cookies as the persistence layer:

- **Signed referral token in URL:** HMAC/JWT with referrer id, issuer time, and expiry—prevents casual tampering with `ref=`.
- **Single-use or bind-on-first-touch (optional):** For high-value campaigns, consider marking a click id server-side; scope carefully to avoid breaking legitimate multi-device users.
- **Hashed IP + coarse User-Agent** stored on **first attribution event** (short retention) as a **fraud signal** only: e.g. detect mass signups from one ASN, or cookie clearing loops. **Do not** use raw IP as the sole key for “who referred whom.”
- **Device-bound signals** (optional, policy-heavy): stub for future **privacy-preserving** device hints only where legal/privacy review approves; default remains cookie + signed URL.

### Conflict and eligibility rules

- **One referred-by per new user:** First valid attachment wins unless product explicitly allows override (usually **no**).
- **Existing users:** Ignore ref cookie if user already has `referred_by` set or already opened packs per policy.
- **Self-referral and collusion:** Reject ref equal to own code; optional velocity limits per referrer and per IP hash cluster.
- **Cross-device:** Same human on phone + laptop may get **two cookies**; policy may allow **one** referral per account only regardless—avoid tying “referral loss” to IP.

### Relation to share pages (§15)

- Share URLs must trigger the **same attribution cookie + mirror** as manual referral links.
- After redirect to marketing or home, the **cookie** retains ref; signup CTA need not restate `?ref=` on every page.

### Future needs

- Formal **attribution window** and rewards SLAs in rewards policy.
- Chargeback / fraud review and referrer clawback rules.
- Jurisdiction-specific privacy review for IP hashing and retention.

## 25. Collector Level And Collector Score

Collector Score is long-term progression earned from achievements. Collector Level is derived from Collector Score.

### Purpose

This adds a permanent status system that is not reset by weekly or monthly tournaments.

Collector Level acts as a status signal showing how far a user has progressed through achievements. As users complete achievements, they earn score points, or experience points, that contribute to their total Collector Level.

### Places It Appears

- Top bar level chip.
- Account modal.
- Collections summary.
- Rewards achievement cards.
- Leaderboards as a metric toggle.
- Leaderboard player badges.

### Leaderboard Role

All-time leaderboards can rank by:

- Packs opened.
- Collector Score.

This lets high-achievement collectors compete separately from high-volume openers.

## 26. Leaderboards / Tournaments

Leaderboards are designed as a competitive arena scoreboard and PVP rewards system, not only a flex board.

### Purpose

Leaderboards should encourage:

- Pack purchases.
- Social competition.
- Shareable moments, such as top 5 finishes.
- Reward-driven participation through Pocket Points.
- Future growth campaigns with grand prizes, such as free packs or card prizes, if economics support them.

### Periods

- All.
- Weekly.
- Monthly.

### Leaderboard Types

- All-time leaderboard: an ongoing flex leaderboard of top pack openers and/or collectors. Future airdrops or free packs could be considered for top all-time users.
- Weekly leaderboard: users compete against one another by pack openings and/or rare pulls. Top winners get Pocket Points.
- Monthly leaderboard: users compete against one another by pack openings and/or rare pulls. Top winners get 5x-10x more Pocket Points than weekly leaderboard winners.

### Reset Rules

- Weekly: Monday through Sunday, reset at 8 PM EST.
- Monthly: first day through end of month, reset at 8 PM EST.

### Metrics

- Packs opened.
- Collector Score.
- Rare pulls, as a future metric.

### All-Time View

Shows total pack volume and long-term collector competition.

The all-time view can toggle between:

- Most packs opened.
- Highest Collector Score.

### Weekly View

Shows weekly pack volume and rewards ranks 1 through 5 with Pocket Points.

Suggested weekly prizes:

- 1st: 10,000 PP.
- 2nd: 7,500 PP.
- 3rd: 5,000 PP.
- 4th: 3,500 PP.
- 5th: 2,500 PP.

### Monthly View

Shows monthly pack volume and rewards ranks 1 through 10 with 10x weekly Pocket Points.

### Desktop Layout

The desktop layout is intended to feel like an arena scoreboard:

- Tournament header.
- Countdown/reset time.
- Prize range.
- Total competitors.
- My rank.
- Hero podium for 1st, 2nd, 3rd.
- Prize ladder side rail.
- Full leaderboard below.

### Mobile Layout

Mobile stacks content into a cleaner sequence:

- Period selector.
- My rank.
- Podium/leader rows.
- Prize ladder.
- Full leaderboard.

### My Rank

Users outside top positions still need to see:

- Current rank.
- Opens this period.
- Opens needed to reach next reward tier.
- Reward cutoff position.

### History

Weekly and monthly views include finished tournament history and share actions for winners.

## 27. Themes

The prototype supports:

- Dark mode, default.
- Light mode.

The latest light-mode sweep improves contrast across text, dropdowns, rarity colors, ticker values, and panels.

Theme control is available in Account.

## 28. Audio

The product includes interface sounds for key interactions.

### Current Audio Moments

- Button/detail click.
- Card detail opening.
- Reveal animation.

### Account Controls

- Mute interface sounds.
- Volume slider.
- Test sound.

Audio and shipping sections are collapsed by default in Account to reduce clutter.

## 29. Prototype API Layer

The Cloudflare Pages Functions layer currently exposes mocked API routes:

- `GET /api/health`
- `GET /api/packs`
- `GET /api/inventory`
- `GET /api/leaderboard`
- `POST /api/open-pack`

These APIs are not production-safe and do not perform real payments, randomness, wallet actions, minting, burning, or shipping.

## 30. Known Product Gaps

- Real Solana wallet authentication and signature verification.
- Real Gmail/OAuth authentication.
- X/Twitter **tweet-and-code verification** and **share-page referral** flows (see §6, §15, §24); optional later: X OAuth as login provider.
- Real card inventory source of truth.
- Real slab pool availability.
- Production odds disclosures.
- Secure pack-opening randomness.
- Payment processing and USDC conversion.
- Withdrawals.
- CNFT minting and transfer.
- Burn-to-redeem verification.
- Shipping fulfillment.
- Instant sell-back pricing.
- P2P marketplace, listings, offers, and trades.
- Admin tools for cards, packs, users, rewards, and disputes.
- Legal, compliance, licensing, and support flows.

## 31. Source Strategy Notes Incorporated

This masterdoc incorporates the following product direction from the supporting notes:

- Pocket Points should not require users to buy gems or manage separate gem types. The reward system should remain: earn PP, then spend PP directly on pack or shipping discounts.
- Pocket Points should focus on only a small number of clear use cases. The primary recommended use cases are pack discounts/free packs and shipping discounts/free shipping.
- Leaderboards should not be only a flex board. They should function as a PVP system with rewards, including all-time flex rankings, weekly reward competition, and monthly reward competition.
- Rewards should be structured into repeatable/time-lapsed Quests and milestone-based Achievements.
- Collector Level should reflect achievement progress and act as a long-term user status signal.
- Collections should be the parent concept that contains both Inventory and Binder.
- Binder should show collected and missing cards, with missing cards visible but locked.
- Set binders are the current focus, but future binder categories can include holo sets, power/type sets, character sets, and other collection views.
- **X identity** is established via **tweet-and-code verification** (paid X API) rather than OAuth-by-default; **share links** use **OG/Twitter Card** pages and **first-party cookies** (with optional mirrors and signed tokens) for **durable referral attribution**—not raw IP as primary (see §6, §15, §24).
