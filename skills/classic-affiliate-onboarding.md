---
name: Onboard a ShapeShift affiliate and read stats
description: Authenticate with Sign-In With Ethereum, register an affiliate/partner, and read attributed swap stats on the ShapeShift Public API.
api: openapi/classic-shapeshift-openapi-original.json
operations: [siweNonce, siweVerify, createAffiliate, getAffiliate, getAffiliateStats, getAffiliateSwaps]
---

# Onboard a ShapeShift affiliate and read stats

Register as an affiliate/partner and read your attributed swap volume. Affiliate
create/update requires authentication; stats/reads follow.

## Steps

1. **Authenticate (SIWE / EIP-4361).** Call `siweNonce` (POST) to get a nonce.
   Have the wallet sign the SIWE message containing that nonce, then call
   `siweVerify` (POST) — it returns `{ token, address }` where `token` is a JWT.
2. **Register.** Call `createAffiliate` (POST) with the JWT as
   `Authorization: Bearer <token>`. A 201 confirms registration; 409 means the
   affiliate already exists (fetch it with `getAffiliate` by `address`).
3. **Configure.** Use `updateAffiliate` (PATCH `/v1/affiliate/{address}`, bearer
   auth) to change the affiliate config. 401/403 mean the token is missing or
   not permitted for that address.
4. **Read performance.** `getAffiliateStats` returns aggregate stats (filter by
   `partnerCode`, `startDate`, `endDate`). `getAffiliateSwaps` lists the
   underlying swaps — cursor paginated via `limit` + `cursor`.

## Rules

- **Auth:** the JWT from `siweVerify` is the bearer credential; only
  create/update affiliate need it, reads of stats/swaps do too when scoped to an
  address you own.
- **Attribution:** downstream swap calls carry `X-Partner-Code`; those partner
  codes resolve via `resolvePartner`.
- **Rate limits / errors:** same envelope and 429/`Retry-After` behavior as the
  rest of the API (see conventions/classic-conventions.yml).
