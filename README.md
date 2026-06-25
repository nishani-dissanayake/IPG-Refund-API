# Payable IPG — Refund API usage guide

---

You need the following credentials from Payable:

- **Business Key** - provided by Payable
- **Business Token** - provided by Payable

---

## Refund APIs

| Item | Value |
|------|--------|
| **Auth endpoint** | `POST https://payable-ipg-qa.web.app/ipg/refund/auth` |
| **Refund endpoint** | `POST https://payable-ipg-qa.web.app/ipg/refund/add` |

---

## Implementation

### 1. Obtain an access token

**`POST /ipg/refund/auth`**

**Headers**

- `Content-Type: application/json`
- `Authorization: <base64(businessKey:businessToken)>` — concatenate **businessKey**, a single colon **`:`**, and **businessToken**; Base64-encode the result.

**Note:** Ensure that you have enabled `Refund API` permission from the Business Integration in the Payable merchant portal.
<img width="1879" height="761" alt="image" src="https://github.com/user-attachments/assets/3c672170-5277-4d98-9634-a79514025a74" />

**Body (JSON)**

| Field | Required | Description |
|-------|----------|-------------|
| `grant_type` | Yes | Grant type string accepted by PAYable for your integration. |

**Example (web)**

```json
{
  "grant_type": "client_credentials"
}
```

**Success (200)** — example shape:

```json
{
  "accessToken": "<access token>",
  "tokenType": "Bearer",
  "expiresIn": "300",
  "environment": "<env>"
}
```

Note: The Access Token expires in 5 minutes.

---

### 2. Create a refund request

**`POST /ipg/refund/add`**

**Headers**

- `Content-Type: application/json`
- `Authorization: Bearer <accessToken>` — Must be the same token returned by step 1. It should not be expired.

**Body**

Send a JSON object with the following **mandatory** fields.

| Field | Notes |
|-------|--------|
| `orderId` | Can be obtained from the payment notification data, or via the payment details in the Payable merchant portal. |
| `refundAmount` | The refund amount must not exceed the initial transaction amount. |
| `notifyUrl` | A valid `https` URL for the refund notification callback. |

**Note:** The transaction must be settled before initiating a refund request.

**Success (200)** — example:

```json
{
    "requestStatus": "PENDING",
    "refundStatus": "NOT_STARTED",
    "requestedAmount": 123.00,
    "requestedTime": "2026-06-24T12:06:00.411340"
}
```

---

## Sample refund payload

```json
{
    "orderId": "oid-xxxxxxxx-xxxx-xxxx-xxxx-xxxxxxxxxxxx",
    "refundAmount": "10.00",
    "notifyUrl": "https://yoursite.com/webhook/payment"
}
```

---

## Refund-related errors

Validation failures return **`400`** with a per-field map:

```json
{
  "status": 400,
  "errors": {
    "orderId": ["Invalid format for Order Id."]
  }
}
```

Authentication or permission issues may return **`401`** or **`400`** with a single `error` string. Upstream or configuration issues may return **`404`** or **`500`**.

---

## Listening to payment notifications

PAYable calls your **webhook** URL server-to-server (the URL you supply as **`notifyUrl`**).

- The callback is **not** loaded in the browser; test by updating your database when the endpoint is hit.
- **`notifyUrl`** must use a **public HTTPS** host; **localhost** will not receive production callbacks due to security concerns.

**Note:** You will receive the refund notification only after the refund has been processed and completed.

### Typical callback payload

```json
{
  "payableOriginalTxId": "xxxxxxxx-xxxx-xxxx-xxxx-xxxxxxxxxxxx",
  "payableOriginalOrderId": "oid-xxxxxxxx-xxxx-xxxx-xxxx-xxxxxxxxxxxx",
  "payableRefundTxId": "xxxxxxxx-xxxx-xxxx-xxxx-xxxxxxxxxxxx",
  "refundedDateTime": "2026-06-24 11:45:22.0",
  "refundedAmount": "123.00",
  "totalRefundedAmount": "123.00",
  "orderId": 'oid-xxxxxxxx-xxxx-xxxx-xxxx-xxxxxxxxxxxx"
  "checkValue": "XXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXX"
}
```

### Acknowledging the callback

Generate your own check value by adhering to the following format: </br>
`UPPERCASE(SHA512[<orderId>|<refundTransactionId>|UPPERCASE(SHA512[<businessToken>])])`

If the `checkValue` validation is passed, respond with JSON such as:

```json
{
  "Status": 200
}
```

---
