# Architecture — Seventh Sky Store

#architecture #backend #frontend

---

## Overview

Seventh Sky Store uses a **headless architecture** with a clear separation between backend (Laravel REST API) and frontend (Next.js App Router).

```mermaid
graph TD
    subgraph Frontend [Next.js 16 Frontend]
        Site[(site) Pages]
        Auth[(auth) Pages]
        Admin[(admin) Pages]
        API_Client[API Client / Axios]
        State[Zustand + React Query]
        Visual[WebGL + Framer Motion]
    end

    subgraph Backend [Laravel 13 API]
        AuthCtrl[AuthController]
        ProdCtrl[ProductController]
        CatCtrl[CategoryController]
        CartCtrl[CartController]
        ChkCtrl[CheckoutController]
        OrdCtrl[OrderController]
        AdminCtrl[AdminDashboardController]
        MidCtrl[MidtransController]
        UpCtrl[UploadController]
        Services[Services Layer]
        Middleware[Middleware]
    end

    subgraph External [External Services]
        Midtrans[Midtrans Payment]
        Cloudinary[Cloudinary Images]
    end

    Site --> API_Client
    Auth --> API_Client
    Admin --> API_Client
    API_Client --> AuthCtrl
    API_Client --> ProdCtrl
    API_Client --> CatCtrl
    API_Client --> CartCtrl
    API_Client --> ChkCtrl
    API_Client --> OrdCtrl
    API_Client --> AdminCtrl
    API_Client --> MidCtrl
    API_Client --> UpCtrl

    AuthCtrl --> Services
    ProdCtrl --> Services
    CatCtrl --> Services
    CartCtrl --> Services
    ChkCtrl --> Services
    OrdCtrl --> Services
    AdminCtrl --> Services

    Services --> Midtrans
    Services --> Cloudinary
```

---

## Frontend Architecture (Next.js 16)

### Route Groups (App Router)

```
app/
├── (site)/           # Public pages — no auth required
│   ├── page.tsx              # Homepage
│   ├── products/page.tsx     # Product listing
│   ├── products/[slug]/page.tsx  # Product detail
│   ├── collections/page.tsx  # Collections with WebGL
│   ├── cart/page.tsx         # Shopping cart
│   ├── checkout/page.tsx     # Checkout form
│   ├── orders/page.tsx       # Order history
│   ├── orders/[id]/page.tsx  # Order detail
│   └── contact/page.tsx      # Contact page
├── (auth)/           # Auth pages — guest only
│   ├── layout.tsx
│   ├── login/page.tsx
│   └── register/page.tsx
└── (admin)/          # Admin pages — admin role required
    ├── layout.tsx
    ├── page.tsx              # Dashboard
    ├── products/page.tsx     # Product list
    ├── products/create/page.tsx
    ├── products/[id]/edit/page.tsx
    ├── categories/page.tsx   # Category list
    ├── categories/create/page.tsx
    ├── categories/[id]/edit/page.tsx
    ├── orders/page.tsx       # Order list
    └── orders/[id]/page.tsx  # Order detail
```

### State Management Strategy

| Layer | Tool | Responsibility |
|-------|------|----------------|
| **Client State** | Zustand | Cart items, UI state (modals, sidebar), custom cursor |
| **Server State** | TanStack React Query | Products, categories, orders, user data — caching, sync, deduping |
| **Auth State** | React Context | Current user, token, login/logout actions |
| **Toast State** | React Context | Global toast notifications |

### Component Organization

```
src/components/
├── admin/           # Admin-specific components
│   ├── AdminDashboardView.tsx
│   ├── AdminProductsView.tsx
│   ├── AdminCategoriesView.tsx
│   ├── AdminOrders.tsx
│   ├── AdminOrderDetail.tsx
│   ├── AdminSidebar.tsx
│   ├── CreateProductForm.tsx
│   ├── EditProductForm.tsx
│   ├── CreateCategoryForm.tsx
│   └── EditCategoryForm.tsx
├── auth/            # Auth forms
│   ├── LoginForm.tsx
│   └── RegisterForm.tsx
├── cart/            # Cart components
│   └── CartView.tsx
├── checkout/        # Checkout flow
│   └── CheckoutForm.tsx
├── collections/     # Visual effects
│   ├── Galaxy.tsx           # WebGL starfield (OGL + GLSL)
│   └── FlyingPosters.tsx    # 3D poster animations
├── home/            # Homepage sections
│   ├── HeroSection.tsx
│   ├── HeroCarousel.tsx
│   ├── FeaturedCollection.tsx
│   ├── BrandStatement.tsx
│   ├── NewArrival.tsx
│   ├── TypographyShowcase.tsx
│   └── BrandIntro.tsx
├── layout/          # Layout components
│   ├── Navbar.tsx
│   ├── Footer.tsx
│   ├── ConditionalFooter.tsx
│   └── CustomCursor.tsx
├── order/           # Order components
│   ├── OrdersView.tsx
│   └── OrderDetailView.tsx
├── product/         # Product components
│   ├── ProductCard.tsx
│   ├── ProductsView.tsx
│   ├── QuantitySelector.tsx
│   ├── ProductActions.tsx
│   └── loading.tsx
└── ui/              # Reusable UI primitives
    ├── Toast.tsx
    ├── ConfirmModal.tsx
    ├── ClickRipple.tsx
    └── SmoothScroll.tsx
```

---

## Backend Architecture (Laravel 13)

### Controller Structure

```
app/Http/Controllers/Api/
├── AuthController.php           # Register, login, me, logout, logout-all
├── ProductController.php        # Public + admin product CRUD
├── CategoryController.php       # Public + admin category CRUD
├── CartController.php           # Cart CRUD (authenticated)
├── CheckoutController.php       # Order creation + Midtrans token
├── OrderController.php          # Customer + admin order management
├── AdminDashboardController.php # Admin stats
├── MidtransController.php       # Webhook handler
└── UploadController.php         # Cloudinary test upload
```

### Service Layer

| Service | Responsibility |
|---------|----------------|
| `MidtransService` | Snap token generation, webhook signature verification |
| `CloudinaryService` | Image upload, destroy, transformation |
| `CategoryService` | Slug generation, category CRUD logic |

### Middleware Pipeline

```
Request
  │
  ├─► SecurityHeaders (global)
  │     ├─► X-Content-Type-Options: nosniff
  │     ├─► X-Frame-Options: DENY
  │     ├─► X-XSS-Protection: 1; mode=block
  │     └─► Referrer-Policy: strict-origin-when-cross-origin
  │
  ├─► Public Routes (throttle: 5/min register, 10/min login)
  │     ├─► POST /register
  │     ├─► POST /login
  │     ├─► GET /products
  │     ├─► GET /products/{slug}
  │     ├─► GET /categories
  │     ├─► GET /categories/{slug}
  │     └─► POST /midtrans/notification
  │
  ├─► auth:sanctum (token validation)
  │     ├─► GET /me
  │     ├─► POST /logout
  │     ├─► POST /logout-all
  │     ├─► GET /cart
  │     ├─► POST /cart/items
  │     ├─► PUT /cart/items/{id}
  │     ├─► DELETE /cart/items/{id}
  │     ├─► POST /checkout (throttle: 10/min)
  │     ├─► GET /orders
  │     └─► GET /orders/{id}
  │
  └─► auth:sanctum + admin (role check)
        ├─► GET /admin/dashboard
        ├─► CRUD /admin/categories
        ├─► CRUD /admin/products
        ├─► GET /admin/orders
        ├─► GET /admin/orders/{id}
        ├─► PUT /admin/orders/{id}/status
        └─► POST /admin/upload-test
```

---

## Data Flow Examples

### 5.1 Product Listing Flow

```
User visits /products
       │
       ▼
Frontend: ProductsView.tsx
       │
       ▼
React Query: useQuery(['products'], getProducts)
       │
       ▼
GET /api/products?search=&category=&page=1
       │
       ▼
Laravel: ProductController@index
       │
       ├─► Product::with('category')
       │     ├─► where('is_active', true)
       │     ├─► when(search) → where name LIKE
       │     └─► when(category) → whereHas category slug
       │
       └─► paginate(15)
       │
       ▼
JSON Response: { success, data: [...], meta: {...} }
       │
       ▼
React Query caches → ProductsView renders ProductCard[]
```

### 5.2 Checkout Flow

```
User submits CheckoutForm
       │
       ▼
POST /api/checkout { items, shipping }
       │
       ▼
Laravel: CheckoutController@store
       │
       ├─► Validate request (CheckoutRequest)
       ├─► DB Transaction
       │     ├─► Create Order (pending, unpaid)
       │     ├─► Create OrderItems (snapshot product data)
       │     ├─► Decrement product stock
       │     └─► Clear user's cart
       │
       ├─► MidtransService::createSnapToken()
       │     ├─► Build transaction params
       │     └─► Snap::getSnapToken()
       │
       ▼
JSON: { success, data: { snap_token, redirect_url, order } }
       │
       ▼
Frontend: loadMidtransScript() → snap.pay(snap_token)
       │
       ▼
Midtrans Popup → User pays
       │
       ▼
Midtrans → POST /api/midtrans/notification (webhook)
       │
       ▼
Laravel: MidtransController@notification
       │
       ├─► Verify SHA-512 signature
       ├─► Parse transaction_status
       ├─► Update Order: payment_status, paid_at, midtrans_*
       └─► If failed/expired → Restore stock
```

---

## Database Relationships

```
User (1) ─────< (M) Cart
Cart (1) ─────< (M) CartItem >───── (1) Product
Product (M) ──> (1) Category
User (1) ─────< (M) Order
Order (1) ─────< (M) OrderItem
OrderItem (M) ──> (1) Product (snapshot at purchase time)
```

---

## Security Architecture

### Authentication
- **Laravel Sanctum** — Personal access tokens (API tokens)
- Tokens stored in `personal_access_tokens` table
- Token abilities: `*` (full access)
- Token expiration: configurable (default none)

### Authorization
- **Admin Middleware** — Checks `user->role === 'admin'`
- Applied to all `/admin/*` routes
- Returns 403 if not admin

### Rate Limiting
- Register: 5 requests/minute per IP
- Login: 10 requests/minute per IP
- Checkout: 10 requests/minute per user

### Webhook Security
- Midtrans sends `signature_key` (SHA-512)
- Server recomputes: `hash('sha512', order_id + status_code + gross_amount + server_key)`
- Constant-time comparison to prevent timing attacks

---

## External Integrations

### Midtrans
- **SDK:** `midtrans/midtrans-php` v2.6
- **Mode:** Sandbox (configurable to production)
- **Features:** Snap token, webhook handling, status mapping

### Cloudinary
- **SDK:** `cloudinary/cloudinary_php` v3.1
- **Usage:** Product image upload, automatic optimization
- **Cleanup:** Destroys old images on product update/delete

---

## Deployment Architecture

### Development
```
┌─────────────┐     ┌─────────────┐
│  Frontend   │────►│  Backend    │
│  :3000      │     │  :8000      │
└─────────────┘     └─────────────┘
                           │
                    ┌──────┴──────┐
                    ▼             ▼
               SQLite DB     External APIs
```

### Production (Recommended)
```
                    ┌─────────────┐
                    │   CDN/Edge  │
                    │  (Vercel)   │
                    └──────┬──────┘
                           │
         ┌─────────────────┼─────────────────┐
         ▼                 ▼                 ▼
┌─────────────┐    ┌─────────────┐    ┌─────────────┐
│  Frontend   │    │   Backend   │    │  Database   │
│  (Static/   │    │  (Laravel   │    │ (PostgreSQL │
│   Standalone)    │   on VPS/   │    │  / MySQL)   │
└─────────────┘    │   Forge)    │    └─────────────┘
                   └──────┬──────┘           │
                          │                  │
                    ┌──────┴──────┐          │
                    ▼             ▼          ▼
               Midtrans       Cloudinary   Redis/Queue
```

---

## Related Notes

- [[README]] — Project overview
- [[PRD]] — Product requirements
- [[API Reference]] — Complete endpoint docs
- [[Database Schema]] — ERD & migrations
- [[Backend Structure]] — Controllers, services, models
- [[Frontend Structure]] — Components, state, hooks
- [[Deployment Guide]] — Production setup