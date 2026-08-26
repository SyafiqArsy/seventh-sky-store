# Admin Dashboard — Seventh Sky Store

#admin #backend #frontend

---

## Overview

Admin-only dashboard with analytics, product/category/order management, and role-based access control.

---

## Access Control

### Middleware

```php
// app/Http/Middleware/AdminMiddleware.php
public function handle(Request $request, Closure $next)
{
    if (! $request->user() || $request->user()->role !== 'admin') {
        return response()->json([
            'success' => false,
            'message' => 'Forbidden: Admin access required'
        ], 403);
    }
    return $next($request);
}
```

### Route Protection

```php
// routes/api.php
Route::middleware(['auth:sanctum', 'admin'])->prefix('admin')->group(function () {
    // All admin routes here
});
```

### Frontend Guard

```typescript
// src/components/admin/AdminGuard.tsx
export function AdminGuard({ children }) {
  const { isAdmin, isAuthenticated } = useAuth();
  const router = useRouter();
  
  useEffect(() => {
    if (!isAuthenticated) router.push('/login');
    else if (!isAdmin) router.push('/');
  }, [isAuthenticated, isAdmin]);
  
  if (!isAuthenticated || !isAdmin) return null;
  
  return <>{children}</>;
}
```

---

## Dashboard Analytics

### Endpoint

```http
GET /api/admin/dashboard
Authorization: Bearer <admin_token>
```

### Response

```json
{
  "success": true,
  "data": {
    "total_products": 50,
    "total_categories": 8,
    "total_orders": 120,
    "total_revenue": "45000000.00"
  }
}
```

### Implementation

```php
// AdminDashboardController@index
public function index()
{
    return response()->json([
        'success' => true,
        'data' => [
            'total_products' => Product::count(),
            'total_categories' => Category::count(),
            'total_orders' => Order::count(),
            'total_revenue' => Order::where('payment_status', 'paid')->sum('grand_total'),
        ]
    ]);
}
```

---

## Product Management (Admin)

### List Products

```http
GET /api/admin/products?search=&page=1&per_page=15
```

- Returns ALL products (including inactive)
- Search by name
- Paginated

### Create Product

```http
POST /api/admin/products
Content-Type: multipart/form-data
{
  "category_id": 1,
  "name": "Product Name",
  "description": "Description",
  "price": 299000,
  "stock": 10,
  "is_active": true,
  "image": <file>
}
```

**Auto-generates:**
- `sku`: `SKU-` + 8 random chars
- `slug`: from name

### Update Product

```http
PUT /api/admin/products/{id}
Content-Type: multipart/form-data
{
  "category_id": 1,
  "name": "Updated Name",
  "description": "...",
  "price": 349000,
  "stock": 15,
  "is_active": true,
  "image": <file> // optional
}
```

- Updates slug if name changed
- Replaces Cloudinary image if new file uploaded

### Delete Product

```http
DELETE /api/admin/products/{id}
```

- Deletes Cloudinary image
- Cascades to cart_items, order_items (restrict/protect)

---

## Category Management (Admin)

### List Categories

```http
GET /api/admin/categories?search=
```

- All categories, no pagination

### CRUD Endpoints

| Method | Endpoint | Body |
|--------|----------|------|
| POST | `/api/admin/categories` | `{ "name": "..." }` |
| PUT | `/api/admin/categories/{id}` | `{ "name": "..." }` |
| DELETE | `/api/admin/categories/{id}` | — |

**Auto-generates slug** from name.

---

## Order Management (Admin)

### List Orders

```http
GET /api/admin/orders?search=&page=1&per_page=15
```

- Search by order_number
- Includes user + items
- Paginated

### Order Detail

```http
GET /api/admin/orders/{id}
```

- Full order with user + items

### Update Status

```http
PUT /api/admin/orders/{id}/status
{ "order_status": "processing" }
```

**Validated by state machine** (see [[Order Management]])

---

## Frontend Admin Pages

### Layout

```
app/(admin)/layout.tsx
├── AdminSidebar (navigation)
├── Header (user, logout)
└── {children} (page content)
```

### Pages

| Page | Component | Features |
|------|-----------|----------|
| `/admin` | AdminDashboardView | Stats cards, quick links |
| `/admin/products` | AdminProductsView | Table, search, paginate, actions |
| `/admin/products/create` | CreateProductForm | Full form + image upload |
| `/admin/products/[id]/edit` | EditProductForm | Pre-filled + image replace |
| `/admin/categories` | AdminCategoriesView | List, create, edit, delete |
| `/admin/categories/create` | CreateCategoryForm | Name input |
| `/admin/categories/[id]/edit` | EditCategoryForm | Name input |
| `/admin/orders` | AdminOrders | Table, search, status dropdown |
| `/admin/orders/[id]` | AdminOrderDetail | Full detail + status update |

---

## AdminSidebar Navigation

```
AdminSidebar.tsx
├── Dashboard (/admin)
├── Products (/admin/products)
│   ├── List
│   └── Create
├── Categories (/admin/categories)
│   ├── List
│   └── Create
└── Orders (/admin/orders)
    ├── List
    └── Detail (via row click)
```

---

## Cloudinary Upload Test

```http
POST /api/admin/upload-test
Content-Type: multipart/form-data
{ "image": <file> }
```

Returns Cloudinary response for debugging.

---

## Related Notes

- [[Authentication#Role-Based Access]] — Admin middleware
- [[API Reference#Admin Endpoints]] — All admin endpoints
- [[Product Catalog]] — Product CRUD details
- [[Order Management]] — Order state machine
- [[Frontend Structure#Admin Components]] — Admin UI