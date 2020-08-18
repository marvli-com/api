---
title: Public REST API

language_tabs: # must be one of https://git.io/vQNgJ
  - curl examples

toc_footers:
  - <a href='https://merch.transcoin.me'>Sign Up for a Developer Key</a>
  - <a href='https://github.com/slatedocs/slate'>Documentation Powered by Slate</a>

search: false
---

# Public REST API

All the endpoints below have a prefix of https://cabinet.marvli.com/api/market/v1/ ( e.g. https://cabinet.marvli.com/api/market/v1/new_payment)

You will need your API secret key that you can obtain in API section of your company profile. Use it as a bearer token 
in `Authorization` header included with every request: `Authorization:` `Bearer <secret key>`

# Webhooks

For creating and modifying webhooks see the Webhook <a href='https://github.com/slatedocs/slate'>CRUD API specification.</a>CRUD API specification.

Webhook callback payloads of each individual Webhook are signed using a dedicated key pair. The public key can be retrieved from Webhook.public_key. 
See Authentication below for more details.

# Delivery protocol

In case a callback is not successfully delivered (received by the target server and responded to with a 200 series HTTP response code) further 
attempts will be made to deliver the callback, up to a total of 8 attempts, at exponentially increasing intervals between attempts. If the callback
 is not successfully delivered 36 hours after triggering, no further delivery attempts will be made.

Please note that due to the asynchronous nature of network requests it is possible for a callback delivery confirmation (HTTP response with a 200 series
 status code) to not properly arrive from the callback's target server. Therefore it is possible in case of severe network faults for the target server
 to receive a callback, respond to it with a 200 series HTTP status code and then receive the same callback after an interval.

Callback deliveries are guaranteed to be sequential in relation to events triggered on their source objects. For example, if webhooks have been registered
 for both the purchase.created and purchase.paid events, no purchase.paid callbacks for this Purchase will be delivered until all purchase.created 
 callbacks for this Purchase have been successfully delivered.

# Authentication

To guarantee the authenticity of delivered callbacks payloads are signed using asymmetric A.K.A. public-key cryptography. Each callback delivery 
request includes an X-Signature header field. This field contains a base64-encoded RSA PKCS#1 v1.5 signature of the SHA3-256 digest of the 
request body buffer.

The public key for authenticating a Webhooks is available on Webhook.public_key of the corresponding Webhook.
The public key for authenticating success callbacks can be retrieved with GET /public_key/.
Please note the Gateway provider is not responsible for any financial loses incurred due to not implementing payload signature verification.

# Purchases

## /purchases/ Create a Purchase. The main request for any e-commerce integration.

To run payments in your application use POST /purchases/, request to register payments and receive the checkout link (checkout_url). 
After the payment is processed, gateway will redirect the client back to your website (take note of success_redirect, failure_redirect).

You have three options to check payment status: 1) use success_calback parameter of Purchase object; 2) use GET /purchases/<purchase_id>/ request; 3)
 set up a Webhook using the UI or Webhook API to listen to purchase.paid or purchase.payment_failure event on your server.

Using skip_capture flag allows you to separate the authentication and payment execution steps, allowing you to reserve funds on payer’s card 
account for some time. This flag can also enable preauthorization capability, allowing you to save the card without a financial transaction, if available.

In case making a purchase client agrees to store his card for the upcoming purchases, next time he will be able to pay in a single click.

Instead of a redirect you can also utilize Direct Post checkout: you can create an HTML `<form>` on your website with method="POST" and action 
pointing to direct_post_url of a created Purchase. You will also need to saturate form with `<input>`-s for card data fields. As a result, when
 a payer submits their card data, it will be posted straight to our system, allowing you to customize the checkout as you wish while your PCI 
 DSS requirement is only raised to SAQ A-EP, as your system doesn't receive or process card data. For more details, see the documentation on 
 Purchase's direct_post_url field.

To pay for test Purchases, use 4444 3333 2222 1111 as the card number, 123 as CVC, any date/month greater than now as expiry and any (Latin)
 cardholder name. Any other card number/CVC/expiry not greater or equal than the current month will all fail a test payment.
 
Method has no parameters

## Request body


> Request body
 
```javascript
{
  "client": {
    "email": "test@test.com"
  },
  "purchase": {
    "products": [
      {
        "name": "test",
        "price": 100
      }
    ]
  },
  "brand_id": "409eb80e-3782-4b1d-afa8-b779759266a5"
}
```

>

| № | Parameter name | Type   | Description                                           |   |
|---|----------------|--------|-------------------------------------------------------|---|
| 1 | client         | object | Client data                                           |   |
| 2 | email          | string | Client email                                          |   |
| 3 | purchase       | object | Purchase data                                         |   |
| 4 | name           | string | Product name                                          |   |
| 5 | price          | float  | Product price                                         |   |
| 6 | brand_id       | string | Brand ID in our system                                |   |

## Responses

| № | Code | Description                                         |   |
|---|------|-----------------------------------------------------|---|
| 1 | 201  | OK                                                  |   |
| 2 | 400  | Invalid data submitted or request processing error  |   |

## Success Response

> Success Response
 
```javascript
{
  "type": "string",
  "id": "3fa85f64-5717-4562-b3fc-2c963f66afa6",
  "created_on": 1619740800,
  "updated_on": 1619740800,
  "client": {
    "bank_account": "string",
    "bank_code": "string",
    "email": "user@example.com",
    "phone": "+44 45643564564",
    "full_name": "string",
    "personal_code": "string",
    "street_address": "string",
    "country": "string",
    "city": "string",
    "zip_code": "string",
    "shipping_street_address": "string",
    "shipping_country": "string",
    "shipping_city": "string",
    "shipping_zip_code": "string",
    "cc": [
      "user@example.com"
    ],
    "bcc": [
      "user@example.com"
    ],
    "legal_name": "string",
    "brand_name": "string",
    "registration_number": "string",
    "tax_number": "string"
  },
  "purchase": {
    "currency": "string",
    "products": [
      {
        "name": "string",
        "quantity": "string",
        "price": 0,
        "discount": 0,
        "tax_percent": "string"
      }
    ],
    "total": 0,
    "language": "string",
    "notes": "string",
    "debt": 0,
    "subtotal_override": 0,
    "total_tax_override": 0,
    "total_discount_override": 0,
    "total_override": 0,
    "request_client_details": [
      "email"
    ],
    "timezone": "Europe/Oslo",
    "due_strict": false,
    "email_message": "string"
  },
  "payment": {
    "is_outgoing": false,
    "payment_type": "purchase",
    "amount": 0,
    "currency": "string",
    "net_amount": 0,
    "fee_amount": 0,
    "pending_amount": 0,
    "pending_unfreeze_on": 1619740800,
    "description": "string",
    "paid_on": 1619740800,
    "remote_paid_on": 1619740800
  },
  "issuer_details": {
    "website": "string",
    "legal_street_address": "string",
    "legal_country": "string",
    "legal_city": "string",
    "legal_zip_code": "string",
    "bank_accounts": [
      {
        "bank_account": "string",
        "bank_code": "string"
      }
    ],
    "legal_name": "string",
    "brand_name": "string",
    "registration_number": "string",
    "tax_number": "string"
  },
  "transaction_data": {
    "payment_method": "string",
    "extra": {},
    "country": "string",
    "attempts": [
      {
        "type": "execute",
        "successful": true,
        "payment_method": "string",
        "extra": {},
        "country": "string",
        "client_ip": "string",
        "processing_time": 1619740800,
        "error": {
          "code": "string",
          "message": "string"
        }
      }
    ]
  },
  "status": "created",
  "status_history": [
    {
      "status": "created",
      "timestamp": 1619740800,
      "related_object": {
        "type": "string",
        "id": "3fa85f64-5717-4562-b3fc-2c963f66afa6"
      }
    }
  ],
  "viewed_on": 1619740800,
  "company_id": "3fa85f64-5717-4562-b3fc-2c963f66afa6",
  "is_test": true,
  "user_id": "3fa85f64-5717-4562-b3fc-2c963f66afa6",
  "brand_id": "3fa85f64-5717-4562-b3fc-2c963f66afa6",
  "billing_template_id": "3fa85f64-5717-4562-b3fc-2c963f66afa6",
  "client_id": "3fa85f64-5717-4562-b3fc-2c963f66afa6",
  "send_receipt": false,
  "is_recurring_token": true,
  "recurring_token": "3fa85f64-5717-4562-b3fc-2c963f66afa6",
  "skip_capture": false,
  "reference_generated": "string",
  "reference": "string",
  "issued": "2020-04-30",
  "due": 1619740800,
  "refund_availability": "all",
  "refundable_amount": 0,
  "currency_conversion": {
    "original_currency": "string",
    "original_amount": 0,
    "exchange_rate": 0
  },
  "payment_method_whitelist": [
    "string"
  ],
  "success_redirect": "string",
  "failure_redirect": "string",
  "cancel_redirect": "string",
  "success_callback": "string",
  "creator_agent": "string",
  "platform": "web",
  "product": "purchases",
  "created_from_ip": "string",
  "invoice_url": "string",
  "checkout_url": "string",
  "direct_post_url": "string"
}
```

>

| № | Parameter name          | Type   | Description                   |   |
|---|-------------------------|--------|-------------------------------|---|
| 1 | type                    | string |                               |   |
| 2 | id                      | string | Client email                  |   |
| 3 | created_on              | time   | Creation date                 |   |
| 4 | updated_on              | time   | UPdate date                   |   |
| 6 | bank_account            | string |                               |   |
| 6 | bank_code               | string |                               |   |
| 6 | email                   | string |                               |   |
| 6 | phone                   | string |                               |   |
| 6 | full_name               | string |                               |   |
| 6 | personal_code           | string |                               |   |
| 6 | street_address          | string |                               |   |
| 6 | country                 | string |                               |   |
| 6 | city                    | string |                               |   |
| 6 | zip_code                | string |                               |   |
| 6 | shipping_street_address | string |                               |   |
| 6 | shipping_country        | string |                               |   |
| 6 | shipping_city           | string |                               |   |
| 6 | shipping_zip_code       | string |                               |   |
| 6 | cc                      | string |                               |   |
| 6 | bcc                     | string |                               |   |
| 6 | legal_name              | string |                               |   |
| 6 | brand_name              | string |                               |   |
| 6 | registration_number     | string |                               |   |
| 6 | tax_number              | string |                               |   |
| 6 | purchase                | object |                               |   |
| 6 | currency                | string |                               |   |
| 6 | products                | object |                               |   |
| 6 | name                    | string |                               |   |
| 6 | quantity                | string |                               |   |
| 6 | price                   | float  |                               |   |
| 6 | discount                | float  |                               |   |
| 6 | tax_percent             | string |                               |   |
| 6 | total                   | float  |                               |   |
| 6 | language                | string |                               |   |
| 6 | notes                   | string |                               |   |
| 6 | debt                    | float  |                               |   |
| 6 | subtotal_override       | float  |                               |   |
| 6 | total_tax_override      | float  |                               |   |
| 6 | total_discount_override | float  |                               |   |
| 6 | total_override          | float  |                               |   |
| 6 | total_override          | float  |                               |   |
| 6 | total_override          | float  |                               |   |
| 6 | total_override          | float  |                               |   |
| 6 | total_override          | float  |                               |   |
| 6 | request_client_details  | array  |                               |   |
| 6 | timezone                | string |                               |   |
| 6 | due_strict              | bool   |                               |   |
| 6 | email_message           | string |                               |   |
| 6 | payment                 | object |                               |   |
| 6 | is_outgoing             | bool   |                               |   |
| 6 | payment_type            | string |                               |   |
| 6 | amount                  | float  |                               |   |
| 6 | currency                | string |                               |   |
| 6 | net_amount              | float  |                               |   |
| 6 | fee_amount              | float  |                               |   |
| 6 | pending_amount          | float  |                               |   |
| 6 | pending_unfreeze_on     | time   |                               |   |
| 6 | description             | string |                               |   |
| 6 | paid_on                 | time   |                               |   |
| 6 | remote_paid_on          | time   |                               |   |
| 6 | issuer_details          | object |                               |   |
| 6 | website                 | string |                               |   |
| 6 | legal_street_address    | string |                               |   |
| 6 | legal_country           | string |                               |   |
| 6 | legal_city              | string |                               |   |
| 6 | legal_zip_code          | string |                               |   |
| 6 | bank_accounts           | object |                               |   |
| 6 | bank_account            | string |                               |   |
| 6 | bank_code               | string |                               |   |
| 6 | legal_name              | string |                               |   |
| 6 | brand_name              | string |                               |   |
| 6 | registration_number     | string |                               |   |
| 6 | tax_number              | string |                               |   |
| 6 | transaction_data        | object |                               |   |
| 6 | payment_method          | string |                               |   |
| 6 | extra                   | array` |                               |   |
| 6 | country                 | string |                               |   |
| 6 | attempts                | object |                               |   |
| 6 | type                    | string |                               |   |
| 6 | successful              | bool   |                               |   |
| 6 | payment_method          | string |                               |   |
| 6 | extra                   | bool   |                               |   |
| 6 | country                 | string |                               |   |
| 6 | client_ip               | string |                               |   |
| 6 | processing_time         | time   |                               |   |
| 6 | error                   | object |                               |   |
| 6 | code                    | string |                               |   |
| 6 | message                 | string |                               |   |
| 6 | status                  | string |                               |   |
| 6 | status_history          | object |                               |   |
| 6 | status                  | string |                               |   |
| 6 | timestamp               | time   |                               |   |
| 6 | related_object          | object |                               |   |
| 6 | type                    | string |                               |   |
| 6 | id                      | string |                               |   |
| 6 | viewed_on               | time   |                               |   |
| 6 | company_id              | string |                               |   |
| 6 | is_test                 | bool   |                               |   |
| 6 | user_id                 | string |                               |   |
| 6 | brand_id                | string |                               |   |
| 6 | billing_template_id     | string |                               |   |
| 6 | client_id               | string |                               |   |
| 6 | send_receipt            | bool   |                               |   |
| 6 | is_recurring_token      | bool   |                               |   |
| 6 | recurring_token         | string |                               |   |
| 6 | skip_capture            | bool   |                               |   |
| 6 | reference_generated     | string |                               |   |
| 6 | reference               | string |                               |   |
| 6 | issued                  | date   |                               |   |
| 6 | due                     | time   |                               |   |
| 6 | refund_availability     | string |                               |   |
| 6 | refundable_amount       | float  |                               |   |
| 6 | currency_conversion     | object |                               |   |
| 6 | original_currency       | string |                               |   |
| 6 | original_amount         | float  |                               |   |
| 6 | exchange_rate           | float  |                               |   |
| 6 | success_redirect        | array  |                               |   |
| 6 | failure_redirect        | string |                               |   |
| 6 | cancel_redirect         | string |                               |   |
| 6 | success_callback        | string |                               |   |
| 6 | creator_agent           | string |                               |   |
| 6 | platform                | string |                               |   |
| 6 | product                 | string |                               |   |
| 6 | created_from_ip         | string |                               |   |
| 6 | invoice_url             | string |                               |   |
| 6 | checkout_url            | string |                               |   |
| 6 | direct_post_url         | string |                               |   |


## Error Response

> Error Response
 
```javascript
{
  "__all__": {
    "message": "descriptive error message",
    "code": "error_code"
  }
}
```

>

| № | Parameter name | Type   | Description                                           |   |
|---|----------------|--------|-------------------------------------------------------|---|
| 1 | "__all__"      | object | Object whitch stores error data                       |   |
| 2 | code           | int    | Error Code                                            |   |
| 3 | message        | string | Error message                                         |   |
