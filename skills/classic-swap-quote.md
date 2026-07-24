---
name: Get and track a ShapeShift swap
description: Discover a supported asset pair, fetch indicative rates, get an executable quote, and track the swap to completion using the ShapeShift Public API.
api: openapi/classic-shapeshift-openapi-original.json
operations: [listAssets, getAssetById, getSwapRates, getSwapQuote, getSwapStatus]
---

# Get and track a ShapeShift swap

Use the ShapeShift Public API (`https://api.shapeshift.com`, base path `/v1`) to
quote and track a crypto swap. Read endpoints are public — no auth required.

## Steps

1. **Resolve the assets.** Assets are identified with CAIP-19 ids (e.g.
   `eip155:1/slip44:60` for ETH). Use `listAssets` (offset paginated via
   `limit`/`offset`, filter by `chainId`) or `getAssetById` to confirm the
   `sellAssetId` and `buyAssetId` you intend to trade.
2. **Get indicative rates.** Call `getSwapRates` with `sellAssetId`,
   `buyAssetId`, `sellAmountCryptoBaseUnit` (amount as a base-unit decimal
   string) and optional `slippageTolerancePercentageDecimal`. Pass your
   `X-Partner-Code` header for attribution.
3. **Get an executable quote.** Call `getSwapQuote` (POST) for a firm
   `QuoteResponse` — it returns a `quoteId` (uuid), `swapperName`, `rate`, and
   `buyAmountAfterFeesCryptoBaseUnit`. Quotes are priced at request time; there
   is no idempotency key, so re-request rather than replay.
4. **Track the swap.** After execution, poll `getSwapStatus` with the `quoteId`
   or the on-chain `txHash` until it reaches a terminal status.

## Rules

- **Rate limits:** on HTTP 429 (`code: RATE_LIMIT_EXCEEDED`) honor the
  `Retry-After` header and back off; watch `RateLimit-Remaining`.
- **Errors:** envelope is `{ "error": string, "code": string }` (not RFC 9457).
  Handle 400 (bad params), 404 (unknown asset/quote), 409 (status conflict).
- **Amounts** are crypto base units as decimal strings — never floats.
