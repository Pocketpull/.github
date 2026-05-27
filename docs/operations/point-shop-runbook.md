# Point Shop runbook

Operations for Pocket Points redemptions, benefit consumption, and Collector Crypt partner fee injection.

**Full runbook:** [docs/ops-store-benefits-runbook.md](../../docs/ops-store-benefits-runbook.md)

## Tables

| Table | Purpose |
| ----- | ------- |
| `store_items` | Catalog: PP cost, `shop_category`, `benefit_kind`, `benefit_value_bps`, `max_uses` |
| `store_redemptions` | Purchase rows; `redemption_status`: `active` \| `consumed` \| `cancelled` |
| `store_benefit_consumption` | Idempotent ledger when shipping/buyback benefit applied |

## Active benefits query

```sql
SELECT sr.id, sr.remaining_uses, si.title, si.shop_category, si.benefit_value_bps
FROM store_redemptions sr
JOIN store_items si ON si.id = sr.item_id
WHERE sr.user_id = '<uuid>'
  AND sr.redemption_status = 'active'
  AND sr.remaining_uses > 0;
```

Same shape as `GET /me/store-benefits`.

## Redeem flow

1. User: `POST /gamification/store/redeem`  
2. PP deducted, `store_redemptions` inserted  
3. Fulfillment at ship/buyback/marketplace time should decrement `remaining_uses` — **verify** per benefit kind (known gap in [rewards-system-spec](../../docs/rewards-system-spec-and-tasks.md))

## CC integration env

| Variable | Purpose |
| -------- | ------- |
| `REWARDS_CC_OPEN_PP_ENABLED` | PP + pack quests on CC `openPack` success |
| `REWARDS_CC_MARKETPLACE_FEE_INJECT` | Merge active marketplace_fee bps into CC `POST /v2` |
| `REWARDS_CC_MARKETPLACE_FEE_METHODS` | Allowed CC methods for injection |
| `COLLECTORCRYPT_MARKETPLACE_FEE_PARAM_KEY` | Default `partnerFeeDiscountBps` |

Coordinate with Collector Crypt for stable partner fee param names.

## Product rules

- Shipping rewards: **one shipment** per redemption  
- No PP from free packs  
- See [rewards.md](../reference/rewards.md) for catalog PP costs
