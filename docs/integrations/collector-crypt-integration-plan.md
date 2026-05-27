# Collector Crypt integration plan (Pocketpull `frontend/` + `backend/`)

## Implementation status (in repo)

- **Backend:** All documented Gacha endpoints are proxied under `/integrations/collector-crypt/*` with `COLLECTORCRYPT_API_BASE_URL` + `COLLECTORCRYPT_API_KEY` (see `backend/src/routes/collector-crypt.ts`, `backend/src/lib/collector-crypt.ts`).
- **Frontend:** `/gacha` — Phantom wallet, purchase → sign → submit, open pack, yolo, buyback, stock/status, `getNfts`, winners, purchased/gifted summaries (`frontend/src/pages/Gacha.tsx`); all traffic via `api()` → `VITE_API_URL`.
- **Marketplace** (vault/listings) is **not** wired here — still separate per Collector Crypt docs.

---

This document is the **live (non-mock)** integration blueprint for using **Collector Crypt** services from our Vite app and Fastify API. The **Gacha** stack is implemented as above; marketplace can follow the same proxy pattern when APIs are confirmed.

**Official docs (source of truth):** [docs.collectorcrypt.com](https://docs.collectorcrypt.com/) — in particular **Gacha → API Docs** for the HTTP API described below.

---

## 1. Does Collector Crypt have a marketplace?

**Yes.** Their documentation site includes a **Marketplace** section (e.g. overview, depositing cards, user guides, withdrawals). That is a **separate product track** from the **Gacha Machine REST API** (which is fully specified with `x-api-key`, JSON bodies, and Solana transaction flows).

For **this plan**, we treat:

| Area | What we know | Integration approach |
|------|----------------|----------------------|
| **Gacha** | Documented REST API: `generatePack`, `openPack`, `buyback`, `status`, `stock`, `getNfts`, `getAllWinners`, gifts, purchased packs, etc. ([Gacha API](https://docs.collectorcrypt.com/gacha/api)) | Primary target for a **thin vertical** (see §6). |
| **Marketplace** | Docs exist; flows are vaulting / trading oriented | **Phase later**: read Marketplace pages end-to-end, confirm whether integration is **on-chain programs**, **partner APIs**, or **operator-only**. Do not assume the same auth/header pattern as Gacha until verified. |

**Action:** Assign someone to read Marketplace + any “Program documentation” pages and list **concrete endpoints or program IDs** before scoping marketplace work.

---

## 2. Goals and constraints

### 2.1 Product goals

- Offer a **slice** of CC Gacha: e.g. **one pack type** on devnet first, then prod.
- Keep **Pocketpull** as the UX shell (navigation, branding, optional account/credits for non-crypto paths).
- Use CC for **truth** on pack purchase, open, NFT receipt, and optional **buyback**.

### 2.2 Technical constraints (non-negotiable)

1. **`COLLECTORCRYPT_*` API keys never ship to the browser.** Only `backend/` may attach `x-api-key` (or future auth headers).
2. **Gacha is Solana + USDC + wallet signatures.** Our current “USD credits + server-side open” model does **not** map 1:1 without a **bridge** (custodial wallet, sponsorship, or separate SKU). The plan assumes **explicit wallet connect** for the CC slice unless product defines a bridge.
3. **CORS:** Browser must call **our** API (`VITE_API_URL`), not `gacha.collectorcrypt.com` directly for privileged routes.
4. **Idempotency / memos:** CC uses **memo** strings to correlate purchase → open; persist memos server-side if we associate them with `users.id`.

---

## 3. Current architecture (baseline)

| Layer | Role today |
|-------|------------|
| `frontend/` | Vite, React, `api()` → `VITE_API_URL` with cookies. |
| `backend/` | Fastify, sessions, Postgres, ledger, `/opens/pack` simulation for “credits” pulls. |

**Integration adds:** a **Collector Crypt adapter** in the backend and **new frontend flows** that use wallet + CC-backed endpoints.

---

## 4. Target architecture

```
[Browser]
  ├─ Pocketpull UI (existing)
  ├─ Solana wallet adapter (new) — devnet/mainnet per env
  └─ HTTPS → only Pocketpull API (cookies + JSON)

[backend/]
  ├─ Existing auth, DB, CORS
  ├─ New: CollectorCryptClient (fetch to CC base URL + x-api-key)
  ├─ New routes: /integrations/collector-crypt/... (proxy + normalize errors)
  └─ Optional: store memo ↔ user mapping, audit logs

[Collector Crypt]
  └─ gacha.collectorcrypt.com (confirm dev vs prod base URL in CC docs / dashboard)
```

**Why proxy?** Hides API key, centralizes logging, lets us attach Pocketpull `userId` to CC operations for support and analytics.

---

## 5. Environment and configuration

### 5.1 Backend (`backend/.env`)

Add (names illustrative — align with CC dashboard):

| Variable | Purpose |
|----------|---------|
| `COLLECTORCRYPT_API_BASE_URL` | e.g. `https://gacha.collectorcrypt.com` — **confirm** devnet host if different from prod. |
| `COLLECTORCRYPT_API_KEY` | Devnet or prod partner key; **server-only**. |
| `COLLECTORCRYPT_NETWORK` | `devnet` \| `mainnet-beta` (for validation messages to frontend). |

Optional:

| Variable | Purpose |
|----------|---------|
| `COLLECTORCRYPT_DEFAULT_PACK_TYPE` | e.g. `pokemon_50` for first slice. |
| `COLLECTORCRYPT_WEBHOOK_SECRET` | Only if CC sends webhooks to us later. |

### 5.2 Frontend (`frontend/.env`)

**Do not** add the API key. Only:

| Variable | Purpose |
|----------|---------|
| `VITE_API_URL` | Unchanged. |
| `VITE_SOLANA_NETWORK` or `VITE_COLLECTOR_CRYPT_ENABLED` | Feature flag / network hint for wallet UI. |

---

## 6. Phased implementation (recommended “small slice”)

### Phase A — Foundation (backend only)

**Deliverables**

- [ ] `collectorCryptFetch(path, init)` helper: sets `Content-Type`, `x-api-key`, timeouts, structured errors.
- [ ] Health/read-only routes (no user wallet yet):
  - `GET /integrations/collector-crypt/status` → CC `GET /api/status`
  - `GET /integrations/collector-crypt/stock` → CC `GET /api/stock`
- [ ] `GET /integrations/collector-crypt/nfts?code=...&rarity=...` → CC `GET /api/getNfts` (query passthrough with allowlist).
- [ ] Unit smoke: devnet key works from server.

**Acceptance:** curl/Postman against Pocketpull API returns CC-shaped JSON; key never exposed.

### Phase B — “Recent winners” / marketing (optional but cheap)

**Deliverables**

- [ ] `GET /integrations/collector-crypt/winners?since=ISO&slug=&limit=` → CC `GET /api/getAllWinners`
- [ ] Frontend home or rewards rail: consume normalized DTO (wallet addresses may be truncated for privacy).

**Acceptance:** Live data on landing without wallet connect.

### Phase C — Core Gacha slice (wallet required)

**User story:** Connect wallet → buy one pack type → sign txs → open pack → see NFT in UI.

**Backend**

- [ ] `POST /integrations/collector-crypt/packs/generate`  
  Body: `{ playerAddress, packType?, turbo? }`  
  Proxies to CC `POST /api/generatePack`; returns `{ memo, transaction }` (base64) **or** error mapping.
- [ ] `POST /integrations/collector-crypt/transactions/submit`  
  Body: `{ signedTransaction }` → CC `POST /api/submitTransaction`.
- [ ] `POST /integrations/collector-crypt/packs/open`  
  Body: `{ memo }` → CC `POST /api/openPack`; return normalized `{ nft, rarity, metadata, ... }`.
- [ ] Optional: persist `{ userId, memo, createdAt }` when `auth/me` is present (requires session).

**Frontend**

- [ ] Add Solana wallet provider (e.g. `@solana/wallet-adapter-react` + Phantom).
- [ ] New route or subflow: `PackOpenCC` or extend `Reveal` behind feature flag:
  1. Call `generate` → deserialize tx → **user signs** in wallet → `submit`.
  2. Poll or wait confirmation (wallet/web3 commitment).
  3. Call `open` with `memo`.
  4. Map response to a **Reveal** UI (image, name, rarity from `nftWon` / metadata).
- [ ] Devnet: link to USDC faucet and CC dev UI from docs ([dev-gacha](https://dev-gacha.collectorcrypt.com), SPL faucet per CC gacha page).

**Acceptance:** End-to-end on devnet with test wallet + devnet USDC.

### Phase D — Buyback (optional)

- [ ] `POST /integrations/collector-crypt/buyback/prepare` → CC `POST /api/buyback`
- [ ] `GET /integrations/collector-crypt/buyback/check?memo=...` → CC `GET /api/buyback/check`
- [ ] UI: 24h window messaging from CC docs.

### Phase E — YOLO / gifts / purchased packs (later)

- Implement `generateYoloPacks`, `generateGift`, `getGifted`, `purchasedPacks`, `generatePurchasedPack`, `usePurchasedPack` only when product needs them — each adds UX and edge cases (Ledger, turbo, etc.).

---

## 7. API mapping (CC → Pocketpull)

| Collector Crypt | Pocketpull backend route (proposed) | Notes |
|-----------------|-------------------------------------|--------|
| `POST /api/generatePack` | `POST .../packs/generate` | Requires `playerAddress`. |
| `POST /api/submitTransaction` | `POST .../transactions/submit` | After user signs. |
| `POST /api/openPack` | `POST .../packs/open` | Body `{ memo }`. |
| `POST /api/buyback` | `POST .../buyback/prepare` | |
| `GET /api/buyback/check` | `GET .../buyback/check` | |
| `GET /api/status` | `GET .../status` | |
| `GET /api/stock` | `GET .../stock` | |
| `GET /api/getNfts` | `GET .../nfts` | Allowlist query params. |
| `GET /api/getAllWinners` | `GET .../winners` | |

Normalize errors: map CC HTTP codes to stable `{ error: string, code?: string }` for the frontend.

---

## 8. Data model and coexistence with today’s “pulls”

- Today’s `PullItem` / `/opens/pack` is **our** simulation tied to ledger credits.
- CC pulls produce **on-chain NFTs** + metadata; different shape.

**Recommendation**

- Introduce **`CollectorCryptPull`** (or `OnChainPull`) TypeScript type in `frontend/src/shared` / `backend/src/shared` for CC responses.
- If product needs both credit-based reveal and CC reveal, use an explicit **feature flag** or route split (e.g. `/reveal/:slug?source=cc` vs default credit pull).

---

## 9. Security checklist

- [ ] API key only in backend env; rotate if leaked.
- [ ] Rate-limit new integration routes stricter than generic `/` if needed.
- [ ] Validate `playerAddress` as valid Solana pubkey before proxying.
- [ ] Never log full API key or signed transactions in production logs (truncate).
- [ ] CORS: only `WEB_ORIGIN` for browser.

---

## 10. Testing strategy

| Layer | What to test |
|-------|----------------|
| Unit | CC client error parsing; param allowlists. |
| Integration | Against CC devnet with dev key; use [dev-gacha](https://dev-gacha.collectorcrypt.com) as reference. |
| E2E | Manual: Phantom devnet + USDC faucet → one full pack flow. |

---

## 11. Ownership and milestones (suggested)

| Milestone | Owner | Output |
|-----------|-------|--------|
| M1 | Backend | Client + status/stock/nfts proxy |
| M2 | Frontend | Wallet connect + generate/submit/open UI slice |
| M3 | Full stack | Buyback + persistence (memo/user) |
| M4 | PM + eng | Marketplace doc review + separate RFC |

---

## 12. References

- [Collector Crypt — Gacha API](https://docs.collectorcrypt.com/gacha/api) — endpoints, `x-api-key`, flows, devnet links.
- [Collector Crypt — Introduction](https://docs.collectorcrypt.com/) — full doc tree including Marketplace, Swap, etc.
- [Gacha starter demo (GitHub)](https://github.com/daxherrera/gacha-starter) — reference implementation.

---

*This file is scoped to implementation planning for `frontend/` and `backend/` in this repo. Update phases as Collector Crypt ships API changes or as Pocketpull product defines credit-vs-wallet bridging.*
