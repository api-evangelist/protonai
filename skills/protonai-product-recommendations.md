---
name: Serve AI product recommendations for a distributor cart
description: Use the Proton AI recommendation endpoints to surface reorder, cross-sell, and similar-item suggestions for a distributor's customer, and log the search that produced them.
api: openapi/protonai-openapi.yml
operations: [newRecommendations, completeCartRecommendations, boughtAlsoBought, similarItems, trackProductSearch]
---

# Serve AI product recommendations

Proton returns AI-ranked product recommendations for a given customer of a distributor tenant. All requests go to `https://api.proton.ai/{company}/...` with these headers:

- `X-Api-Key`: static key supplied by Proton (required)
- `X-Company`: tenant company slug (required; also the first path segment)
- `X-User-Id`: acting user id (optional, for attribution)

Recommendation reads take `customer_id`, `count`, and `user_id` as query parameters. There is no idempotency key; reads are safe to retry.

## Steps

1. **General recommendations** — call `newRecommendations` (`GET /{company}/product/recommendations?customer_id=&count=&user_id=`) for a customer's default recommended products.
2. **Cross-sell from a product** — call `boughtAlsoBought` (`GET /{company}/product/recommendations/bought`) or `similarItems` (`GET /{company}/product/recommendations/similar`) with a `product_id` to expand a cart line.
3. **Score a full cart** — POST the cart to `completeCartRecommendations` (`POST /{company}/product/recommendations/cart`) to get complete-the-cart suggestions for the current basket.
4. **Log the query** — call `trackProductSearch` (`POST /{company}/track`) so Proton's models learn from what was searched/shown.

## Notes

- Responses use the `{ "status_message": "...", "data": { ... } }` envelope (see `conventions/protonai-conventions.yml`).
- On `401` re-check the `X-Api-Key`; on `403` re-check the `X-Company` tenant matches the key's scope (`errors/protonai-problem-types.yml`).
