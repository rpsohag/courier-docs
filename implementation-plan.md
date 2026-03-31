# Laravel Courier Package Implementation Plan

## Context

This plan details the complete implementation of a Laravel courier package that integrates multiple Bangladeshi courier services (Steadfast, RedX, Carrybee, Pathao, Paperfly) following the same architectural patterns as Laravel Socialite.

**Why this change is being made:**
- Multiple courier APIs have been documented in the `docs/` folder
- Need a unified, consistent interface for all courier services
- Follow Laravel Socialite's proven patterns for multi-provider packages
- Provide developers with a familiar, elegant API

**Intended outcome:**
- A production-ready Laravel package supporting 5 courier providers
- Consistent API across all providers despite their differences
- Easy provider switching via configuration
- Full OAuth 2.0 support for Pathao with automatic token refresh
- Comprehensive testing and documentation

---

## Recommended Approach

### Directory Structure

```
courier-pkg/
├── src/
│   ├── CourierServiceProvider.php          # Laravel integration
│   ├── Contracts/
│   │   └── CourierProviderInterface.php    # Provider contract
│   ├── Exceptions/
│   │   ├── CourierException.php
│   │   ├── AuthenticationException.php
│   │   ├── ValidationException.php
│   │   ├── ProviderNotSupportedException.php
│   │   └── WebhookVerificationException.php
│   ├── Facades/
│   │   └── Courier.php                      # Static facade
│   ├── Factory/
│   │   ├── CourierManager.php               # Main coordinator
│   │   └── CourierFactory.php               # Provider factory
│   ├── Providers/
│   │   ├── AbstractCourierProvider.php      # Base class
│   │   ├── SteadfastProvider.php            # API Key auth
│   │   ├── RedXProvider.php                 # Bearer token auth
│   │   ├── CarrybeeProvider.php             # Multi-header auth
│   │   ├── PathaoProvider.php               # OAuth 2.0 auth
│   │   └── PaperflyProvider.php             # Basic auth
│   ├── Results/
│   │   ├── OrderResult.php
│   │   ├── TrackingResult.php
│   │   ├── PriceResult.php
│   │   ├── BulkOrderResult.php
│   │   ├── StoreResult.php
│   │   └── LocationResult.php
│   ├── Support/
│   │   ├── HttpClient.php
│   │   ├── CacheManager.php
│   │   ├── TokenManager.php                 # OAuth token management
│   │   └── ResponseNormalizer.php
│   ├── Webhook/
│   │   ├── WebhookController.php            # Handle incoming webhooks
│   │   ├── WebhookEvent.php                 # Unified event model
│   │   ├── Verifiers/
│   │   │   ├── PaperflyVerifier.php
│   │   │   ├── RedXVerifier.php
│   │   │   ├── CarrybeeVerifier.php
│   │   │   └── PathaoVerifier.php
│   │   ├── Normalizers/
│   │   │   ├── PaperflyNormalizer.php
│   │   │   ├── RedXNormalizer.php
│   │   │   ├── CarrybeeNormalizer.php
│   │   │   └── PathaoNormalizer.php
│   │   ├── Jobs/
│   │   │   └── ProcessWebhookEvent.php      # Async webhook processing
│   │   └── Listeners/
│   │       ├── ParcelCreatedListener.php
│   │       ├── ParcelDeliveredListener.php
│   │       ├── ParcelReturnedListener.php
│   │       └── ParcelFailedListener.php
│   └── config/
│       └── courier.php
├── database/
│   └── migrations/
│       ├── create_courier_orders_table.php
│       └── create_webhook_logs_table.php
├── tests/
│   ├── Unit/
│   │   ├── Providers/
│   │   ├── Webhook/
│   │   │   ├── Verifiers/
│   │   │   └── Normalizers/
│   │   └── Support/
│   └── Feature/
│       ├── OrderCreationTest.php
│       ├── WebhookHandlingTest.php
│       └── WebhookIntegrationTest.php
├── composer.json
├── README.md
└── LICENSE.md
```

### Implementation Phases

#### Phase 1: Core Architecture (Days 1-3)

**Critical files to create:**

1. **`composer.json`** - Package configuration with Laravel integration
2. **`src/CourierServiceProvider.php`** - Register services with Laravel
3. **`src/Facades/Courier.php`** - Static interface
4. **`src/Factory/CourierManager.php`** - Main coordinator class
5. **`src/Contracts/CourierProviderInterface.php`** - Provider contract
6. **`src/Providers/AbstractCourierProvider.php`** - Base functionality
7. **`config/courier.php`** - Configuration for all providers

**Key implementation details:**

**CourierManager** (d:\Project Files\courier-pkg\src\Factory\CourierManager.php)
```php
class CourierManager
{
    protected array $drivers = [];
    protected string $defaultDriver;

    public function driver(?string $driver = null): CourierProviderInterface
    {
        $driver = $driver ?: $this->defaultDriver;

        if (!isset($this->drivers[$driver])) {
            $this->drivers[$driver] = $this->createDriver($driver);
        }

        return $this->drivers[$driver];
    }

    protected function createDriver(string $driver): CourierProviderInterface
    {
        $providerClass = match($driver) {
            'steadfast' => SteadfastProvider::class,
            'redx' => RedXProvider::class,
            'carrybee' => CarrybeeProvider::class,
            'pathao' => PathaoProvider::class,
            'paperfly' => PaperflyProvider::class,
            default => throw new ProviderNotSupportedException($driver)
        };

        return new $providerClass($this->config["drivers.{$driver}"]);
    }
}
```

**AbstractCourierProvider** (d:\Project Files\courier-pkg\src\Providers\AbstractCourierProvider.php)
```php
abstract class AbstractCourierProvider implements CourierProviderInterface
{
    protected array $config;
    protected HttpClient $httpClient;
    protected ResponseNormalizer $normalizer;

    abstract protected function getAuthHeaders(): array;
    abstract protected function getApiBaseUrl(): string;

    protected function request(string $method, string $endpoint, array $data = []): array
    {
        $url = $this->getApiBaseUrl() . $endpoint;
        $headers = $this->getAuthHeaders();

        return $this->httpClient->request($method, $url, [
            'headers' => $headers,
            'json' => $data
        ]);
    }

    public function createOrder(array $data): OrderResult
    {
        $this->validateOrderData($data);
        $mappedData = $this->mapOrderData($data);
        $response = $this->request('POST', $this->getOrderEndpoint(), $mappedData);
        return $this->mapOrderResponse($response);
    }
}
```

#### Phase 2: First Provider Implementation - Steadfast (Days 4-7)

**Files to create:**

1. **`src/Providers/SteadfastProvider.php`** - First concrete implementation
2. **`src/Results/OrderResult.php`** - Order creation result
3. **`src/Results/TrackingResult.php`** - Tracking information
4. **`src/Support/HttpClient.php`** - Guzzle wrapper
5. **`src/Support/ResponseNormalizer.php`** - Response standardization
6. **`src/Exceptions/`** - All exception classes

**Authentication (Steadfast):**
```php
protected function getAuthHeaders(): array
{
    return [
        'Api-Key' => $this->config['api_key'],
        'Secret-Key' => $this->config['secret_key'],
        'Content-Type' => 'application/json'
    ];
}
```

**Key endpoints:**
- `POST /create_order` - Create single order
- `POST /create_order/bulk-order` - Bulk orders (max 500)
- `GET /status_by_trackingcode/{code}` - Track order
- `GET /get_balance` - Check balance
- `POST /create_return_request` - Create return

#### Phase 3: Remaining Providers (Days 8-14)

**Implementation order:**

1. **RedXProvider** - Bearer token authentication, webhooks, price calculation
2. **CarrybeeProvider** - Multi-header auth, hierarchical locations, 24 webhook events
3. **PathaoProvider** - OAuth 2.0 with TokenManager for automatic refresh
4. **PaperflyProvider** - Basic auth, exchange orders

**Authentication variations:**

**RedX:**
```php
protected function getAuthHeaders(): array
{
    return ['API-ACCESS-TOKEN' => 'Bearer ' . $this->config['api_token']];
}
```

**Carrybee:**
```php
protected function getAuthHeaders(): array
{
    return [
        'Client-ID' => $this->config['client_id'],
        'Client-Secret' => $this->config['client_secret'],
        'Client-Context' => $this->config['client_context']
    ];
}
```

**Pathao (OAuth 2.0):**
```php
protected function getAuthHeaders(): array
{
    $token = $this->tokenManager->getValidAccessToken();
    return [
        'Authorization' => 'Bearer ' . $token,
        'Content-Type' => 'application/json'
    ];
}
```

**TokenManager for Pathao:**
- Store access_token and refresh_token
- Check token expiry before use
- Auto-refresh when token expires (5 day TTL)
- Cache tokens to reduce API calls

#### Phase 4: Webhook Integration System (Days 15-18)

**Features to implement:**

1. **WebhookController** - Handle webhook events from all couriers
2. **WebhookNormalizer** - Normalize different webhook payload formats
3. **WebhookVerifier** - Authentication/verification for each courier
4. **WebhookEvent** - Unified event structure
5. **WebhookLogger** - Database logging for all webhooks
6. **CacheManager** - Cache locations, stores, tokens
7. **TokenManager** - OAuth token lifecycle management
8. **Bulk operation enhancements** - Chunking, progress tracking

**Webhook Architecture:**

```
POST /webhooks/{courier}
    ↓
WebhookController (verify → normalize → dispatch)
    ↓
Queue Job (async processing)
    ↓
Event Handler (update order status, trigger business logic)
    ↓
Database (courier_orders, webhook_logs)
```

**Supported Couriers Webhook Summary:**

| Courier | Events | Authentication | Timeout | Retry |
|---------|--------|----------------|---------|-------|
| **Paperfly** | 14 events | X-Paperfly-Secret header | 30s | 3x |
| **RedX** | 7 events | Query parameter token | 10s | Standard |
| **Carrybee** | 24 events | Signature header | Standard | Standard |
| **Pathao** | 21 events | Signature + response header | 10s | Standard |
| **Steadfast** | 0 events | Polling only | - | - |

**Key Implementation Files:**

1. **`src/Webhook/WebhookController.php`** - Main webhook entry point
2. **`src/Webhook/Verifiers/`** - Authentication for each courier
3. **`src/Webhook/Normalizers/`** - Payload normalization
4. **`src/Webhook/WebhookEvent.php`** - Unified event model
5. **`src/Webhook/Jobs/ProcessWebhookEvent.php`** - Async processing
6. **`src/Webhook/Listeners/`** - Event handlers
7. **database/migrations/create_webhook_tables.php`** - Database schema

**Authentication Per Courier:**

```php
// Paperfly: Header-based secret
protected function verifyPaperfly(Request $request): bool
{
    $secret = $request->header('X-Paperfly-Secret');
    return hash_equals(config('courier.paperfly.webhook_secret'), $secret);
}

// RedX: Query parameter
protected function verifyRedX(Request $request): bool
{
    $token = $request->query('token');
    return hash_equals(config('courier.redx.webhook_token'), $token);
}

// Carrybee: Signature header
protected function verifyCarrybee(Request $request): bool
{
    $signature = $request->header('X-Carrybee-Webhook-Signature');
    return hash_equals(config('courier.carrybee.webhook_signature'), $signature);
}

// Pathao: Signature header + special response header
protected function verifyPathao(Request $request): bool
{
    $signature = $request->header('X-PATHAO-Signature');
    return hash_equals(config('courier.pathao.webhook_secret'), $signature);
}

// Pathao response
protected function sendPathaoResponse(Response $response): Response
{
    return $response->header(
        'X-Pathao-Merchant-Webhook-Integration-Secret',
        config('courier.pathao.webhook_secret')
    );
}
```

**Event Mapping:**

| Internal Event | Paperfly | RedX | Carrybee | Pathao |
|----------------|----------|------|----------|--------|
| `parcel.created` | parcel.created | - | order.created | order.created |
| `parcel.picked_up` | parcel.picked_up | - | order.picked | - |
| `parcel.in_transit` | parcel.in_transit | delivery-in-progress | order.in-transit | in.transit |
| `parcel.delivered` | parcel.delivered | delivered | order.delivered | delivered |
| `parcel.partial` | parcel.partial | - | order.partial-delivery | partial.delivery |
| `parcel.returned` | parcel.return | returned | order.returned | return |
| `parcel.on_hold` | parcel.on_hold | agent-hold | order.delivery-on-hold | on.hold |

#### Phase 5: Testing & Documentation (Days 19-21)

**Testing structure:**
- Unit tests for each provider
- Mock HTTP responses using Guzzle handlers
- Feature tests for complete workflows
- Integration tests with Laravel

**Documentation:**
- Installation guide
- Configuration examples
- Usage examples for each provider
- API reference
- Troubleshooting guide

---

## Configuration

**`config/courier.php`:**

```php
return [
    'default' => env('COURIER_DRIVER', 'steadfast'),

    'drivers' => [
        'steadfast' => [
            'base_url' => env('STEADFAST_BASE_URL', 'https://portal.packzy.com/api/v1'),
            'api_key' => env('STEADFAST_API_KEY'),
            'secret_key' => env('STEADFAST_SECRET_KEY'),
            'webhook' => [
                'enabled' => false, // Steadfast doesn't support webhooks
            ],
        ],

        'redx' => [
            'base_url' => env('REDX_BASE_URL', 'https://openapi.redx.com.bd/v1.0.0-beta'),
            'sandbox_url' => env('REDX_SANDBOX_URL', 'https://sandbox.redx.com.bd/v1.0.0-beta'),
            'api_token' => env('REDX_API_TOKEN'),
            'sandbox' => env('REDX_SANDBOX', false),
            'webhook' => [
                'enabled' => true,
                'token' => env('REDX_WEBHOOK_TOKEN'),
                'timeout' => 10, // seconds
            ],
        ],

        'carrybee' => [
            'base_url' => env('CARRYBEE_BASE_URL', 'https://developers.carrybee.com'),
            'client_id' => env('CARRYBEE_CLIENT_ID'),
            'client_secret' => env('CARRYBEE_CLIENT_SECRET'),
            'client_context' => env('CARRYBEE_CLIENT_CONTEXT'),
            'webhook' => [
                'enabled' => true,
                'signature' => env('CARRYBEE_WEBHOOK_SIGNATURE'),
                'timeout' => 10,
            ],
        ],

        'pathao' => [
            'base_url' => env('PATHAO_BASE_URL', 'https://api-hermes.pathao.com'),
            'sandbox_url' => env('PATHAO_SANDBOX_URL', 'https://courier-api-sandbox.pathao.com'),
            'client_id' => env('PATHAO_CLIENT_ID'),
            'client_secret' => env('PATHAO_CLIENT_SECRET'),
            'username' => env('PATHAO_USERNAME'),
            'password' => env('PATHAO_PASSWORD'),
            'sandbox' => env('PATHAO_SANDBOX', false),
            'webhook' => [
                'enabled' => true,
                'secret' => env('PATHAO_WEBHOOK_SECRET', 'f3992ecc-59da-4cbe-a049-a13da2018d51'),
                'timeout' => 10,
                'response_secret' => env('PATHAO_WEBHOOK_SECRET'),
            ],
        ],

        'paperfly' => [
            'base_url' => env('PAPERFLY_BASE_URL', 'https://api.paperfly.com.bd'),
            'username' => env('PAPERFLY_USERNAME'),
            'password' => env('PAPERFLY_PASSWORD'),
            'webhook' => [
                'enabled' => true,
                'secret' => env('PAPERFLY_WEBHOOK_SECRET'),
                'timeout' => 30, // Paperfly has 30s timeout
                'retry_max' => 3, // Paperfly retries up to 3 times
            ],
        ],
    ],

    'cache' => [
        'enabled' => env('COURIER_CACHE_ENABLED', true),
        'ttl' => env('COURIER_CACHE_TTL', 86400), // 24 hours
    ],

    'webhook' => [
        'queue' => env('COURIER_WEBHOOK_QUEUE', 'webhooks'),
        'middleware' => ['throttle:webhooks'],
        'log_retention_days' => 90,
    ],
];
```

**Environment Variables (.env):**

```env
# Courier Selection
COURIER_DRIVER=steadfast

# Steadfast
STEADFAST_API_KEY=your_api_key
STEADFAST_SECRET_KEY=your_secret_key

# RedX
REDX_API_TOKEN=your_api_token
REDX_WEBHOOK_TOKEN=your_webhook_token

# Carrybee
CARRYBEE_CLIENT_ID=your_client_id
CARRYBEE_CLIENT_SECRET=your_client_secret
CARRYBEE_CLIENT_CONTEXT=your_client_context
CARRYBEE_WEBHOOK_SIGNATURE=your_webhook_signature

# Pathao
PATHAO_CLIENT_ID=your_client_id
PATHAO_CLIENT_SECRET=your_client_secret
PATHAO_USERNAME=your_email
PATHAO_PASSWORD=your_password
PATHAO_WEBHOOK_SECRET=f3992ecc-59da-4cbe-a049-a13da2018d51

# Paperfly
PAPERFLY_USERNAME=your_username
PAPERFLY_PASSWORD=your_password
PAPERFLY_WEBHOOK_SECRET=your_webhook_secret

# Webhook Settings
COURIER_WEBHOOK_QUEUE=webhooks
COURIER_CACHE_ENABLED=true
COURIER_CACHE_TTL=86400
```

**`composer.json`:**

```json
{
    "name": "your-vendor/courier",
    "description": "Laravel courier package with multiple provider support",
    "type": "library",
    "license": "MIT",
    "require": {
        "php": "^8.2",
        "laravel/framework": "^11.0",
        "guzzlehttp/guzzle": "^7.8"
    },
    "autoload": {
        "psr-4": {
            "Courier\\": "src/"
        }
    },
    "extra": {
        "laravel": {
            "providers": ["Courier\\CourierServiceProvider"],
            "aliases": {"Courier": "Courier\\Facades\\Courier"}
        }
    }
}
```

---

## Usage Examples

### Basic Usage

```php
use Courier\Facades\Courier;

// Create order with default driver
$result = Courier::createOrder([
    'recipient_name' => 'John Doe',
    'recipient_phone' => '01712345678',
    'recipient_address' => 'House 123, Road 4, Dhaka',
    'cod_amount' => 1500,
    'item_weight' => 500,
]);

echo $result->trackingId;

// Track order
$tracking = Courier::trackOrder($result->trackingId);
echo $tracking->currentStatus;
```

### Switching Providers

```php
// Use specific provider
$steadfast = Courier::driver('steadfast');
$order = $steadfast->createOrder([...]);

$redx = Courier::driver('redx');
$order = $redx->createOrder([...]);
```

### Bulk Orders

```php
$result = Courier::driver('steadfast')->createBulkOrders([
    ['recipient_name' => 'Customer 1', ...],
    ['recipient_name' => 'Customer 2', ...],
]);

echo "Successful: {$result->getSuccessCount()}";
echo "Failed: {$result->getFailureCount()}";
```

### Error Handling

```php
use Courier\Exceptions\ValidationException;
use Courier\Exceptions\AuthenticationException;

try {
    $result = Courier::createOrder($data);
} catch (ValidationException $e) {
    $errors = $e->getErrors();
} catch (AuthenticationException $e) {
    // Handle auth failure
}
```

### Webhook Setup

**1. Define Routes:**

```php
// routes/web.php or routes/api.php
Route::post('/webhooks/paperfly', [WebhookController::class, 'handlePaperfly']);
Route::post('/webhooks/redx', [WebhookController::class, 'handleRedX']);
Route::post('/webhooks/carrybee', [WebhookController::class, 'handleCarrybee']);
Route::post('/webhooks/pathao', [WebhookController::class, 'handlePathao']);
```

**2. Webhook Controller:**

```php
use Courier\Webhook\WebhookController;
use Illuminate\Http\Request;
use Courier\Facades\Courier;

class WebhookController extends Controller
{
    public function handle(Request $request, string $courier)
    {
        // Package will handle:
        // 1. Authentication/verification
        // 2. Payload normalization
        // 3. Duplicate detection
        // 4. Async dispatching

        return Courier::handleWebhook($request, $courier);
    }
}
```

**3. Event Listeners:**

```php
// app/Providers/EventServiceProvider.php
protected $listen = [
    \Courier\Webhook\Events\ParcelDelivered::class => [
        \App\Listeners\UpdateOrderStatus::class,
        \App\Listeners\SendDeliveryNotification::class,
        \App\Listeners\CreateInvoice::class,
    ],
    \Courier\Webhook\Events\ParcelReturned::class => [
        \App\Listeners\ProcessReturn::class,
        \App\Listeners\InitiateRefund::class,
    ],
];
```

**4. Custom Event Handler:**

```php
use Courier\Webhook\Events\WebhookReceived;
use Courier\Webhook\WebhookEvent;

class UpdateOrderStatus
{
    public function handle(WebhookEvent $event): void
    {
        // Access normalized event data
        $order = Order::where('tracking_number', $event->trackingNumber)
            ->firstOrFail();

        // Update status
        $order->update([
            'status' => $this->mapStatus($event->eventType),
            'last_webhook_at' => now()
        ]);

        // Trigger business logic
        if ($event->eventType === 'parcel.delivered') {
            // Send notification, update inventory, etc.
        }
    }

    protected function mapStatus(string $eventType): string
    {
        return match($eventType) {
            'parcel.created' => 'pending_pickup',
            'parcel.delivered' => 'delivered',
            'parcel.returned' => 'returned',
            default => 'processing'
        };
    }
}
```

**5. Webhook Querying:**

```php
use Courier\Webhook\Models\WebhookLog;
use Courier\Webhook\Models\CourierOrder;

// Get webhook history for an order
$webhooks = WebhookLog::where('tracking_number', $trackingId)
    ->orderBy('timestamp', 'desc')
    ->get();

// Get all orders by status
$deliveredOrders = CourierOrder::where('status', 'delivered')
    ->where('courier', 'paperfly')
    ->get();

// Get failed webhooks
$failedWebhooks = WebhookLog::where('processed', false)
    ->where('retry_count', '>=', 3)
    ->get();
```

---

## Verification

### Testing Strategy

**1. Unit Tests** (tests/Unit/)
- Test each provider method independently
- Mock HTTP responses using Guzzle MockHandler
- Test data transformation and validation
- Test error scenarios

**2. Feature Tests** (tests/Feature/)
- Test complete workflows (Create → Track → Cancel)
- Test multi-provider scenarios
- Test facade and manager functionality
- Test configuration loading

**3. Integration Tests**
- Test with real Laravel application
- Test cache integration
- Test token refresh flow (Pathao)
- Test webhook handling

### End-to-End Verification

1. **Install package** in fresh Laravel project
2. **Configure** `.env` with sandbox credentials
3. **Test order creation** with each provider
4. **Test tracking** for created orders
5. **Test bulk operations** (where supported)
6. **Test error handling** with invalid data
7. **Test token refresh** (Pathao - wait for token expiry)
8. **Verify caching** - check cache hit/miss
9. **Test provider switching** - create orders with different providers
10. **Test webhooks** for all providers that support them
11. **Run test suite** - ensure 100% coverage

### Webhook Testing

**1. Unit Tests:**

```php
class WebhookVerifierTest extends TestCase
{
    public function test_paperfly_webhook_verification()
    {
        $payload = [
            'event' => 'parcel.created',
            'timestamp' => now()->toIso8601String(),
            'data' => [
                'order_number' => 'Z-241225-174131-A1-A7',
                'merchant_order_reference' => 'test123',
                'barcode' => '231814375965',
                'package_price' => 10,
            ]
        ];

        $response = $this->postJson('/webhooks/paperfly', $payload, [
            'X-Paperfly-Secret' => config('courier.paperfly.webhook.secret')
        ]);

        $response->assertStatus(200)
            ->assertJson(['success' => true]);

        $this->assertDatabaseHas('webhook_logs', [
            'courier' => 'paperfly',
            'event_type' => 'parcel.created',
            'tracking_number' => 'Z-241225-174131-A1-A7'
        ]);
    }

    public function test_invalid_webhook_secret_rejected()
    {
        $response = $this->postJson('/webhooks/paperfly', [], [
            'X-Paperfly-Secret' => 'invalid_secret'
        ]);

        $response->assertStatus(401);
    }

    public function test_duplicate_webhook_handling()
    {
        $payload = [/* ... */];

        // Send same webhook twice
        $this->postJson('/webhooks/paperfly', $payload, [
            'X-Paperfly-Secret' => config('courier.paperfly.webhook.secret')
        ]);

        $response = $this->postJson('/webhooks/paperfly', $payload, [
            'X-Paperfly-Secret' => config('courier.paperfly.webhook.secret')
        ]);

        $response->assertStatus(200);
        $this->assertEquals(1, WebhookLog::count());
    }
}
```

**2. Integration Tests:**

```php
class WebhookIntegrationTest extends TestCase
{
    public function test_end_to_end_paperfly_webhook()
    {
        Event::fake();

        $payload = [/* ... */];

        $this->postJson('/webhooks/paperfly', $payload, [
            'X-Paperfly-Secret' => config('courier.paperfly.webhook.secret')
        ]);

        // Assert job was dispatched
        Event::assertDispatched(WebhookReceived::class);

        // Process job
        Queue::assertPushed(ProcessWebhookEvent::class);

        // Run job
        $job = Queue::pushed(ProcessWebhookEvent::class)[0];
        $job->handle(app(WebhookService::class));

        // Assert order was created/updated
        $this->assertDatabaseHas('courier_orders', [
            'tracking_number' => 'Z-241225-174131-A1-A7',
            'status' => 'pending_pickup'
        ]);
    }
}
```

**3. Testing with Real Webhooks:**

```bash
# Use ngrok for local testing
ngrok http 8000

# Configure webhook URL with courier
# https://abc123.ngrok.io/webhooks/paperfly

# Send test webhook
curl -X POST https://abc123.ngrok.io/webhooks/paperfly \
  -H "X-Paperfly-Secret: your_secret" \
  -H "Content-Type: application/json" \
  -d '{
    "event": "parcel.created",
    "timestamp": "2025-12-24T17:28:24+00:00",
    "data": {
      "order_number": "TEST-123",
      "merchant_order_reference": "test123",
      "barcode": "123456789",
      "package_price": 100,
      "recipient": {
        "name": "Test Customer",
        "phone": "01685048848",
        "address": "Test Address"
      }
    }
  }'
```

### Critical Files to Test

- `src/Factory/CourierManager.php` - Core coordination
- `src/Providers/AbstractCourierProvider.php` - Base functionality
- `src/Providers/SteadfastProvider.php` - First implementation
- `src/Support/TokenManager.php` - OAuth token management
- `src/Support/HttpClient.php` - HTTP client wrapper

---

## Summary

This plan provides a complete roadmap for building a Laravel courier package following Socialite's architecture. The implementation is broken into 5 phases over 21 days:

1. **Phase 1 (Days 1-3):** Core architecture - Manager, Facade, ServiceProvider, Interfaces
2. **Phase 2 (Days 4-7):** First provider - Steadfast with all result classes and support utilities
3. **Phase 3 (Days 8-14):** Remaining providers - RedX, Carrybee, Pathao, Paperfly
4. **Phase 4 (Days 15-18):** Webhook integration system - Unified webhook handling for all providers
5. **Phase 5 (Days 19-21):** Testing and documentation

The package will:
- Handle 5 different authentication methods (API Key, Bearer Token, Multi-header, OAuth 2.0, Basic Auth)
- Normalize responses across all providers
- Support bulk operations (where available)
- Manage OAuth tokens automatically with refresh
- Process webhooks from 4 couriers with 66+ total events
- Provide a consistent, elegant API for developers
- Support real-time order status updates via webhooks
- Include comprehensive logging and monitoring

### Webhook Coverage Summary

| Courier | Webhook Support | Events | Authentication |
|---------|----------------|--------|----------------|
| **Paperfly** | ✅ Full | 14 events | Secret Header |
| **RedX** | ✅ Full | 7 events | Query Token |
| **Carrybee** | ✅ Full | 24 events | Signature Header |
| **Pathao** | ✅ Full | 21 events | Signature + Response |
| **Steadfast** | ❌ None | 0 events | Polling required |
| **Total** | 4/5 providers | 66 events | 4 methods |

### Key Webhook Features

1. **Unified Interface:** Single endpoint pattern for all couriers
2. **Automatic Normalization:** Convert different payload formats to unified events
3. **Authentication:** Verify webhooks from each courier using their specific method
4. **Idempotency:** Prevent duplicate webhook processing
5. **Async Processing:** Queue-based processing for performance
6. **Comprehensive Logging:** Full audit trail of all webhooks
7. **Event Mapping:** Map courier-specific events to internal events
8. **Error Handling:** Robust error handling with retry logic
9. **Testing Support:** Complete testing suite with mocks and integration tests
10. **Monitoring:** Built-in metrics and alerting capabilities
