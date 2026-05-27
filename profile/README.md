<div align="center">

[![Pocketpull banner](https://raw.githubusercontent.com/Pocketpull/.github/main/profile/assets/banner.svg)](https://pocketpull.io)

<br />

<img src="https://raw.githubusercontent.com/Pocketpull/.github/main/profile/assets/logosymbol.png" alt="" width="88" height="88" />
&nbsp;&nbsp;
<img src="https://raw.githubusercontent.com/Pocketpull/.github/main/profile/assets/logotext.png" alt="Pocketpull" width="220" />

<br /><br />

**Open packs. Pull slabs. Vault your collection — on Solana.**

<br />

[![Website](https://img.shields.io/badge/website-pocketpull.io-2f7cf6?style=for-the-badge)](https://pocketpull.io)
[![App](https://img.shields.io/badge/app-frontend--9a5.pages.dev-0f172a?style=for-the-badge&logo=cloudflare&logoColor=f38020)](https://frontend-9a5.pages.dev)
[![API](https://img.shields.io/badge/api-Railway-0B0D0E?style=for-the-badge&logo=railway&logoColor=white)](https://pocketpull-production.up.railway.app/health)

<br />

[![React](https://img.shields.io/badge/React-19-61DAFB?style=flat-square&logo=react&logoColor=111)](https://github.com/Pocketpull/Frontend)
[![TypeScript](https://img.shields.io/badge/TypeScript-5-3178C6?style=flat-square&logo=typescript&logoColor=white)](https://github.com/Pocketpull/Frontend)
[![Vite](https://img.shields.io/badge/Vite-SPA-646CFF?style=flat-square&logo=vite&logoColor=white)](https://github.com/Pocketpull/Frontend)
[![Fastify](https://img.shields.io/badge/Fastify-API-000000?style=flat-square&logo=fastify&logoColor=white)](https://github.com/Pocketpull/Backend)
[![PostgreSQL](https://img.shields.io/badge/PostgreSQL-Drizzle-4169E1?style=flat-square&logo=postgresql&logoColor=white)](https://github.com/Pocketpull/Backend)
[![Solana](https://img.shields.io/badge/Solana-Privy%20wallets-9945FF?style=flat-square&logo=solana&logoColor=white)](https://github.com/Pocketpull/Backend)
[![Cloudflare](https://img.shields.io/badge/Deploy-Cloudflare%20Pages-F38020?style=flat-square&logo=cloudflare&logoColor=white)](https://github.com/Pocketpull/Frontend)

<br />

[Live app](https://pocketpull.io) · [Frontend](https://github.com/Pocketpull/Frontend) · [Backend](https://github.com/Pocketpull/Backend) · [Platform docs](https://github.com/Pocketpull/Pocketpull)

</div>

---

## Platform walkthrough

<div align="center">

<video src="https://github.com/Pocketpull/.github/raw/main/profile/assets/platform-walkthrough.mp4" width="720" controls>
  <a href="https://github.com/Pocketpull/.github/raw/main/profile/assets/platform-walkthrough.mp4">Download platform walkthrough</a>
</video>

<br />

Local dev · Google sign-in · Privy wallet · staging vs production

</div>

---

## Repositories

| | Repository | Stack | Deploy |
| :---: | --- | --- | --- |
| 🖥️ | [**Frontend**](https://github.com/Pocketpull/Frontend) | Vite · React 19 · TypeScript · Privy | Cloudflare Pages |
| ⚙️ | [**Backend**](https://github.com/Pocketpull/Backend) | Node · Fastify · Drizzle · Postgres | Railway |
| 📘 | [**Pocketpull**](https://github.com/Pocketpull/Pocketpull) | Architecture & env matrices | Docs |
| 🏠 | [**`.github`**](https://github.com/Pocketpull/.github) | Org profile (this page) | — |

---

## Architecture

```mermaid
flowchart LR
  subgraph users [Users]
    Browser[Browser]
  end
  subgraph cf [Cloudflare Pages]
    SPA[Vite SPA]
  end
  subgraph rw [Railway]
    API[Fastify API]
    DB[(Postgres)]
  end
  subgraph external [Integrations]
    Google[Google OAuth]
    Privy[Privy wallets]
    CC[Collector Crypt]
    R2[Cloudflare R2]
  end
  Browser --> SPA
  SPA -->|Bearer session| API
  API --> DB
  API --> Google
  API --> Privy
  API --> CC
  API --> R2
```

| Step | What happens |
| :--: | --- |
| 1 | User opens the **SPA** (production, staging preview, or local). |
| 2 | **Sign in with Google** → API OAuth → redirect with `#pp_token=`. |
| 3 | API provisions a **Privy embedded Solana wallet** and persists it in Postgres. |
| 4 | Packs, gacha, vault, marketplace, and rewards call the **API** with `Authorization: Bearer`. |

<details>
<summary><strong>Auth & wallets</strong></summary>

<br />

| | |
| --- | --- |
| **Login** | Google OAuth only *(Apple Sign-In planned)* |
| **Session** | Bearer token after OAuth — required for Cloudflare Pages + Railway cross-origin |
| **Wallet** | Privy server-side on sign-in; Phantom / Solflare optional |
| **Not used** | Privy login modal, email / password |

</details>

---

## Environments

| Environment | Frontend | API |
| --- | --- | --- |
| **Production** | [pocketpull.io](https://pocketpull.io) · [frontend-9a5.pages.dev](https://frontend-9a5.pages.dev) | [pocketpull-production.up.railway.app](https://pocketpull-production.up.railway.app) |
| **Staging** | Cloudflare Preview (`staging` branch) | [staging-pp-production.up.railway.app](https://staging-pp-production.up.railway.app) |
| **Local** | `http://localhost:8008` | `http://localhost:8080` |

<details>
<summary><strong>Staging without duplicate repos</strong></summary>

<br />

- Git: `main` → production · `staging` → preview builds  
- Railway: `pocketpull` + `Staging-pp` (separate Postgres)  
- Cloudflare: **Production** vs **Preview** env vars (`VITE_API_URL` must match the API)  
- Templates: `Frontend/.staging.env` · `Backend/.staging.env`

</details>

---

## Quick start

<details>
<summary><strong>Clone & run locally</strong></summary>

<br />

```bash
# Terminal 1 — API
git clone https://github.com/Pocketpull/Backend.git
cd Backend && cp .env.example .env
npm install && npm run db:migrate && npm run db:seed && npm run dev

# Terminal 2 — Web
git clone https://github.com/Pocketpull/Frontend.git
cd Frontend && cp .env.example .env
npm install && npm run dev
# → http://localhost:8008
```

Set `VITE_API_URL=http://localhost:8080` and backend `WEB_ORIGIN=http://localhost:8008`.

Full guides: [Frontend README](https://github.com/Pocketpull/Frontend) · [Backend README](https://github.com/Pocketpull/Backend) · [Platform README](https://github.com/Pocketpull/Pocketpull/blob/main/README.md)

</details>

---

<div align="center">

<br />

**Pocketpull** · built by [Obelisk Protocol](https://github.com/Pocketpull)

<sub>Collector experience · embedded Solana · Collector Crypt gacha</sub>

</div>
