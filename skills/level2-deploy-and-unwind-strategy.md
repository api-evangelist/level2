---
name: level2-deploy-and-unwind-strategy
description: >-
  Take a Level2 strategy from backtest to a live deployment and back out again safely. Use when an
  agent must rehearse, deploy, monitor or unwind an automated trading strategy on Level2.
api: Level2 Hub API
base_url: https://hub2.trylevel2.com
generated: '2026-09-17'
method: generated
source: >-
  openapi/level2-hub-controller-openapi.json;
  https://learn.trylevel2.com/docs/technical/virtual-trading;
  https://learn.trylevel2.com/docs/Broker/backtest
operations:
  - build_new_strategy_build_new_strategy_post
  - save_strategy_save_strategy_post
  - backtest_strategy_backtest_strategy_post
  - get_report_data_get_report_data_get
  - deploy_live_strategy_deploy_live_strategy_post
  - get_deploy_live_stats_get_deploy_live_stats_get
  - undeploy_live_strategy_undeploy_live_strategy_post
  - get_strategy_details_get_strategy_details_get
---

# Deploy a Level2 strategy — and be able to unwind it

**This skill moves real money once step 5 runs.** `deploy_live_strategy` places the strategy against
the trader's connected brokerage account. Do not skip the rehearsal steps.

## Steps

1. **Create or load the strategy.** `POST /build_new_strategy`
   (`build_new_strategy_build_new_strategy_post`), or `GET /get_strategy_details`
   (`get_strategy_details_get_strategy_details_get`) for one that already exists. Persist edits with
   `POST /save_strategy` (`save_strategy_save_strategy_post`).

2. **Backtest it.** `POST /backtest_strategy` (`backtest_strategy_backtest_strategy_post`). Level2
   documents two modes: Realtime Backtest over 1,500 historical datapoints (fast), and Comprehensive
   Backtest over more datapoints (slower).

3. **Read the report.** `GET /get_report_data` (`get_report_data_get_report_data_get`).

4. **Rehearse without capital.** Level2's own documentation warns that a backtest can be
   over-optimised to historic data and recommends virtual (paper) trading for a period before going
   live. Virtual trading routes orders to a free Alpaca account instead of the trader's real broker.
   Treat this as the mandatory dry run — there is no `dry_run` request flag anywhere in the contract,
   so this product mode is the only rehearsal available.

5. **Deploy live.** `POST /deploy_live_strategy` (`deploy_live_strategy_deploy_live_strategy_post`).

6. **Monitor.** `GET /get_deploy_live_stats` (`get_deploy_live_stats_get_deploy_live_stats_get`).

7. **Unwind when required.** `POST /undeploy_live_strategy`
   (`undeploy_live_strategy_undeploy_live_strategy_post`) is the documented inverse of step 5. For a
   multi-asset strategy use `undeploy_live_multi_asset_strategy`.

## What is NOT guaranteed

- **No reversal window is published.** The inverse operation exists, but Level2 states nowhere how
  long after an undeploy a position remains recoverable, or whether redeploying restores the same
  broker position. Never promise a user that a deploy can be taken back within some period — the
  provider has not said so.
- **No idempotency.** There is no `Idempotency-Key` header. A retried `POST /deploy_live_strategy`
  after a timeout may deploy twice. Always call `get_deploy_live_stats` to establish current state
  before retrying a deploy.
- **Errors.** `422` with `{ "detail": [ { "loc", "msg", "type" } ] }` is the only declared failure
  shape; there is no problem+json envelope and no declared 429.
