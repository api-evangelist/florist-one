---
name: Browse and price Florist One flowers
description: List flower or gift-basket products, confirm a delivery date for the recipient's ZIP code, and get an authoritative order total before anything is charged. This is the whole read-only surface and the only part of Florist One an agent can safely exercise.
api: openapi/florist-one-flowershop-api-openapi.yml
operations:
  - GET /flowershop/getproducts
  - GET /flowershop/checkdeliverydate
  - GET /flowershop/gettotal
  - GET /giftbaskets/getproducts
  - GET /giftbaskets/gettotal
generated: '2026-09-10'
method: generated
source: openapi/ in this repo, plus https://www.floristone.com/api/how-it-works/ and Florist One's published sample code
---

# Browse and price Florist One flowers

Base URL: `https://www.floristone.com/api/rest`. `/api/` on its own is the marketing site,
not an API host.

## Before you call

- Authenticate with HTTP Basic: your Florist One API Key is the username, your assigned
  password is the password. Florist One's own samples send
  `Authorization: <base64(key:password)>` **without** the `Basic ` prefix. If a standard
  basic-auth helper returns 403, try the raw form before assuming the key is wrong.
- There is no sandbox and no test key. Every call is a live call against production. The
  operations in this skill are all reads, which is why they are safe to run; do not extend
  this skill with `placeorder`.
- One credential grants the entire surface, including order placement. There is no
  read-only key. Treat the credential accordingly.
- Responses are JSON with **UPPERCASE** keys. Read `PRODUCTS`, `CODE`, `PRICE`,
  `SUBTOTAL`, `ORDERTOTAL` — not their lowercase spellings.

## Steps

1. **List products.** `GET /flowershop/getproducts?category=<code>&count=<n>&start=<n>`, or
   `?code=<sku>` for one product. Paging is offset-based on `start` (1-based) and `count`,
   and no total or cursor is returned — you know you have reached the end only when a page
   comes back short. For gift baskets use `GET /giftbaskets/getproducts` with the same
   parameters.
   Each product carries `CODE`, `NAME`, `DESCRIPTION`, `PRICE` and image URLs in `SMALL`,
   `THUMBNAIL` and `IMAGE`.
2. **Confirm delivery is possible.** `GET /flowershop/checkdeliverydate?zipcode=<zip>` and
   optionally `&date=<YYYY-MM-DD>`. Florist One delivers every day except Sundays and
   holidays, and only within the United States and Canada. Never present a delivery date to
   a user that this call did not return.
3. **Price the basket.** `GET /flowershop/gettotal?products=<json>` where `products` is a
   URL-encoded JSON array of `{code, price, recipient:{zipcode}}` objects. Optional
   `affiliateservicecharge` and `masterservicecharge` add your own service charges.
   Read `SUBTOTAL` and `ORDERTOTAL` from the response — do not compute the total yourself,
   because tax and service charges are applied server-side. Gift baskets price through
   `GET /giftbaskets/gettotal`.

## Rules

- The category vocabulary is not published anywhere public. If you do not already know a
  valid category code, discover products by `code` instead of guessing categories.
- `gettotal` returns an `ORDERNO` even though no order has been placed. It is not proof of
  an order. Do not report it to a user as a confirmation number.
- Currency follows the destination: US deliveries are charged in USD, Canadian deliveries
  in CAD, and an integrator selling into both maintains two separate API keys and swaps
  them by destination.
- No rate limit is published and no rate-limit header is returned, so pace yourself
  conservatively and do not parallelise aggressively.

## When it fails

A failure returns an untyped HTML page, not JSON — the observed 403 is a stock IIS
"Forbidden" page with no error code, no `WWW-Authenticate` challenge and no JSON body.
Check the `Content-Type` before parsing. You cannot distinguish a missing credential from
a revoked one from the response; stop and ask a human rather than retrying.
