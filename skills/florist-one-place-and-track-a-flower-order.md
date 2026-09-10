---
name: Place and track a Florist One flower order
description: Submit a real flower order for delivery and look it up afterwards. This spends money and dispatches a florist, and Florist One publishes no way to undo it — read the reversibility section before running any of it.
api: openapi/florist-one-flowershop-api-openapi.yml
operations:
  - GET /flowershop/gettotal
  - POST /flowershop/placeorder
  - GET /flowershop/getorderinfo
  - POST /giftbaskets/placeorder
generated: '2026-09-10'
method: generated
source: openapi/ in this repo, https://www.floristone.com/api/how-it-works/, https://www.floristone.com/api/print_api_legal/, and Florist One's published sample code
---

# Place and track a Florist One flower order

Base URL: `https://www.floristone.com/api/rest`.

## Read this before you act

**This flow is irreversible.** `POST /flowershop/placeorder` charges a payment method and
dispatches a local florist. Florist One publishes:

- **no cancel, void, refund or amend operation** — the API has none;
- **no reversal window** — the API Agreement says only that "All credits and refunds by
  Provider are at Provider's sole discretion";
- **no idempotency key** — a retried or duplicated request produces a second real
  delivery to a real address, with no mechanism to collapse the two;
- **no sandbox or test mode** — there is nowhere to rehearse this.

Get explicit human confirmation of the product, price, recipient address and delivery date
before calling `placeorder`, and never retry a `placeorder` request whose outcome you are
unsure of. Verify with `getorderinfo` instead.

## Steps

1. **Price first.** `GET /flowershop/gettotal?products=<json>&affiliateservicecharge=<n>&masterservicecharge=<n>`.
   Take `ORDERTOTAL` from the response verbatim; you will send it back as `ordertotal`.
2. **Confirm the delivery date** with `GET /flowershop/checkdeliverydate?zipcode=<zip>` if
   you have not already. A date the API did not return will not be honoured.
3. **Place the order.** `POST /flowershop/placeorder` with an
   `application/x-www-form-urlencoded` body. Each field is a JSON document encoded as a
   string — this is not a JSON request body:
   - `products` — array of `{code, price, deliverydate, cardmessage, specialinstructions, recipient:{name, institution, address1, address2, city, state, country, phone, zipcode}}`.
     Each item carries its own recipient and delivery date, so one order can fan out to
     several addresses.
   - `customer` — `{name, address1, address2, city, state, zipcode, country, phone, email, ip}`.
     `ip` is the buyer's IP address and is used only for Florist One's fraud detection.
   - `ccinfo` — the payment credential. **Confirm the current form with Florist One before
     sending anything.** The how-it-works page states you tokenize with Authorize.Net or
     Stripe and pass only a token, so Florist One never sees card data; the published
     sample code, last updated in 2017, instead posts raw `ccnum` and `cvv2`. These
     contradict each other on the public surface. Do not send raw card data on the strength
     of the old sample.
   - `ordertotal` — the `ORDERTOTAL` from step 1.
   Gift baskets follow the same shape at `POST /giftbaskets/placeorder`.
4. **Record the order number** returned in `ORDERNO` immediately. It is the only handle you
   will have on the order.
5. **Track it.** `GET /flowershop/getorderinfo?orderno=<n>` returns `CUSTOMER`, the `ITEMS`
   array with each item's `RECIPIENT`, `CARDMSG`, `DELIVERYDATE` and `INSTRUCTIONS`, plus
   `SUBTOTAL`, `TAX`, `SERVICECHARGE`, `DISCOUNT` and `TOTAL`. Response keys are UPPERCASE.

## If something goes wrong

There is no error catalogue. Declined cards, undeliverable ZIP codes, unavailable delivery
dates and out-of-stock products have no documented representation, and the one failure
response actually observed on this API is a stock HTML 403 page with no code in it.

So: do not infer failure from a non-200 and retry. Call `getorderinfo` with the order
number, or — if you never received one — stop and escalate to a human. Cancellations and
refunds are a conversation with Florist One customer service, not an API call.
