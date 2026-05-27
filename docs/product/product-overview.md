# Product overview

Pocketpull is a **digital pack-opening and collector marketplace** for physical-backed cards: fund an account, open slab-roll packs, hold inventory, progress through Pocket Points and binders, and (roadmap) redeem physical cards via burn-to-redeem.

## Production stack (today)

| Capability | Status |
| ------------ | ------ |
| Google OAuth sign-in | **Live** |
| Privy embedded Solana wallet on sign-up | **Live** |
| Fastify API + Postgres | **Live** |
| Cloudflare Pages SPA | **Live** |
| Collector Crypt gacha proxy | **Live** (env-dependent) |
| Stripe deposits | **Wired** (webhook + routes) |
| Pocket Points, quests, Point Shop | **Partial** — see [rewards-and-gamification.md](rewards-and-gamification.md) |
| Binder / collection quests | **Partial** — see [binder-and-collections.md](binder-and-collections.md) |
| CNFT minting, burn-to-redeem shipping | **Roadmap** |

## Core principles

From [Masterdoc §2](../reference/Masterdoc.md):

- **Physical-backed** — cards map to real inventory / fulfillment.
- **Solana CNFT ownership** — planned representation on-chain.
- **Balance-first** — roll with account balance before wallet required.
- **Wallet optional early, important later** — Google account first; link Phantom or use Privy embedded.
- **Social proof** — live pulls, leaderboards, sharing.
- **Collector progression** — Pocket Points, quests, achievements, Collector Level / Score.
- **Binder nostalgia** — set completion, locked missing slots.

## Prototype vs production

[Masterdoc](../reference/Masterdoc.md) was written for a **UX-first prototype** (mock APIs, no real payments). Sections **1–28** remain the product UX bible. Treat these as **outdated for engineering**:

| Masterdoc section | Issue |
| ----------------- | ----- |
| §6 Login | Says Gmail + wallet login; production = **Google OAuth** + Privy provision |
| §29 Prototype API | Lists Cloudflare Functions mocks; production = **Fastify** routes |
| §30 Known gaps | Many items now partially implemented (OAuth, pools, referrals) |

Use [implementation-status.md](../changelog/implementation-status.md) for shipped vs backlog.

## Full specification

- **[Masterdoc](../reference/Masterdoc.md)** — all screens, flows, copy, rarity UX, sharing, referrals (31 sections)
- **[rewards](../reference/rewards.md)** — client PP earn/spend numbers
- **[badges](../reference/badges.md)** — badge catalogue (49 items)

## Information architecture

Five primary product areas: **Hub**, **Market**, **Collections**, **Rewards**, **Leaderboards** — detailed in [information-architecture.md](information-architecture.md).
