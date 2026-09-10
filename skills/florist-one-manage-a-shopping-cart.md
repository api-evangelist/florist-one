---
name: Manage a Florist One shopping cart
description: Build a server-side cart with Florist One so a checkout flow does not have to store basket state locally. Fully reversible, pre-transaction, and the safest write surface on this API.
api: openapi/florist-one-shoppingcart-api-openapi.yml
operations:
  - POST /shoppingcart
  - GET /shoppingcart
  - DELETE /shoppingcart
generated: '2026-09-10'
method: generated
source: openapi/ in this repo, https://www.floristone.com/api/how-it-works/, and Florist One's published sample code
---

# Manage a Florist One shopping cart

Base URL: `https://www.floristone.com/api/rest`. Florist One hosts the cart, so a storefront
can run a checkout without a database of its own.

Every operation here is reversible and nothing is charged, which makes this the one write
surface on the API an agent can exercise without spending money.

## Steps

1. **Create a cart.** `POST /shoppingcart` with `sessionid=<your id>` in the body. You
   choose the id. Florist One documents no format, no entropy requirement and no expiry —
   generate a long random value, because a guessable `sessionid` is a cart somebody else
   can read and mutate.
2. **Add an item.** `POST /shoppingcart?sessionid=<id>&productcode=<sku>&action=add`.
   Note a discrepancy in the provider's own material: the published sample
   `php/shoppingcart/addtocart.php` issues this with HTTP `PUT`, while the description in
   this repo models it as `POST`. No authoritative reference is published to settle which
   the server expects. If one verb 403s or 405s, try the other before concluding the call
   is wrong.
3. **Read the cart.** `GET /shoppingcart?sessionid=<id>`.
4. **Remove one item** with `action=remove`, or **empty the cart** with `action=clear`,
   using the same `POST /shoppingcart` shape.
5. **Destroy the cart** when the session ends: `DELETE /shoppingcart?sessionid=<id>`.

## Rules

- Authenticate on every call — HTTP Basic, API Key as username. See
  `authentication/florist-one-authentication.yml` for the non-standard header form.
- A cart is not an order. Nothing here reserves inventory, holds a price, or guarantees a
  delivery date. Re-price with `GET /flowershop/gettotal` and re-check the delivery date
  before placing the order.
- Handing off to checkout is a one-way door — see
  `skills/florist-one-place-and-track-a-flower-order.md` before calling `placeorder`.
- Responses are JSON with UPPERCASE keys; errors are untyped HTML. Check `Content-Type`
  before parsing.
