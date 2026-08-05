---
name: Check a Therabody order and request a return
description: Use the Therabody customer-account MCP server to look up order status, read store credit balances, and open a return request for a signed-in customer.
api: mcp/therabody-mcp.yml
server: https://account.therabody.com/customer/api/mcp
operations: [get_most_recent_order_status, get_order_status, get_store_credit_balances, request_return]
generated: '2026-08-05'
method: generated
---

# Check a Therabody order and request a return

Therabody runs a second, separate MCP server for post-purchase work at
`https://account.therabody.com/customer/api/mcp`. Tool discovery is anonymous, but every
tool here acts on a specific customer's data and requires authentication to execute.
The tool contract is captured in `mcp/therabody-customer-account-mcp-tools.json`.

## Authentication

Execution requires an OAuth 2.0 bearer token for a logged-in customer, sent in the
`Authorization` header. Get one through the authorization-code flow with PKCE:

- Authorization endpoint: `https://account.therabody.com/authentication/oauth/authorize`
- Token endpoint: `https://account.therabody.com/authentication/oauth/token`
- PKCE: `S256` (required)
- Scope needed for these tools: `customer-account-mcp-api:full`

Full discovery document: `well-known/therabody-openid-configuration.json`.
Scope reference: `scopes/therabody-scopes.yml`.

Never ask the user to paste an order number as a substitute for signing in — these tools
resolve the customer from the token, not from the argument.

## Steps

### Order status

- For "where is my order?" with no order number, call `get_most_recent_order_status`.
  It takes no arguments and resolves the most recent order for the token holder.
- When the user names an order, call `get_order_status` with `order_number` (string,
  required).

Read fulfillment state and tracking back plainly. Do not speculate about delivery dates
beyond what the tool returns.

### Store credit

Call `get_store_credit_balances` (no arguments) before quoting a price on a replacement
purchase — a customer may have a balance from an earlier return. It returns nothing when
the customer has no balance; treat that as zero, not as an error.

### Returns

Call `request_return`. Supply **either** `order_id` **or** `order_number` — one is
required — plus `line_items` identifying what is going back.

Before calling it:

1. Confirm the specific items and quantities with the user.
2. Tell them a return request is being opened on their account.

A return request is a state change on a real customer account. Treat it with the same
care as checkout: get an explicit go-ahead first, and do not retry a request that may
have already succeeded without first re-reading order status.

## Related

Catalog, cart and checkout live on a different server entirely — see
`skills/therabody-agentic-checkout.md`.
