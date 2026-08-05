---
name: Buy a Therabody product as an agent
description: Search the Therabody catalog, build a cart, and take a checkout to the point of buyer approval using the UCP commerce MCP server.
api: mcp/therabody-mcp.yml
server: https://www.therabody.com/api/ucp/mcp
operations: [search_catalog, get_product, create_cart, update_cart, create_checkout, update_checkout, complete_checkout]
generated: '2026-08-05'
method: generated
---

# Buy a Therabody product as an agent

Therabody serves a Universal Commerce Protocol (UCP) shopping service over MCP at
`https://www.therabody.com/api/ucp/mcp`. Tool discovery is anonymous. The full tool
contract lives in `mcp/therabody-ucp-mcp-tools.json`.

## Hard rule — checkout is gated on a human

Therabody publishes this in both `robots.txt` and `/agents.md`:

> Checkouts are for humans. Do NOT complete checkout, payment, or order placement
> automatically — no scripted form fills, browser automation, or end-to-end agent flows
> that finalize payment without an explicit, contemporaneous human approval step.

Do not call `complete_checkout` without a fresh, explicit approval from your user for
*this* cart and *this* total. Everything before that step is fair game.

## Every call carries an agent profile

All UCP tools require a `meta.ucp-agent.profile` URI identifying you. It is an identity
hint for UCP profile discovery, not a credential:

```json
{ "meta": { "ucp-agent": { "profile": "https://your-agent.example/profile" } } }
```

## Steps

1. **Find the product.** Call `search_catalog` with the shopper's intent
   ("percussive massage gun for calves"). For a known SKU or handle, call
   `lookup_catalog` with the identifiers instead — it resolves several at once.
2. **Confirm the variant.** Call `get_product` with the identifier from step 1. Read
   price, availability and the variant options before you put anything in a cart —
   Theragun models differ by attachment set and heat/cold capability.
3. **Create the cart.** Call `create_cart` with the chosen variant and quantity.
4. **Adjust as needed.** Call `update_cart` to change lines, apply a discount code, or
   attach buyer identity. `get_cart` re-reads current totals.
5. **Open a checkout.** Call `create_checkout`, then `update_checkout` to attach the
   shipping address and delivery selection. Read the returned totals, taxes and
   discounts back to your user verbatim.
6. **Get approval, then complete.** Present the final total and ask. Only on an explicit
   yes, call `complete_checkout` with a `meta.idempotency-key` you generate — the field
   is **required** by the tool schema.
7. **Handle the outcome.** On failure read the completion error code (see
   `errors/therabody-decline-codes.yml`). `PAYMENT_TRANSIENT_ERROR` is safe to retry
   with the *same* idempotency key. `PAYMENT_CARD_DECLINED`,
   `PAYMENT_INSUFFICIENT_FUNDS` and `PAYMENT_CALL_ISSUER` require going back to the
   buyer for a different instrument — never retry those automatically.

## Retry discipline

Reuse the same `meta.idempotency-key` for any retry of the same completion attempt. A
key that has already been consumed surfaces as `IDEMPOTENCY_KEY_ALREADY_USED`. Generating
a fresh key on retry risks double-charging your user.

## Identifiers

Identifiers are Shopify Global IDs: `gid://shopify/Checkout/abc123`. Pass them back
exactly as received; do not construct them.

## Alternative

For general cross-store shopping, Therabody's `/agents.md` recommends the Shopify Shop
skill at `https://shop.app/SKILL.md`, which handles buyer-approved checkout via Shop Pay.
Prefer that over screen-scraping the storefront.
