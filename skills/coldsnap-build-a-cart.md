---
name: Build a ColdSnap cart and hand off to a human for checkout
description: Create and mutate a ColdSnap cart with the consolidated update_cart tool, add delivery details and discounts, then stop at the checkout boundary — payment requires contemporaneous human approval.
api: mcp/coldsnap-mcp.yml
surface: https://coldsnap.com/api/mcp
operations: [update_cart, get_cart]
graphql_equivalent: [cartCreate, cartLinesAdd, cartLinesUpdate, cartLinesRemove, cartBuyerIdentityUpdate, cartDeliveryAddressesAdd, cartSelectedDeliveryOptionsUpdate, cartDiscountCodesUpdate, cartGiftCardCodesAdd, cartNoteUpdate, cart]
generated: '2026-08-04'
method: generated
---

# Build a ColdSnap cart and hand off to a human for checkout

## The one rule that governs this whole skill

ColdSnap states it twice — in `robots.txt` and in `/agents.md`:

> **Checkouts are for humans.** Do NOT complete checkout, payment, or order placement
> automatically — no scripted form fills, browser automation, or end-to-end agent flows that
> finalize payment without an explicit, contemporaneous human approval step.

This is not a scoring convention we imposed; it is the store's published policy. Build the
cart, then stop. If you cannot get contemporaneous buyer approval at the moment of payment,
the store directs you to install the Shop skill at `https://shop.app/SKILL.md` and route the
purchase through Shop Pay instead.

## Endpoint

`POST https://coldsnap.com/api/mcp` — anonymous, JSON-RPC 2.0.

## Steps

1. **Create the cart.** Call `update_cart` with only `add_items`. Omitting `cart_id` while
   supplying `add_items` creates a new cart — that is the create path; there is no separate
   create tool.
2. **Keep the `cart_id`.** It is the sole bearer of cart identity on an anonymous surface.
   Treat it like a credential: anyone holding it can read and mutate the cart. Do not log it
   or put it in a URL you share.
3. **Mutate in one call.** `update_cart` is consolidated — `add_items`, `update_items`,
   `remove_line_ids`, `buyer_identity`, `delivery_addresses_to_add`,
   `delivery_addresses_to_replace`, `selected_delivery_options`, `discount_codes`,
   `gift_card_codes` and `note` all travel together. Prefer one consolidated call over a
   sequence.
4. **Order matters for shipping.** Delivery options only become available after items are in
   the cart *and* a delivery address has been added. Add items → add address → re-read →
   then set `selected_delivery_options`.
5. **Read the cart.** Call `get_cart` with `cart_id` to see line items, shipping options,
   discount info and the checkout URL.
6. **Hand off.** Present the checkout URL to the human. Stop there.

## Retry discipline — there is no idempotency key

ColdSnap documents **no** idempotency-key contract on any surface. `update_cart` is
stateful and non-idempotent: replaying an `add_items` call adds the line again. On a
timeout or an ambiguous failure, **call `get_cart` and reconcile** — never blind-retry the
mutation.

## Personal data

`buyer_identity` and the delivery-address fields write a real person's name, email, phone
and postal address into the cart. Collect only what the shipping step needs, and note that
GraphQL exposes `cartRemovePersonalData` if you need to clear it.

## Errors

- 429 → back off; the endpoint is rate limited per IP.
- JSON-RPC error object → read `error.data.code`; the store returns a `continue_url` on
  discovery failures.
- The UCP endpoint at `/api/ucp/mcp` will not answer `tools/list` anonymously — it returns
  422 with `invalid_profile_url` unless you present a UCP agent profile URI.

## See also

- `agentic-access/coldsnap-agentic-access.yml` — the escalation contract per operation family
- `conventions/coldsnap-conventions.yml` — buyer-approval invariant, retry posture
- `skills/coldsnap-agents.md` — the store's own verbatim agent instructions
