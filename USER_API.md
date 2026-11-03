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

`OrderNumber` is the Wheels America order number for this line. Save it and use it to check status and tracking. Each request creates one order.

```json
{
  "success": true,
  "message": "Order placed successfully",
  "OrderNumber": 482910,
  "PO": "PO-12345"
}
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

### Get Order

**GET** `/orders/:orderNumber`

Retrieve status, quantity sent, carrier, and tracking numbers for one order placed through this API.

#### Request Headers

| Header | Required | Description |
|--------|----------|-------------|
| `x-api-key` | Yes | Your API key |

#### Path Parameters

| Param | Required | Description |
|-------|----------|-------------|
| `orderNumber` | Yes | `OrderNumber` returned when the order was placed |

#### Example Request

```bash
curl https://api.wheelsamerica.com/orders/482910 \
  -H "x-api-key: YOUR_API_KEY"
```

#### Response Fields

| Field | Type | Description |
|-------|------|-------------|
| `OrderNumber` | number | Wheels America order number |
| `PO` | string | Your purchase order number |
| `Status` | string | `Open`, `Processing`, `Partial`, `Shipped`, `Cancelled`, `Refunded`, or `Closed` |
| `SKU` | string | Wheel SKU |
| `Interchange` | string | Interchange number |
| `Qty` | number | Quantity ordered |
| `QtySent` | number | Quantity shipped. `0` until a shipment has a carrier and tracking number |
| `OrderDate` | string \| null | Order date in UTC |
| `Shipments` | array | Shipped shipments. Empty until a shipment has a carrier and tracking number |
| `Shipments[].QtySent` | number | Quantity on that shipment |
| `Shipments[].ShipDate` | string \| null | Ship date in UTC |
| `Shipments[].Packages` | array | Labels for that shipment. Quantity is on the shipment, not on each label |
| `Shipments[].Packages[].Carrier` | string | Carrier name |
| `Shipments[].Packages[].TrackingNumber` | string | Tracking number |

`Processing` covers orders that have been received and are being filled. `Partial` means some of the ordered quantity has shipped.

#### Responses

**200 OK**

```json
{
  "success": true,
  "order": {
    "OrderNumber": 482910,
    "PO": "PO-12345",
    "Status": "Shipped",
    "SKU": "W004821",
    "Interchange": "70235",
    "Qty": 2,
    "QtySent": 2,
    "OrderDate": "2026-11-03T15:04:00.000Z",
    "Shipments": [
      {
        "QtySent": 2,
        "ShipDate": "2026-11-04T18:10:00.000Z",
        "Packages": [
          { "Carrier": "UPS", "TrackingNumber": "1Z999AA10123456784" }
        ]
      }
    ]
  }
}
```

**400 Bad Request** — Order number is not a positive integer.

```json
{ "error": "Invalid order number" }
```

**404 Not Found** — No order with that number for your account.

```json
{ "error": "Order not found" }
```

**500 Internal Server Error**

```json
{ "error": "Failed to retrieve order" }
```

---

### Get Orders by PO

**GET** `/orders?po=`

Retrieve every order placed through this API for your account with that purchase order number. A PO can match more than one order, because each place-order request is one line.

#### Request Headers

| Header | Required | Description |
|--------|----------|-------------|
| `x-api-key` | Yes | Your API key |

#### Query Parameters

| Param | Required | Description |
|-------|----------|-------------|
| `po` | Yes | Your purchase order number |

#### Example Request

```bash
curl "https://api.wheelsamerica.com/orders?po=PO-12345" \
  -H "x-api-key: YOUR_API_KEY"
```

The `orders` array uses the same order object as Get Order.

#### Responses

**200 OK**

```json
{
  "success": true,
  "count": 1,
  "orders": [
    {
      "OrderNumber": 482910,
      "PO": "PO-12345",
      "Status": "Open",
      "SKU": "W004821",
      "Interchange": "70235",
      "Qty": 2,
      "QtySent": 0,
      "OrderDate": "2026-11-03T15:04:00.000Z",
      "Shipments": []
    }
  ]
}
```

A PO with no matching orders returns `count: 0` and an empty `orders` array.

**400 Bad Request** — Missing PO.

```json
{ "error": "Missing required parameter: po" }
```

**500 Internal Server Error**

```json
{ "error": "Failed to retrieve orders" }
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
| `VariationID` | number | Wheel variation ID (determines finish/style) |
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
      "VariationID": 20388,
      "Interchange": "10001",
      "Available": 1,
      "Finished": 1,
      "Price": 241.38,
      "Finish": "Gloss Black Gloss Black Powder Coat",
      "Image": "https://res.cloudinary.com/wheels-america/image/upload/t_Wheel_400_2/f_auto/v1234567890/example.jpg"
    },
    {
      "SKU": "W020511",
      "VariationID": 48305,
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

### Get Fitment

**GET** `/inventory/fitment`

Retrieve vehicle fitment data (Year, Make, Model, Submodel, Engines) for wheels. Optionally filter by interchange number or SKU.

#### Request Headers

| Header | Required | Description |
|--------|----------|-------------|
| `x-api-key` | Yes | Your API key |

#### Query Parameters

| Param | Required | Description |
|-------|----------|-------------|
| `interchange` | No | Filter results to a specific interchange number |
| `sku` | No | Filter results by SKU (decoded to VariationID). Cannot be combined with `interchange` |

#### Example Requests

Get all fitment data:

```bash
curl https://api.wheelsamerica.com/inventory/fitment \
  -H "x-api-key: YOUR_API_KEY"
```

Filter by interchange number:

```bash
curl "https://api.wheelsamerica.com/inventory/fitment?interchange=70235" \
  -H "x-api-key: YOUR_API_KEY"
```

Filter by SKU:

```bash
curl "https://api.wheelsamerica.com/inventory/fitment?sku=W004821" \
  -H "x-api-key: YOUR_API_KEY"
```

#### Response Fields

| Field | Type | Description |
|-------|------|-------------|
| `Year` | number | Vehicle model year |
| `Make` | string | Vehicle manufacturer (e.g. `Toyota`) |
| `Model` | string | Vehicle model (e.g. `Camry`) |
| `Submodel` | string | Vehicle submodel/trim (e.g. `LE`, `Sport`) |
| `Engines` | string | Engine specification (e.g. `2.5L L4`) |

#### Responses

**200 OK**

```json
{
  "success": true,
  "count": 4,
  "fitment": [
    {
      "Year": 2020,
      "Make": "Toyota",
      "Model": "Camry",
      "Submodel": "LE",
      "Engines": "2.5L L4"
    },
    {
      "Year": 2021,
      "Make": "Toyota",
      "Model": "Camry",
      "Submodel": "SE",
      "Engines": "2.5L L4"
    }
  ]
}
```

**400 Bad Request** — Both `interchange` and `sku` were provided, or SKU could not be decoded.

```json
{ "error": "Provide either interchange or sku, not both" }
```

```json
{ "error": "Could not derive VariationID from SKU" }
```

**500 Internal Server Error**

```json
{ "error": "Failed to retrieve fitment data" }
```

---

### Get Locations

**GET** `/inventory/locations`

Retrieve per-store inventory counts for a wheel by SKU. Returns all 6 warehouse locations with their unit count (0 if none in stock at that location).

#### Request Headers

| Header | Required | Description |
|--------|----------|-------------|
| `x-api-key` | Yes | Your API key |

#### Query Parameters

| Param | Required | Description |
|-------|----------|-------------|
| `sku` | Yes | Wheel SKU (e.g. `W004821`) |

#### Example Request

```bash
curl "https://api.wheelsamerica.com/inventory/locations?sku=W004821" \
  -H "x-api-key: YOUR_API_KEY"
```

#### Response Fields

| Field | Type | Description |
|-------|------|-------------|
| `sku` | string | The SKU that was queried |
| `locations` | object | Per-warehouse breakdown |
| `locations.<STORE>.qty` | number | Total units at that warehouse |
| `locations.<STORE>.finished` | number | Finished (ready to ship) units |
| `locations.<STORE>.cores` | number | Core (unfinished) units |

Store codes: `DA` (Dallas), `SA` (San Antonio), `HO` (Houston), `LA` (Los Angeles), `SF` (San Francisco), `PH` (Philadelphia).

#### Responses

**200 OK**

```json
{
  "success": true,
  "sku": "W004821",
  "locations": {
    "DA": { "qty": 3, "finished": 2, "cores": 1 },
    "SA": { "qty": 0, "finished": 0, "cores": 0 },
    "HO": { "qty": 1, "finished": 1, "cores": 0 },
    "LA": { "qty": 0, "finished": 0, "cores": 0 },
    "SF": { "qty": 2, "finished": 2, "cores": 0 },
    "PH": { "qty": 0, "finished": 0, "cores": 0 }
  }
}
```

**400 Bad Request** — Missing SKU or could not decode it.

```json
{ "error": "Missing required parameter: sku" }
```

```json
{ "error": "Could not derive VariationID from SKU" }
```

**500 Internal Server Error**

```json
{ "error": "Failed to retrieve location data" }
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
