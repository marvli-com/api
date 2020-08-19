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
curl --location --request POST 'https://cabinet.marvli.com/api/market/v1/new_payment' \
--header 'Authorization: Bearer 1_12cb0ab9695a4f09fd74d61c659d3c87' \
--header 'Content-Type: application/json' \
--data-raw {
  "client": {
    "email":"<client email>",
    "phone":"",
    "full_name":"",
    "personal_code":"",
    "legal_name":"",
    "brand_name":"",
    "registration_number":"",
    "tax_number":"",
    "bank_account":"",
    "bank_code":"",
    "street_address":"",
    "city":"",
    "zip_code":"",
    "country":"",
    "shipping_street_address":"",
    "shipping_city":"",
    "shipping_zip_code":"",
    "shipping_country":"",
    "cc":[],
    "bcc":[]
  },
  "purchase":   {
    "currency":"EUR",
    "products":[
        {"name":"test",
        "price":100,
        "quantity":"1.0000",
        "discount":0,
        "tax_percent":"0.00"
        }
    ],
    "language":"en",
    "notes":"",
    "debt":0,
    "subtotal_override":null,
    "total_tax_override":null,
    "total_discount_override":null,
    "total_override":null,
    "total":100,
    "request_client_details":[],
    "timezone":"Europe/Riga",
    "due_strict":false,
    "email_message":"",
    "shipping_options":[]
  },
  "send_receipt":false,
  "skip_capture":false,
  "reference":"",
  "due":1597737053,
  "payment_method_whitelist":null,
  "success_redirect":"<your success url>",
  "failure_redirect":"<your failture url>",
  "cancel_redirect":"",
  "success_callback":"",
  "creator_agent":"",
  "platform":"api",
  "market_id": "5"
}
```

>

| № | Parameter name          | Type   | Description                                                    |   |
|---|-------------------------|--------|----------------------------------------------------------------|---|
| 1 | client*                 | object | Object with Client data (required)                             |   |
| 2 | purchase*               | object | Object with Purchase data (required)                           |   |
| 3 | send_receipt            | bool   | Whether to send receipt email for this Purchase when it's paid.|   |
| 4 | skip_capture            | float  | Card payment-specific: if set to true, only authorize the payment (place funds on hold) when payer enters his card data and pays. This option requires a POST /capture/ or POST /release/ later on.                                            |   |
| 5 | reference               | string | Invoice reference.                                             |   |
| 6 | due                     | int    | When the payment is due for this Purchase. The default behaviour is to still allow payment once this moment passes. To change that, set purchase.due_strict to true.               |   |
| 7 | payment_method_whitelist| object | An optional whitelist of payment methods                       |   |
| 8 | success_redirect        | string | When Purchase is paid for successfully, your customer will be taken to this link. Otherwise a standard screen will be displayed. (maxLength: 500)                                    |   |
| 9 | failure_redirect        | string | If there's a payment failure for this Purchase, your customer will be taken to this link. Otherwise a standard screen will be displayed. (maxLength: 500)                                 |   |
|10 | cancel_redirect         | string | If you provide this link, customer will have an option to go to it instead of making payment (a button with 'Return to seller' text will be displayed). Can't contain any of the following symbols: <>'". (maxLength: 500) |   |
|11 | success_callback        | string | When Purchase is paid for successfully, the success_callback URL will receive a POST request with the Purchase object's data in body. (maxLength: 500)                                 |   |
|12 | creator_agent           | string | Identification of software (e.g. an ecommerce module and version) used to create this purchase, if any. (maxLength: 32) |   |
|13 | platform                | enum   | Platform this Purchase was created on. Possible values is [ web, api, ios, android, macos, windows ]                     |   |
|14 | market_id               | int    | ID of the brand to create this Purchase for. You can copy it down in the API section, see the "specify the ID of the Brand" link in answer to "How to setup payments on website or in mobile app?".                                                  |   |

### Client object structure

Contains customer data

| № | Parameter name          | Type   | Description                                                    |   |
|---|-------------------------|--------|----------------------------------------------------------------|---|
| 1 | email*                  | string | Email address (required,maxLength: 254)                        |   |
| 2 | phone                   | string | Phone number in the <country_code> <number> format (maxLength: 32)|   |
| 3 | full_name               | string | Name and surname of client (maxLength: 128)                    |   |
| 4 | personal_code           | string | Personal identification code of client (maxLength: 32)         |   |
| 5 | legal_name              | string | Legal name of company (maxLength: 128)                         |   |
| 6 | brand_name              | string | Company brand name (maxLength: 128)                            |   |
| 7 | registration_number     | string | Registration number of company (maxLength: 32)                 |   |
| 8 | tax_number              | string | Tax payer registration number (maxLength: 32)                  |   |
| 9 | bank_account            | string | Bank account number (e.g. IBAN) (maxLength: 34)                |   |
|10 | bank_code               | string | SWIFT/BIC code of the bank (maxLength: 11)                     |   |
|11 | street_address          | string | Street house number and flat address where applicable (maxLength: 128)|   |
|12 | city                    | string | City name (maxLength: 128)                                     |   |
|13 | zip_code                | string | ZIP or postal code (maxLength: 32)                             |   |
|14 | country                 | string | Country code in the ISO 3166-1 alpha-2 format (e.g. 'GB') (maxLength: 2)|   |
|15 | shipping_street_address | string | Street house number and flat address where applicable (maxLength: 128)|   |
|16 | shipping_city           | string | City name (maxLength: 128)                                     |   |
|17 | shipping_zip_code       | string | ZIP or postal code (maxLength: 32)                             |   |
|18 | shipping_country        | string | Country code in the ISO 3166-1 alpha-2 format (e.g. 'GB') (maxLength: 2)|   |
|18 | cc                      | object | Email addresses to receive a carbon copy of all notification emails |   |
|18 | bcc                     | object | Email addresses to receive a blind carbon copy of all notification emails |   |

### Purchase object structure

Core information about the Purchase, including the products, total, currency and invoice fields. If you're using invoicing via /billing/ or /billing_templates/, this object will be copied 1:1 from BillingTemplate you specify to the resulting Purchases (also to subscription Purchases).

| № | Parameter name          | Type   | Description                                                    |   |
|---|-------------------------|--------|----------------------------------------------------------------|---|
| 1 | currency                | string | Currency code in the ISO 4217 standard (e.g. 'EUR').(maxLength: 3)|   |
| 2 | products*               | object | Line items of the invoice. In case of a transaction with no invoice sent, specify a single Product forming the cost of transaction.|   |
| 3 | language                | string | Language code in the ISO 639-1 format (e.g. 'en',maxLength: 2) |   |
| 4 | notes                   | string | Invoice notes. (maxLength: 1000)                               |   |
| 5 | debt                    | int    | Will be added/substracted to the invoice total, if present.    |   |
| 6 | subtotal_override       | int    | If specified and not null, will override the grand subtotal. This field is visual-only, setting it won't impact `total`.|   |
| 7 | total_tax_override      | int    | If specified and not null, will override the total tax. This field is visual-only, setting it won't impact `total`.|   |
| 8 | total_discount_override | int    | If specified and not null, will override the total discount. This field is visual-only, setting it won't impact `total`.|   |
| 9 | total_override          | int    | If specified and not null, will override the total (unlike the rest of `total_*_override` fields). You can use this field or `products[].total` with a value of 0 to activate preauthorization scenario. See the description of the `Purchase.skip_capture` field.|   |
|10 | total                   | int    | Calculated from `products`. You don't need to specify it.      |   |
|11 | request_client_details  | object | ClientDetails fields to request from the client before the payment. If a value is passed for a field in ClientDetails, it will be automatically removed from this list. [ email, phone, full_name, personal_code, brand_name, legal_name, registration_number, tax_number, country, city, street_address, zip_code, bank_account, bank_code, shipping_country, shipping_city, shipping_street_address, shipping_zip_code ]|   |
|12 | timezone                | string | Timezone to localize invoice-specific timestamps in, e.g. to display a concrete date for a due timestamp on the invoice.|   |
|13 | due_strict              | bool   | Whether to permit payments when Purchase's due has passed. By default those are permitted (and status will be set to overdue once due moment is passed). If this is set to true, it won't be possible to pay for an overdue invoice, and when due is passed the Purchase's status will be set to expired. |   |
|14 | email_message           | string | An optional message to display to your customer in invoice email, e.g. "Your invoice for June". (maxLength: 256)|   |

### Purchase Products object structure	

Line items of the invoice. In case of a transaction with no invoice sent, specify a single Product forming the cost of transaction.

| № | Parameter name          | Type   | Description                                                    |   |
|---|-------------------------|--------|----------------------------------------------------------------|---|
| 1 | name*                   | string | Product name (maxLength: 256)                                  |   |
| 2 | price*                  | int    | You can use this field or total_override with a value of 0 to activate preauthorization scenario. See the description of the Purchase.skip_capture field.|   |
| 3 | quantity                | float  | Quantity of these products in invoice                          |   |
| 4 | discount                | int    | Total discount per this product in invoice                     |   |
| 5 | tax_percent             | float  | Percent of tax added to the price of this product              |   |

## Responses

All fields listed above in the request body will also be included in the response. However, the answer will add additional parameters.
These parameters are read-only, they are described in the table below.

### Success Response

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

| № | Parameter name          | Type   | Description                                         |   |
|---|-------------------------|--------|-----------------------------------------------------|---|
| 1 | type                    | string |  Object type identifier                             |   |
| 2 | id                      | string | Request uid                                         |   |
| 3 | created_on              | int    | Object creation time                                |   |
| 4 | updated_on              | int    | Object last modification time                       |   |
| 5 | payment                 | object | Details of an executed transaction. Read-only for Purchases and Payouts. For an unpaid Purchase, this object will be null. |   |
| 6 | issuer_details          | object | Read-only details of issuer company/brand, persisted for invoice display. |   |
| 7 | transaction_data        | object | Payment method-specific, read-only transaction data. Will contain information about all the transaction attempts and possible errors, if available. |   |
| 8 | status                  | string | Purchase status. Can have the following values: [ created, sent, viewed, error, cancelled, overdue, expired, blocked, hold, released, preauthorized, paid, cleared, settled, chargeback, refunded ] |   |
| 9 | status_history          | ozbject | History of status changes, latest last. Might contain entry about a related object, e.g. status change to refunded will contain a reference to the refund Payment. |   |
|10 | viewed_on               | string | Time the payment form or invoice page was first viewed on |   |
|11 | company_id              | string | Uid of cimpany                                      |   |
|12 | is_test                 | bool   | Indicates this is a test object, created using test API keys or using Billing section of UI while in test mode. |   |
|13 | user_id                 | string | ID of user who has created this object in the Billing UI, if applicable. |   |
|14 | billing_template_id     | string | ID of a BillingTemplate that has spawned this Purchase, if any. |   |
|15 | client_id               | string | ID of a Client object used to initialize ClientDetails (.client) of this Purchase. Either this field or specifying .client object is required (you can only specify a value for one of these fields). All ClientDetails fields from the Client will be copied to .client object. Note that editing Client object won't change the respective fields in already created Purchases. |   |
|16 | is_recurring_token      | string | Indicates whether a recurring token (e.g. for card payments - card token) was saved for this Purchase. If this is true, the id of this Purchase can be used as a recurring_token in POST /purchases/{id}/charge/, enabling you to pay for that Purchase using the same method (same card for card payments) that this one was paid with. |   |
|17 | recurring_token         | string | ID of a recurring token (Purchase having is_recurring_token == true) that was used to pay this Purchase, if any. |   |
|18 | reference_generated     | string | If you don't provide an invoice reference yourself, this autogenerated value will be used as a reference instead. |   |
|19 | refund_availability     | string | Specifies, if the purchase can be refunded fully and partially, only fully, partially or not at all: [ all, full_only, partial_only, none ]  |   |
|20 | refundable_amount       | string | Amount available for refunds. Amount of money as the smallest indivisible units of the currency. Examples: 1 cent for EUR and 1 Yen for JPY. |   |
|21 | currency_conversion     | object | This object is present when automatic currency conversion has occurred upon creation of the purchase. Purchase's original currency was changed and its original amount was converted using the exchange rate shown here. |   |
|22 | product                 | string | Defines which gateway product was used to create this Purchase: [ purchases, billing_invoices, billing_subscriptions, billing_subscriptions_invoice ] |   |
|23 | created_from_ip         | string | IP the Purchase was created from.                    |   |
|24 | invoice_url             | string | URL you will be able to access invoice for this Purchase at, if applicable |   |
|25 | checkout_url            | object | URL you will be able to access the checkout for this Purchase at, if payment for it is possible. When building integrations, redirect the customer to this URL once purchase is created. |   |
|26 | direct_post_url         | string | URL that can be used for Direct Post integration.    |   |

### Payment object structure

Details of an executed transaction. Read-only for Purchases and Payouts. For an unpaid Purchase, this object will be null.

| № | Parameter name          | Type   | Description                                                                                           |   |
|---|-------------------------|--------|-------------------------------------------------------------------------------------------------------|---|
| 1 | is_outgoing             | bool   | Denotes the direction of payment, e.g. for a paid Purchase, is granted to be false, true for payouts. |   |
| 2 | payment_type            | string | Can take the following values [ purchase, purchase_charge, payout, bank_payment, refund, custom ]     |   |
| 3 | amount                  | int    | Amount of money as the smallest indivisible units of the currency. Examples: 1 cent for EUR and 1 Yen for JPY. |   |
| 4 | currency                | string | Currency code in the ISO 4217 standard (e.g. 'EUR'). (maxLength: 3)                                   |   |
| 5 | net_amount              | int    | Net amount of payment with all fees and pending amount subtracted. `amount` = `net_amount` + `fee_amount` + `pending_amount`. The respective account is credited or debited with this value. |   |
| 6 | fee_amount              | int    | Amount of fees for this payment. For a Purchase's PurchaseDetails this is the calculated transaction fee. |   |
| 7 | pending_amount          | int    | Pending amount for this payment that will be unfrozen later. If e.g. it's a Purchase's PaymentDetails and a part of transaction sum is withheld to form a rolling reserve, this field will be equal to the frozen part amount. |   |
| 8 | pending_unfreeze_on     | int    | Informs when the `pending_amount` will be unfrozen.                                                   |   |
| 9 | description             | string | Payment description                                                                                   |   |
|10 | paid_on                 | int    | When the payment was accepted in (is_outgoing == false) or sent from (is_outgoing == true) the gateway system. |   |
|11 | remote_paid_on          | int    | If available, this field will report the date the payment was sent by the remote payer (is_outgoing == false) or when funds arrived to the remote beneficiary (is_outgoing == true). |   |

### Issuer_details object structure

Read-only details of issuer company/brand, persisted for invoice display.

| № | Parameter name          | Type   | Description                                                                                           |   |
|---|-------------------------|--------|-------------------------------------------------------------------------------------------------------|---|
| 1 | website                 | string | Company website URL (maxLength: 200)                                                                  |   |
| 2 | legal_street_address    | string | Street house number and flat address where applicable (maxLength: 128)                                |   |
| 3 | legal_country           | string | Country code in the ISO 3166-1 alpha-2 format (e.g. 'GB') (maxLength: 2)                              |   |
| 4 | legal_city              | string | City name (maxLength: 128)                                                                            |   |
| 5 | legal_zip_code          | string | ZIP or postal code (maxLength: 32)                                                                    |   |
| 6 | bank_accounts           | object | Object of bank accounts                                                                               |   |
| 7 | legal_name              | string | Legal name of company (maxLength: 128)                                                                |   |
| 8 | brand_name              | string | Company brand name (maxLength: 128)                                                                   |   |
| 9 | registration_number     | string | Registration number of company (maxLength: 32)                                                        |   |
|10 | tax_number              | string | Tax payer registration number (maxLength: 32)                                                         |   |

bank_accounts - object of company bank accounts, which contain bank account row: 
bank_account  - Bank account number (e.g. IBAN) string maxLength: 34 
bank_code     - SWIFT/BIC code of the bank (e.g. IBAN) string maxLength: 11

### Transaction_data object structure

Payment method-specific, read-only transaction data. Will contain information about all the transaction attempts and possible errors, if available.

| № | Parameter name          | Type   | Description                                                                                           |   |
|---|-------------------------|--------|-------------------------------------------------------------------------------------------------------|---|
| 1 | payment_method          | string | Payment method used if Purchase was paid, blank string otherwise.                                     |   |
| 2 | extra                   | object | Extra data associated with selected payment method if Purchase was paid, empty object otherwise. Dataset depends on payment method. E.g. for card payment methods like visa or mastercard it will contain properties masked_pan: string, three_d_secure: boolean and cardholder_name: string. |   |
| 3 | country                 | string | Country code (in the ISO 3166-1 alpha-2 format e.g. 'GB') where payment tool used originates (e.g. in case of card payments, the card issuing country). Will be blank if Purchase was not paid or country could not be detected. |   |
| 4 | attempts                | object | Will contain information about all the payment attempts made and errors encountered, if any.          |   |

### Transaction_data Attempts object structure  

Will contain information about all the payment attempts made and errors encountered, if any.

| № | Parameter name          | Type   | Description                                                                                           |   |
|---|-------------------------|--------|-------------------------------------------------------------------------------------------------------|---|
| 1 | type                    | string | Type of action attempted, takes following values: [ execute, authorize, release, capture, recurring_execute, delete_recurring_token, refund ] |   |
| 2 | successful              | bool   | If this attempt was successful or not. For false, error of this attempt will be not null.             |   |
| 3 | payment_method          | string | Payment method used for this attempt.                                                                 |   |
| 4 | extra                   | object | Extra data associated with selected payment method. Dataset depends on payment method. E.g. for card payment methods like visa or mastercard it will contain properties masked_pan: string, three_d_secure: boolean and cardholder_name: string. |   |
| 5 | country                 | string | Country code (in the ISO 3166-1 alpha-2 format e.g. 'GB') where payment tool used originates (e.g. in case of card payments, the card issuing country). Will be blank if country could not be detected. |   |
| 6 | client_ip               | string | IP the paying client made this attempt from, if available.                                            |   |
| 7 | processing_time         | int    | Time (if possible, fetched from the remot processing system) this attempt happened at.                |   |
| 8 | error                   | object | Contains code and message fields. Code - contains an error code, message - contains an error message  |   |

### Transaction_data Attempts error codes

| № | Code                                            | Description                                                                   |   |
|---|-------------------------------------------------|-------------------------------------------------------------------------------|---|
| 1 | unknown_payment_method                          | Unknown payment method                                                        |   |
| 2 | invalid_card_number                             | Invalid card number                                                           |   |
| 3 | invalid_expires                                 | Invalid expires                                                               |   |
| 4 | no_matching_terminal                            | No matching terminal                                                          |   |
| 5 | blacklisted_tx                                  | Blacklisted transaction: blocked (general)                                    |   |
| 6 | timeout_3ds_enrollment_check                    | 3DS enrollment check timeout.                                                 |   |
| 7 | timeout_acquirer_status_check                   | Timeout checking payment status with acquirer                                 |   |
| 8 | validation_card_details_missing                 | Card data field values are missing from request                               |   |
| 8 | validation_cvc_not_provided                     | cvc field not provided                                                        |   |
| 8 | validation_cardholder_name_not_provided         | cardholder_name field not provided                                            |   |
| 8 | validation_card_number_not_provided             | card_number field not provided                                                |   |
| 8 | validation_expires_not_provided                 | expires field not provided                                                    |   |
| 8 | validation_cvc_too_long                         | cvc is too long                                                               |   |
| 8 | validation_cardholder_name_too_long             | cardholder_name is too long                                                   |   |
| 8 | validation_card_number_too_long                 | card_number is too long                                                       |   |
| 8 | validation_expires_too_long                     | expires is too long                                                           |   |
| 8 | validation_cvc_invalid                          | cvc is invalid                                                                |   |
| 8 | validation_cardholder_name_invalid              | cardholder_name is too long or invalid                                        |   |
| 8 | validation_card_number_invalid                  | card_number is invalid                                                        |   |
| 8 | validation_expires_invalid                      | expires is invalid                                                            |   |
| 8 | acquirer_connection_error                       | Acquirer connection error                                                     |   |
| 8 | blacklisted_tx_issuing_country                  | Blacklisted transaction: issuing country                                      |   |
| 8 | s2s_not_supported                               | Server-to-server flow not supported by processing                             |   |
| 8 | timeout                                         | Operation timeout                                                             |   |
| 8 | general_transaction_error                       | Unrecognized transaction error                                                |   |
| 8 | antifraud_general                               | Decline, fraud                                                                |   |
| 8 | 3ds_authentication_failed                       | 3DS authentication failed                                                     |   |
| 8 | do_not_honour                                   | Do not honour (the transaction was declined by the Issuer without definition or reason). |   |
| 8 | payment_rejected_other_reason                   | Payment rejected (other reason)                                               |   |
| 8 | exceeds_withdrawal_limit                        | Exceeds withdrawal limit                                                      |   |
| 8 | exceeded_account_limit                          | Exceeded account limit                                                        |   |
| 8 | expired_card                                    | Expired card                                                                  |   |
| 8 | blacklisted_tx_risk_score                       | Blacklisted transaction: risk score                                           |   |
| 8 | transaction_not_supported_or_not_valid_for_card | The transaction request presented is not supported or is not valid for the card number presented. |   |
| 8 | exceeded_acquirer_refund_amount                 | Exceeded refundable amount defined by acquirer                                |   |
| 8 | transaction_not_permitted_on_terminal           | Transaction not permitted on terminal (this card does not support the type of transaction requested). |   |
| 8 | acquirer_internal_error                         | Acquirer internal error                                                       |   |
| 8 | transaction_not_permitted_to_cardholder         | Transaction not permitted to cardholder                                       |   |
| 8 | invalid_issuer_number                           | No such issuer (the Issuer number is not valid).                              |   |
| 8 | restricted_card                                 | Restricted card                                                               |   |
| 8 | merchant_response_timeout                       | Timeout of merchant response exceeded                                         |   |
| 8 | reconcile_error                                 | Reconcilation error                                                           |   |
| 8 | lost_card                                       | Lost card                                                                     |   |
| 8 | stolen_card                                     | Stolen card                                                                   |   |
| 8 | invalid_amount                                  | Invalid amount                                                                |   |
| 8 | re_enter_transaction                            | Re enter transaction                                                          |   |
| 8 | security_violation                              | Security violation                                                            |   |
| 8 | partial_forbidden                               | Intervene, bank approval required for partial amount                          |   |
| 8 | suspected_fraud                                 | Decline, suspected fraud                                                      |   |
| 8 | acquirer_routing_error                          | Acquirer routing error                                                        |   |
| 8 | acquirer_configuration_error                    | Acquirer configuration error                                                  |   |
| 8 | invalid_card_data                               | Invalid card data provided                                                    |   |
| 8 | issuer_not_available                            | Issuer Not Available                                                          |   |
| 8 | exceeded_terminal_limit                         | Exceeded terminal limit                                                       |   |

### Error Response

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
