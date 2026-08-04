---
name: Find and price a ColdSnap product
description: Search the ColdSnap catalog for machines, pods or accessories and resolve a specific purchasable variant with its price and availability.
api: mcp/coldsnap-mcp.yml
surface: https://coldsnap.com/api/mcp
operations: [search_catalog, get_product_details]
graphql_equivalent: [products, search, predictiveSearch, product, variantBySelectedOptions]
generated: '2026-08-04'
method: generated
---

# Find and price a ColdSnap product

ColdSnap sells three things: the countertop frozen-treat machine, shelf-stable pods (ice
cream, non-dairy, frozen lattes, smoothies, protein shakes, boozy cocktails), and
accessories/merchandise. This skill takes a shopper's intent to a specific purchasable
variant.

## Endpoint

`POST https://coldsnap.com/api/mcp` — JSON-RPC 2.0, MCP protocol `2025-06-18`. **No
authentication.** Send `Accept: application/json, text/event-stream`.

## Steps

1. **Search.** Call `search_catalog`. At least one of `query` or `filters` must be
   supplied — a call with neither is rejected. Natural language works: "non-dairy pods",
   "commercial machine". Results are a limited first page.
2. **Page if needed.** The response carries an opaque cursor. Pass it back through
   `pagination.cursor` on the next `search_catalog` call. Do not construct cursors.
3. **Resolve the variant.** Call `get_product_details` with the `product_id` from the search
   result. `product_id` is the only required field. Without `options`, the first available
   variant is returned; pass `options` to select a specific one (pod flavor, pack size,
   machine bundle).
4. **Localize.** Pass `country` and `language` on `get_product_details` for correct pricing
   and availability. The store's `paymentSettings` report USD, US, and USD as the only
   enabled presentment currency — expect no other currency to price.
5. **Hand the variant to the cart skill.** See `coldsnap-build-a-cart.md`.

## Rules

- **Check the body, not the status code.** The GraphQL surface returns HTTP 200 even for a
  failed query, with a top-level `errors[]` array. The MCP surface returns JSON-RPC error
  objects.
- **Back off on 429.** The store documents per-IP rate limiting on the MCP endpoint in
  `/agents.md`. There are no `RateLimit-*` headers to read ahead of the limit.
- **Quote `x-request-id`.** Every response carries one; include it in any support contact.
- **The catalog has test residue.** The live product list includes `[Test] Product Redirect`
  and a duplicated `PRE-ORDER Margarita ... - OLD`. Do not surface these to a buyer.

## When to use GraphQL instead

Use `https://coldsnap.com/api/2026-07/graphql.json` when you need something no tool
exposes: collections as merchandising groupings, product tags and types, product
recommendations, selling plans, or metafields. Introspection is open and the SDL is at
`graphql/coldsnap-storefront.graphql`.

## See also

- `conventions/coldsnap-conventions.yml` — pagination, versioning, tracing, error envelopes
- `errors/coldsnap-problem-types.yml` — the error catalog
- `mcp/coldsnap-tool-crosswalk.yml` — which GraphQL fields back each tool
