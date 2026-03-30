# Paperfly API Documentation

> Complete API reference for building a Socialite-like courier package integration with Paperfly

## Overview

Paperfly is Bangladesh's largest e-commerce logistics network, providing comprehensive delivery solutions for merchants. The API offers simple REST-based endpoints for creating orders, tracking shipments, and managing cancellations using Basic Authentication.

## Base URL

**Base URL:** `https://api.paperfly.com.bd`

> Paperfly uses a single production environment. Contact Paperfly to obtain your merchant panel credentials and API key.

## Authentication

Paperfly uses Basic Authentication combined with a special API key header.

### Authentication Method

**Type:** Basic Auth

| Field | Description |
|-------|-------------|
| Username | Merchant Panel User Name |
| Password | Merchant Panel Password |

### Required Headers

```http
paperflykey: Paperfly_~La?Rj73FcLm
Content-Type: application/json
Authorization: Basic {{base64_credentials}}
```

### Authentication Example
```bash
--header 'paperflykey: Paperfly_~La?Rj73FcLm' \
--header 'Content-Type: application/json' \
--header 'Authorization: Basic {{base64(username:password)}}'
```

---

## API Endpoints

### 1. Create Order

Create a new delivery order.

**Method:** `POST`

**Endpoint:** `/merchant/api/service/new_order_v2.php`

**Parameters:**

| Field Name | Type | Required | Description |
|------------|------|----------|-------------|
| merchantOrderReference | string | Yes | Unique merchant order reference ID |
| storeName | string | Yes | Name of the store |
| productBrief | string | Yes | Brief description of the product |
| packagePrice | string | Yes | Package price |
| max_weight | string | Yes | Maximum weight of package |
| customerName | string | Yes | Customer name |
| customerAddress | string | Yes | Customer address |
| customerPhone | string | Yes | Customer phone number |

**Sample Request:**
```bash
curl --location 'https://api.paperfly.com.bd/merchant/api/service/new_order_v2.php' \
--header 'paperflykey: Paperfly_~La?Rj73FcLm' \
--header 'Content-Type: application/json' \
--header 'Authorization: Basic {{base64_credentials}}' \
--data '{
  "merchantOrderReference": "Test_01610",
  "storeName": "Ovi",
  "productBrief": "Test Product",
  "packagePrice": "10",
  "max_weight": "0.3",
  "customerName": "Liton Ovi",
  "customerAddress": "Banani, Dhaka",
  "customerPhone": "01610202717"
}'
```

**Sample Response:**
```json
{
  "success": {
    "message": "successfully inserted",
    "tracking_number": "Z-051125-63821-A3-A1",
    "tracking_barcode": "751820115459"
  },
  "response_code": 200
}
```

**Response Fields:**

| Field Name | Type | Description |
|------------|------|-------------|
| success.message | string | Response message |
| success.tracking_number | string | Unique tracking number for the order |
| success.tracking_barcode | string | Barcode number for tracking |
| response_code | integer | HTTP response code |

---

### 2. Create Exchange Order

Create an exchange order (special order type for product exchanges).

**Method:** `POST`

**Endpoint:** `/merchant/api/service/new_order_v2.php`

**Parameters:**

| Field Name | Type | Required | Description |
|------------|------|----------|-------------|
| merchantOrderReference | string | Yes | Unique merchant order reference ID |
| storeName | string | Yes | Name of the store |
| productBrief | string | Yes | Brief description of the product |
| packagePrice | string | Yes | Package price |
| max_weight | string | Yes | Maximum weight of package |
| customerName | string | Yes | Customer name |
| customerAddress | string | Yes | Customer address |
| customerPhone | string | Yes | Customer phone number |
| orderType | string | Yes | Must be "Exchange" |
| exchangeDescription | string | Yes | Description of exchange product |
| exchangePrice | string | Yes | Exchange amount |
| exchangeWeight | string | Yes | Package weight for exchange |

**Sample Request:**
```bash
curl --location 'https://api.paperfly.com.bd/merchant/api/service/new_order_v2.php' \
--header 'paperflykey: Paperfly_~La?Rj73FcLm' \
--header 'Content-Type: application/json' \
--header 'Authorization: Basic {{base64_credentials}}' \
--data '{
  "merchantOrderReference": "Test_01615",
  "storeName": "Ovi",
  "productBrief": "Test Product",
  "packagePrice": "10",
  "max_weight": "0.3",
  "customerName": "Liton Ovi",
  "customerAddress": "Banani, Dhaka",
  "customerPhone": "01610202717",
  "orderType": "Exchange",
  "exchangeDescription": "exchange product",
  "exchangePrice": "100",
  "exchangeWeight": "1.5"
}'
```

**Sample Response:**
```json
{
  "success": {
    "message": "successfully inserted",
    "tracking_number": "Z-051125-63822-A3-A1",
    "tracking_barcode": "751820115460"
  },
  "response_code": 200
}
```

---

### 3. Track Order

Get tracking information for an order using merchant order reference.

**Method:** `POST`

**Endpoint:** `/API-Order-Tracking`

**Parameters:**

| Field Name | Type | Required | Description |
|------------|------|----------|-------------|
| ReferenceNumber | string | Yes | Merchant order reference ID |

**Sample Request:**
```bash
curl --location 'https://api.paperfly.com.bd/API-Order-Tracking' \
--header 'paperflykey: Paperfly_~La?Rj73FcLm' \
--header 'Content-Type: application/json' \
--header 'Authorization: Basic {{base64_credentials}}' \
--data '{
  "ReferenceNumber": "Test_01610"
}'
```

**Sample Response:**
```json
{
  "success": {
    "message": "success",
    "trackingStatus": [
      {
        "invNum": "",
        "receivedAmount": "",
        "Pick": null,
        "PickTime": null,
        "inTransit": "",
        "inTransitTime": "",
        "ReceivedAtPoint": "",
        "ReceivedAtPointTime": "",
        "PickedForDelivery": "",
        "PickedForDeliveryTime": "",
        "Delivered": "",
        "DeliveredTime": "",
        "Returned": "",
        "ReturnedTime": "",
        "Partial": "",
        "PartialTime": ""
      }
    ]
  }
}
```

**Tracking Status Fields:**

| Field Name | Type | Description |
|------------|------|-------------|
| invNum | string | Invoice number |
| receivedAmount | string | Amount received (for COD orders) |
| Pick | string/null | Pick status |
| PickTime | string/null | Pick timestamp |
| inTransit | string | In transit status |
| inTransitTime | string | In transit timestamp |
| ReceivedAtPoint | string | Received at delivery point status |
| ReceivedAtPointTime | string | Received at point timestamp |
| PickedForDelivery | string | Picked for delivery status |
| PickedForDeliveryTime | string | Picked for delivery timestamp |
| Delivered | string | Delivered status |
| DeliveredTime | string | Delivered timestamp |
| Returned | string | Returned status |
| ReturnedTime | string | Returned timestamp |
| Partial | string | Partial delivery status |
| PartialTime | string | Partial delivery timestamp |

---

### 4. Cancel Order

Cancel an existing order using merchant order reference.

**Method:** `POST`

**Endpoint:** `/api/v1/cancel-order`

**Parameters:**

| Field Name | Type | Required | Description |
|------------|------|----------|-------------|
| order_id | string | Yes | Merchant order reference ID |

**Sample Request:**
```bash
curl --location 'https://api.paperfly.com.bd/api/v1/cancel-order' \
--header 'paperflykey: Paperfly_~La?Rj73FcLm' \
--header 'Content-Type: application/json' \
--header 'Authorization: Basic {{base64_credentials}}' \
--data '{
  "order_id": "Test_01610"
}'
```

**Sample Response:**
```json
{
  "success": {
    "message": "Order cancelled successfully"
  },
  "response_code": 200
}
```

---

## Quick Reference - All Endpoints

### Base URL
- **Production:** `https://api.paperfly.com.bd`

### All API Endpoints

| # | Method | Endpoint | Description |
|---|--------|----------|-------------|
| 1 | `POST` | `/merchant/api/service/new_order_v2.php` | Create new order |
| 2 | `POST` | `/merchant/api/service/new_order_v2.php` | Create exchange order |
| 3 | `POST` | `/API-Order-Tracking` | Track order by reference |
| 4 | `POST` | `/api/v1/cancel-order` | Cancel order |

### Endpoint Categories

**Order Management:**
```
POST   /merchant/api/service/new_order_v2.php (Regular)
POST   /merchant/api/service/new_order_v2.php (Exchange)
POST   /api/v1/cancel-order
```

**Tracking:**
```
POST   /API-Order-Tracking
```

### Authentication Headers
```http
paperflykey: Paperfly_~La?Rj73FcLm
Content-Type: application/json
Authorization: Basic {{base64(username:password)}}
```

### Order Types

| Type | Description |
|------|-------------|
| Regular | Standard delivery order |
| Exchange | Product exchange order |

### Tracking Status Fields

| Status | Description |
|--------|-------------|
| Pick | Package picked up from merchant |
| inTransit | Package in transit to delivery point |
| ReceivedAtPoint | Package received at delivery point |
| PickedForDelivery | Package picked up for final delivery |
| Delivered | Package delivered successfully |
| Returned | Package returned to merchant |
| Partial | Partially delivered |

---

## Integration Notes for Package Development

### Key Differences from Other Couriers:

1. **Basic Authentication:** Uses standard HTTP Basic Auth with username/password
2. **Special API Key Header:** Requires `paperflykey` header with fixed value
3. **Order Type Support:** Supports regular and exchange orders
4. **Simple Tracking:** Track by merchant reference number only
5. **No Location Hierarchy:** No city/zone/area structure needed
6. **Minimal Endpoints:** Only 3 main operations (create, track, cancel)
7. **Detailed Tracking:** Comprehensive tracking status with timestamps
8. **No Bulk Orders:** No bulk order creation endpoint
9. **No Webhooks:** No webhook system (polling required for updates)
10. **Fixed API Key:** Paperfly key appears to be constant (not merchant-specific)

### Key Considerations for a Socialite-like Package:

1. **Authentication Management:**
   - Store merchant panel username and password securely
   - Encode credentials to Base64 for Basic Auth
   - Include paperflykey header in all requests

2. **Order Creation:**
   - Support both regular and exchange orders
   - Generate unique merchantOrderReference IDs
   - Handle exchange-specific fields when orderType is "Exchange"

3. **Tracking Implementation:**
   - Implement polling mechanism for status updates
   - Parse all tracking status fields
   - Handle null/empty timestamps gracefully
   - Map tracking statuses to application states

4. **Error Handling:**
   - Validate all required fields
   - Handle duplicate order reference errors
   - Parse error messages from API responses

5. **Data Validation:**
   - Ensure merchantOrderReference is unique
   - Validate phone number format
   - Validate weight and price values

6. **Response Normalization:**
   - Normalize API responses for consistent interface
   - Map Paperfly-specific fields to standard format
   - Handle different response formats

7. **Order Type Handling:**
   - Detect when to use exchange order type
   - Validate exchange-specific fields
   - Handle pricing differences for exchanges

8. **Polling Strategy:**
   - Implement periodic status checking
   - Cache tracking results to reduce API calls
   - Handle rate limiting if present

### Suggested Package Structure:

```
- Authentication Manager (Basic Auth + paperflykey)
- Order Manager (Create regular/exchange orders, cancel)
- Tracking Manager (Track by reference, parse status)
- Response Builder (Normalize API responses)
- Exception Handler (Custom exceptions and errors)
- Validator (Input validation)
- Polling Manager (Status update polling)
```

### Basic Authentication Encoding:

```php
// Encode credentials for Basic Auth
$credentials = base64_encode($username . ':' . $password);

// Use in request
$headers = [
    'paperflykey: Paperfly_~La?Rj73FcLm',
    'Content-Type: application/json',
    'Authorization: Basic ' . $credentials
];
```

### Order Creation Pattern:

```php
// Regular Order
$orderData = [
    'merchantOrderReference' => generateUniqueReference(),
    'storeName' => $storeName,
    'productBrief' => $productDescription,
    'packagePrice' => $price,
    'max_weight' => $weight,
    'customerName' => $customerName,
    'customerAddress' => $customerAddress,
    'customerPhone' => $customerPhone
];

// Exchange Order (additional fields)
if ($isExchange) {
    $orderData['orderType'] = 'Exchange';
    $orderData['exchangeDescription'] = $exchangeDesc;
    $orderData['exchangePrice'] = $exchangePrice;
    $orderData['exchangeWeight'] = $exchangeWeight;
}
```

### Tracking Status Parsing:

```php
function parseTrackingStatus($trackingData) {
    $status = [
        'current_status' => null,
        'timeline' => []
    ];

    // Check each status field
    $statuses = [
        'Pick' => 'picked_up',
        'inTransit' => 'in_transit',
        'ReceivedAtPoint' => 'at_delivery_point',
        'PickedForDelivery' => 'out_for_delivery',
        'Delivered' => 'delivered',
        'Returned' => 'returned',
        'Partial' => 'partial_delivered'
    ];

    foreach ($statuses as $field => $mappedStatus) {
        if (!empty($trackingData[$field])) {
            $status['timeline'][] = [
                'status' => $mappedStatus,
                'timestamp' => $trackingData[$field . 'Time'] ?? null
            ];
        }
    }

    return $status;
}
```

---

## Comparison with Other Couriers

| Feature | Paperfly | Steadfast | RedX | Carrybee | Pathao |
|---------|----------|-----------|------|----------|--------|
| Authentication | Basic Auth + API Key | API Key + Secret Key | Bearer Token | Client ID/Secret/Context | OAuth 2.0 |
| Token Refresh | Not needed | No | No | No | Yes (5 days) |
| Order Types | Regular + Exchange | Single type | Single type | Single type | Single type |
| Bulk Orders | No | Yes (max 500) | No | Yes | Yes |
| Location Structure | None | None | Areas | City → Zone → Area | City → Zone → Area |
| Webhooks | No | No | Yes | Yes (24 events) | No |
| Tracking | By Reference | 3 methods | By tracking_id | By consignment_id | By consignment_id |
| Price Calculation | No | No | Yes | No | Yes |
| Balance Check | No | Yes | No | No | No |
| Special Features | Exchange orders | Returns management | Pickup stores | Address parsing | Token refresh |

---

## Paperfly-Specific Features

### Exchange Orders

Paperfly supports a dedicated exchange order type for handling product returns and exchanges:

**When to Use Exchange Orders:**
- Customer returns a product for exchange
- Size/color variations of the same product
- Replacement orders for defective items

**Exchange-Specific Fields:**
- `orderType`: Must be "Exchange"
- `exchangeDescription`: Description of the exchange product
- `exchangePrice`: Price of the exchange item
- `exchangeWeight`: Weight of the exchange package

### Tracking Timeline

Paperfly provides detailed tracking with timestamps for each stage:

1. **Pick** - Package picked up from merchant location
2. **In Transit** - Package moving to delivery point
3. **Received at Point** - Package reached local delivery point
4. **Picked for Delivery** - Out for final delivery
5. **Delivered** - Successfully delivered
6. **Returned** - Returned to merchant
7. **Partial** - Partially delivered (some items)

---

*Last Updated: 2026-03-29*
*API Version: v2*
