---
name: level2-onboard-broker-user
description: >-
  Register a brokerage customer into the Level2 platform and confirm the account is usable, using the
  Level2 Broker API. Use when a broker partner needs to provision, correct or audit an end-user
  account on Level2.
api: Level2 Hub API
base_url: https://hub2.trylevel2.com
generated: '2026-09-17'
method: generated
source: >-
  openapi/level2-hub-controller-openapi.json;
  https://learn.trylevel2.com/docs/Broker/API/authentication;
  https://learn.trylevel2.com/broker_apis.json
operations:
  - register_broker_user_broker_register_user_post
  - broker_get_registered_users_broker_get_registered_users_get
  - broker_update_user_email_broker_update_user_email_put
  - broker_reset_user_password_broker_reset_user_password_put
  - get_user_strategies_broker_get_user_strategies_get
---

# Onboard a broker user onto Level2

## Before you start

Mint a token. The Broker API takes an HS256 JSON Web Token that **you** generate with the shared
secret Level2 issued you, carrying a `domain` claim and a `token_expiry` claim. Level2 automatically
invalidates any token claiming more than **180 minutes** of validity, so mint for 180 minutes or less
and re-mint rather than extending. Send it as `Authorization: Bearer <jwt>` on every request.

Base URL is `https://hub2.trylevel2.com`. There is no sandbox host and no test-mode key — these calls
hit production.

## Steps

1. **Register the user.** `POST /broker/register_user`
   (`register_broker_user_broker_register_user_post`). The body is
   `application/x-www-form-urlencoded`, not JSON.

2. **Confirm the registration landed.** `GET /broker/get_registered_users`
   (`broker_get_registered_users_broker_get_registered_users_get`) and look for the address you just
   registered. This endpoint is paginated with `page` and `size`, and returns the
   `{items,total,page,size,pages}` envelope — read `total`, do not assume page 1 is the whole set.

3. **Correct the address if it is wrong.** `PUT /broker/update_user_email?user_email=<old>&new_user_email=<new>`
   (`broker_update_user_email_broker_update_user_email_put`). The broker surface addresses users by
   **email**, not by id, so a mistyped address is the identifier — fix it before anything else
   references the account.

4. **Reset a password on request.** `PUT /broker/reset_user_password?user_email=<email>&new_password=<pw>`
   (`broker_reset_user_password_broker_reset_user_password_put`).

5. **Verify the account is usable.** `GET /broker/get_user_strategies?user_email=<email>`
   (`get_user_strategies_broker_get_user_strategies_get`). An empty paginated result is the correct
   answer for a brand-new user; an error here means the account is not wired up.

## Rules that apply to every call

- **No idempotency key exists.** There is no `Idempotency-Key` header on any Level2 operation. Do not
  blind-retry `POST /broker/register_user` on a timeout — call `GET /broker/get_registered_users`
  first and check whether the user is already there.
- **Errors are FastAPI validation errors, not RFC 9457.** The only declared failure is `422` with
  `{ "detail": [ { "loc", "msg", "type" } ] }`. Read `detail[].loc` to find the offending parameter.
  Nothing in the contract declares what a `401` or an expired token looks like — treat any non-200,
  non-422 response as an auth or transport problem and re-mint the token before retrying.
- **No rate-limit headers are published.** Nothing tells you how much headroom is left, so pace
  bulk onboarding conservatively and back off on any non-200.
