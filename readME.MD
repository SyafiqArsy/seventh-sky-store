<br />

<div align="center">

  <!-- Logo / Title -->
  <h1>☁️ SEVENTH SKY STORE</h1>
  <p><strong>Elevate Your Style.</strong></p>
  <p>A full-stack e-commerce platform for a premium streetwear brand — built with <strong>Laravel</strong> & <strong>Next.js</strong>, featuring Midtrans payment integration, WebGL visual effects, and a complete admin dashboard.</p>

  <br />

  <!-- Tech Badges -->
  <img src="https://img.shields.io/badge/Laravel-13-FF2D20?style=for-the-badge&logo=laravel&logoColor=white" alt="Laravel" />
  <img src="https://img.shields.io/badge/Next.js-16-000000?style=for-the-badge&logo=next.js&logoColor=white" alt="Next.js" />
  <img src="https://img.shields.io/badge/React-19-61DAFB?style=for-the-badge&logo=react&logoColor=black" alt="React" />
  <img src="https://img.shields.io/badge/Tailwind_CSS-4-06B6D4?style=for-the-badge&logo=tailwindcss&logoColor=white" alt="Tailwind CSS" />
  <img src="https://img.shields.io/badge/TypeScript-5-3178C6?style=for-the-badge&logo=typescript&logoColor=white" alt="TypeScript" />
  <img src="https://img.shields.io/badge/Midtrans-Payment-00CED1?style=for-the-badge&logoColor=white" alt="Midtrans" />

</div>

<br />

---

## 📖 Table of Contents

- [About](#-about)
- [Key Features](#-key-features)
- [Tech Stack](#-tech-stack)
- [Architecture](#-architecture)
- [Database Schema](#-database-schema)
- [API Endpoints](#-api-endpoints)
- [Getting Started](#-getting-started)
- [Project Structure](#-project-structure)
- [Screenshots](#-screenshots)
- [License](#-license)

---

## 🚀 About

**Seventh Sky Store** is a modern, full-stack e-commerce web application designed for a premium streetwear fashion brand. Built with a **headless architecture**, the backend (Laravel REST API) and frontend (Next.js App Router) are fully decoupled, communicating through JSON endpoints.

The platform delivers a seamless shopping experience — from browsing products with immersive WebGL visual effects, through a smooth checkout process with **Midtrans Snap** payment gateway, to real-time order tracking. An integrated admin dashboard provides complete control over products, categories, orders, and revenue analytics.

> 🇮🇩 Built with the Indonesian market in mind — prices in Rupiah, Midtrans payment gateway, and localized address format.

---

## ✨ Key Features

### 🛍️ Customer Experience
- **Product Catalog** — Browse products with search and category filtering
- **Product Detail** — Rich product pages with image galleries and quantity selectors
- **Shopping Cart** — Add, update, and remove items with real-time cart management (Zustand + localStorage sync)
- **Secure Checkout** — Shipping form with Midtrans Snap payment popup
- **Order History** — Track order status (pending → processing → shipped → completed)
- **User Authentication** — Register, login, and token-based session management (Laravel Sanctum)

### 🎨 Visual & UI
- **WebGL Galaxy Effects** — Immersive 3D starfield shader on the Collections page (built with OGL + custom GLSL)
- **Parallax Animations** — Smooth parallax scrolling on the homepage hero
- **Custom Cursor System** — Interactive cursor with explore and pointer modes
- **Smooth Scrolling** — Butter-smooth scroll behavior powered by Lenis
- **Framer Motion Animations** — Fluid page transitions and hover effects
- **Responsive Design** — Fully responsive across all device sizes
- **Flying Posters** — Animated 3D poster effects on collections page
- **Hero Carousel** — Auto-rotating hero section with embla-carousel

### 🔧 Admin Dashboard
- **Revenue Analytics** — Real-time stats for products, categories, orders, and total revenue
- **Product Management** — Full CRUD with Cloudinary image upload and stock management
- **Category Management** — Create, edit, and delete product categories
- **Order Management** — View all orders, update status with state machine validation
- **Role-Based Access** — Admin-only routes with middleware protection

### 💳 Payment & Orders
- **Midtrans Snap Integration** — Full payment flow with Snap popup
- **Webhook Handling** — Server-to-server payment verification (SHA-512 signature)
- **Stock Management** — Auto-decrement on checkout, restore on payment failure/expiration
- **Order State Machine** — Enforced status transitions with validation (pending → processing → shipped → completed, with cancellation at any stage)

---

## 🛠️ Tech Stack

### Backend

| Technology | Purpose |
|---|---|
| **Laravel 13** | PHP framework for the REST API |
| **Laravel Sanctum** | Token-based API authentication |
| **SQLite** | Lightweight database (easily switchable to MySQL/PostgreSQL) |
| **Midtrans PHP SDK** | Payment gateway integration |
| **Cloudinary PHP SDK** | Cloud-based image hosting and management |
| **Eloquent Sluggable** | Automatic URL slug generation |

### Frontend

| Technology | Purpose |
|---|---|
| **Next.js 16** | React framework with App Router |
| **React 19** | UI component library |
| **TypeScript** | Type-safe JavaScript |
| **Tailwind CSS 4** | Utility-first styling |
| **Framer Motion** | Animation library |
| **Zustand** | Lightweight state management |
| **TanStack React Query** | Server state & data fetching |
| **React Hook Form + Zod** | Form handling & validation |
| **OGL** | WebGL library for 3D effects |
| **Lenis** | Smooth scrolling |
| **Axios** | HTTP client |
| **Lucide React** | Icon library |
| **Embla Carousel** | Carousel/slider component |

---

## 🏗️ Architecture

```
┌─────────────────────────────────────────────────────────┐
│                      FRONTEND                           │
│                  Next.js 16 (App Router)                 │
│  ┌──────────┐  ┌──────────┐  ┌──────────┐  ┌────────┐  │
│  │  (site)   │  │  (auth)  │  │ (admin)  │  │  API   │  │
│  │  Pages    │  │  Pages   │  │  Pages   │  │ Client │  │
│  └──────────┘  └──────────┘  └──────────┘  └───┬────┘  │
│                                                  │       │
│         Zustand · React Query · Framer Motion     │       │
└──────────────────────────────────┬───────────────┘       │
                                   │ REST API (JSON)
                                   ▼
┌─────────────────────────────────────────────────────────┐
│                      BACKEND                            │
│                   Laravel 13 API                         │
│  ┌──────────┐  ┌──────────┐  ┌──────────┐  ┌────────┐  │
│  │  Auth     │  │ Products │  │  Orders  │  │ Admin  │  │
│  │Controller │  │Controller│  │Controller│  │Dashboard│  │
│  └──────────┘  └──────────┘  └──────────┘  └────────┘  │
│                                                  │       │
│         Sanctum Auth · Admin Middleware            │       │
└───────────┬──────────────────┬───────────────────┘       │
            │                  │
            ▼                  ▼
     ┌────────────┐    ┌─────────────┐
     │   SQLite   │    │  External   │
     │  Database  │    │  Services   │
     └────────────┘    ├─────────────┤
                       │  Midtrans   │
                       │  Cloudinary │
                       └─────────────┘
```

---

## 🗄️ Database Schema

```
┌──────────────┐     ┌──────────────┐     ┌──────────────┐
│    users      │     │  categories  │     │   products   │
├──────────────┤     ├──────────────┤     ├──────────────┤
│ id           │     │ id           │     │ id           │
│ name         │     │ name         │◄────│ category_id  │
│ email        │     │ slug         │     │ name         │
│ password     │     └──────────────┘     │ slug         │
│ role         │                           │ description  │
│ (admin/      │                           │ sku          │
│  customer)   │                           │ price        │
└──────┬───────┘                           │ stock        │
       │                                   │ image        │
       │                                   │ image_public_id│
       ▼                                   │ is_active    │
┌──────────────┐     ┌──────────────┘     └──────────────┘
│    carts      │     ┌──────────────┐
├──────────────┤     │  cart_items  │
│ id           │◄────│ cart_id      │
│ user_id      │     │ product_id   │
└──────────────┘     │ quantity     │
                     └──────────────┘

┌──────────────┐     ┌──────────────┐
│   orders      │     │ order_items  │
├──────────────┤     ├──────────────┤
│ id           │◄────│ order_id     │
│ user_id      │     │ product_id   │
│ order_number │     │ product_name │
│ recipient    │     │ product_price│
│ phone        │     │ quantity     │
│ address      │     │ subtotal     │
│ city         │     └──────────────┘
│ postal_code  │
│ total_price  │
│ shipping_cost│
│ grand_total  │
│ payment_status│
│ order_status │
│ midtrans_*   │
│ paid_at      │
└──────────────┘
```

---

## 🔌 API Endpoints

### Public

| Method | Endpoint | Description |
|---|---|---|
| `POST` | `/api/register` | Register a new account |
| `POST` | `/api/login` | Login and receive token |
| `GET` | `/api/products` | List active products (search, filter, pagination) |
| `GET` | `/api/products/{slug}` | Get product by slug |
| `GET` | `/api/categories` | List all categories |
| `GET` | `/api/categories/{slug}` | Get category by slug |
| `POST` | `/api/midtrans/notification` | Payment webhook |

### Authenticated (requires `Bearer` token)

| Method | Endpoint | Description |
|---|---|---|
| `GET` | `/api/me` | Get current user profile |
| `POST` | `/api/logout` | Revoke current token |
| `POST` | `/api/logout-all` | Revoke all tokens |
| `GET` | `/api/cart` | Get user's cart |
| `POST` | `/api/cart/items` | Add item to cart |
| `PUT` | `/api/cart/items/{id}` | Update cart item quantity |
| `DELETE` | `/api/cart/items/{id}` | Remove item from cart |
| `POST` | `/api/checkout` | Create order & get payment token |
| `GET` | `/api/orders` | Get order history (paginated) |
| `GET` | `/api/orders/{id}` | Get order detail |

### Admin (requires `admin` role)

| Method | Endpoint | Description |
|---|---|---|
| `GET` | `/api/admin/dashboard` | Dashboard statistics |
| `GET` | `/api/admin/categories` | List all categories (admin) |
| `POST` | `/api/admin/categories` | Create category |
| `PUT` | `/api/admin/categories/{id}` | Update category |
| `DELETE` | `/api/admin/categories/{id}` | Delete category |
| `GET` | `/api/admin/products` | List all products (admin, paginated) |
| `GET` | `/api/admin/products/{id}` | Get product detail (admin) |
| `POST` | `/api/admin/products` | Create product |
| `PUT` | `/api/admin/products/{id}` | Update product |
| `DELETE` | `/api/admin/products/{id}` | Delete product |
| `GET` | `/api/admin/orders` | List all orders (admin, paginated) |
| `GET` | `/api/admin/orders/{id}` | Get order detail (admin) |
| `PUT` | `/api/admin/orders/{id}/status` | Update order status |
| `POST` | `/api/admin/upload-test` | Test Cloudinary upload |

---

## 🏁 Getting Started

### Prerequisites

- **PHP** >= 8.3
- **Composer** >= 2.x
- **Node.js** >= 18.x
- **npm** >= 9.x

### 1. Clone the Repository

```bash
git clone https://github.com/your-username/seventh-sky-store.git
cd seventh-sky-store
```

### 2. Backend Setup

```bash
cd backend

# Install dependencies
composer install

# Create environment file
cp .env.example .env

# Generate application key
php artisan key:generate

# Create SQLite database
touch database/database.sqlite

# Run migrations
php artisan migrate

# Seed database (optional)
php artisan db:seed

# Start development server
php artisan serve
```

#### Backend `.env` Configuration

```env
DB_CONNECTION=sqlite

MIDTRANS_SERVER_KEY=your_midtrans_server_key
MIDTRANS_CLIENT_KEY=your_midtrans_client_key
MIDTRANS_IS_PRODUCTION=false

CLOUDINARY_CLOUD_NAME=your_cloud_name
CLOUDINARY_API_KEY=your_api_key
CLOUDINARY_API_SECRET=your_api_secret
```

### 3. Frontend Setup

```bash
cd frontend

# Install dependencies
npm install

# Create environment file
cp .env.example .env.local

# Start development server
npm run dev
```

#### Frontend `.env.local` Configuration

```env
NEXT_PUBLIC_API_URL=http://localhost:8000/api
NEXT_PUBLIC_MIDTRANS_CLIENT_KEY=your_midtrans_client_key
```

### 4. Access the Application

| URL | Description |
|---|---|
| `http://localhost:3000` | Frontend (Next.js) |
| `http://localhost:8000` | Backend API (Laravel) |

---

## 📁 Project Structure

```
seventh-sky-store/
├── backend/                        # Laravel 13 API
│   ├── app/
│   │   ├── Http/
│   │   │   ├── Controllers/Api/   # API controllers (Auth, Product, Category, Cart, Order, Checkout, AdminDashboard, Midtrans, Upload)
│   │   │   ├── Middleware/        # AdminMiddleware, SecurityHeaders
│   │   │   └── Requests/          # Form requests for validation
│   │   ├── Models/                # Eloquent models (User, Category, Product, Cart, CartItem, Order, OrderItem)
│   │   ├── Services/              # Business logic (MidtransService, CloudinaryService, CategoryService)
│   │   └── Providers/             # AppServiceProvider
│   ├── database/
│   │   ├── migrations/            # Database schema (12 migrations)
│   │   ├── factories/             # Model factories
│   │   └── seeders/               # Database seeders
│   ├── routes/
│   │   └── api.php                # API route definitions
│   └── config/
│       └── midtrans.php           # Midtrans configuration
│
├── frontend/                       # Next.js 16 App
│   ├── app/
│   │   ├── (site)/                # Public pages (home, products, product detail, collections, cart, checkout, orders, contact)
│   │   ├── (auth)/                # Auth pages (login, register)
│   │   └── (admin)/               # Admin pages (dashboard, products CRUD, categories CRUD, orders)
│   ├── src/
│   │   ├── components/
│   │   │   ├── admin/             # Admin components (Dashboard, Products, Categories, Orders, Sidebar, Forms)
│   │   │   ├── auth/              # Login/Register forms
│   │   │   ├── cart/              # CartView
│   │   │   ├── checkout/          # CheckoutForm
│   │   │   ├── collections/       # Galaxy (WebGL), FlyingPosters
│   │   │   ├── home/              # HeroSection, HeroCarousel, FeaturedCollection, BrandStatement, NewArrival, TypographyShowcase, BrandIntro
│   │   │   ├── layout/            # Navbar, Footer, ConditionalFooter, CustomCursor
│   │   │   ├── order/             # OrdersView, OrderDetailView
│   │   │   ├── product/           # ProductCard, ProductsView, QuantitySelector, ProductActions
│   │   │   └── ui/                # Toast, ConfirmModal, ClickRipple, SmoothScroll
│   │   ├── context/               # AuthContext, CartContext, ToastContext
│   │   ├── hooks/                 # Custom hooks
│   │   ├── lib/                   # API client, auth, cart, checkout, order, admin, cursorLoading, loadMidtrans
│   │   ├── stores/                # Zustand stores
│   │   └── types/                 # TypeScript types (product, midtrans)
│   └── public/                    # Static assets
│
├── docs/                          # Documentation
│   ├── PRD.md                     # Product Requirements Document
│   └── SeventhSkyStore-Obsidian/  # Obsidian vault
│
└── README.md                      # This file
```

---

## 📸 Screenshots

> _Add screenshots of the application here to showcase the UI._

| Homepage | Collections | Product Detail |
|:---:|:---:|:---:|
| ![Home](screenshots/home.png) | ![Collections](screenshots/collections.png) | ![Product](screenshots/product.png) |

| Cart | Checkout | Admin Dashboard |
|:---:|:---:|:---:|
| ![Cart](screenshots/cart.png) | ![Checkout](screenshots/checkout.png) | ![Admin](screenshots/admin.png) |

---

## 📄 License

This project is open-sourced under the [MIT License](https://opensource.org/licenses/MIT).

---

<div align="center">

  <p>Built with ❤️ by <strong>SyafiqArsy</strong></p>

  <p>
    <img src="https://img.shields.io/badge/Laravel-FF2D20?style=for-the-badge&logo=laravel&logoColor=white" />
    <img src="https://img.shields.io/badge/Next.js-000000?style=for-the-badge&logo=next.js&logoColor=white" />
    <img src="https://img.shields.io/badge/React-61DAFB?style=for-the-badge&logo=react&logoColor=black" />
    <img src="https://img.shields.io/badge/Tailwind-06B6D4?style=for-the-badge&logo=tailwindcss&logoColor=white" />
  </p>

</div>