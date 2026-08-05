---
name: Query the Therabody storefront with GraphQL
description: Read the Therabody product catalog, collections, content and shop configuration directly from the Shopify Storefront GraphQL API served on the Therabody host.
api: graphql/therabody-storefront.graphql
endpoint: https://www.therabody.com/api/2026-04/graphql.json
operations: [products, product, productByHandle, collections, search, predictiveSearch, shop, paymentSettings, blogs, articles, pages]
generated: '2026-08-05'
method: generated
---

# Query the Therabody storefront with GraphQL

Therabody serves a Shopify Storefront GraphQL API from its own host at
`https://www.therabody.com/api/{version}/graphql.json`. During probing on 2026-08-05,
anonymous introspection and anonymous queries both returned HTTP 200 without a Storefront
access token. The full SDL — 416 types, 35 query root fields, 41 mutations — is captured
in `graphql/therabody-storefront.graphql`.

Use this surface for **reading**. For agent-driven cart and checkout, prefer the UCP MCP
server (`skills/therabody-agentic-checkout.md`), which is the interface Therabody
actually advertises to agents in `robots.txt` and `/agents.md`.

## Versioning

The version is a date-based path segment. Supported at time of capture: `2025-10`,
`2026-01`, `2026-04`, `2026-07` (latest). Pin a version explicitly — do not rely on a
default. Confirm the current train yourself:

```graphql
{ publicApiVersions { handle supported displayName } }
```

See `lifecycle/therabody-lifecycle.yml`.

## Steps

1. **Identify the shop.** `shop { name primaryDomain { url } }` and
   `paymentSettings { currencyCode acceptedCardBrands supportedDigitalWallets }`.
2. **Browse products.** Use the `products` connection with cursor pagination, or
   `product` / `productByHandle` for a single item.
3. **Search.** `search` for full-text queries, `predictiveSearch` for typeahead.
4. **Merchandising.** `collections`, `collectionByHandle` and `productRecommendations`
   for category and related-item browsing.
5. **Content.** `blogs`, `articles`, `pages` and `menu` cover the editorial surface —
   Therabody publishes a substantial recovery-and-wellness content library this way.

## Pagination

Every list is a Relay-style connection. Page with `first`/`after` (or `last`/`before`)
and follow `pageInfo.hasNextPage` + `pageInfo.endCursor`. Never assume a fixed page size.

```graphql
{ products(first: 20, after: $cursor) {
    edges { cursor node { id title handle } }
    pageInfo { hasNextPage endCursor } } }
```

## Rate limiting

This API is metered by **calculated query cost**, not request count. Every response
carries an `extensions.cost` object with `requestedQueryCost`. Read it and back off on
cost, not on a header — there is no `X-RateLimit-*` signal here. Request only the fields
you need; deep nested connections get expensive fast.

## Errors

Mutations return a payload with a typed `userErrors[]` carrying a `code` enum, a `field`
path and a `message`; transport and validation failures land in the top-level `errors[]`.
The full code vocabulary — 197 codes across 7 enums — is in
`errors/therabody-problem-types.yml`.

## Identifiers

Global IDs of the form `gid://shopify/Product/123`. Resolve arbitrary ones with `node`
or `nodes`.
