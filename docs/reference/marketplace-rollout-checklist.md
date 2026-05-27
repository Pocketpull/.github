# Collector Crypt Marketplace Rollout Checklist

## Staging Validation
- Call `GET /integrations/collector-crypt/marketplace/rollout-health` and confirm configured route + allowlist.
- Verify listing feed loads from `GET /integrations/collector-crypt/marketplace`.
- Validate wallet-signed list flow (`createListTxV2` -> sign -> `/broadcast`).
- Validate wallet-signed buy flow (`detectMarketplaceSource` + `createQuickBuyTxV2` -> sign -> `/broadcast`).
- Confirm query refetch after each action updates UI state.

## Metrics and Compatibility
- Proxy logs emit `cc_marketplace_proxy` records for each marketplace request.
- Client posts `/integrations/collector-crypt/marketplace/volume-event` after successful buy/list.
- Buy events continue updating `users.marketplace_volume_cents` for leaderboard compatibility.

## Rollout Gates
- Gate public launch behind environment readiness (CC keys + cluster routing).
- Run internal-wallet smoke tests first on staging and production.
- Keep previous marketplace tables intact during stabilization window for rollback safety.
