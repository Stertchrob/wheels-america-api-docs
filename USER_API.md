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

Submit a purchase order for a wheel by interchange number and variation.

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
{ "error": "Missing required field: Interchange" }
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

### Get Inventory

**GET** `/inventory/get`

Retrieve current available wheel inventory. Optionally filter by interchange number.

#### Request Headers

| Header | Required | Description |
|--------|----------|-------------|
| `x-api-key` | Yes | Your API key |

#### Query Parameters

| Param | Required | Description |
|-------|----------|-------------|
| `interchange` | No | Filter results to a specific interchange number |

#### Example Requests

Get all inventory:

```bash
curl https://api.wheelsamerica.com/inventory/get \
  -H "x-api-key: YOUR_API_KEY"
```

Filter by interchange number:

```bash
curl "https://api.wheelsamerica.com/inventory/get?interchange=10001" \
  -H "x-api-key: YOUR_API_KEY"
```

#### Response Fields

| Field | Type | Description |
|-------|------|-------------|
| `SKU` | string | Wheel SKU (e.g. `W020388`) |
| `Interchange` | string | Interchange number |
| `Available` | number | Total units available |
| `Finished` | number | Total finished units |
| `Price` | number | Unit price |
| `Finish` | string | Wheel finish description |
| `Image` | string \| null | Wheel image URL (Cloudinary); `null` if no image is on file |

#### Responses

**200 OK**

```json
{
  "success": true,
  "count": 3,
  "inventory": [
    {
      "SKU": "W020388",
      "Interchange": "10001",
      "Available": 1,
      "Finished": 1,
      "Price": 241.38,
      "Finish": "Gloss Black Gloss Black Powder Coat",
      "Image": "https://res.cloudinary.com/wheels-america/image/upload/t_Wheel_400_2/f_auto/v1234567890/example.jpg"
    },
    {
      "SKU": "W020511",
      "Interchange": "10001",
      "Available": 4,
      "Finished": 4,
      "Price": 189.77,
      "Finish": "Machined Lip w/ Charcoal Spokes",
      "Image": null
    }
  ]
}
```

**404 Not Found** — Inventory data not yet available.

```json
{ "error": "Inventory file not available" }
```

**500 Internal Server Error**

```json
{ "error": "Failed to retrieve inventory" }
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
