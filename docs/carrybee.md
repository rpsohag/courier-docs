# Carrybee API Documentation v2.0

> Complete API reference for building a Socialite-like courier package integration with Carrybee

## Overview

Carrybee Developers' API v2.0 allows merchants to connect through secure and reliable APIs for seamless parcel operations. With these APIs, you can create orders, manage stores, handle locations, and receive real-time webhook updates.

## Base URLs

### Production Environment
**Base URL:** `https://developers.carrybee.com/`

### Sandbox Environment
**Base URL:** `https://stage-sandbox.carrybee.com/`

> The sandbox environment is designed for testing and experimentation. Use the sandbox credentials for development and testing before switching to production.

## Authentication

All API requests require authentication using three headers:

```http
Client-ID: {{client_id}}
Client-Secret: {{client_secret}}
Client-Context: {{client_context}}
```

### Sandbox Credentials
- **Client ID:** `1a89c1a6-fc68-4395-9c09-628e0d3eaafc`
- **Client Secret:** `1d7152c9-5b2d-4e4e-9c20-652b93333704`
- **Client Context:** `DzJwPsx31WaTbS745XZoBjmQLcNqwK`

### Production Credentials
Contact Carrybee to obtain your production credentials.

### Authentication Headers Example
```bash
--header 'Client-ID: {{client_id}}' \
--header 'Client-Secret: {{client_secret}}' \
--header 'Client-Context: {{client_context}}'
```

---

## API Endpoints

### 1. Get Cities

Retrieve all available cities for delivery.

**Method:** `GET`

**Endpoint:** `/api/v2/cities`

**Sample Request:**
```bash
curl --location '{{base_url}}/api/v2/cities' \
--header 'Client-ID: {{client_id}}' \
--header 'Client-Secret: {{client_secret}}' \
--header 'Client-Context: {{client_context}}'
```

**Sample Response:**
```json
{
  "error": false,
  "message": "City list fetched successfully",
  "data": {
    "cities": [
      {
        "id": 1,
        "name": "Bagerhat"
      },
      {
        "id": 14,
        "name": "Dhaka"
      },
      {
        "id": 15,
        "name": "Dinajpur"
      }
    ]
  }
}
```

**Response Fields:**

| Field Name | Field Type | Description |
|------------|------------|-------------|
| error | boolean | Indicates if the request was successful |
| message | string | Descriptive message about the request |
| data.cities | array | List of available cities |
| data.cities[].id | integer | Unique identifier of the city |
| data.cities[].name | string | Name of the city |

**Error Response:**
```json
{
  "error": true,
  "message": "variable response message text"
}
```

---

### 2. Get Zones

Retrieve zones for a specific city.

**Method:** `GET`

**Endpoint:** `/api/v2/cities/{{city_id}}/zones`

**Parameters:**

| Field Name | Field Type | Required | Description |
|------------|------------|----------|-------------|
| city_id | integer | Yes | ID from Get Cities response (data.cities[*].id) |

**Sample Request:**
```bash
curl --location '{{base_url}}/api/v2/cities/{{city_id}}/zones' \
--header 'Client-ID: {{client_id}}' \
--header 'Client-Secret: {{client_secret}}' \
--header 'Client-Context: {{client_context}}'
```

**Sample Response:**
```json
{
  "error": false,
  "message": "Zones",
  "data": {
    "zones": [
      {
        "id": 290,
        "name": "Zoo Road",
        "city_id": 14
      },
      {
        "id": 356,
        "name": "Zirani",
        "city_id": 14
      },
      {
        "id": 468,
        "name": "Zirabo",
        "city_id": 14
      }
    ]
  }
}
```

**Response Fields:**

| Field Name | Field Type | Description |
|------------|------------|-------------|
| data.zones | array | List of zones in the city |
| data.zones[].id | integer | Unique identifier of the zone |
| data.zones[].name | string | Name of the zone |
| data.zones[].city_id | integer | ID of the city the zone belongs to |

---

### 3. Get Areas

Retrieve areas for a specific zone.

**Method:** `GET`

**Endpoint:** `/api/v2/cities/{{city_id}}/zones/{{zone_id}}/areas`

**Parameters:**

| Field Name | Field Type | Required | Description |
|------------|------------|----------|-------------|
| city_id | integer | Yes | ID from Get Cities response |
| zone_id | integer | Yes | ID from Get Zones response (data.zones[*].id) |

**Sample Request:**
```bash
curl --location '{{base_url}}/api/v2/cities/{{city_id}}/zones/{{zone_id}}/areas' \
--header 'Client-ID: {{client_id}}' \
--header 'Client-Secret: {{client_secret}}' \
--header 'Client-Context: {{client_context}}'
```

**Sample Response:**
```json
{
  "error": false,
  "message": "Area list fetched successfully",
  "data": {
    "areas": [
      {
        "id": 8803,
        "name": "Zirani Kata",
        "zone_id": 356
      },
      {
        "id": 8748,
        "name": "Walton building",
        "zone_id": 356
      },
      {
        "id": 8809,
        "name": "Vobani Pur Bottla",
        "zone_id": 356
      }
    ]
  }
}
```

**Response Fields:**

| Field Name | Field Type | Description |
|------------|------------|-------------|
| data.areas | array | List of areas in the zone |
| data.areas[].id | integer | Unique identifier of the area |
| data.areas[].name | string | Name of the area |
| data.areas[].zone_id | integer | ID of the zone the area belongs to |

---

### 4. Get Area Suggestion

Search for areas by text (searches across area/zone/city).

**Method:** `GET`

**Endpoint:** `/api/v2/area-suggestion?search={{search_text}}`

**Parameters:**

| Field Name | Field Type | Required | Description |
|------------|------------|----------|-------------|
| search | string | Yes | Search text (minimum 3 characters) |

**Sample Request:**
```bash
curl --location '{{base_url}}/api/v2/area-suggestion?search=sector' \
--header 'Client-ID: {{client_id}}' \
--header 'Client-Secret: {{client_secret}}' \
--header 'Client-Context: {{client_context}}'
```

**Sample Response:**
```json
{
  "error": false,
  "message": "Suggested area",
  "data": {
    "items": [
      {
        "city_id": 14,
        "city_name": "Dhaka",
        "zone_id": 151,
        "zone_name": "Uttara sector 6",
        "area_id": 5100,
        "area_name": "Alal Avenue"
      },
      {
        "city_id": 14,
        "city_name": "Dhaka",
        "zone_id": 151,
        "zone_name": "Uttara sector 6",
        "area_id": 5102,
        "area_name": "BNS Centre"
      },
      {
        "city_id": 14,
        "city_name": "Dhaka",
        "zone_id": 151,
        "zone_name": "Uttara sector 6",
        "area_id": 5101,
        "area_name": "House Building"
      }
    ]
  }
}
```

**Response Fields:**

| Field Name | Field Type | Description |
|------------|------------|-------------|
| data.items | array | List of suggested areas |
| data.items[].city_id | integer | ID of the city |
| data.items[].city_name | string | Name of the city |
| data.items[].zone_id | integer | ID of the zone |
| data.items[].zone_name | string | Name of the zone |
| data.items[].area_id | integer | ID of the area |
| data.items[].area_name | string | Name of the area |

---

### 5. Get City & Zone ID from Address

Extract city and zone IDs from a text address.

**Method:** `POST`

**Endpoint:** `/api/v2/address-details`

**Parameters:**

| Field Name | Field Type | Required | Description |
|------------|------------|----------|-------------|
| query | string | Yes | Address text (minimum 10 characters) |

**Sample Request:**
```bash
curl --location '{{base_url}}/api/v2/address-details' \
--header 'Content-Type: application/json' \
--header 'Client-ID: {{client_id}}' \
--header 'Client-Secret: {{client_secret}}' \
--header 'Client-Context: {{client_context}}' \
--data '{
    "query": "Baridhara Jame Masjid, baridhara, Dhaka"
}'
```

**Sample Response:**
```json
{
  "error": false,
  "message": "Address details",
  "data": {
    "city_id": 14,
    "zone_id": 161
  }
}
```

**Response Fields:**

| Field Name | Field Type | Description |
|------------|------------|-------------|
| data.city_id | integer | ID of the city |
| data.zone_id | integer | ID of the zone |

**Validation Error (422):**
```json
{
  "error": true,
  "message": "Validation error",
  "causes": {
    "query": [
      {
        "type": "min",
        "attribute": {
          "value": 10
        }
      }
    ]
  }
}
```

---

### 6. Create a New Store

Create a new pickup/return store location.

**Method:** `POST`

**Endpoint:** `/api/v2/stores`

**Parameters:**

| Field Name | Field Type | Required | Description |
|------------|------------|----------|-------------|
| name | string | Yes | Store name (3-30 characters) |
| contact_person_name | string | Yes | Contact person name (3-30 characters) |
| contact_person_number | string | Yes | Contact person phone number |
| contact_person_secondary_number | string | No | Secondary contact phone number |
| address | string | Yes | Store address (3-100 characters) |
| city_id | integer | Yes | City ID from Get Cities response |
| zone_id | integer | Yes | Zone ID from Get Zones response |
| area_id | integer | Yes | Area ID from Get Areas response |
| lat | float | No | Latitude of the store location |
| lng | float | No | Longitude of the store location |

**Sample Request:**
```bash
curl --location '{{base_url}}/api/v2/stores' \
--header 'Content-Type: application/json' \
--header 'Client-ID: {{client_id}}' \
--header 'Client-Secret: {{client_secret}}' \
--header 'Client-Context: {{client_context}}' \
--data '{
    "name": "store_name",
    "contact_person_name": "store_contact_person_s_name",
    "contact_person_number": "store_contact_person_s_number",
    "contact_person_secondary_number": "store_contact_person_s_secondary_number_otherwise_do_not_include_or_null",
    "address": "address_of_the_store",
    "city_id": {{city_id}},
    "zone_id": {{zone_id}},
    "area_id": {{area_id}}
}'
```

**Sample Response:**
```json
{
  "error": false,
  "message": "Store created successfully"
}
```

**Validation Error (422):**
```json
{
  "error": true,
  "message": "Validation error",
  "causes": {
    "city_id": [
      {
        "type": "exists"
      }
    ],
    "name": [
      {
        "type": "required"
      }
    ],
    "address": [
      {
        "type": "max",
        "attribute": {
          "value": 100
        }
      }
    ]
  }
}
```

---

### 7. Get List of Stores

Retrieve all stores associated with your account.

**Method:** `GET`

**Endpoint:** `/api/v2/stores`

**Sample Request:**
```bash
curl --location '{{base_url}}/api/v2/stores' \
--header 'Client-ID: {{client_id}}' \
--header 'Client-Secret: {{client_secret}}' \
--header 'Client-Context: {{client_context}}'
```

**Sample Response:**
```json
{
  "error": false,
  "message": "Store list",
  "data": {
    "stores": [
      {
        "id": "abcd-1234",
        "name": "Store of Anik at 20250729-085709",
        "contact_person_name": "Anik",
        "contact_person_number": "8801652241276",
        "contact_person_secondary_number": "8801652241274",
        "address": "address of Store of Anik at 20250729-085709",
        "city_id": 14,
        "zone_id": 5,
        "area_id": 282,
        "is_active": false,
        "is_approved": false,
        "is_default_pickup_store": false,
        "is_default_return_store": false
      },
      {
        "id": "wxyz-1234",
        "name": "Store of Anik at 20250729-105709",
        "contact_person_name": "Anik",
        "contact_person_number": "8801652241271",
        "address": "address of Store of Anik at 20250729-105709",
        "city_id": 14,
        "zone_id": 5,
        "area_id": 111,
        "is_active": true,
        "is_approved": true,
        "is_default_pickup_store": true,
        "is_default_return_store": true
      }
    ],
    "pending_count": 1
  }
}
```

**Response Fields:**

| Field Name | Field Type | Description |
|------------|------------|-------------|
| data.stores | array | List of stores |
| data.stores[].id | string | Unique store identifier |
| data.stores[].name | string | Store name |
| data.stores[].contact_person_name | string | Contact person name |
| data.stores[].contact_person_number | string | Contact person phone number |
| data.stores[].contact_person_secondary_number | string | Secondary contact number (optional) |
| data.stores[].address | string | Store address |
| data.stores[].city_id | integer | City ID |
| data.stores[].zone_id | integer | Zone ID |
| data.stores[].area_id | integer | Area ID |
| data.stores[].is_active | boolean | Store active status |
| data.stores[].is_approved | boolean | Store approval status |
| data.stores[].is_default_pickup_store | boolean | Default pickup store flag |
| data.stores[].is_default_return_store | boolean | Default return store flag |
| data.pending_count | integer | Count of stores pending approval |

---

### 8. Create a Single Order

Create a new delivery order.

**Method:** `POST`

**Endpoint:** `/api/v2/orders`

**Parameters:**

| Field Name | Field Type | Required | Description |
|------------|------------|----------|-------------|
| store_id | string | Yes | Store ID from Get Stores response |
| merchant_order_id | string | No | Merchant's order ID (max 50 characters) |
| delivery_type | integer | Yes | 1 for Normal, 2 for Express |
| product_type | integer | Yes | 1 for Parcel, 2 for Book, 3 for Document |
| recipient_phone | string | Yes | Recipient phone number |
| recipient_secendary_phone | string | No | Recipient secondary phone number |
| recipient_name | string | Yes | Recipient name (2-99 characters) |
| recipient_address | string | Yes | Recipient address (10-200 characters) |
| city_id | integer | Yes | City ID from Get Cities response |
| zone_id | integer | Yes | Zone ID from Get Zones response |
| area_id | integer | No | Area ID from Get Areas response |
| special_instruction | string | No | Special instructions (max 256 characters) |
| product_description | string | No | Product description (max 256 characters) |
| item_weight | integer | Yes | Item weight in grams (1-25000) |
| item_quantity | integer | No | Item quantity (1-200) |
| collectable_amount | integer | No | Amount to collect in Taka (0-100000) |
| is_closed | boolean | No | Closed box item (default: false) |

**Sample Request:**
```bash
curl --location '{{base_url}}/api/v2/orders' \
--header 'Content-Type: application/json' \
--header 'Client-ID: {{client_id}}' \
--header 'Client-Secret: {{client_secret}}' \
--header 'Client-Context: {{client_context}}' \
--data '{
    "store_id": "{{store_id}}",
    "merchant_order_id": "order-1234",
    "delivery_type": 1,
    "product_type": 1,
    "recipient_phone": "01652241276",
    "recipient_secendary_phone": "nullable recipient secondary phone number",
    "recipient_name": "recipient name",
    "recipient_address": "receipient address",
    "city_id": {{city_id}},
    "zone_id": {{zone_id}},
    "area_id": {{area_id}},
    "special_instruction": "null_or_string",
    "product_description": "null_or_string",
    "item_weight": 500,
    "item_quantity": 2,
    "collectable_amount": 15000,
    "is_closed": false
}'
```

**Sample Response:**
```json
{
  "error": false,
  "message": "Order Created Successfully",
  "data": {
    "order": {
      "consignment_id": "FX1212124433",
      "store_id": "a1b2c3d4",
      "merchant_order_id": "order-1234",
      "collectable_amount": "15000",
      "cod_fee": 15,
      "delivery_fee": "60"
    }
  }
}
```

**Response Fields:**

| Field Name | Field Type | Description |
|------------|------------|-------------|
| data.order.consignment_id | string | Carrybee consignment ID |
| data.order.store_id | string | Store ID |
| data.order.merchant_order_id | string | Merchant order ID |
| data.order.collectable_amount | string | Amount to collect |
| data.order.cod_fee | float | COD fee |
| data.order.delivery_fee | string | Delivery fee |

**Validation Error (422):**
```json
{
  "error": true,
  "message": "Validation error",
  "causes": {
    "city_id": [
      {
        "type": "exists"
      }
    ],
    "recipient_phone": [
      {
        "type": "required"
      }
    ],
    "recipient_secendary_phone": [
      {
        "type": "phone"
      }
    ],
    "product_type": [
      {
        "type": "in",
        "attribute": {
          "values": [1, 2, 3]
        }
      }
    ],
    "address": [
      {
        "type": "max",
        "attribute": {
          "value": 255
        }
      }
    ]
  }
}
```

---

### 9. Create Bulk Order

Create multiple orders in a single request.

**Method:** `POST`

**Endpoint:** `/api/v2/orders-bulk`

**Parameters:**

An array of order objects. Each order object follows the same validation as Create a Single Order.

| Field Name | Field Type | Required | Description |
|------------|------------|----------|-------------|
| orders | array | Yes | Array of order objects |

**Sample Request:**
```bash
curl --location '{{base_url}}/api/v2/orders-bulk' \
--header 'Content-Type: application/json' \
--header 'Client-ID: {{client_id}}' \
--header 'Client-Secret: {{client_secret}}' \
--header 'Client-Context: {{client_context}}' \
--data '{
    "orders": [
        {
            "store_id": "{{store_id}}",
            "merchant_order_id": "order-1234",
            "delivery_type": 1,
            "product_type": 1,
            "recipient_phone": "01652241276",
            "recipient_secendary_phone": "nullable recipient secondary phone number",
            "recipient_name": "recipient name",
            "recipient_address": "receipient address",
            "city_id": {{city_id}},
            "zone_id": {{zone_id}},
            "area_id": {{area_id}},
            "special_instruction": "null_or_string",
            "product_description": "null_or_string",
            "item_weight": 500,
            "item_quantity": 2,
            "collectable_amount": 15000,
            "is_closed": false
        },
        {
            "store_id": "{{store_id}}",
            "merchant_order_id": "order-1234",
            "delivery_type": 1,
            "product_type": 1,
            "recipient_phone": "01652241276",
            "recipient_secendary_phone": "nullable recipient secondary phone number",
            "recipient_name": "recipient name",
            "recipient_address": "receipient address",
            "city_id": "{{city_id}}",
            "zone_id": "{{zone_id}}",
            "area_id": "{{area_id}}",
            "special_instruction": "null_or_string",
            "product_description": "null_or_string",
            "item_weight": 500,
            "item_quantity": 2,
            "collectable_amount": 15000
        }
    ]
}'
```

**Sample Response:**
```json
{
  "error": false,
  "message": "Order list accepted to be processed"
}
```

> You will receive webhook for each of the orders created if subscribed.

---

### 10. Cancel an Order

Cancel an existing order.

**Method:** `POST`

**Endpoint:** `/api/v2/orders/{{consignment_id}}/cancel`

**Parameters:**

| Field Name | Field Type | Required | Description |
|------------|------------|----------|-------------|
| cancellation_reason | string | Yes | Reason for cancellation (max 200 characters) |

**Sample Request:**
```bash
curl --location '{{base_url}}/api/v2/orders/{{consignment_id}}/cancel' \
--header 'Content-Type: application/json' \
--header 'Client-ID: {{client_id}}' \
--header 'Client-Secret: {{client_secret}}' \
--header 'Client-Context: {{client_context}}' \
--data '{
    "cancellation_reason": "cancellation reason"
}'
```

**Sample Response:**
```json
{
  "error": false,
  "message": "Order cancelled successfully"
}
```

---

### 11. Get Order Details

Retrieve detailed information about an order.

**Method:** `GET`

**Endpoint:** `/api/v2/orders/{{consignment_id}}/details`

**Parameters:**

| Field Name | Field Type | Required | Description |
|------------|------------|----------|-------------|
| consignment_id | string | Yes | Carrybee consignment ID or merchant order ID (min 3 characters) |

**Sample Request:**
```bash
curl --location '{{base_url}}/api/v2/orders/{{consignment_id}}/details' \
--header 'Client-ID: {{client_id}}' \
--header 'Client-Secret: {{client_secret}}' \
--header 'Client-Context: {{client_context}}'
```

**Sample Response:**
```json
{
  "error": false,
  "message": "Order details",
  "data": {
    "transfer_status": "In transit",
    "store_id": "a1b2c3",
    "consignment_id": "FX1212124433",
    "merchant_order_id": "order-1234",
    "recipient_name": "recipient name",
    "recipient_phone": "8801652241276",
    "recipient_secondary_phone": "8801652241276",
    "recipient_address": "recipient address",
    "collectable_amount": "1000",
    "collected_amount": "0",
    "cod_fee": 0,
    "delivery_fee": "105",
    "attempt": 0,
    "updated_at": "2025-07-30T10:11:12+00:00"
  }
}
```

**Response Fields:**

| Field Name | Field Type | Description |
|------------|------------|-------------|
| data.transfer_status | string | Current transfer status |
| data.store_id | string | Store ID |
| data.consignment_id | string | Consignment ID |
| data.merchant_order_id | string | Merchant order ID (optional) |
| data.recipient_name | string | Recipient name |
| data.recipient_phone | string | Recipient phone number |
| data.recipient_secondary_phone | string | Recipient secondary phone (optional) |
| data.recipient_address | string | Recipient address |
| data.collectable_amount | string | Amount to collect |
| data.collected_amount | string | Amount collected |
| data.cod_fee | float | COD fee |
| data.delivery_fee | string | Delivery fee |
| data.attempt | integer | Number of delivery attempts |
| data.reason | string | Cancellation/reason (optional) |
| data.updated_at | string | Last update timestamp (ISO 8601) |
| data.invoice_id | string | Invoice ID (optional) |
| data.payment_status | string | Payment status (optional) |

> All amounts are in Taka.

**Error Response (404):**
```json
{
  "error": true,
  "message": "order not found"
}
```

---

## Webhook Integration

Configure your webhook URL in your Carrybee dashboard to receive real-time updates about order status changes.

### Webhook Configuration

**Method:** `POST`

**Content-Type:** `application/json`

**Webhook URL:** Configure in Carrybee dashboard

### Webhook Authentication

Carrybee sends webhook requests with a signature header for verification:

```http
X-Carrybee-Webhook-Signature: {{WEBHOOK_SIGNATURE}}
```

> **Note:** The signature/credential is provided by you so that you can verify that the request is sent from the Carrybee server and not from someone else.

### Webhook Event Payloads

#### 1. Order Created

**Event:** `order.created`

**Payload:**
```json
{
  "event": "order.created",
  "store_id": "a1b2c3d4",
  "consignment_id": "FX1212124433",
  "merchant_order_id": "order-1234",
  "timestamptz": "2025-07-30T10:11:12+00:00",
  "collectable_amount": "1592",
  "cod_fee": 15.92,
  "delivery_fee": "85"
}
```

**Field Notes:**
- `event` (string): The current event name
- `store_id`, `consignment_id` (string): Store and consignment identifiers
- `merchant_order_id` (null|string): Your merchant order ID
- `timestamptz` (string): Timestamp when the event occurred (ISO 8601)
- `collectable_amount` (string): Amount to collect, wrapping an integer
- `cod_fee` (float): Cash on delivery fee
- `delivery_fee` (string): Delivery charge, wrapping an integer
- **All amounts are in Taka**

---

#### 2. Order Updated

**Event:** `order.updated`

**Payload:**
```json
{
  "event": "order.updated",
  "store_id": "a1b2c3d4",
  "consignment_id": "FX1212124433",
  "merchant_order_id": "order-1234",
  "timestamptz": "2025-07-30T10:11:12+00:00",
  "collectable_amount": "1592",
  "cod_fee": 15.92,
  "delivery_fee": "85"
}
```

**Field Notes:** Same as Order Created

---

#### 3. Pickup Requested

**Event:** `order.pickup-requested`

**Payload:**
```json
{
  "event": "order.pickup-requested",
  "store_id": "a1b2c3d4",
  "consignment_id": "FX1212124433",
  "merchant_order_id": "order-1234",
  "timestamptz": "2025-07-30T10:11:12+00:00"
}
```

**Field Notes:**
- `event` (string): The current event name
- `store_id`, `consignment_id` (string): Store and consignment identifiers
- `merchant_order_id` (null|string): Your merchant order ID
- `timestamptz` (string): Timestamp when the event occurred (ISO 8601)

---

#### 4. Assigned for Pickup

**Event:** `order.assigned-for-pickup`

**Payload:**
```json
{
  "event": "order.assigned-for-pickup",
  "store_id": "a1b2c3d4",
  "consignment_id": "FX1212124433",
  "merchant_order_id": "order-1234",
  "timestamptz": "2025-07-30T10:11:12+00:00"
}
```

**Field Notes:** Same as Pickup Requested

---

#### 5. Picked

**Event:** `order.picked`

**Payload:**
```json
{
  "event": "order.picked",
  "store_id": "a1b2c3d4",
  "consignment_id": "FX1212124433",
  "merchant_order_id": "order-1234",
  "timestamptz": "2025-07-30T10:11:12+00:00"
}
```

**Field Notes:** Same as Pickup Requested

---

#### 6. Pickup Failed

**Event:** `order.pickup-failed`

**Payload:**
```json
{
  "event": "order.pickup-failed",
  "store_id": "a1b2c3d4",
  "consignment_id": "FX1212124433",
  "merchant_order_id": "order-1234",
  "timestamptz": "2025-07-30T10:11:12+00:00"
}
```

**Field Notes:** Same as Pickup Requested

---

#### 7. Pickup Cancelled

**Event:** `order.pickup-cancelled`

**Payload:**
```json
{
  "event": "order.pickup-cancelled",
  "store_id": "a1b2c3d4",
  "consignment_id": "FX1212124433",
  "merchant_order_id": "order-1234",
  "timestamptz": "2025-07-30T10:11:12+00:00"
}
```

**Field Notes:** Same as Pickup Requested

---

#### 8. At the Sorting Hub

**Event:** `order.at-the-sorting-hub`

**Payload:**
```json
{
  "event": "order.at-the-sorting-hub",
  "store_id": "a1b2c3d4",
  "consignment_id": "FX1212124433",
  "merchant_order_id": "order-1234",
  "timestamptz": "2025-07-30T10:11:12+00:00"
}
```

**Field Notes:** Same as Pickup Requested

---

#### 9. On the Way to Central Warehouse

**Event:** `order.on-the-way-to-central-warehouse`

**Payload:**
```json
{
  "event": "order.on-the-way-to-central-warehouse",
  "store_id": "a1b2c3d4",
  "consignment_id": "FX1212124433",
  "merchant_order_id": "order-1234",
  "timestamptz": "2025-07-30T10:11:12+00:00"
}
```

**Field Notes:** Same as Pickup Requested

---

#### 10. At Central Warehouse

**Event:** `order.at-central-warehouse`

**Payload:**
```json
{
  "event": "order.at-central-warehouse",
  "store_id": "a1b2c3d4",
  "consignment_id": "FX1212124433",
  "merchant_order_id": "order-1234",
  "timestamptz": "2025-07-30T10:11:12+00:00"
}
```

**Field Notes:** Same as Pickup Requested

---

#### 11. In Transit

**Event:** `order.in-transit`

**Payload:**
```json
{
  "event": "order.in-transit",
  "store_id": "a1b2c3d4",
  "consignment_id": "FX1212124433",
  "merchant_order_id": "order-1234",
  "timestamptz": "2025-07-30T10:11:12+00:00"
}
```

**Field Notes:** Same as Pickup Requested

---

#### 12. Received at Last Mile Hub

**Event:** `order.received-at-last-mile-hub`

**Payload:**
```json
{
  "event": "order.received-at-last-mile-hub",
  "store_id": "a1b2c3d4",
  "consignment_id": "FX1212124433",
  "merchant_order_id": "order-1234",
  "timestamptz": "2025-07-30T10:11:12+00:00"
}
```

**Field Notes:** Same as Pickup Requested

---

#### 13. Assigned for Delivery

**Event:** `order.assigned-for-delivery`

**Payload:**
```json
{
  "event": "order.assigned-for-delivery",
  "store_id": "a1b2c3d4",
  "consignment_id": "FX1212124433",
  "merchant_order_id": "order-1234",
  "timestamptz": "2025-07-30T10:11:12+00:00",
  "attempt": 1
}
```

**Field Notes:**
- Same basic fields as Pickup Requested
- `attempt` (integer): Number of delivery attempts

---

#### 14. Delivery On Hold

**Event:** `order.delivery-on-hold`

**Payload:**
```json
{
  "event": "order.delivery-on-hold",
  "store_id": "a1b2c3d4",
  "consignment_id": "FX1212124433",
  "merchant_order_id": "order-1234",
  "timestamptz": "2025-07-30T10:11:12+00:00",
  "attempt": 1,
  "reason": "This field may or may not contain data"
}
```

**Field Notes:**
- Same basic fields as Pickup Requested
- `attempt` (integer): Number of delivery attempts
- `reason` (null|string): Reason why delivery is on hold (may be null or contain data)

---

#### 15. Delivered

**Event:** `order.delivered`

**Payload:**
```json
{
  "event": "order.delivered",
  "store_id": "a1b2c3d4",
  "consignment_id": "FX1212124433",
  "merchant_order_id": "order-1234",
  "timestamptz": "2025-07-30T10:11:12+00:00",
  "collected_amount": "60",
  "attempt": 1
}
```

**Field Notes:**
- Same basic fields as Pickup Requested
- `collected_amount` (string): Amount collected from customer, wrapping an integer
- `attempt` (integer): Number of delivery attempts
- **All amounts are in Taka**

---

#### 16. Partial Delivery

**Event:** `order.partial-delivery`

**Payload:**
```json
{
  "event": "order.partial-delivery",
  "store_id": "a1b2c3d4",
  "consignment_id": "FX1212124433",
  "merchant_order_id": "order-1234",
  "timestamptz": "2025-07-30T10:11:12+00:00",
  "collected_amount": "60",
  "reason": "This field may or may not contain data",
  "attempt": 1
}
```

**Field Notes:**
- Same basic fields as Pickup Requested
- `collected_amount` (string): Amount collected from customer, wrapping an integer
- `reason` (null|string): Reason for partial delivery (may be null or contain data)
- `attempt` (integer): Number of delivery attempts
- **All amounts are in Taka**

---

#### 17. Delivery Failed

**Event:** `order.delivery-failed`

**Payload:**
```json
{
  "event": "order.delivery-failed",
  "store_id": "a1b2c3d4",
  "consignment_id": "FX1212124433",
  "merchant_order_id": "order-1234",
  "timestamptz": "2025-07-30T10:11:12+00:00",
  "reason": "This field may or may not contain data",
  "attempt": 1
}
```

**Field Notes:**
- Same basic fields as Pickup Requested
- `reason` (null|string): Reason for delivery failure (may be null or contain data)
- `attempt` (integer): Number of delivery attempts

---

#### 18. Returned

**Event:** `order.returned`

**Payload:**
```json
{
  "event": "order.returned",
  "store_id": "a1b2c3d4",
  "consignment_id": "FX1212124433",
  "merchant_order_id": "order-1234",
  "timestamptz": "2025-07-30T10:11:12+00:00",
  "reason": "This field may or may not contain data"
}
```

**Field Notes:**
- Same basic fields as Pickup Requested
- `reason` (null|string): Reason for return (may be null or contain data)

---

#### 19. Paid Return

**Event:** `order.paid-return`

**Payload:**
```json
{
  "event": "order.paid-return",
  "store_id": "a1b2c3d4",
  "consignment_id": "FX1212124433",
  "merchant_order_id": "order-1234",
  "timestamptz": "2025-07-30T10:11:12+00:00",
  "collected_amount": "60",
  "reason": "This field may or may not contain data",
  "attempt": 1
}
```

**Field Notes:**
- Same basic fields as Pickup Requested
- `collected_amount` (string): Amount collected, wrapping an integer
- `reason` (null|string): Reason for paid return (may be null or contain data)
- `attempt` (integer): Number of delivery attempts
- **All amounts are in Taka**

---

#### 20. Exchange

**Event:** `order.exchange`

**Payload:**
```json
{
  "event": "order.exchange",
  "store_id": "a1b2c3d4",
  "consignment_id": "EX1212124433",
  "merchant_order_id": "order-1234",
  "timestamptz": "2025-07-30T10:11:12+00:00",
  "collected_amount": "60",
  "reason": "This field may or may not contain data"
}
```

**Field Notes:**
- Same basic fields as Pickup Requested
- `collected_amount` (string): Amount collected, wrapping an integer
- `reason` (null|string): Reason for exchange (may be null or contain data)
- **All amounts are in Taka**

---

#### 21. Paid

**Event:** `order.paid`

**Payload:**
```json
{
  "event": "order.paid",
  "store_id": "a1b2c3d4",
  "consignment_id": "FX1212124433",
  "merchant_order_id": "order-1234",
  "timestamptz": "2025-07-30T10:11:12+00:00",
  "invoice_id": "IX212124433"
}
```

**Field Notes:**
- Same basic fields as Pickup Requested
- `invoice_id` (string): Invoice identifier

---

#### 22. Returned at Sorting

**Event:** `order.returned-at-sorting`

**Payload:**
```json
{
  "event": "order.returned-at-sorting",
  "store_id": "a1b2c3d4",
  "consignment_id": "FX1212124433",
  "merchant_order_id": "order-1234",
  "timestamptz": "2025-07-30T10:11:12+00:00"
}
```

**Field Notes:** Same as Pickup Requested

---

#### 23. Returned In Transit

**Event:** `order.returned-in-transit`

**Payload:**
```json
{
  "event": "order.returned-in-transit",
  "store_id": "a1b2c3d4",
  "consignment_id": "FX1212124433",
  "merchant_order_id": "order-1234",
  "timestamptz": "2025-07-30T10:11:12+00:00"
}
```

**Field Notes:** Same as Pickup Requested

---

#### 24. Returned to Merchant

**Event:** `order.returned-to-merchant`

**Payload:**
```json
{
  "event": "order.returned-to-merchant",
  "store_id": "a1b2c3d4",
  "consignment_id": "FX1212124433",
  "merchant_order_id": "order-1234",
  "timestamptz": "2025-07-30T10:11:12+00:00"
}
```

**Field Notes:** Same as Pickup Requested

---

### Testing Your Webhook

You can test your webhook endpoint using curl:

**Request:**
```bash
curl --location '{{webhook_url}}' \
--header 'X-Carrybee-Webhook-Signature: "{{WEBHOOK_SIGNATURE}}"' \
--data '{
     "content": "your webhook event data"
}'
```

**Parameters:**
- `{{webhook_url}}`: The URL you provided - this is where you'll receive the data based on your subscription
- `{{WEBHOOK_SIGNATURE}}`: The signature/credential you provided - use this to verify requests are from Carrybee server
- **content**: The data section will contain the actual event data based on the event you have subscribed to, exactly as specified in the event section above

### Common Webhook Fields

| Field Name | Field Type | Description |
|------------|------------|-------------|
| `event` | string | Event name identifier |
| `store_id` | string | Store ID |
| `consignment_id` | string | Consignment ID |
| `merchant_order_id` | string/null | Merchant order ID (nullable) |
| `timestamptz` | string | Event timestamp (ISO 8601 format) |
| `collectable_amount` | string | Amount to collect in Taka (string wrapping integer) |
| `cod_fee` | float | Cash on delivery fee |
| `delivery_fee` | string | Delivery fee in Taka (string wrapping integer) |
| `collected_amount` | string | Amount collected in Taka (string wrapping integer) |
| `attempt` | integer | Delivery attempt number |
| `reason` | string/null | Reason for status change (nullable) |
| `invoice_id` | string | Invoice ID |

### Webhook Best Practices

1. **Signature Verification:** Always verify the `X-Carrybee-Webhook-Signature` header to ensure requests are from Carrybee
2. **Idempotency:** Handle duplicate webhook deliveries gracefully (check `consignment_id` and `timestamptz`)
3. **Async Processing:** Process webhooks asynchronously to avoid timeouts
4. **Event Logging:** Log all incoming webhooks with timestamps for audit trails
5. **Error Handling:** Implement robust error handling without exposing sensitive information
6. **Response Time:** Respond quickly (within 5 seconds) with appropriate status codes
7. **Event Mapping:** Map Carrybee events to your internal order statuses
8. **Database Transactions:** Use transactions when updating order status to maintain data consistency
9. **Testing:** Thoroughly test webhook handling before production deployment
10. **Monitoring:** Set up alerts for webhook delivery failures

---

## Quick Reference - All Endpoints

### Base URLs
- **Sandbox:** `https://stage-sandbox.carrybee.com/`
- **Production:** `https://developers.carrybee.com/`

### All API Endpoints

| # | Method | Endpoint | Description |
|---|--------|----------|-------------|
| 1 | `GET` | `/api/v2/cities` | Get all cities |
| 2 | `GET` | `/api/v2/cities/{{city_id}}/zones` | Get zones by city |
| 3 | `GET` | `/api/v2/cities/{{city_id}}/zones/{{zone_id}}/areas` | Get areas by zone |
| 4 | `GET` | `/api/v2/area-suggestion?search={{search_text}}` | Search areas |
| 5 | `POST` | `/api/v2/address-details` | Get city & zone from address |
| 6 | `POST` | `/api/v2/stores` | Create new store |
| 7 | `GET` | `/api/v2/stores` | Get all stores |
| 8 | `POST` | `/api/v2/orders` | Create single order |
| 9 | `POST` | `/api/v2/orders-bulk` | Create bulk orders |
| 10 | `POST` | `/api/v2/orders/{{consignment_id}}/cancel` | Cancel order |
| 11 | `GET` | `/api/v2/orders/{{consignment_id}}/details` | Get order details |

### Endpoint Categories

**Location Management:**
```
GET    /api/v2/cities
GET    /api/v2/cities/{{city_id}}/zones
GET    /api/v2/cities/{{city_id}}/zones/{{zone_id}}/areas
GET    /api/v2/area-suggestion?search={{search_text}}
POST   /api/v2/address-details
```

**Store Management:**
```
POST   /api/v2/stores
GET    /api/v2/stores
```

**Order Management:**
```
POST   /api/v2/orders
POST   /api/v2/orders-bulk
POST   /api/v2/orders/{{consignment_id}}/cancel
GET    /api/v2/orders/{{consignment_id}}/details
```

### Authentication Headers
```http
Client-ID: {{client_id}}
Client-Secret: {{client_secret}}
Client-Context: {{client_context}}
```

### Sandbox Credentials
```
Client ID: 1a89c1a6-fc68-4395-9c09-628e0d3eaafc
Client Secret: 1d7152c9-5b2d-4e4e-9c20-652b93333704
Client Context: DzJwPsx31WaTbS745XZoBjmQLcNqwK
```

---

## Integration Notes for Package Development

### Key Differences from RedX:

1. **Three-Header Authentication:** Carrybee uses Client-ID, Client-Secret, and Client-Context instead of Bearer token
2. **Hierarchical Location Structure:** City → Zone → Area (3-level hierarchy)
3. **Weight in Grams:** All weights must be specified in grams (1-25000 range)
4. **Delivery Types:** 1 (Normal) or 2 (Express)
5. **Product Types:** 1 (Parcel), 2 (Book), 3 (Document)
6. **Bulk Orders:** Separate endpoint for bulk order creation
7. **24 Webhook Events:** Comprehensive order lifecycle events

### Key Considerations for a Socialite-like Package:

1. **Environment Management:** Support both sandbox and production environments with easy credential switching
2. **Location Caching:** Implement caching for cities/zones/areas to reduce API calls
3. **Address Parsing:** Use the address-details endpoint to extract location IDs from raw addresses
4. **Bulk Operations:** Support bulk order creation with individual order validation
5. **Error Handling:** Handle validation errors with detailed field-level error messages
6. **Webhook Verification:** Implement signature verification using X-Carrybee-Webhook-Signature header
7. **Event Mapping:** Map webhook events to application-specific order statuses
8. **Rate Limiting:** Respect API rate limits if any
9. **Data Validation:** Validate all request data before sending to API (phone numbers, weight ranges, etc.)
10. **Response Normalization:** Normalize API responses for consistent interface across different courier providers

### Suggested Package Structure:

```
- Authentication Manager (Client-ID/Secret/Context handling)
- Location Manager (Cities, Zones, Areas with caching)
- Store Manager (Create, list stores)
- Order Manager (Create single/bulk orders, cancel, track)
- Address Parser (Extract location IDs from addresses)
- Webhook Handler (Process and verify webhook events)
- Response Builder (Normalize API responses)
- Exception Handler (Custom exceptions and validation errors)
- Validator (Request data validation)
```

---

*Last Updated: 2026-03-29*
*API Version: v2.0*
