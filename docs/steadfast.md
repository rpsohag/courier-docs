# Steadfast Courier API Documentation

> Complete API reference for building a Socialite-like courier package integration with Steadfast Courier Limited

## Overview

Steadfast Courier API provides a simple REST API for creating orders, tracking shipments, managing returns, and checking payments. The API uses API Key and Secret Key authentication for all requests.

## Base URL

**Base URL:** `https://portal.packzy.com/api/v1`

> Steadfast uses a single production environment. Contact Steadfast to obtain your API credentials.

## Authentication

All API requests require authentication using two headers:

```http
Api-Key: {{api_key}}
Secret-Key: {{secret_key}}
Content-Type: application/json
```

### Authentication Headers

| Header Name | Type | Description |
|-------------|------|-------------|
| Api-Key | String | API Key provided by Steadfast Courier Ltd. |
| Secret-Key | String | Secret Key provided by Steadfast Courier Ltd. |
| Content-Type | String | Request Content Type (must be application/json) |

### Authentication Example
```bash
--header 'Api-Key: {{api_key}}' \
--header 'Secret-Key: {{secret_key}}' \
--header 'Content-Type: application/json'
```

---

## API Endpoints

### 1. Placing an Order (Create Order)

Create a new delivery order.

**Method:** `POST`

**Endpoint:** `/create_order`

**Parameters:**

| Field Name | Type | Required | Description |
|------------|------|----------|-------------|
| invoice | string | Yes | Unique invoice ID (alpha-numeric, hyphens, underscores) |
| recipient_name | string | Yes | Recipient name (max 100 characters) |
| recipient_phone | string | Yes | Recipient phone number (11 digits) |
| recipient_address | string | Yes | Recipient address (max 250 characters) |
| alternative_phone | string | No | Alternative phone number (11 digits) |
| recipient_email | string | No | Recipient email address |
| cod_amount | numeric | Yes | Cash on delivery amount in BDT (cannot be less than 0) |
| note | string | No | Delivery instructions or notes |
| item_description | string | No | Item name and description |
| total_lot | numeric | No | Total lot of items |
| delivery_type | numeric | No | 0 for home delivery, 1 for Point Delivery/Hub Pick Up |

**Sample Request:**
```bash
curl --location 'https://portal.packzy.com/api/v1/create_order' \
--header 'Api-Key: {{api_key}}' \
--header 'Secret-Key: {{secret_key}}' \
--header 'Content-Type: application/json' \
--data '{
    "invoice": "12366",
    "recipient_name": "John Smith",
    "recipient_phone": "01234567890",
    "recipient_address": "Fla# A1, House# 17/1, Road# 3/A, Dhanmondi, Dhaka-1209",
    "cod_amount": 1060,
    "note": "Deliver within 3PM",
    "item_description": "Clothing items",
    "total_lot": 2,
    "delivery_type": 0
}'
```

**Sample Response:**
```json
{
  "status": 200,
  "message": "Consignment has been created successfully.",
  "consignment": {
    "consignment_id": 1424107,
    "invoice": "Aa12-das4",
    "tracking_code": "15BAEB8A",
    "recipient_name": "John Smith",
    "recipient_phone": "01234567890",
    "recipient_address": "Fla# A1,House# 17/1, Road# 3/A, Dhanmondi,Dhaka-1209",
    "cod_amount": 1060,
    "status": "in_review",
    "note": "Deliver within 3PM",
    "created_at": "2021-03-21T07:05:31.000000Z",
    "updated_at": "2021-03-21T07:05:31.000000Z"
  }
}
```

**Response Fields:**

| Field Name | Type | Description |
|------------|------|-------------|
| status | integer | HTTP status code |
| message | string | Response message |
| consignment.consignment_id | integer | Unique consignment ID |
| consignment.invoice | string | Invoice number |
| consignment.tracking_code | string | Tracking code for the shipment |
| consignment.recipient_name | string | Recipient name |
| consignment.recipient_phone | string | Recipient phone |
| consignment.recipient_address | string | Recipient address |
| consignment.cod_amount | numeric | COD amount |
| consignment.status | string | Current status |
| consignment.created_at | string | Creation timestamp |
| consignment.updated_at | string | Last update timestamp |

---

### 2. Bulk Order Create

Create multiple orders in a single request (maximum 500 orders).

**Method:** `POST`

**Endpoint:** `/create_order/bulk-order`

**Parameters:**

| Field Name | Type | Required | Description |
|------------|------|----------|-------------|
| data | JSON | Yes | JSON encoded array of order objects (max 500 items) |

**Order Object Fields:**
Same as single order creation (invoice, recipient_name, recipient_phone, recipient_address, cod_amount, note, etc.)

**Sample Request (PHP Example):**
```php
$orders = Order::with('address')->where('status', 1)->take(500)->get();

$data = array();

foreach($orders as $order){
    $item = [
        'invoice' => $order->id,
        'recipient_name' => $order->address ? $order->address->name : 'N/A',
        'recipient_address' => $order->address ? $order->address->address : 'N/A',
        'recipient_phone' => $order->address ? $order->address->phone : '',
        'cod_amount' => $order->due_amount,
        'note' => $order->note,
    ];
    $data[] = $item;
}

$response = Http::withHeaders([
    'Api-Key' => $api_key,
    'Secret-Key' => $secret_key,
    'Content-Type' => 'application/json'
])->post('https://portal.packzy.com/api/v1/create_order/bulk-order', [
    'data' => json_encode($data),
]);
```

**Sample Response:**
```json
[
  {
    "invoice": "230822-1",
    "recipient_name": "John Doe",
    "recipient_address": "House 44, Road 2/A, Dhanmondi, Dhaka 1209",
    "recipient_phone": "0171111111",
    "cod_amount": "0.00",
    "note": null,
    "consignment_id": 11543968,
    "tracking_code": "B025A038",
    "status": "success"
  },
  {
    "invoice": "230822-2",
    "recipient_name": "Jane Doe",
    "recipient_address": "House 45, Road 2/B, Dhanmondi, Dhaka 1209",
    "recipient_phone": "0172222222",
    "cod_amount": "500.00",
    "note": "Handle with care",
    "consignment_id": 11543969,
    "tracking_code": "B025A1DC",
    "status": "success"
  }
]
```

**Error Response:**
```json
[
  {
    "invoice": "230822-1",
    "recipient_name": "John Doe",
    "recipient_address": "House 44, Road 2/A, Dhanmondi, Dhaka 1209",
    "recipient_phone": "0171111111",
    "cod_amount": "0.00",
    "note": null,
    "consignment_id": null,
    "tracking_code": null,
    "status": "error"
  }
]
```

---

### 3. Check Delivery Status

Get the current status of a consignment using different identifiers.

#### i) By Consignment ID
**Method:** `GET`

**Endpoint:** `/status_by_cid/{id}`

**Parameters:**

| Field Name | Type | Required | Description |
|------------|------|----------|-------------|
| id | integer | Yes | Consignment ID |

**Sample Request:**
```bash
curl --location 'https://portal.packzy.com/api/v1/status_by_cid/1424107' \
--header 'Api-Key: {{api_key}}' \
--header 'Secret-Key: {{secret_key}}'
```

#### ii) By Invoice ID
**Method:** `GET`

**Endpoint:** `/status_by_invoice/{invoice}`

**Parameters:**

| Field Name | Type | Required | Description |
|------------|------|----------|-------------|
| invoice | string | Yes | Your invoice ID |

**Sample Request:**
```bash
curl --location 'https://portal.packzy.com/api/v1/status_by_invoice/12366' \
--header 'Api-Key: {{api_key}}' \
--header 'Secret-Key: {{secret_key}}'
```

#### iii) By Tracking Code
**Method:** `GET`

**Endpoint:** `/status_by_trackingcode/{trackingCode}`

**Parameters:**

| Field Name | Type | Required | Description |
|------------|------|----------|-------------|
| trackingCode | string | Yes | Tracking code |

**Sample Request:**
```bash
curl --location 'https://portal.packzy.com/api/v1/status_by_trackingcode/15BAEB8A' \
--header 'Api-Key: {{api_key}}' \
--header 'Secret-Key: {{secret_key}}'
```

**Sample Response:**
```json
{
  "status": 200,
  "delivery_status": "in_review"
}
```

**Delivery Status Values:**

| Status | Description |
|--------|-------------|
| `pending` | Consignment is not delivered or cancelled yet |
| `delivered_approval_pending` | Consignment is delivered but waiting for admin approval |
| `partial_delivered_approval_pending` | Partially delivered, waiting for admin approval |
| `cancelled_approval_pending` | Cancelled and waiting for admin approval |
| `unknown_approval_pending` | Unknown pending status. Contact support team |
| `delivered` | Delivered and balance added |
| `partial_delivered` | Partially delivered and balance added |
| `cancelled` | Cancelled and balance updated |
| `hold` | Consignment is held |
| `in_review` | Order placed and waiting to be reviewed |
| `unknown` | Unknown status. Contact support team |

---

### 4. Check Current Balance

Get the current account balance.

**Method:** `GET`

**Endpoint:** `/get_balance`

**Sample Request:**
```bash
curl --location 'https://portal.packzy.com/api/v1/get_balance' \
--header 'Api-Key: {{api_key}}' \
--header 'Secret-Key: {{secret_key}}'
```

**Sample Response:**
```json
{
  "status": 200,
  "current_balance": 0
}
```

**Response Fields:**

| Field Name | Type | Description |
|------------|------|-------------|
| status | integer | HTTP status code |
| current_balance | numeric | Current account balance in BDT |

---

### 5. Create Return Request

Create a return request for a consignment.

**Method:** `POST`

**Endpoint:** `/create_return_request`

**Parameters:**

| Field Name | Type | Required | Description |
|------------|------|----------|-------------|
| consignment_id or invoice or tracking_code | numeric/string | Yes | Consignment ID, invoice ID, or tracking code |
| reason | string | No | Reason for return request |

**Sample Request:**
```bash
curl --location 'https://portal.packzy.com/api/v1/create_return_request' \
--header 'Api-Key: {{api_key}}' \
--header 'Secret-Key: {{secret_key}}' \
--header 'Content-Type: application/json' \
--data '{
    "consignment_id": "10000042",
    "reason": "Product damaged during delivery"
}'
```

**Sample Response:**
```json
{
  "id": 1,
  "user_id": 1,
  "consignment_id": 10000042,
  "reason": "Product damaged during delivery",
  "status": "pending",
  "created_at": "2025-07-30T23:11:45.000000Z",
  "updated_at": "2025-07-30T23:11:45.000000Z"
}
```

**Return Request Status Values:**
- `pending` - Request submitted and pending review
- `approved` - Return request approved
- `processing` - Return is being processed
- `completed` - Return process completed
- `cancelled` - Return request cancelled

---

### 6. Get Single Return Request

Get details of a specific return request.

**Method:** `GET`

**Endpoint:** `/get_return_request/{id}`

**Parameters:**

| Field Name | Type | Required | Description |
|------------|------|----------|-------------|
| id | integer | Yes | Return request ID |

**Sample Request:**
```bash
curl --location 'https://portal.packzy.com/api/v1/get_return_request/1' \
--header 'Api-Key: {{api_key}}' \
--header 'Secret-Key: {{secret_key}}'
```

---

### 7. Get Return Requests

Get all return requests for your account.

**Method:** `GET`

**Endpoint:** `/get_return_requests`

**Sample Request:**
```bash
curl --location 'https://portal.packzy.com/api/v1/get_return_requests' \
--header 'Api-Key: {{api_key}}' \
--header 'Secret-Key: {{secret_key}}'
```

---

### 8. Get Payments

Get all payment records.

**Method:** `GET`

**Endpoint:** `/payments`

**Sample Request:**
```bash
curl --location 'https://portal.packzy.com/api/v1/payments' \
--header 'Api-Key: {{api_key}}' \
--header 'Secret-Key: {{secret_key}}'
```

---

### 9. Get Single Payment with Consignments

Get details of a specific payment including associated consignments.

**Method:** `GET`

**Endpoint:** `/payments/{payment_id}`

**Parameters:**

| Field Name | Type | Required | Description |
|------------|------|----------|-------------|
| payment_id | integer | Yes | Payment ID |

**Sample Request:**
```bash
curl --location 'https://portal.packzy.com/api/v1/payments/123' \
--header 'Api-Key: {{api_key}}' \
--header 'Secret-Key: {{secret_key}}'
```

---

### 10. Get Police Stations

Get list of police stations (recently added).

**Method:** `GET`

**Endpoint:** `/police_stations`

**Sample Request:**
```bash
curl --location 'https://portal.packzy.com/api/v1/police_stations' \
--header 'Api-Key: {{api_key}}' \
--header 'Secret-Key: {{secret_key}}'
```

---

## Quick Reference - All Endpoints

### Base URL
- **Production:** `https://portal.packzy.com/api/v1`

### All API Endpoints

| # | Method | Endpoint | Description |
|---|--------|----------|-------------|
| 1 | `POST` | `/create_order` | Create single order |
| 2 | `POST` | `/create_order/bulk-order` | Create bulk orders (max 500) |
| 3 | `GET` | `/status_by_cid/{id}` | Get status by consignment ID |
| 4 | `GET` | `/status_by_invoice/{invoice}` | Get status by invoice ID |
| 5 | `GET` | `/status_by_trackingcode/{trackingCode}` | Get status by tracking code |
| 6 | `GET` | `/get_balance` | Get current balance |
| 7 | `POST` | `/create_return_request` | Create return request |
| 8 | `GET` | `/get_return_request/{id}` | Get single return request |
| 9 | `GET` | `/get_return_requests` | Get all return requests |
| 10 | `GET` | `/payments` | Get all payments |
| 11 | `GET` | `/payments/{payment_id}` | Get single payment with consignments |
| 12 | `GET` | `/police_stations` | Get police stations |

### Endpoint Categories

**Order Management:**
```
POST   /create_order
POST   /create_order/bulk-order
```

**Status Tracking:**
```
GET    /status_by_cid/{id}
GET    /status_by_invoice/{invoice}
GET    /status_by_trackingcode/{trackingCode}
```

**Returns Management:**
```
POST   /create_return_request
GET    /get_return_request/{id}
GET    /get_return_requests
```

**Financial:**
```
GET    /get_balance
GET    /payments
GET    /payments/{payment_id}
```

**Utilities:**
```
GET    /police_stations
```

### Authentication Headers
```http
Api-Key: {{api_key}}
Secret-Key: {{secret_key}}
Content-Type: application/json
```

### Delivery Status Values

| Status | Description |
|--------|-------------|
| `pending` | Not delivered or cancelled yet |
| `delivered_approval_pending` | Delivered, awaiting approval |
| `partial_delivered_approval_pending` | Partially delivered, awaiting approval |
| `cancelled_approval_pending` | Cancelled, awaiting approval |
| `unknown_approval_pending` | Unknown pending status |
| `delivered` | Delivered and balance added |
| `partial_delivered` | Partially delivered and balance added |
| `cancelled` | Cancelled and balance updated |
| `hold` | Consignment held |
| `in_review` | Order placed, awaiting review |
| `unknown` | Unknown status |

### Return Request Status Values

| Status | Description |
|--------|-------------|
| `pending` | Request submitted, pending review |
| `approved` | Return request approved |
| `processing` | Return being processed |
| `completed` | Return process completed |
| `cancelled` | Return request cancelled |

### Delivery Types

| Value | Description |
|-------|-------------|
| `0` | Home delivery |
| `1` | Point Delivery/Steadfast Hub Pick Up |

### Input Validation Rules

| Field | Validation |
|-------|------------|
| invoice | Unique, alpha-numeric (hyphens, underscores allowed) |
| recipient_name | Max 100 characters |
| recipient_phone | 11 digits |
| alternative_phone | 11 digits (optional) |
| recipient_address | Max 250 characters |
| cod_amount | Numeric, cannot be less than 0 |

---

## Integration Notes for Package Development

### Key Differences from Other Couriers:

1. **Simple Authentication:** Uses API Key and Secret Key (no OAuth, no token refresh)
2. **Multiple Tracking Methods:** Three different ways to check status (by ID, invoice, or tracking code)
3. **Bulk Order Limit:** Maximum 500 orders per bulk request
4. **Delivery Types:** Simple binary choice (0 = home delivery, 1 = hub pickup)
5. **No Location Hierarchy:** No city/zone/area structure needed
6. **Status Workflow:** Multiple approval pending states for different actions
7. **Return Management:** Built-in return request system
8. **Payment Tracking:** Detailed payment and consignment tracking
9. **Balance Checking:** Direct API to check current balance
10. **No Webhooks:** No webhook system mentioned (polling required for updates)

### Key Considerations for a Socialite-like Package:

1. **Simple Authentication:**
   - Store API Key and Secret Key securely
   - No token refresh mechanism needed
   - Simple header-based authentication

2. **Invoice Management:**
   - Generate unique invoice IDs (alpha-numeric with hyphens/underscores)
   - Track invoice IDs for order lookup
   - Handle invoice uniqueness validation

3. **Bulk Order Processing:**
   - Limit to 500 orders per batch
   - Process response array for individual order status
   - Handle partial success/failure in bulk operations

4. **Status Tracking:**
   - Support three tracking methods (ID, invoice, tracking code)
   - Implement polling mechanism for status updates
   - Map status codes to application states
   - Handle approval pending states

5. **Return Management:**
   - Create return requests for consignments
   - Track return request status
   - Handle return workflow (pending → approved → processing → completed)

6. **Financial Management:**
   - Check balance before creating orders
   - Track payments and associated consignments
   - Handle COD amounts correctly

7. **Error Handling:**
   - Validate all input data (phone numbers, addresses, etc.)
   - Handle bulk order errors with individual status tracking
   - Parse and display meaningful error messages

8. **Data Validation:**
   - Enforce character limits (name: 100, address: 250)
   - Validate phone numbers (11 digits)
   - Ensure invoice uniqueness

9. **Response Normalization:**
   - Normalize API responses for consistent interface
   - Map Steadfast-specific fields to standard format
   - Handle different response formats

10. **Polling Strategy:**
    - Implement periodic status checking for orders
    - Cache status to reduce API calls
    - Handle rate limiting if present

### Suggested Package Structure:

```
- Authentication Manager (API Key/Secret Key handling)
- Order Manager (Create single/bulk orders)
- Status Manager (Track by ID/invoice/tracking code)
- Return Manager (Create and track return requests)
- Payment Manager (Check balance, get payments)
- Validator (Input validation and sanitization)
- Response Builder (Normalize API responses)
- Exception Handler (Custom exceptions and errors)
- Polling Manager (Status update polling)
```

### Bulk Order Processing Pattern:

```php
// 1. Prepare orders
$orders = prepareOrders(); // Max 500

// 2. Create bulk order
$response = createBulkOrder($orders);

// 3. Process results
foreach ($response as $result) {
    if ($result['status'] === 'success') {
        // Store consignment_id and tracking_code
        updateOrder($result['invoice'], [
            'consignment_id' => $result['consignment_id'],
            'tracking_code' => $result['tracking_code']
        ]);
    } else {
        // Handle error
        logError($result['invoice'], $result);
    }
}
```

### Status Tracking Pattern:

```php
// Method 1: By Consignment ID
$status = getStatusByConsignmentId($consignment_id);

// Method 2: By Invoice ID
$status = getStatusByInvoice($invoice);

// Method 3: By Tracking Code
$status = getStatusByTrackingCode($tracking_code);

// Map status to application state
$appState = mapSteadfastStatus($status['delivery_status']);
```

---

## Comparison with Other Couriers

| Feature | Steadfast | RedX | Carrybee | Pathao |
|---------|-----------|------|----------|--------|
| Authentication | API Key + Secret Key | Bearer Token | Client ID/Secret/Context | OAuth 2.0 |
| Token Refresh | Not needed | No | No | Yes (5 days) |
| Bulk Orders | Yes (max 500) | No | Yes | Yes |
| Location Structure | None | Areas | City → Zone → Area | City → Zone → Area |
| Webhooks | No | Yes | Yes (24 events) | No |
| Returns | Built-in | Via Update | Via Cancel | Via Status |
| Price Calculation | No | Yes | No | Yes |
| Balance Check | Yes | No | No | No |

---

*Last Updated: 2026-03-29*
*API Version: v1*
