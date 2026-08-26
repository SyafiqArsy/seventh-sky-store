# Product Catalog — Seventh Sky Store

#backend #frontend #database

---

## Overview

Product catalog system with categories, search, filtering, and pagination.

---

## Backend

### ProductController

| Method | Endpoint | Access | Description |
|--------|----------|--------|-------------|
| `index` | GET `/api/products` | Public | List active products |
| `show` | GET `/api/products/{slug}` | Public | Product detail |
| `adminIndex` | GET `/api/admin/products` | Admin | List all products |
| `adminShow` | GET `/api/admin/products/{id}` | Admin | Product detail |
| `store` | POST `/api/admin/products` | Admin | Create product |
| `update` | PUT `/api/admin/products/{id}` | Admin | Update product |
| `destroy` | DELETE `/api/admin/products/{id}` | Admin | Delete product |

### Product Model

```php
// app/Models/Product.php
protected $fillable = [
    'category_id', 'name', 'slug', 'description',
    'sku', 'price', 'stock', 'image', 'image_public_id', 'is_active'
];

protected $casts = [
    'price' => 'decimal:2',
    'is_active' => 'boolean',
];

public function category(): BelongsTo
{
    return $this->belongsTo(Category::class);
}
```

### Query Logic (Public)

```php
Product::with('category')
    ->when($search, fn($q) => $q->where('name', 'like', "%{$search}%"))
    ->when($category, fn($q) => $q->whereHas('category', fn($c) => $c->where('slug', $category)))
    ->where('is_active', true)
    ->latest()
    ->paginate($perPage);
```

### Auto-Generation

```php
// On create
'sku' => 'SKU-' . strtoupper(Str::random(8)),
'slug' => Str::slug($request->name),

// On update (if name changed)
if (isset($data['name'])) {
    $data['slug'] = Str::slug($data['name']);
}
```

---

## CategoryController

| Method | Endpoint | Access | Description |
|--------|----------|--------|-------------|
| `index` | GET `/api/categories` | Public | All categories |
| `show` | GET `/api/categories/{slug}` | Public | Category detail |
| `adminIndex` | GET `/api/admin/categories` | Admin | All categories |
| `store` | POST `/api/admin/categories` | Admin | Create |
| `update` | PUT `/api/admin/categories/{id}` | Admin | Update |
| `destroy` | DELETE `/api/admin/categories/{id}` | Admin | Delete |

### CategoryService

```php
// app/Services/CategoryService.php
public function create(array $data): Category
{
    $data['slug'] = Str::slug($data['name']);
    return Category::create($data);
}

public function update(Category $category, array $data): Category
{
    if (isset($data['name'])) {
        $data['slug'] = Str::slug($data['name']);
    }
    $category->update($data);
    return $category->fresh();
}
```

---

## Frontend

### Data Fetching (React Query)

```typescript
// src/lib/api.ts
export async function getProducts(params?: { search?: string; category?: string; page?: number }) {
  const searchParams = new URLSearchParams();
  if (params?.search) searchParams.set('search', params.search);
  if (params?.category) searchParams.set('category', params.category);
  if (params?.page) searchParams.set('page', params.page.toString());
  
  const response = await fetch(`${API_URL}/products?${searchParams}`, { cache: 'no-store' });
  return response.json();
}

export async function getProduct(slug: string) {
  const response = await fetch(`${API_URL}/products/${slug}`, { cache: 'no-store' });
  return response.json();
}
```

### Components

| Component | File | Purpose |
|-----------|------|---------|
| `ProductsView` | `src/components/product/ProductsView.tsx` | Product grid with search/filter |
| `ProductCard` | `src/components/product/ProductCard.tsx` | Individual product display |
| `ProductActions` | `src/components/product/ProductActions.tsx` | Add to cart, quantity |
| `QuantitySelector` | `src/components/product/QuantitySelector.tsx` | +/- quantity buttons |

### Product Detail Page

```
app/(site)/products/[slug]/page.tsx
├── Hero image
├── Product info (name, price, description)
├── QuantitySelector
├── ProductActions (Add to Cart)
└── Related products
```

### Featured Products (Homepage)

```typescript
// src/lib/api.ts
export async function getLatestProducts(limit = 3) {
  const response = await fetch(`${API_URL}/products`, { cache: 'no-store' });
  const result = await response.json();
  return result.data.slice(0, limit);
}
```

---

## Admin Forms

### CreateProductForm

- Category dropdown (fetches from `/api/admin/categories`)
- Name (auto-generates slug)
- Description (textarea)
- Price (number, IDR)
- Stock (number)
- Image upload (Cloudinary)
- Active toggle

### EditProductForm

- Pre-filled with existing data
- Image optional (keeps current if not changed)
- Deletes old Cloudinary image on replace

---

## Search & Filter UX

### Public Product Page
- Search input (debounced)
- Category filter pills
- Pagination controls
- Loading skeletons

### Admin Product Page
- Search input
- Pagination
- Inline edit/delete actions
- Bulk actions (future)

---

## Cloudinary Integration

### Upload Flow

```php
// ProductController@store
$upload = $this->cloudinaryService->upload($request->file('image')->getRealPath());

$product = Product::create([
    ...
    'image' => $upload['url'],
    'image_public_id' => $upload['public_id'],
]);
```

### CloudinaryService

```php
// app/Services/CloudinaryService.php
public function upload(string $filePath): array
{
    $result = (new UploadApi())->upload($filePath, [
        'folder' => 'products',
        'transformation' => [
            'width' => 800,
            'height' => 800,
            'crop' => 'limit',
            'quality' => 'auto',
            'format' => 'webp',
        ],
    ]);
    
    return [
        'url' => $result['secure_url'],
        'public_id' => $result['public_id'],
    ];
}

public function destroy(string $publicId): void
{
    (new UploadApi())->destroy($publicId);
}
```

---

## Related Notes

- [[Database Schema#products]] — Product table structure
- [[API Reference#Products]] — Product endpoints
- [[Architecture#Data Flow Examples]] — Product listing flow
- [[Admin Dashboard#Product Management]] — Admin CRUD
- [[Visual Effects]] — Collections page with products