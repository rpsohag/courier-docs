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
│   │   └── ProviderNotSupportedException.php
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
│   └── config/
│       └── courier.php
├── tests/
│   ├── Unit/
│   └── Feature/
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

#### Phase 4: Advanced Features (Days 15-18)

**Features to implement:**

1. **CacheManager** - Cache locations, stores, tokens
2. **TokenManager** - OAuth token lifecycle management
3. **WebhookController** - Handle webhook events
4. **Bulk operation enhancements** - Chunking, progress tracking

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
        ],

        'redx' => [
            'base_url' => env('REDX_BASE_URL', 'https://openapi.redx.com.bd/v1.0.0-beta'),
            'sandbox_url' => env('REDX_SANDBOX_URL', 'https://sandbox.redx.com.bd/v1.0.0-beta'),
            'api_token' => env('REDX_API_TOKEN'),
            'sandbox' => env('REDX_SANDBOX', false),
        ],

        'carrybee' => [
            'base_url' => env('CARRYBEE_BASE_URL', 'https://developers.carrybee.com'),
            'client_id' => env('CARRYBEE_CLIENT_ID'),
            'client_secret' => env('CARRYBEE_CLIENT_SECRET'),
            'client_context' => env('CARRYBEE_CLIENT_CONTEXT'),
        ],

        'pathao' => [
            'base_url' => env('PATHAO_BASE_URL', 'https://api-hermes.pathao.com'),
            'sandbox_url' => env('PATHAO_SANDBOX_URL', 'https://courier-api-sandbox.pathao.com'),
            'client_id' => env('PATHAO_CLIENT_ID'),
            'client_secret' => env('PATHAO_CLIENT_SECRET'),
            'username' => env('PATHAO_USERNAME'),
            'password' => env('PATHAO_PASSWORD'),
            'sandbox' => env('PATHAO_SANDBOX', false),
        ],

        'paperfly' => [
            'base_url' => env('PAPERFLY_BASE_URL', 'https://api.paperfly.com.bd'),
            'username' => env('PAPERFLY_USERNAME'),
            'password' => env('PAPERFLY_PASSWORD'),
        ],
    ],

    'cache' => [
        'enabled' => env('COURIER_CACHE_ENABLED', true),
        'ttl' => env('COURIER_CACHE_TTL', 86400), // 24 hours
    ],
];
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
10. **Run test suite** - ensure 100% coverage

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
4. **Phase 4 (Days 15-18):** Advanced features - Caching, token management, webhooks
5. **Phase 5 (Days 19-21):** Testing and documentation

The package will handle 5 different authentication methods, normalize responses across all providers, support bulk operations, manage OAuth tokens automatically, and provide a consistent, elegant API for developers.
