# Ops runbook — Point Shop redemptions & benefits

## Tables

- `store_items` — catalog (PP cost, `shop_category`, `benefit_kind`, `benefit_value_bps`, `max_uses`).
- `store_redemptions` — one row per purchase; `redemption_status` `active` | `consumed` | `cancelled`; `remaining_uses` decremented on fulfillment.
- `store_benefit_consumption` — idempotent ledger rows when shipping or buyback boost is applied to an `inventory_items` row.

## Useful queries

**Active benefits for a user** (same payload as `GET /me/store-benefits`):

```sql
SELECT sr.id, sr.remaining_uses, si.title, si.shop_category, si.benefit_value_bps
FROM store_redemptions sr
JOIN store_items si ON si.id = sr.item_id
WHERE sr.user_id = '<uuid>'
  AND sr.redemption_status = 'active'
  AND sr.remaining_uses > 0;
```

**Consumption history for an inventory item**:

```sql
SELECT * FROM store_benefit_consumption
WHERE inventory_item_id = '<uuid>'
ORDER BY created_at;
```

## Collector Crypt API — partner fields (discovery)

Collector Crypt does **not** publish a stable public JSON contract for partner marketplace fee overrides or buyback quotes in this repo. Coordinate with CC for the exact `POST /v2` `params` key (Pocketpull default env: `COLLECTORCRYPT_MARKETPLACE_FEE_PARAM_KEY` = `partnerFeeDiscountBps`) and for which methods accept it.

## Env — CC rewards integration

| Variable | Purpose |
|----------|---------|
| `REWARDS_CC_OPEN_PP_ENABLED` | `true` to grant Pocket Points + `pack_opens` quests on successful CC `openPack` (idempotent per memo via `won_nfts`). |
| `REWARDS_CC_MARKETPLACE_FEE_INJECT` | `true` to merge active `marketplace_fee` bps into marketplace `POST /v2` `params`. |
| `REWARDS_CC_MARKETPLACE_FEE_METHODS` | Comma-separated CC method names allowed for injection (empty = no injection). |
| `REWARDS_CC_MARKETPLACE_FEE_CONSUME_METHODS` | Comma-separated methods where a **successful** `/v2` response decrements the redemption (use **narrow** `send*` methods — not tx builders). |
| `COLLECTORCRYPT_MARKETPLACE_FEE_PARAM_KEY` | JSON key under `params` for the bps value (partner-confirmed). |
| `REWARDS_CC_BUYBACK_BOOST_DISPLAY` | `true` to append `pocketpullBuybackBoost` to proxied `POST /buyback` JSON when a boost redemption is active and a quote is parsed. |

**Correlation:** Open PP uses `cc_generate_pack` `platform_events` rows matched by `memo` on `request_summary` / `response_summary` for paid amount (`amount_usdc_cents` / price fields). If correlation fails, check logs for `cc_open_gamification_spend_unknown`.

**Marketplace consumption:** Rows in `store_benefit_consumption` with `kind = marketplace_fee_v2` and `external_ref` set (e.g. signature) — idempotent per `external_ref`.

## Marketplace fee SKUs

Fee discounts are stored like other redemptions. With injection off or upstream rejecting unknown keys, apply discounts manually at settlement and use `GET /me/store-benefits` for the active `marketplace_fee` row (`benefit_value_bps`: 25 = 0.25%, 100 bps = 1%).
