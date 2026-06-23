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
| `orderId` | Can be obtained from the payment notification data. |
| `refundAmount` | The refund amount. |
| `notificationUrl` | A valid `https` URL for the refund notification callback. |

**Success (200)** — example:

```json
{
  "status": "PENDING",
  "uid": "XXXXXXXXXXX",
  "statusIndicator": "XXXXXXX",
  "paymentPage": "https://xxxxxx/ipg/{environment}}/?uid=xxxxxxx-..."
}
```

---

## Sample refund payload

```json
{
    "orderId": "oid-xxxxxxxx-xxxx-xxxx-xxxx-xxxxxxxxxxxx",
    "refundAmount": "10.00",
    "notificationUrl": "https://yoursite.com/webhook/payment"
}
```

---

## Refund-related errors

Validation failures return **`400`** with a per-field map:

```json
{
  "status": 400,
  "errors": {
    "isRetry": ["Invalid format for Is Retry."]
  }
}
```

Authentication or origin/package mismatch may return **`401`** or **`400`** with a single `error` string. Upstream or configuration issues may return **`404`** or **`500`**.

---

## Listening to payment notifications

PAYable calls your **webhook** URL server-to-server (the URL you supply as **`notifyUrl`**).

- The callback is **not** loaded in the browser; test by updating your database when the endpoint is hit.
- **`notifyUrl`** must use a **public HTTPS** host; **localhost** will not receive production callbacks.

### Typical callback payload

| Field | Description |
|-------|-------------|
| `merchantKey` | Merchant identifier |
| `payableOrderId` | PAYable order id |
| `payableTransactionId` | Transaction reference |
| `payableAmount` | Amount |
| `payableCurrency` | Currency |
| `invoiceNo` | Your invoice id |
| `statusCode` | Numeric status |
| `statusMessage` | e.g. `SUCCESS` / `FAILURE` |
| `paymentType`, `paymentMethod`, `paymentScheme` | Payment metadata |
| `custom1`, `custom2` |  |
| `cardHolderName`, `cardNumber` | Present when applicable (masked PAN) |
| `checkValue` |  |

### Acknowledging the callback

Respond with JSON such as:

```json
{
  "Status": 200
}
```

---
