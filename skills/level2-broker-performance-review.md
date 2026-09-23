---
name: level2-broker-performance-review
description: >-
  Pull the execution-quality and engagement picture for a broker partner's Level2 users — strategies,
  deployments, trades, backtests, latency and slippage. Use when a broker needs a periodic review of
  how its customers are performing on Level2.
api: Level2 Hub API
base_url: https://hub2.trylevel2.com
generated: '2026-09-17'
method: generated
source: openapi/level2-hub-controller-openapi.json; https://learn.trylevel2.com/broker_apis.json
operations:
  - broker_get_registered_users_broker_get_registered_users_get
  - get_user_strategies_broker_get_user_strategies_get
  - broker_get_all_user_deployments_broker_get_all_user_deployments_get
  - broker_get_all_trades_by_user_deployments_broker_get_all_trades_by_user_deployments_get
  - broker_get_all_trades_by_user_strategy_broker_get_all_trades_by_user_strategy_get
  - broker_get_recent_backtest_details_broker_get_recent_backtest_details_get
  - get_broker_user_strategy_performance_broker_get_user_strategy_performance_get
  - get_broker_user_latency_by_email_broker_get_broker_user_latency_by_email_get
  - get_broker_user_slippage_by_email_broker_get_broker_user_slippage_by_email_get
  - broker_get_all_user_scanners_broker_get_all_user_scanners_get
---

# Review how a broker's users are performing on Level2

All of this is read-only. Mint an HS256 JWT (180 minutes maximum) and send it as
`Authorization: Bearer <jwt>` against `https://hub2.trylevel2.com`.

## Steps

1. **Enumerate the cohort.** `GET /broker/get_registered_users`
   (`broker_get_registered_users_broker_get_registered_users_get`). Paginated — walk `page`/`size`
   until you have collected `total` items. Users are keyed by **email** on this surface.

2. **Per user, list what they built.**
   - Strategies: `GET /broker/get_user_strategies?user_email=<email>`
     (`get_user_strategies_broker_get_user_strategies_get`)
   - Scanners: `GET /broker/get_all_user_scanners?user_email=<email>`
     (`broker_get_all_user_scanners_broker_get_all_user_scanners_get`)
   - Deployments: `GET /broker/get_all_user_deployments?user_email=<email>`
     (`broker_get_all_user_deployments_broker_get_all_user_deployments_get`)

3. **Pull the trades.** By deployment,
   `GET /broker/get_all_trades_by_user_deployments?user_email=<email>`
   (`broker_get_all_trades_by_user_deployments_broker_get_all_trades_by_user_deployments_get`); by
   strategy, `GET /broker/get_all_trades_by_user_strategy?strategy_keygen=<uuid>`
   (`broker_get_all_trades_by_user_strategy_broker_get_all_trades_by_user_strategy_get`). Note the
   change of key: this one takes the strategy's `keygen` UUID, not its numeric id and not an email.

4. **Compare intent against outcome.**
   `GET /broker/get_recent_backtest_details?strategy_keygen=<uuid>`
   (`broker_get_recent_backtest_details_broker_get_recent_backtest_details_get`) gives the backtested
   expectation; `GET /broker/get_user_strategy_performance?strategy_id=<id>&deployment_id=<id>`
   (`get_broker_user_strategy_performance_broker_get_user_strategy_performance_get`) gives the
   realised one.

5. **Measure execution quality.**
   `GET /broker/get_broker_user_latency_by_email?user_email=<email>&limit=50`
   (`get_broker_user_latency_by_email_broker_get_broker_user_latency_by_email_get`) and
   `GET /broker/get_broker_user_slippage_by_email?user_email=<email>&limit=50`
   (`get_broker_user_slippage_by_email_broker_get_broker_user_slippage_by_email_get`). These are the
   two numbers that say whether the broker's own fill quality, not the strategy, is the problem.

## Cautions

- **Three identifier styles.** The broker surface uses `user_email`; trades-by-strategy uses the
  `keygen` UUID; performance uses numeric `strategy_id` and `deployment_id`. Carry all three when you
  join the data.
- **No rate-limit signal.** Nothing in the contract or docs publishes a limit, a header or a 429. A
  per-user fan-out across a large cohort is many hundreds of calls — pace it and back off on any
  non-200.
- **Read-only.** Nothing in this skill writes. The mutating siblings on the same surface
  (`update_user_strategy`, `update_user_deployment`, `update_marketplace_strategy`) take a boolean
  `should_delete` / `should_publish` flag and are deliberately not used here.
