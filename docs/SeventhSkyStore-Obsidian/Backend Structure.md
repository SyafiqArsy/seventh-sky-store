# Backend Structure — Seventh Sky Store

#backend #laravel

---

## Directory Structure

```
backend/
├── app/
│   ├── Http/
│   │   ├── Controllers/Api/      # API Controllers
│   │   ├── Middleware/           # Custom Middleware
│   │   └── Requests/             # Form Requests (Validation)
│   ├── Models/                   # Eloquent Models
│   ├── Services/                 # Business Logic Services
│   └── Providers/                # Service Providers
├── database/
│   ├── migrations/               # Database Migrations
│   ├── factories/                # Model Factories
│   └── seeders/                  # Database Seeders
├── routes/
│   └── api.php                   # API Routes
├── config/
│   └── midtrans.php              # Midtrans Config
└── config/                       # Laravel Configs
```

---

## Controllers

### AuthController
`app/Http/Controllers/Api/AuthController.php`

| Method | Endpoint | Description |
|--------|----------|-------------|
| `register` | POST `/api/register` | Create user, return token |
| `login` | POST `/api/login` | Validate, return token |
| `me` | GET `/api/me` | Current user |
| `logout` | POST `/api/logout` | Revoke current token |
| `logoutAll` | POST `/api/logout-all` | Revoke all tokens |

### ProductController
`app/Http/Controllers/Api/ProductController.php`

| Method | Endpoint | Access |
|--------|----------|--------|
| `index` | GET `/api/products` | Public |
| `show` | GET `/api/products/{slug}` | Public |
| `adminIndex` | GET `/api/admin/products` | Admin |
| `adminShow` | GET `/api/admin/products/{id}` | Admin |
| `store` | POST `/api/admin/products` | Admin |
| `update` | PUT `/api/admin/products/{id}` | Admin |
| `destroy` | DELETE `/api/admin/products/{id}` | Admin |

### CategoryController
`app/Http/Controllers/Api/CategoryController.php`

| Method | Endpoint | Access |
|--------|----------|--------|
| `index` | GET `/api/categories` | Public |
| `show` | GET `/api/categories/{slug}` | Public |
| `adminIndex` | GET `/api/admin/categories` | Admin |
| `store` | POST `/api/admin/categories` | Admin |
| `update` | PUT `/api/admin/categories/{id}` | Admin |
| `destroy` | DELETE `/api/admin/categories/{id}` | Admin |

### CartController
`app/Http/Controllers/Api/CartController.php`

| Method | Endpoint | Description |
|--------|----------|-------------|
| `index` | GET `/api/cart` | Get cart with items |
| `store` | POST `/api/cart/items` | Add item |
| `update` | PUT `/api/cart/items/{id}` | Update quantity |
| `destroy` | DELETE `/api/cart/items/{id}` | Remove item |

### CheckoutController
`app/Http/Controllers/Api/CheckoutController.php`

| Method | Endpoint | Description |
|--------|----------|-------------|
| `store` | POST `/api/checkout` | Create order + Snap token |

### OrderController
`app/Http/Controllers/Api/OrderController.php`

| Method | Endpoint | Access |
|--------|----------|--------|
| `index` | GET `/api/orders` | Auth |
| `show` | GET `/api/orders/{id}` | Auth |
| `adminIndex` | GET `/api/admin/orders` | Admin |
| `adminShow` | GET `/api/admin/orders/{id}` | Admin |
| `updateStatus` | PUT `/api/admin/orders/{id}/status` | Admin |

### AdminDashboardController
`app/Http/Controllers/Api/AdminDashboardController.php`

| Method | Endpoint | Description |
|--------|----------|-------------|
| `index` | GET `/api/admin/dashboard` | Stats |

### MidtransController
`app/Http/Controllers/Api/MidtransController.php`

| Method | Endpoint | Description |
|--------|----------|-------------|
| `notification` | POST `/api/midtrans/notification` | Webhook handler |

### UploadController
`app/Http/Controllers/Api/UploadController.php`

| Method | Endpoint | Description |
|--------|----------|-------------|
| `test` | POST `/api/admin/upload-test` | Cloudinary test |

---

## Services

### MidtransService
`app/Services/MidtransService.php`

```php
class MidtransService {
    public static function init(): void
    public static function createSnapToken(array $params): string
    public static function verifySignature(array $notification): bool
}
```

### CloudinaryService
`app/Services/CloudinaryService.php`

```php
class CloudinaryService {
    public function upload(string $filePath): array
    public function destroy(string $publicId): void
}
```

### CategoryService
`app/Services/CategoryService.php`

```php
class CategoryService {
    public function create(array $data): Category
    public function update(Category $category, array $data): Category
}
```

---

## Models

### User
`app/Models/User.php`
- `HasApiTokens` (Sanctum)
- Fillable: name, email, password, role
- Casts: role (string)

### Category
`app/Models/Category.php`
- Fillable: name, slug
- Relationship: `products()` → HasMany

### Product
`app/Models/Product.php`
- Fillable: category_id, name, slug, description, sku, price, stock, image, image_public_id, is_active
- Casts: price (decimal:2), is_active (boolean)
- Relationship: `category()` → BelongsTo

### Cart
`app/Models/Cart.php`
- Fillable: user_id
- Relationships: `user()` → BelongsTo, `items()` → HasMany

### CartItem
`app/Models/CartItem.php`
- Fillable: cart_id, product_id, quantity
- Relationships: `cart()` → BelongsTo, `product()` → BelongsTo

### Order
`app/Models/Order.php`
- Fillable: user_id, order_number, recipient_name, phone, address, city, postal_code, total_price, shipping_cost, grand_total, payment_status, order_status, midtrans_*, paid_at
- Casts: paid_at (datetime)
- Relationships: `user()` → BelongsTo, `items()` → HasMany

### OrderItem
`app/Models/OrderItem.php`
- Fillable: order_id, product_id, product_name, product_price, quantity, subtotal
- Relationships: `order()` → BelongsTo, `product()` → BelongsTo

---

## Middleware

### AdminMiddleware
`app/Http/Middleware/AdminMiddleware.php`
- Checks `user->role === 'admin'`
- Returns 403 if not admin

### SecurityHeaders
`app/Http/Middleware/SecurityHeaders.php`
- Adds security headers to all responses:
  - X-Content-Type-Options: nosniff
  - X-Frame-Options: DENY
  - X-XSS-Protection: 1; mode=block
  - Referrer-Policy: strict-origin-when-cross-origin
  - Content-Security-Policy (configurable)

---

## Form Requests

### Auth
- `RegisterRequest` — name, email, password, password_confirmation
- `LoginRequest` — email, password

### Cart
- `AddToCartRequest` — product_id (exists), quantity (min:1)
- `UpdateCartItemRequest` — quantity (min:1)

### Category
- `StoreCategoryRequest` — name (required, unique)
- `UpdateCategoryRequest` — name (required, unique)

### Product
- `ProductStoreRequest` — category_id, name, description, price, stock, image (file), is_active
- `ProductUpdateRequest` — same, all optional, image optional

### Checkout
- `CheckoutRequest` — recipient_name, phone, address, city, postal_code, shipping_cost

---

## Routes

```php
// routes/api.php

// Health / Webhook
POST /midtrans/notification → MidtransController@notification

// Public (throttled)
POST /register → AuthController@register (5/min)
POST /login → AuthController@login (10/min)
GET /categories → CategoryController@index
GET /categories/{slug} → CategoryController@show
GET /products → ProductController@index
GET /products/{slug} → ProductController@show

// Authenticated (auth:sanctum)
GET /me → AuthController@me
POST /logout → AuthController@logout
POST /logout-all → AuthController@logoutAll
GET /cart → CartController@index
POST /cart/items → CartController@store
PUT /cart/items/{id} → CartController@update
DELETE /cart/items/{id} → CartController@destroy
POST /checkout → CheckoutController@store (10/min)
GET /orders → OrderController@index
GET /orders/{id} → OrderController@show

// Admin (auth:sanctum + admin)
GET /admin/dashboard → AdminDashboardController@index
POST /admin/upload-test → UploadController@test
CRUD /admin/categories → CategoryController
CRUD /admin/products → ProductController
GET /admin/orders → OrderController@adminIndex
GET /admin/orders/{id} → OrderController@adminShow
PUT /admin/orders/{id}/status → OrderController@updateStatus
```

---

## Configuration

### config/midtrans.php

```php
return [
    'server_key' => env('MIDTRANS_SERVER_KEY'),
    'client_key' => env('MIDTRANS_CLIENT_KEY'),
    'is_production' => env('MIDTRANS_IS_PRODUCTION', false),
    'is_sanitized' => env('MIDTRANS_IS_SANITIZED', true),
    'is_3ds' => env('MIDTRANS_IS_3DS', true),
];
```

---

## Related Notes

- [[Architecture#Backend Architecture]] — Controller/Service overview
- [[API Reference]] — All endpoints
- [[Database Schema]] — Models & relationships
- [[Authentication]] — Sanctum setup
- [[Admin Dashboard]] — Admin routes