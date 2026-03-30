# RedX API Documentation

> Complete API reference for building a Socialite-like courier package integration with RedX Bangladesh

## Overview

RedX OpenAPI allows merchants to connect through secure and reliable APIs for seamless parcel operations. With these APIs, you can create and manage parcels and pickup stores, streamlining your logistics workflow.

## Base URLs

### Sandbox Environment
**Base URL:** `https://sandbox.redx.com.bd/v1.0.0-beta`

> The sandbox environment is designed for testing and experimentation. All transactions and operations performed here are simulated and do not impact real data.

### Production Environment
**Base URL:** `https://openapi.redx.com.bd/v1.0.0-beta`

> The production environment is intended for live API operations that interact with real data. Ensure all integrations are thoroughly tested in the sandbox environment before transitioning to production.

## Authentication

All API requests require authentication using a Bearer token in the request header.

```http
API-ACCESS-TOKEN: Bearer {{jwt_token}}
```

### Authentication Header Example
```bash
--header 'API-ACCESS-TOKEN: Bearer {{jwt_token}}'
```

---

## API Endpoints

### 1. Track Parcel

Get tracking information for a specific parcel.

**Method:** `GET`

**Endpoint:** `/parcel/track/<:parcel_id>`

**Parameters:**

| Field Name | Field Type | Required | Description |
|------------|------------|----------|-------------|
| tracking_id | string | Yes | Unique identifier assigned to a parcel for tracking |

**Sample Request:**
```bash
curl --location '{{base_url}}/v1.0.0-beta/parcel/track/{{tracking_id}}' \
  --header 'API-ACCESS-TOKEN: Bearer {{jwt_token}}'
```

**Sample Response:**
```json
{
  "tracking": [
    {
      "message_en": "Package is created successfully",
      "message_bn": "পার্সেলটি সফল ভাবে প্লেস করা হয়েছে",
      "time": "2020-02-04T21:19:41.000Z"
    },
    {
      "message_en": "Package is picked up",
      "message_bn": "পার্সেল পিকাপ সম্পন্ন হয়েছে",
      "time": "2020-02-05T11:41:03.000Z"
    }
  ]
}
```

**Response Fields:**

| Field Name | Field Type | Optional | Description |
|------------|------------|----------|-------------|
| tracking | array | No | List of tracking updates for the parcel |
| tracking[].message_en | string | No | Tracking update message in English |
| tracking[].message_bn | string | No | Tracking update message in Bengali |
| tracking[].time | string (ISO 8601) | No | Timestamp of the tracking update |

---

### 2. Get Parcel Details

Retrieve detailed information about a specific parcel.

**Method:** `GET`

**Endpoint:** `/parcel/info/<:tracking_id>`

**Parameters:**

| Field Name | Field Type | Required | Description |
|------------|------------|----------|-------------|
| tracking_id | string | Yes | Unique identifier assigned to a parcel for tracking |

**Sample Request:**
```bash
curl --location '{{base_url}}/v1.0.0-beta/parcel/info/{{tracking_id}}' \
  --header 'API-ACCESS-TOKEN: Bearer {{jwt_token}}'
```

**Sample Response:**
```json
{
  "parcel": {
    "tracking_id": "21A427TU4BN3R",
    "customer_address": "Test Address",
    "delivery_area": "Mirpur DOHS",
    "delivery_area_id": 12,
    "charge": 60,
    "customer_name": "Test Customer",
    "customer_phone": "01987654321",
    "cash_collection_amount": 13293,
    "parcel_weight": 500,
    "merchant_invoice_id": "ACBD1234TEST",
    "status": "pickup-pending",
    "instruction": "",
    "created_at": "2021-04-27T08:29:14.000Z",
    "delivery_type": "regular",
    "value": "0",
    "pickup_location": {
      "id": 1,
      "name": "Malibag",
      "address": "Malibagh",
      "area_name": "Malibag",
      "area_id": 1
    }
  }
}
```

**Response Fields:**

| Field Name | Field Type | Optional | Description |
|------------|------------|----------|-------------|
| parcel | object | No | Parcel details |
| parcel.tracking_id | string | No | Unique tracking ID of the parcel |
| parcel.customer_address | string | No | Delivery address of the customer |
| parcel.delivery_area | string | No | Name of the delivery area |
| parcel.delivery_area_id | number | No | Unique ID of the delivery area |
| parcel.charge | number | No | Delivery charge for the parcel |
| parcel.customer_name | string | No | Name of the customer receiving the parcel |
| parcel.customer_phone | string | No | Phone number of the customer |
| parcel.cash_collection_amount | number | No | Amount to be collected from the customer |
| parcel.parcel_weight | number | No | Weight of the parcel in grams |
| parcel.merchant_invoice_id | string | No | Invoice ID provided by the merchant |
| parcel.status | string | No | Current status of the parcel |
| parcel.instruction | string | Yes | Additional delivery instructions |
| parcel.created_at | string (ISO 8601) | No | Timestamp when the parcel was created |
| parcel.delivery_type | string | No | Type of delivery (e.g., regular, express) |
| parcel.value | string | No | Declared value of the parcel |
| parcel.pickup_location | object | No | Details of the pickup location |
| parcel.pickup_location.id | number | No | Unique ID of the pickup location |
| parcel.pickup_location.name | string | No | Name of the pickup location |
| parcel.pickup_location.address | string | No | Address of the pickup location |
| parcel.pickup_location.area_name | string | No | Name of the area where the pickup location is situated |
| parcel.pickup_location.area_id | number | No | Unique ID of the pickup location area |

---

### 3. Create Parcel

Create a new parcel for delivery.

**Method:** `POST`

**Endpoint:** `/parcel`

**Parameters:**

| Field Name | Field Type | Required | Description |
|------------|------------|----------|-------------|
| customer_name | string | Yes | Full name of the customer receiving the parcel |
| customer_phone | string | Yes | Contact phone number of the customer |
| delivery_area | string | Yes | Name of the delivery area where the parcel will be sent |
| delivery_area_id | integer | Yes | Unique identifier of the delivery area |
| customer_address | string | Yes | Complete address of the customer for parcel delivery |
| cash_collection_amount | string | Yes | Amount to be collected from the customer upon delivery |
| parcel_weight | string | Yes | Weight of the parcel in appropriate units (e.g., kg, g) |
| value | string | Yes | Declared value of the parcel for compensation calculation |
| merchant_invoice_id | string | No | Invoice ID generated by the merchant for reference |
| instruction | string | No | Special instructions for the delivery |
| type | string | No | Defines the parcel type, mainly used for reverse shipments |
| parcel_details_json | object | No | JSON object containing additional parcel details |
| pickup_store_id | string | No | Identifier of the pickup store where the parcel is collected from |

**Parcel Details Object (parcel_details_json):**

| Field Name | Field Type | Required | Description |
|------------|------------|----------|-------------|
| name | string | Yes | Name of the item inside the parcel |
| category | string | Yes | Category of the item (e.g., electronics, clothing, etc.) |
| value | integer | Yes | Declared value of the individual item |

**Sample Request:**
```bash
curl --location '{{base_url}}/v1.0.0-beta/parcel' \
  --header 'API-ACCESS-TOKEN: Bearer {{jwt_token}}' \
  --header 'Content-Type: application/json' \
  --data '{
    "customer_name": "{{customer_name}}",
    "customer_phone": "{{customer_phone}}",
    "delivery_area": "{{delivery_area}}",
    "delivery_area_id": {{delivery_area_id}},
    "customer_address": "{{customer_address}}",
    "merchant_invoice_id": "{{merchant_invoice_id}}",
    "cash_collection_amount": "{{cash_collection_amount}}",
    "parcel_weight": {{parcel_weight}},
    "instruction": "{{instruction}}",
    "value": {{value}},
    "is_closed_box": "{{is_closed_box}}",
    "pickup_store_id": {{pickup_store_id}},
    "parcel_details_json": [
      {
        "name": "{{name}}",
        "category": "{{category}}",
        "value": {{value}}
      },
      {
        "name": "{{name}}",
        "category": "{{category}}",
        "value": {{value}}
      }
    ]
  }'
```

**Sample Response:**
```json
{
  "tracking_id": "20A312THJDJ8"
}
```

**Response Fields:**

| Field Name | Field Type | Optional | Description |
|------------|------------|----------|-------------|
| tracking_id | string | No | Unique tracking ID of the parcel |

---

### 4. Update Parcel

Update an existing parcel's information.

**Method:** `PATCH`

**Endpoint:** `/parcels`

**Parameters:**

| Field Name | Field Type | Required | Description |
|------------|------------|----------|-------------|
| entity_type | string | Yes | Type of entity being updated, e.g., 'parcel-tracking-id' |
| entity_id | string | Yes | Unique identifier of the parcel to be updated |
| update_details | object | Yes | Object containing details of the update to be applied |

**Update Details Object:**

| Field Name | Field Type | Required | Description |
|------------|------------|----------|-------------|
| property_name | string | Yes | Name of the parcel property to be updated (e.g., status, delivery_address) |
| new_value | string | Yes | New value to be assigned to the specified property |
| reason | string | No | Optional reason for the update, useful for logging changes |

**Sample Request:**
```bash
curl --location --request PATCH '{{base_url}}/v1.0.0-beta/parcels' \
  --header 'API-ACCESS-TOKEN: Bearer {{jwt_token}}' \
  --header 'Content-Type: application/json' \
  --data '{
    "entity_type": "parcel-tracking-id",
    "entity_id": "{{tracking_id}}",
    "update_details": {
      "property_name": "status",
      "new_value": "cancelled",
      "reason": "Your Cancellation Reason"
    }
  }'
```

**Sample Response:**
```json
{
  "success": true,
  "message": "Request Accepted"
}
```

**Response Fields:**

| Field Name | Field Type | Optional | Description |
|------------|------------|----------|-------------|
| success | boolean | No | Indicates whether the request was successful |
| message | string | No | Descriptive message about the request status |

---

### 5. Get Areas

Retrieve all available delivery areas.

**Method:** `GET`

**Endpoint:** `/areas`

**Sample Request:**
```bash
curl --location '{{base_url}}/v1.0.0-beta/areas' \
  --header 'API-ACCESS-TOKEN: Bearer {{jwt_token}}'
```

**Sample Response:**
```json
{
  "areas": [
    {
      "id": 1,
      "name": "Mohammadpur(Dhaka)",
      "post_code": 1207,
      "division_name": "Dhaka",
      "zone_id": 1
    },
    {
      "id": 2,
      "name": "Dhanmondi",
      "post_code": 1209,
      "division_name": "Dhaka",
      "zone_id": 1
    },
    {
      "id": 3,
      "name": "Gulshan",
      "post_code": 1212,
      "division_name": "Dhaka",
      "zone_id": 1
    }
  ]
}
```

**Response Fields:**

| Field Name | Field Type | Optional | Description |
|------------|------------|----------|-------------|
| areas | array | No | List of areas available for delivery |
| areas[].id | number | No | Unique identifier of the area |
| areas[].name | string | No | Name of the area |
| areas[].post_code | number | No | Postal code of the area |
| areas[].division_name | string | No | Name of the division where the area is located |
| areas[].zone_id | number | No | Zone identifier for the area |

---

### 6. Get Areas by Postal Code

Retrieve delivery areas for a specific postal code.

**Method:** `GET`

**Endpoint:** `/areas?post_code=<:postal_code>`

**Parameters:**

| Field Name | Field Type | Required | Description |
|------------|------------|----------|-------------|
| post_code | integer | Yes | The postal code for which areas need to be retrieved |

**Sample Request:**
```bash
curl --location '{{base_url}}/v1.0.0-beta/areas?post_code={{post_code}}' \
  --header 'API-ACCESS-TOKEN: Bearer {{jwt_token}}'
```

**Sample Response:**
```json
{
  "areas": [
    {
      "id": 13,
      "name": "Kochukhet",
      "post_code": 1206,
      "division_name": "Dhaka",
      "zone_id": 1
    },
    {
      "id": 20,
      "name": "Ibrahimpur",
      "post_code": 1206,
      "division_name": "Dhaka",
      "zone_id": 1
    }
  ]
}
```

---

### 7. Get Areas by District Name

Retrieve delivery areas for a specific district.

**Method:** `GET`

**Endpoint:** `/areas?district_name=<:district_name>`

**Parameters:**

| Field Name | Field Type | Required | Description |
|------------|------------|----------|-------------|
| district_name | string | Yes | The name of the district for which areas need to be retrieved |

**Sample Request:**
```bash
curl --location '{{base_url}}/v1.0.0-beta/areas?district_name={{district_name}}' \
  --header 'API-ACCESS-TOKEN: Bearer {{jwt_token}}'
```

**Sample Response:**
```json
{
  "areas": [
    {
      "id": 13,
      "name": "Kochukhet",
      "post_code": 1206,
      "division_name": "Dhaka",
      "zone_id": 1
    },
    {
      "id": 20,
      "name": "Ibrahimpur",
      "post_code": 1206,
      "division_name": "Dhaka",
      "zone_id": 1
    },
    {
      "id": 23,
      "name": "Kafrul",
      "post_code": 1206,
      "division_name": "Dhaka",
      "zone_id": 1
    }
  ]
}
```

---

### 8. Create Pickup Store

Create a new pickup store location.

**Method:** `POST`

**Endpoint:** `/pickup/store`

**Parameters:**

| Field Name | Field Type | Required | Description |
|------------|------------|----------|-------------|
| name | string | Yes | The name of the pickup store |
| phone | string | Yes | The contact phone number for the pickup store |
| address | string | Yes | The physical address of the pickup store |
| area_id | integer | Yes | The unique identifier of the area where the pickup store is located |

**Sample Request:**
```bash
curl --location '{{base_url}}/v1.0.0-beta/pickup/store' \
  --header 'API-ACCESS-TOKEN: Bearer {{jwt_token}}' \
  --data '{
    "name": "{{name}}",
    "phone": "{{phone}}",
    "address": "{{address}}",
    "area_id": {{area_id}}
  }'
```

**Sample Response:**
```json
{
  "id": 1,
  "name": "Test Pickup Store",
  "address": "Test Address",
  "area_name": "Mohammadpur(Dhaka)",
  "area_id": 1,
  "phone": "8801898000999"
}
```

**Response Fields:**

| Field Name | Field Type | Optional | Description |
|------------|------------|----------|-------------|
| id | number | No | Unique identifier of the pickup store |
| name | string | No | Name of the pickup store |
| address | string | No | Address of the pickup store |
| area_name | string | No | Name of the area where the pickup store is located |
| area_id | number | No | Unique identifier of the area |
| phone | string | No | Contact phone number of the pickup store |

---

### 9. Get Pickup Stores

Retrieve all pickup stores associated with the merchant account.

**Method:** `GET`

**Endpoint:** `/pickup/stores`

**Sample Request:**
```bash
curl --location '{{base_url}}/v1.0.0-beta/pickup/stores' \
  --header 'API-ACCESS-TOKEN: Bearer {{jwt_token}}' \
  --data ''
```

**Sample Response:**
```json
{
  "pickup_stores": [
    {
      "id": {{pickup_store_id}},
      "name": "{{name}}",
      "address": "{{address}}",
      "area_name": "{{area_name}}",
      "area_id": {{area_id}},
      "phone": "{{phone}}",
      "created_at": "{{created_at}}"
    },
    {
      "id": {{pickup_store_id}},
      "name": "{{name}}",
      "address": "{{address}}",
      "area_name": "{{area_name}}",
      "area_id": {{area_id}},
      "phone": "{{phone}}",
      "created_at": "{{created_at}}"
    }
  ]
}
```

**Response Fields:**

| Field Name | Field Type | Optional | Description |
|------------|------------|----------|-------------|
| pickup_stores | array | No | List of available pickup stores |
| pickup_stores[].id | number | No | Unique identifier of the pickup store |
| pickup_stores[].name | string | No | Name of the pickup store |
| pickup_stores[].address | string | No | Address of the pickup store |
| pickup_stores[].area_name | string | No | Name of the area where the pickup store is located |
| pickup_stores[].area_id | number | No | Unique identifier of the area |
| pickup_stores[].phone | string | No | Contact phone number of the pickup store |
| pickup_stores[].created_at | string | No | Timestamp indicating when the pickup store was created |

---

### 10. Get Pickup Store Details

Retrieve detailed information about a specific pickup store.

**Method:** `GET`

**Endpoint:** `/pickup/store/info/<:pickup_store_id>`

**Parameters:**

| Field Name | Field Type | Required | Description |
|------------|------------|----------|-------------|
| pickup_store_id | integer | Yes | The unique identifier of the pickup store whose details need to be retrieved |

**Sample Request:**
```bash
curl --location '{{base_url}}/v1.0.0-beta/pickup/store/info/{{pickup_store_id}}' \
  --header 'API-ACCESS-TOKEN: Bearer {{jwt_token}}' \
  --data ''
```

**Sample Response:**
```json
{
  "pickup_store": {
    "id": 1,
    "name": "Test Pickup Store",
    "address": "Test Address",
    "area_name": "Mohammadpur(Dhaka)",
    "area_id": 1,
    "phone": "8801898000999",
    "created_at": "2021-09-13T10:39:15.000Z"
  }
}
```

**Response Fields:**

| Field Name | Field Type | Optional | Description |
|------------|------------|----------|-------------|
| pickup_store | object | No | Details of the pickup store |
| pickup_store.id | number | No | Unique identifier of the pickup store |
| pickup_store.name | string | No | Name of the pickup store |
| pickup_store.address | string | No | Address of the pickup store |
| pickup_store.area_name | string | No | Name of the area where the pickup store is located |
| pickup_store.area_id | number | No | Unique identifier of the area |
| pickup_store.phone | string | No | Contact phone number of the pickup store |
| pickup_store.created_at | string | No | Timestamp indicating when the pickup store was created |

---

### 11. Calculate Parcel Charge

Calculate delivery and COD charges for a parcel before creating it.

**Method:** `GET`

**Endpoint:** `/charge/charge_calculator`

**Parameters:**

| Field Name | Field Type | Required | Description |
|------------|------------|----------|-------------|
| delivery_area_id | number | Yes | The unique identifier of the area where the parcel will be delivered |
| pickup_area_id | number | Yes | The unique identifier of the area where the parcel will be picked up |
| cash_collection_amount | number | Yes | The total cash amount to be collected upon delivery |
| weight | number | Yes | The weight of the parcel in grams |

**Sample Request:**
```bash
curl --location '{{base_url}}/v1.0.0-beta/charge/charge_calculator?delivery_area_id={{delivery_area_id}}&pickup_area_id={{pickup_area_id}}&cash_collection_amount={{cash_collection_amount}}&weight={{weight}}' \
  --header 'API-ACCESS-TOKEN: Bearer {{jwt_token}}'
```

**Sample Response:**
```json
{
  "deliveryCharge": 60,
  "codCharge": 0
}
```

**Response Fields:**

| Field Name | Field Type | Optional | Description |
|------------|------------|----------|-------------|
| deliveryCharge | number | No | The cost of delivery for the parcel |
| codCharge | number | No | The cash-on-delivery (COD) charge |

---

## Webhook Integration

Configure your webhook URL to receive real-time updates about parcel status changes directly from RedX.

### Webhook Configuration

**Method:** `POST`

**Content-Type:** `application/json`

**Callback URL Structure:**
```
https://example.com/callback?token=<token>
```

> Any required credentials should be included in the query parameters of the URL

### Webhook Payload

When a parcel status changes, RedX will send a POST request to your configured Callback URL with the following payload:

```json
{
  "tracking_number": "<REDX_TRACKING_ID>",
  "timestamp": "<TIMESTAMP>",
  "status": "<STATUS>",
  "message_en": "<MESSAGE_EN>",
  "message_bn": "<MESSAGE_BN>",
  "invoice_number": "<INVOICE_NUMBER>"
}
```

### Status Values

| Status | Meaning |
|--------|---------|
| `ready-for-delivery` | Parcel received from merchants |
| `delivery-in-progress` | Parcels have been dispatched to rider |
| `delivered` | Parcels delivered by rider |
| `agent-hold` | Parcels are on hold to agent |
| `agent-returning` | Parcel return-in-progress |
| `returned` | Parcels returned |
| `agent-area-change` | Area change requested & in progress |

> Merchants should handle these status updates accordingly to keep their systems synchronized with the latest parcel status.

---

## Platform Integrations

### WordPress Plugin

Simplify your parcel creation process with the dedicated WordPress plugin. This plugin allows you to seamlessly create parcels directly from your WordPress website using the OpenAPI.

### Shopify App

Simplify your order fulfillment process with the official Shopify app. The RedX Delivery app connects your Shopify store directly to Bangladesh's leading delivery service. It automatically collects customer phone numbers during checkout and supports batch processing.

---

## Integration Notes for Package Development

### Key Considerations for a Socialite-like Package:

1. **Environment Management**: Support both sandbox and production environments with easy switching
2. **Authentication**: Handle Bearer token authentication seamlessly
3. **Error Handling**: Implement robust error handling for API failures
4. **Webhook Support**: Provide webhook verification and event handling mechanisms
5. **Rate Limiting**: Respect API rate limits if any
6. **Data Validation**: Validate request data before sending to API
7. **Response Normalization**: Normalize API responses for consistent interface
8. **Logging**: Implement comprehensive logging for debugging
9. **Testing**: Provide mock responses for testing without hitting the API
10. **Type Safety**: Use TypeScript/Type definitions for better developer experience

### Suggested Package Structure:

```
- Authentication Manager (Token handling)
- Parcel Manager (Create, update, track parcels)
- Area Manager (Fetch and manage delivery areas)
- Pickup Store Manager (Manage pickup locations)
- Charge Calculator (Calculate delivery costs)
- Webhook Handler (Process webhook events)
- Response Builder (Normalize API responses)
- Exception Handler (Custom exceptions and errors)
```

---

## Quick Reference - All Endpoints

### Base URLs
- **Sandbox:** `https://sandbox.redx.com.bd/v1.0.0-beta`
- **Production:** `https://openapi.redx.com.bd/v1.0.0-beta`

### All API Endpoints

| # | Method | Endpoint | Description |
|---|--------|----------|-------------|
| 1 | `GET` | `/parcel/track/<:parcel_id>` | Track parcel status |
| 2 | `GET` | `/parcel/info/<:tracking_id>` | Get parcel details |
| 3 | `POST` | `/parcel` | Create new parcel |
| 4 | `PATCH` | `/parcels` | Update parcel |
| 5 | `GET` | `/areas` | Get all areas |
| 6 | `GET` | `/areas?post_code=<:postal_code>` | Get areas by postal code |
| 7 | `GET` | `/areas?district_name=<:district_name>` | Get areas by district name |
| 8 | `POST` | `/pickup/store` | Create pickup store |
| 9 | `GET` | `/pickup/stores` | Get all pickup stores |
| 10 | `GET` | `/pickup/store/info/<:pickup_store_id>` | Get pickup store details |
| 11 | `GET` | `/charge/charge_calculator` | Calculate parcel charge |

### Endpoint Categories

**Parcel Operations:**
```
GET    /parcel/track/<:parcel_id>
GET    /parcel/info/<:tracking_id>
POST   /parcel
PATCH  /parcels
```

**Area Management:**
```
GET    /areas
GET    /areas?post_code=<:postal_code>
GET    /areas?district_name=<:district_name>
```

**Pickup Store Management:**
```
POST   /pickup/store
GET    /pickup/stores
GET    /pickup/store/info/<:pickup_store_id>
```

**Pricing:**
```
GET    /charge/charge_calculator
```

### Authentication Header
```http
API-ACCESS-TOKEN: Bearer {{jwt_token}}
```

---

*Last Updated: 2026-03-29*
*API Version: v1.0.0-beta*
