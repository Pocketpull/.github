# Frontend routes

SPA routing from `frontend/src/App.tsx`. Global shell: `AppShellLayout` unless noted.

## Public / marketing

| Path | Page | Notes |
| ---- | ---- | ----- |
| `/` | Home | Live pulls strip, featured actions |
| `/how-it-works` | How it works | |
| `/login` | Login | Google OAuth link to API |
| `/coming-soon` | Coming soon | Site gate bypass |
| `/partners` | Partners | |

## Packs & reveal

| Path | Page |
| ---- | ---- |
| `/packs` | Pack catalog |
| `/packs/:slug` | Pack detail |
| `/packsnew` | New packs UI |
| `/reveal` | Reveal demo |
| `/reveal/:slug` | Pack reveal |
| `/easy/:slug` | Easy opening |
| `/medium/:slug` | Medium opening |
| `/hard/:slug` | Hard opening |

## Gacha & slab roll

| Path | Page |
| ---- | ---- |
| `/gacha` | Gacha hub |
| `/slab-roll` | Machine list |
| `/slab-roll/:machineId` | Machine + pool + purchase |

## Inventory & marketplace

| Path | Page | Notes |
| ---- | ---- | ----- |
| `/vault` | Inventory | Canonical; `/inventory` redirects |
| `/marketplace` | P2P marketplace | `/marketplace/p2p` redirects |

## Rewards & social

| Path | Page |
| ---- | ---- |
| `/rewards` | Rewards hub |
| `/badges` | Badges |
| `/leaderboard` | Leaderboards |
| `/profile` | Account, referrals, wallets |

## Binder

| Path | Page |
| ---- | ---- |
| `/binder` | Pokémon binder home |
| `/bindernew` | Binder WIP |
| `/binder/pokemon/*` | Legacy redirects → `/binder` |

## Admin

| Path | Notes |
| ---- | ----- |
| `/admin/*` | `RequireAdmin` wrapper |

## Dev / internal

| Path | Page |
| ---- | ---- |
| `/animations` | Lottie gallery |
| `/animations/aoe` | AOE effects |

## Providers (app root)

| Provider | Role |
| -------- | ---- |
| `auth-provider.tsx` | Session, `/auth/me` |
| `privy-provider.tsx` | Privy SDK |
| `privy-wallet-bootstrap.tsx` | Post-login `POST /auth/privy/ensure` |
| `solana-wallet-provider.tsx` | Phantom / Solflare |
| `solana-cluster-provider.tsx` | devnet/mainnet toggle |

## API client

- `frontend/src/lib/api.ts` — `api()`, Bearer header, cluster header
- `frontend/src/lib/session-token.ts` — `#pp_token=` bootstrap

## Product mapping (Masterdoc IA)

| Masterdoc section | Primary routes |
| ----------------- | -------------- |
| Hub | `/` |
| Market / slab roll | `/slab-roll`, `/gacha` |
| Collections | `/vault`, `/binder` |
| Rewards | `/rewards`, `/badges` |
| Leaderboards | `/leaderboard` |

See [information-architecture.md](../product/information-architecture.md).
