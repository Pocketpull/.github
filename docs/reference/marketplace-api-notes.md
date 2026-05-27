# Collector Crypt Marketplace API Notes (staging)

Last verified: 2026-04-09

## Hosts

- App host: `https://staging.collectorcrypt.com`
- API host: `https://staging-api.collectorcrypt.com`

The API host is embedded in the staging bundle env (`API_URL`), not inferred.

## What We Confirmed Live

## 1) Card feed endpoint (works unauthenticated)

`GET /marketplace?page=<n>&step=<n>[&filters...]`

Example:

- `GET https://staging-api.collectorcrypt.com/marketplace?page=1&step=12`

Observed response shape:

- `findTotal`, `total`, `totalPages`
- `cardsQtyByCategory` (map)
- `filterNFtCard` (array of card/listing rows)

Observed listing fragment inside each card:

- `listing.price`
- `listing.currency` (`SOL`/`USDC`)
- `listing.marketplace` (currently often `"ME"` in staging data)

## 2) Gemrate options endpoint (works unauthenticated)

`GET /cards/gemrate-options[?pokemon=<...>&sets=<...>]`

Example:

- `GET https://staging-api.collectorcrypt.com/cards/gemrate-options`

Observed response contains large arrays:

- `sets[]`
- `grades[]`
- `cardNames[]`
- `parallels[]`

## 3) V2 RPC endpoint exists

`POST /v2` with body:

```json
{ "method": "<methodName>", "params": { ... } }
```

Examples probed:

- `method=getUserEscrowBalance` -> `401 Unauthorized`
- `method=detectMarketplaceSource` with dummy params -> `200` and:
  - `{ "source": "NONE", "listing": null, "useV2": false }`

So at least some methods on `/v2` may be callable without auth, but escrow/account methods require auth.

## Source-map Discovered Method Map (high confidence)

From `staging` source maps (`shared/api/services/listings.ts` + `shared/api/urls/index.ts`):

- Legacy RPC (POST root): methods like:
  - `getBuyerOffers`
  - `getSellerOffers`
  - `getSellerListings`
  - `getCardOffers`
  - `createListTx`, `createQuickBuyTx`, etc.
- V2 RPC (POST `/v2`): methods like:
  - `createListTxV2`
  - `createQuickBuyTxV2`
  - `detectMarketplaceSource`
  - `getUserEscrowBalance`
  - `createDepositToUserEscrowTx`
  - `createWithdrawFromUserEscrowTx`
  - plus V2 offer/listing tx create/send methods.
- Broadcast endpoint:
  - `POST /broadcast` with `{ signedTransaction }` (used for secure tx broadcast).

## Marketplace Filter Query Contract

`/marketplace` query is built by helper `getFieldsRequestCards(payload, "marketplace")`.

Important params seen in helper:

- paging/search: `page`, `step`, `search`
- classification: `cardType`, `categories`, `blockchain`, `marketplaceSource`
- status/tags: `marketplaceStatus`, `marketplaceTags`
- value bands: `insuredValueMin`, `insuredValueMax`, `listPriceMin`, `listPriceMax`
- grading/year: `gradingCompany`, `grade`, `genericGradeMin`, `genericGradeMax`, `yearMin`, `yearMax`
- gemrate filters: `gemrateSet`, `gemrateGrade`, `gemrateCardName`, `gemrateSpecId`, `gemrateParallel`
- extras: `pokemon`, `favoritesOnly`, `followingOnly`, `followingOf`, `ownerAddress`, `orderBy`, etc.

## Migration Notes for Pocketpull

Current Pocketpull marketplace is local escrow endpoints:

- `GET /p2p/listings`
- `POST /p2p/listings`
- `POST /p2p/orders`
- `POST /p2p/orders/:id/release`

To migrate from ME-backed flow to CC staging API:

1. Read/listing browse:
   - Replace local listing feed with `GET /marketplace`.
2. Filter UI:
   - Map existing controls to `/marketplace` params above.
3. Offers/listings account panels:
   - Use RPC methods via `POST /v2` (or legacy root RPC as needed).
4. Buy/list/cancel tx flow:
   - Use create*TxV2 methods -> wallet sign -> `POST /broadcast`.
5. Source routing:
   - Use `detectMarketplaceSource` before purchase/update flows.

## Caveats

- No formal docs were provided; this file is inferred from:
  - live endpoint probes,
  - publicly accessible source maps.
- Some endpoints require auth and could not be fully schema-verified without session tokens.
- Staging data still includes `listing.marketplace = "ME"` in sample rows; migration may be in progress server-side.
