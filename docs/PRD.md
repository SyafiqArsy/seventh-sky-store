# Product Requirements Document (PRD) — Seventh Sky Store

**Version:** 1.0  
**Last Updated:** 2026-08-26  
**Author:** SyafiqArsy  
**Status:** Active Development

---

## 1. Executive Summary

### 1.1 Product Vision
Seventh Sky Store is a premium streetwear e-commerce platform built with a modern headless architecture. It delivers an immersive shopping experience combining high-performance backend APIs with visually striking frontend interactions powered by WebGL.

### 1.2 Target Audience
- **Primary:** Fashion-forward consumers (ages 18-35) in Indonesia seeking premium streetwear
- **Secondary:** Admin users managing product catalog, orders, and analytics

### 1.3 Key Value Propositions
- **Headless Architecture:** Decoupled Laravel API + Next.js frontend for scalability
- **Immersive Visuals:** WebGL galaxy effects, custom cursor, smooth animations
- **Indonesian Market Ready:** IDR pricing, Midtrans payments, localized addresses
- **Complete Admin Suite:** Full CRUD, order state machine, revenue analytics

---

## 2. Functional Requirements

### 2.1 Customer-Facing Features

#### 2.1.1 Authentication & User Management
| ID | Requirement | Priority | Status |
|----|-------------|----------|--------|
| FR-AUTH-01 | User registration with email/password | Must Have | ✅ Implemented |
| FR-AUTH-02 | User login with token-based auth (Laravel Sanctum) | Must Have | ✅ Implemented |
| FR-AUTH-03 | Token refresh and logout (single/all devices) | Must Have | ✅ Implemented |
| FR-AUTH-04 | Rate limiting on auth endpoints (5/min register, 10/min login) | Must Have | ✅ Implemented |
| FR-AUTH-05 | Role-based access (customer/admin) | Must Have | ✅ Implemented |

#### 2.1.2 Product Catalog
| ID | Requirement | Priority | Status |
|----|-------------|----------|--------|
| FR-CAT-01 | Browse products with pagination (15 per page) | Must Have | ✅ Implemented |
| FR-CAT-02 | Search products by name | Must Have | ✅ Implemented |
| FR-CAT-03 | Filter products by category slug | Must Have | ✅ Implemented |
| FR-CAT-04 | Product detail page with image, description, price, stock | Must Have | ✅ Implemented |
| FR-CAT-05 | Category listing and category detail pages | Must Have | ✅ Implemented |
| FR-CAT-06 | Featured/New arrival products on homepage | Should Have | ✅ Implemented |

#### 2.1.3 Shopping Cart
| ID | Requirement | Priority | Status |
|----|-------------|----------|--------|
| FR-CART-01 | Add product to cart with quantity | Must Have | ✅ Implemented |
| FR-CART-02 | Update cart item quantity | Must Have | ✅ Implemented |
| FR-CART-03 | Remove item from cart | Must Have | ✅ Implemented |
| FR-CART-04 | Persist cart in localStorage + sync with server | Must Have | ✅ Implemented |
| FR-CART-05 | Real-time cart count in navbar | Must Have | ✅ Implemented |

#### 2.1.4 Checkout & Payment
| ID | Requirement | Priority | Status |
|----|-------------|----------|--------|
| FR-CHK-01 | Shipping form (recipient, phone, address, city, postal code) | Must Have | ✅ Implemented |
| FR-CHK-02 | Midtrans Snap payment integration | Must Have | ✅ Implemented |
| FR-CHK-03 | Auto-calculate shipping cost | Must Have | ✅ Implemented |
| FR-CHK-04 | Stock reservation on checkout | Must Have | ✅ Implemented |
| FR-CHK-05 | Order creation with pending status | Must Have | ✅ Implemented |
| FR-CHK-06 | Webhook handling for payment confirmation | Must Have | ✅ Implemented |
| FR-CHK-07 | Stock restoration on payment failure/expiry | Must Have | ✅ Implemented |

#### 2.1.5 Order Management (Customer)
| ID | Requirement | Priority | Status |
|----|-------------|----------|--------|
| FR-ORD-01 | Order history with pagination | Must Have | ✅ Implemented |
| FR-ORD-02 | Order detail view with items | Must Have | ✅ Implemented |
| FR-ORD-03 | Order status tracking (pending→processing→shipped→completed) | Must Have | ✅ Implemented |
| FR-ORD-04 | Payment status display (pending/paid/failed/expired) | Must Have | ✅ Implemented |

#### 2.1.6 Visual Experience
| ID | Requirement | Priority | Status |
|----|-------------|----------|--------|
| FR-VIS-01 | WebGL galaxy starfield on collections page (OGL + GLSL) | Must Have | ✅ Implemented |
| FR-VIS-02 | Custom cursor system (explore/pointer modes) | Should Have | ✅ Implemented |
| FR-VIS-03 | Smooth scrolling with Lenis | Should Have | ✅ Implemented |
| FR-VIS-04 | Framer Motion page transitions & hover effects | Should Have | ✅ Implemented |
| FR-VIS-05 | Parallax hero section with carousel | Should Have | ✅ Implemented |
| FR-VIS-06 | Flying posters 3D effect on collections | Could Have | ✅ Implemented |
| FR-VIS-07 | Click ripple effect | Could Have | ✅ Implemented |

### 2.2 Admin Features

#### 2.2.1 Dashboard Analytics
| ID | Requirement | Priority | Status |
|----|-------------|----------|--------|
| FR-ADM-01 | Total products count | Must Have | ✅ Implemented |
| FR-ADM-02 | Total categories count | Must Have | ✅ Implemented |
| FR-ADM-03 | Total orders count | Must Have | ✅ Implemented |
| FR-ADM-04 | Total revenue (paid orders only) | Must Have | ✅ Implemented |

#### 2.2.2 Product Management
| ID | Requirement | Priority | Status |
|----|-------------|----------|--------|
| FR-ADM-05 | List products with search & pagination | Must Have | ✅ Implemented |
| FR-ADM-06 | Create product with Cloudinary image upload | Must Have | ✅ Implemented |
| FR-ADM-07 | Edit product (name, price, stock, description, image, active status) | Must Have | ✅ Implemented |
| FR-ADM-08 | Delete product with Cloudinary cleanup | Must Have | ✅ Implemented |
| FR-ADM-09 | Auto-generate SKU and slug | Must Have | ✅ Implemented |

#### 2.2.3 Category Management
| ID | Requirement | Priority | Status |
|----|-------------|----------|--------|
| FR-ADM-10 | List all categories | Must Have | ✅ Implemented |
| FR-ADM-11 | Create/Update/Delete categories | Must Have | ✅ Implemented |
| FR-ADM-12 | Auto-generate category slug | Must Have | ✅ Implemented |

#### 2.2.4 Order Management
| ID | Requirement | Priority | Status |
|----|-------------|----------|--------|
| FR-ADM-13 | List all orders with search & pagination | Must Have | ✅ Implemented |
| FR-ADM-14 | View order detail with user & items | Must Have | ✅ Implemented |
| FR-ADM-15 | Update order status with state machine validation | Must Have | ✅ Implemented |

#### 2.2.5 Access Control
| ID | Requirement | Priority | Status |
|----|-------------|----------|--------|
| FR-ADM-16 | Admin middleware protecting /admin/* routes | Must Have | ✅ Implemented |
| FR-ADM-17 | Role check on authenticated user | Must Have | ✅ Implemented |

---

## 3. Non-Functional Requirements

### 3.1 Performance
| ID | Requirement | Target |
|----|-------------|--------|
| NFR-PERF-01 | API response time (p95) | < 500ms |
| NFR-PERF-02 | Frontend First Contentful Paint | < 1.5s |
| NFR-PERF-03 | WebGL shader maintains 60fps | ✅ Target met |
| NFR-PERF-04 | Database queries optimized with eager loading | ✅ Implemented |

### 3.2 Security
| ID | Requirement | Implementation |
|----|-------------|----------------|
| NFR-SEC-01 | API authentication via Laravel Sanctum tokens | ✅ Implemented |
| NFR-SEC-02 | Password hashing (bcrypt) | ✅ Implemented |
| NFR-SEC-03 | Rate limiting on auth & checkout endpoints | ✅ Implemented |
| NFR-SEC-04 | Midtrans webhook SHA-512 signature verification | ✅ Implemented |
| NFR-SEC-05 | SQL injection prevention via Eloquent ORM | ✅ Implemented |
| NFR-SEC-06 | XSS prevention via React auto-escaping | ✅ Implemented |
| NFR-SEC-07 | CORS configuration | ✅ Implemented |
| NFR-SEC-08 | Security headers middleware | ✅ Implemented |

### 3.3 Scalability
| ID | Requirement | Approach |
|----|-------------|----------|
| NFR-SCA-01 | Stateless API for horizontal scaling | ✅ Laravel Sanctum |
| NFR-SCA-02 | Database indexing on foreign keys & search columns | ✅ Migrations |
| NFR-SCA-03 | CDN-ready asset delivery via Cloudinary | ✅ Implemented |
| NFR-SCA-04 | SQLite → MySQL/PostgreSQL migration path | ✅ Configurable |

### 3.4 Reliability
| ID | Requirement | Implementation |
|----|-------------|----------------|
| NFR-REL-01 | Order state machine prevents invalid transitions | ✅ Implemented |
| NFR-REL-02 | Stock consistency (decrement on checkout, restore on failure) | ✅ Implemented |
| NFR-REL-03 | Idempotent webhook processing | ✅ Implemented |
| NFR-REL-04 | Database transactions for order creation | ✅ Implemented |

### 3.5 Usability
| ID | Requirement | Implementation |
|----|-------------|----------------|
| NFR-USE-01 | Fully responsive (mobile-first Tailwind CSS) | ✅ Implemented |
| NFR-USE-02 | Accessible form labels & ARIA attributes | ✅ Implemented |
| NFR-USE-03 | Loading states & error toasts | ✅ Implemented |
| NFR-USE-04 | Indonesian locale (IDR currency, address format) | ✅ Implemented |

---

## 4. Technical Architecture

### 4.1 System Components

```
┌─────────────────────────────────────────────────────────┐
│                    FRONTEND (Next.js 16)                │
├─────────────────────────────────────────────────────────┤
│  Route Groups:                                          │
│  ├── (site)/        → Public pages                      │
│  ├── (auth)/        → Login, Register                   │
│  └── (admin)/       → Protected admin pages             │
│                                                         │
│  State Management:                                      │
│  ├── Zustand        → Client state (cart, UI)           │
│  ├── React Query    → Server state (products, orders)   │
│  └── React Context  → Auth, Toast                       │
│                                                         │
│  Visual Effects:                                        │
│  ├── OGL + GLSL     → Galaxy shader                     │
│  ├── Framer Motion  → Animations                        │
│  └── Lenis          → Smooth scroll                     │
└─────────────────────────────────────────────────────────┘
                          │ REST API (JSON)
                          ▼
┌─────────────────────────────────────────────────────────┐
│                    BACKEND (Laravel 13)                 │
├─────────────────────────────────────────────────────────┤
│  Controllers:                                           │
│  ├── AuthController       → Register, Login, Me, Logout │
│  ├── ProductController    → CRUD + public listing       │
│  ├── CategoryController   → CRUD + public listing       │
│  ├── CartController       → Cart CRUD                   │
│  ├── CheckoutController   → Order creation + payment    │
│  ├── OrderController      → History + admin management  │
│  ├── AdminDashboardController → Analytics               │
│  ├── MidtransController   → Webhook handling            │
│  └── UploadController     → Cloudinary test             │
│                                                         │
│  Services:                                              │
│  ├── MidtransService      → Snap token, webhook verify  │
│  ├── CloudinaryService    → Image upload/destroy        │
│  └── CategoryService      → Slug generation, CRUD       │
│                                                         │
│  Middleware:                                            │
│  ├── auth:sanctum         → Token validation            │
│  ├── admin                → Role check                  │
│  └── SecurityHeaders      → CSP, HSTS, etc.             │
└─────────────────────────────────────────────────────────┘
                          │
              ┌───────────┴───────────────┐
              ▼                           ▼
       ┌────────────┐              ┌─────────────┐
       │  SQLite    │              │  External   │
       │  Database  │              │  Services   │
       └────────────┘              ├─────────────┤
                                   │  Midtrans   │
                                   │  Cloudinary │
                                   └─────────────┘
```

### 4.2 Data Models

#### 4.2.1 Core Entities
- **User:** id, name, email, password, role (admin/customer)
- **Category:** id, name, slug
- **Product:** id, category_id, name, slug, description, sku, price, stock, image, image_public_id, is_active
- **Cart:** id, user_id (unique)
- **CartItem:** id, cart_id, product_id, quantity (unique cart+product)
- **Order:** id, user_id, order_number, recipient_name, phone, address, city, postal_code, total_price, shipping_cost, grand_total, payment_status, order_status, midtrans_*, paid_at
- **OrderItem:** id, order_id, product_id, product_name, product_price, quantity, subtotal

### 4.3 API Contract

#### 4.3.1 Response Format
```json
{
  "success": boolean,
  "message": "string (optional)",
  "data": "object|array",
  "meta": "object (pagination info, optional)"
}
```

#### 4.3.2 Error Format
```json
{
  "success": false,
  "message": "string",
  "errors": "object (validation errors, optional)"
}
```

---

## 5. User Flows

### 5.1 Happy Path: Purchase Flow
```
1. User lands on Homepage
2. Browse Products → Filter by Category → View Product Detail
3. Add to Cart (quantity selector)
4. View Cart → Proceed to Checkout
5. Fill Shipping Form → Submit
6. Midtrans Snap Popup Opens
7. User Completes Payment (VA, E-Wallet, Card, etc.)
8. Midtrans Webhook → Backend verifies signature
9. Order status → paid, order_status → processing
10. Stock decremented
11. User redirected to Order Confirmation
12. Order visible in Order History
```

### 5.2 Admin Order Management Flow
```
1. Admin logs in → redirected to /admin
2. View Dashboard stats
3. Navigate to Orders → View all orders
4. Click order → View detail
5. Update status via dropdown (validated by state machine)
   - pending → processing → shipped → completed
   - Any status → cancelled (if not paid)
6. Changes reflected in real-time
```

---

## 6. Database Migrations History

| Migration | Description |
|-----------|-------------|
| 0001_01_01_000000 | Create users table |
| 0001_01_01_000001 | Create cache table |
| 0001_01_01_000002 | Create jobs table |
| 2026_06_02_000003 | Create personal_access_tokens (Sanctum) |
| 2026_06_02_000004 | Add role to users table |
| 2026_06_02_000005 | Create categories table |
| 2026_06_02_000006 | Create carts table |
| 2026_06_02_000007 | Create products table |
| 2026_06_02_000008 | Create orders table |
| 2026_06_02_162538 | Create cart_items table |
| 2026_06_02_162539 | Create order_items table |
| 2026_06_02_180704 | Add unique user to carts |
| 2026_06_02_180740 | Add unique cart+product to cart_items |
| 2026_06_03_023752 | Add SKU to products |
| 2026_06_03_030409 | Add Cloudinary fields to products |
| 2026_06_03_032123 | Add total fields to orders |
| 2026_06_10_041926 | Add payment history to orders |

---

## 7. Configuration & Environment

### 7.1 Backend (.env)
```env
APP_NAME="Seventh Sky Store"
APP_ENV=local
APP_KEY=base64:...
APP_DEBUG=true
APP_URL=http://localhost:8000

DB_CONNECTION=sqlite
DB_DATABASE=database/database.sqlite

MIDTRANS_SERVER_KEY=
MIDTRANS_CLIENT_KEY=
MIDTRANS_IS_PRODUCTION=false

CLOUDINARY_CLOUD_NAME=
CLOUDINARY_API_KEY=
CLOUDINARY_API_SECRET=

SANCTUM_STATEFUL_DOMAINS=localhost:3000
SESSION_DOMAIN=localhost
```

### 7.2 Frontend (.env.local)
```env
NEXT_PUBLIC_API_URL=http://localhost:8000/api
NEXT_PUBLIC_MIDTRANS_CLIENT_KEY=
```

---

## 8. Testing Strategy

### 8.1 Backend Testing
- **Unit Tests:** Services (MidtransService, CloudinaryService, CategoryService)
- **Feature Tests:** API endpoints (auth, products, cart, checkout, orders, admin)
- **Database Tests:** Migration rollback, factory seeding

### 8.2 Frontend Testing
- **Component Tests:** React Testing Library for UI components
- **Integration Tests:** Cart flow, checkout flow, auth flow
- **E2E Tests:** Playwright/Cypress for critical user journeys

### 8.3 Manual Testing Checklist
- [ ] Register → Login → Browse → Add to Cart → Checkout → Payment → Order Confirmation
- [ ] Admin login → Dashboard → CRUD Products → CRUD Categories → Order Management
- [ ] Mobile responsiveness on all pages
- [ ] WebGL performance on low-end devices
- [ ] Midtrans sandbox payment flow
- [ ] Webhook simulation (success, failure, expiry)

---

## 9. Deployment

### 9.1 Backend Deployment
- **Platform:** Laravel Forge / VPS / Shared Hosting
- **Requirements:** PHP 8.3+, Composer, SQLite/MySQL/PostgreSQL
- **Commands:**
  ```bash
  composer install --optimize-autoloader --no-dev
  php artisan config:cache
  php artisan route:cache
  php artisan view:cache
  php artisan migrate --force
  ```

### 9.2 Frontend Deployment
- **Platform:** Vercel (recommended) / Netlify / VPS
- **Build Command:** `npm run build`
- **Output:** Next.js standalone or static export
- **Environment Variables:** Set in deployment platform

### 9.3 Database
- **Development:** SQLite (file-based)
- **Production:** PostgreSQL (recommended) or MySQL
- **Migrations:** Run on deploy via CI/CD

---

## 10. Future Enhancements (Backlog)

### 10.1 High Priority
- [ ] Email notifications (order confirmation, status updates)
- [ ] Product variants (size, color)
- [ ] Wishlist/Favorites
- [ ] Product reviews & ratings
- [ ] Discount codes & promotions

### 10.2 Medium Priority
- [ ] Multi-language support (EN/ID)
- [ ] Dark mode toggle
- [ ] Advanced analytics (conversion funnel, AOV)
- [ ] Export orders to CSV/Excel
- [ ] Bulk product import/export

### 10.3 Low Priority
- [ ] PWA support (offline browsing, push notifications)
- [ ] Social login (Google, Facebook)
- [ ] Live chat integration
- [ ] Recommendation engine
- [ ] Multi-currency support

---

## 11. Risks & Mitigations

| Risk | Likelihood | Impact | Mitigation |
|------|------------|--------|------------|
| Midtrans API changes | Low | High | SDK version pinning, integration tests |
| Cloudinary pricing changes | Low | Medium | Monitor usage, implement fallback |
| WebGL performance on mobile | Medium | Medium | Graceful degradation, disable option |
| SQLite concurrency limits | Medium | Low | Plan migration to PostgreSQL |
| Third-party dependency vulnerabilities | Medium | High | Dependabot, regular audits |
| SEO for Next.js App Router | Medium | Medium | Implement metadata, sitemap, robots.txt |

---

## 12. Glossary

| Term | Definition |
|------|------------|
| **Headless** | Backend and frontend decoupled, communicate via API |
| **Sanctum** | Laravel's token-based authentication package |
| **Midtrans** | Indonesian payment gateway (GoPay, ShopeePay, Bank Transfer, etc.) |
| **Snap** | Midtrans's pre-built payment popup UI |
| **OGL** | Lightweight WebGL library for 3D graphics |
| **GLSL** | OpenGL Shading Language for GPU shaders |
| **Zustand** | Minimal state management for React |
| **React Query** | Server state management (caching, sync) |
| **State Machine** | Enforced valid transitions between order statuses |

---

## 13. Appendix

### 13.1 Related Documents
- [[README]] — Project overview & setup guide
- [[Architecture]] — Technical architecture deep-dive
- [[API Reference]] — Complete API endpoint documentation
- [[Database Schema]] — ERD & migration details
- [[Deployment Guide]] — Production deployment steps

### 13.2 Links
- **Repository:** https://github.com/your-username/seventh-sky-store
- **Midtrans Docs:** https://docs.midtrans.com
- **Cloudinary Docs:** https://cloudinary.com/documentation
- **Laravel 13 Docs:** https://laravel.com/docs/13.x
- **Next.js 16 Docs:** https://nextjs.org/docs

---

*This PRD is a living document. Update it as features are added, changed, or removed.*