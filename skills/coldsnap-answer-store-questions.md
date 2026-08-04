---
name: Answer questions about ColdSnap's policies, shipping and store facts
description: Use the MCP policy tool for natural-language store questions, and fall back to the GraphQL Shop policy fields and the store's page surface for the authoritative text.
api: mcp/coldsnap-mcp.yml
surface: https://coldsnap.com/api/mcp
operations: [search_shop_policies_and_faqs]
graphql_equivalent: [shop, page, pages]
generated: '2026-08-04'
method: generated
---

# Answer questions about ColdSnap's policies, shipping and store facts

## Steps

1. **Ask the tool.** Call `search_shop_policies_and_faqs` with a `query` — return policy,
   shipping policy, warranty, hours, phone number. An optional `context` field carries the
   conversation so far.
2. **Get the authoritative text when it matters.** The tool is retrieval-backed, not a
   direct field projection, so for anything a buyer will rely on, read the source:
   - GraphQL `shop { privacyPolicy termsOfService refundPolicy shippingPolicy
     subscriptionPolicy termsOfSale contactInformation legalNotice }` — each returns a
     `ShopPolicy` with `title`, `handle`, `url` and `body`.
   - The published policy URLs: `/policies/privacy-policy`, `/policies/terms-of-service`,
     `/policies/refund-policy`.
3. **Use the page surface for everything else.** ColdSnap publishes 38 pages, reachable via
   the GraphQL `pages` connection or `/pages/{handle}`. The ones agents actually need:
   `faq`, `customer-service`, `troubleshooting`, `nutritional-information`,
   `appliance-limited-1-year-warranty`, `refunds-and-returns-policy`,
   `terms-and-conditions-of-purchase`, `get-in-touch`.
4. **Vertical questions have their own pages.** ColdSnap sells into distinct segments and
   each has a page: restaurants and bars, hotels/hospitality and wellness, golf and sports
   clubs, colleges and universities, senior living, car dealerships, offices, micro markets,
   convenience stores, sports and entertainment.

## Rules

- **Quote, don't paraphrase, a policy.** Refunds, warranty and terms are commitments; give
  the buyer the text and the URL.
- **Note what is not a store policy.** Availability of the API surfaces is governed by the
  Shopify platform (`https://www.shopifystatus.com/`), not by ColdSnap — the store operates
  no status page and publishes no SLA.
- **There is no security contact.** `/.well-known/security.txt` returns 404 on both
  ColdSnap hosts. `robots.txt` gives `bots@shopify.com` as the contact for automated
  clients; that is a platform bot contact, not a vulnerability-disclosure channel.

## See also

- `lifecycle/coldsnap-lifecycle.yml` — versioning, deprecation, status, support routes
- `llms/coldsnap-llms.txt` — the store's own published page, collection and product index
