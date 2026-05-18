# Wheels America API — User Guide

Base URL: `https://api.wheelsamerica.com`

## Authentication

All requests require an API key passed via the `x-api-key` header.

```
x-api-key: YOUR_API_KEY
```

Your API key is tied to your company's customer account. Contact Wheels America to obtain one.

---

## Endpoints

### Place Order

**POST** `/orders/place`

Submit a purchase order for a wheel by Hollander number and variation.

#### Request Headers

| Header | Required | Description |
|--------|----------|-------------|
| `x-api-key` | Yes | Your API key |
| `Content-Type` | Yes | `application/json` |

#### Request Body

| Field | Type | Required | Description |
|-------|------|----------|-------------|
| `PO` | string | Yes | Your purchase order number |
| `Interchange` | string | Yes | Interchange number for the wheel |
| `VariationID` | number | Yes* | Wheel variation ID (determines finish/style) |
| `SKU` | string | Yes* | Wheel SKU (e.g. `W012482`). Either `VariationID` or `SKU` is required |
| `Qty` | number | Yes | Quantity to order |
| `CustomerName` | string | Yes | Ship-to contact name |
| `ShipAddress` | string | Yes | Shipping street address |
| `ShipCity` | string | Yes | Shipping city |
| `ShipState` | string | Yes | Shipping state (2-letter abbreviation) |
| `ShipZip` | string | Yes | Shipping zip code |
| `Note` | string | No | Optional order note |

#### Example Request

```bash
curl -X POST https://api.wheelsamerica.com/orders/place \
  -H "x-api-key: YOUR_API_KEY" \
  -H "Content-Type: application/json" \
  -d '{
    "PO": "PO-12345",
    "Interchange": "70235",
    "SKU": "W004821",
    "Qty": 2,
    "CustomerName": "John Smith",
    "ShipAddress": "123 Main St",
    "ShipCity": "Dallas",
    "ShipState": "TX",
    "ShipZip": "75201",
    "Note": "Please ship ASAP"
  }'
```

#### Responses

**201 Created**

```json
{ "success": true, "message": "Order placed successfully" }
```

**400 Bad Request** — Missing a required field.

```json
{ "error": "Missing required field: Hollander" }
```

**404 Not Found** — Invalid VariationID or customer account issue.

```json
{ "error": "Invalid VariationID" }
```

**500 Internal Server Error**

```json
{ "error": "Failed to place order" }
```

---

### Rotate API Key

**POST** `/keys/rotate`

Rotate your current API key. The old key is immediately invalidated and a new key is returned.

#### Request Headers

| Header | Required | Description |
|--------|----------|-------------|
| `x-api-key` | Yes | Your current API key |

#### Example Request

```bash
curl -X POST https://api.wheelsamerica.com/keys/rotate \
  -H "x-api-key: YOUR_API_KEY"
```

#### Responses

**200 OK**

```json
{ "success": true, "apiKey": "NEW_API_KEY_VALUE" }
```

**500 Internal Server Error**

```json
{ "success": false, "error": "Failed to rotate API key" }
```

---

## Error Codes

| HTTP Status | Meaning |
|-------------|---------|
| 401 | Missing API key |
| 403 | Invalid or expired API key |
| 400 | Bad request (missing/invalid fields) |
| 404 | Resource not found |
| 500 | Server error |

---

## Notes

- All requests must include the `x-api-key` header. Requests without it receive a 401 response.
- Pricing is determined server-side based on your account terms. The order total is calculated as unit price multiplied by quantity.
- Billing address and account details are pulled automatically from your customer record on file.
- After rotating your key, update your integration immediately — the previous key will no longer work.
