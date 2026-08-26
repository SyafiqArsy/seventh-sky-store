# Frontend Structure — Seventh Sky Store

#frontend #nextjs #react

---

## Directory Structure

```
frontend/
├── app/                          # Next.js App Router
│   ├── (site)/                   # Public pages (Route Group)
│   │   ├── page.tsx              # Homepage
│   │   ├── products/
│   │   │   ├── page.tsx          # Product listing
│   │   │   └── [slug]/page.tsx   # Product detail
│   │   ├── collections/page.tsx  # Collections (WebGL)
│   │   ├── cart/page.tsx         # Shopping cart
│   │   ├── checkout/page.tsx     # Checkout form
│   │   ├── orders/
│   │   │   ├── page.tsx          # Order history
│   │   │   └── [id]/page.tsx     # Order detail
│   │   └── contact/page.tsx      # Contact page
│   ├── (auth)/                   # Auth pages (Route Group)
│   │   ├── layout.tsx            # Auth layout
│   │   ├── login/page.tsx        # Login
│   │   └── register/page.tsx     # Register
│   ├── (admin)/                  # Admin pages (Route Group)
│   │   ├── layout.tsx            # Admin layout + sidebar
│   │   ├── page.tsx              # Dashboard
│   │   ├── products/
│   │   │   ├── page.tsx          # Product list
│   │   │   ├── create/page.tsx   # Create product
│   │   │   └── [id]/edit/page.tsx # Edit product
│   │   ├── categories/
│   │   │   ├── page.tsx          # Category list
│   │   │   ├── create/page.tsx   # Create category
│   │   │   └── [id]/edit/page.tsx # Edit category
│   │   └── orders/
│   │       ├── page.tsx          # Order list
│   │       └── [id]/page.tsx     # Order detail
│   ├── layout.tsx                # Root layout
│   └── globals.css               # Global styles (Tailwind)
├── src/
│   ├── components/               # React Components
│   │   ├── admin/                # Admin components
│   │   ├── auth/                 # Auth forms
│   │   ├── cart/                 # Cart components
│   │   ├── checkout/             # Checkout components
│   │   ├── collections/          # Visual effects
│   │   ├── home/                 # Homepage sections
│   │   ├── layout/               # Layout components
│   │   ├── order/                # Order components
│   │   ├── product/              # Product components
│   │   └── ui/                   # UI primitives
│   ├── context/                  # React Context Providers
│   │   ├── AuthContext.tsx       # Auth state
│   │   ├── CartContext.tsx       # Cart state (legacy)
│   │   └── ToastContext.tsx      # Toast notifications
│   ├── hooks/                    # Custom Hooks
│   ├── lib/                      # Utilities & API
│   │   ├── api.ts                # Server-side fetch functions
│   │   ├── auth.ts               # Client-side auth helpers
│   │   ├── cart.ts               # Cart API
│   │   ├── checkout.ts           # Checkout API
│   │   ├── order.ts              # Order API
│   │   ├── admin.ts              # Admin API
│   │   ├── cursorLoading.ts      # Cursor loading state
│   │   └── loadMidtrans.ts       # Midtrans script loader
│   ├── stores/                   # Zustand Stores
│   │   └── useCartStore.ts       # Cart state management
│   └── types/                    # TypeScript Types
│       ├── product.ts            # Product types
│       └── midtrans.d.ts         # Midtrans types
├── public/                       # Static Assets
├── package.json
├── tsconfig.json
├── next.config.ts
└── tailwind.config.ts
```

---

## Route Groups Explained

### (site) — Public Pages
No authentication required. Accessible to all visitors.

```
/
├── page.tsx                    → Homepage with Hero, Featured, New Arrivals
├── products/
│   ├── page.tsx                → Product grid with search/filter
│   └── [slug]/page.tsx         → Product detail with add to cart
├── collections/page.tsx        → WebGL Galaxy + Flying Posters
├── cart/page.tsx               → CartView with quantity controls
├── checkout/page.tsx           → Shipping form + Midtrans
├── orders/
│   ├── page.tsx                → Order history (requires auth)
│   └── [id]/page.tsx           → Order detail
└── contact/page.tsx            → Contact form + FAQ
```

### (auth) — Authentication Pages
Guest-only pages. Redirects authenticated users to home.

```
(auth)/
├── layout.tsx                  → Centered card layout
├── login/page.tsx              → LoginForm + register link
└── register/page.tsx           → RegisterForm + login link
```

### (admin) — Admin Pages
Requires admin role. Protected by `AdminGuard`.

```
(admin)/
├── layout.tsx                  → AdminSidebar + Header
├── page.tsx                    → Dashboard stats
├── products/
│   ├── page.tsx                → AdminProductsView (table)
│   ├── create/page.tsx         → CreateProductForm
│   └── [id]/edit/page.tsx      → EditProductForm
├── categories/
│   ├── page.tsx                → AdminCategoriesView
│   ├── create/page.tsx         → CreateCategoryForm
│   └── [id]/edit/page.tsx      → EditCategoryForm
└── orders/
    ├── page.tsx                → AdminOrders (table + status)
    └── [id]/page.tsx           → AdminOrderDetail
```

---

## State Management

### Zustand Store (Client State)

```typescript
// src/stores/useCartStore.ts
interface CartStore {
  items: CartItem[];
  isCartOpen: boolean;
  addItem: (product: Product, qty: number) => Promise<void>;
  updateQuantity: (id: number, qty: number) => Promise<void>;
  removeItem: (id: number) => Promise<void>;
  openCart: () => void;
  closeCart: () => void;
  toggleCart: () => void;
  hydrate: () => Promise<void>;  // Sync from server
}
```

**Persisted to localStorage** for instant UI, synced with server on auth.

### React Query (Server State)

```typescript
// Used throughout components via useQuery/useMutation
// Cache keys: ['products'], ['categories'], ['cart'], ['orders'], etc.
```

**Configuration:** `staleTime: 5 minutes`, `refetchOnWindowFocus: false`

### React Context (Global State)

| Context | Purpose |
|---------|---------|
| `AuthContext` | User, token, login/register/logout, isAdmin |
| `ToastContext` | Global toast notifications (success/error/info) |

---

## Component Reference

### Admin Components
`src/components/admin/`

| Component | Description |
|-----------|-------------|
| `AdminDashboardView` | Stats cards, revenue chart placeholder |
| `AdminProductsView` | Product table with search, pagination, actions |
| `AdminCategoriesView` | Category list with inline create/edit/delete |
| `AdminOrders` | Order table with status dropdown |
| `AdminOrderDetail` | Full order view + status update |
| `AdminSidebar` | Navigation with active state |
| `CreateProductForm` | Multi-part form + image upload |
| `EditProductForm` | Pre-filled form + image replace |
| `CreateCategoryForm` | Simple name input |
| `EditCategoryForm` | Simple name input |
| `AdminGuard` | HOC for admin route protection |

### Auth Components
`src/components/auth/`

| Component | Description |
|-----------|-------------|
| `LoginForm` | Email/password + validation (React Hook Form + Zod) |
| `RegisterForm` | Name/email/password/confirm + validation |

### Cart Components
`src/components/cart/`

| Component | Description |
|-----------|-------------|
| `CartView` | Full cart page: items, quantities, subtotal, checkout button |

### Checkout Components
`src/components/checkout/`

| Component | Description |
|-----------|-------------|
| `CheckoutForm` | Shipping form + order summary + Midtrans integration |

### Collections Components
`src/components/collections/`

| Component | Description |
|-----------|-------------|
| `Galaxy` | WebGL starfield (OGL + GLSL) |
| `FlyingPosters` | 3D animated posters (Framer Motion) |

### Home Components
`src/components/home/`

| Component | Description |
|-----------|-------------|
| `HeroSection` | Parallax hero with carousel |
| `HeroCarousel` | Embla carousel with auto-play |
| `FeaturedCollection` | Featured products grid |
| `BrandStatement` | Brand tagline section |
| `NewArrival` | Latest products |
| `TypographyShowcase` | Decorative typography |
| `BrandIntro` | Brand story section |

### Layout Components
`src/components/layout/`

| Component | Description |
|-----------|-------------|
| `Navbar` | Logo, nav links, cart button, user menu |
| `Footer` | Links, newsletter, social |
| `ConditionalFooter` | Shows/hides based on route |
| `CustomCursor` | Explore/pointer cursor modes |

### Order Components
`src/components/order/`

| Component | Description |
|-----------|-------------|
| `OrdersView` | Customer order history list |
| `OrderDetailView` | Customer order detail |

### Product Components
`src/components/product/`

| Component | Description |
|-----------|-------------|
| `ProductCard` | Product image, name, price, add to cart |
| `ProductsView` | Grid wrapper with search/filter/pagination |
| `QuantitySelector` | +/- buttons with stock limit |
| `ProductActions` | Add to cart button + wishlist (future) |
| `loading.tsx` | Skeleton loaders |

### UI Primitives
`src/components/ui/`

| Component | Description |
|-----------|-------------|
| `Toast` | Animated toast notifications |
| `ConfirmModal` | Confirmation dialog |
| `ClickRipple` | Material-style click ripple |
| `SmoothScroll` | Lenis initialization |

---

## API Layer

### Server-Side (fetch)

```typescript
// src/lib/api.ts
export async function getProducts(params?) { ... }
export async function getProduct(slug: string) { ... }
export async function getCategories() { ... }
export async function getLatestProducts(limit = 3) { ... }
```

**Used in:** Server Components, `generateStaticParams`, `metadata`

### Client-Side (Axios)

```typescript
// src/lib/auth.ts, cart.ts, checkout.ts, order.ts, admin.ts
const api = axios.create({ baseURL: process.env.NEXT_PUBLIC_API_URL });

// Interceptors for auth token, 401 handling
```

**Used in:** Client Components, event handlers, mutations

---

## TypeScript Types

### Product Types
```typescript
// src/types/product.ts
export interface Product {
  id: number;
  category_id: number;
  name: string;
  slug: string;
  description: string | null;
  sku: string;
  price: string;        // Decimal as string
  stock: number;
  image: string | null;
  image_public_id: string | null;
  is_active: boolean;
  category: Category;
}

export interface Category {
  id: number;
  name: string;
  slug: string;
}
```

### Midtrans Types
```typescript
// src/types/midtrans.d.ts
declare global {
  interface Window {
    MidtransNewWindow: boolean;
    snap: {
      pay: (token: string, callbacks: SnapCallbacks) => void;
    };
  }
}

interface SnapCallbacks {
  onSuccess?: () => void;
  onPending?: () => void;
  onError?: () => void;
  onClose?: () => void;
}
```

---

## Styling

### Tailwind CSS 4
- **Config:** `tailwind.config.ts` (minimal, uses CSS variables)
- **Import:** `@import "tailwindcss";` in `globals.css`
- **Custom Properties:** CSS variables for colors, spacing

### Design Tokens
```css
/* globals.css */
:root {
  --color-primary: #1a1a2e;
  --color-secondary: #16213e;
  --color-accent: #e94560;
  --color-background: #0f0f1a;
  --color-surface: #1a1a2e;
  --color-text: #ffffff;
  --color-text-muted: #a0a0b0;
}
```

---

## Key Libraries

| Library | Version | Purpose |
|---------|---------|---------|
| Next.js | 16.2.7 | React framework (App Router) |
| React | 19.2.4 | UI library |
| TypeScript | 5 | Type safety |
| Tailwind CSS | 4 | Utility-first CSS |
| Framer Motion | 12.42.0 | Animations |
| Zustand | 5.0.14 | Client state |
| TanStack React Query | 5.101.0 | Server state |
| React Hook Form | 7.77.0 | Forms |
| Zod | 4.4.3 | Validation |
| OGL | 1.0.11 | WebGL |
| Lenis | 1.3.25 | Smooth scroll |
| Axios | 1.17.0 | HTTP client |
| Lucide React | 1.17.0 | Icons |
| Embla Carousel | 8.6.0 | Carousel |

---

## Related Notes

- [[Architecture#Frontend Architecture]] — Route groups, state strategy
- [[API Reference]] — Endpoints used by frontend
- [[Authentication]] — AuthContext, protected routes
- [[Shopping Cart]] — Cart store, sync logic
- [[Checkout & Payment]] — CheckoutForm, Midtrans
- [[Visual Effects]] — Galaxy, CustomCursor, animations
- [[Product Catalog]] — Product components, data fetching
- [[Admin Dashboard]] — Admin pages, components