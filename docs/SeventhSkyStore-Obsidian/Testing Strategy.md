# Testing Strategy — Seventh Sky Store

#testing #backend #frontend

---

## Overview

Multi-layer testing approach: unit, feature, integration, and E2E.

---

## Backend Testing (PHPUnit)

### Test Structure

```
backend/tests/
├── Unit/                    # Unit tests
│   ├── Services/
│   │   ├── MidtransServiceTest.php
│   │   ├── CloudinaryServiceTest.php
│   │   └── CategoryServiceTest.php
│   └── Models/
│       ├── ProductTest.php
│       ├── OrderTest.php
│       └── CartTest.php
├── Feature/                 # Feature/API tests
│   ├── Auth/
│   │   ├── RegisterTest.php
│   │   ├── LoginTest.php
│   │   └── LogoutTest.php
│   ├── Product/
│   │   ├── ProductIndexTest.php
│   │   ├── ProductShowTest.php
│   │   └── ProductAdminTest.php
│   ├── Cart/
│   │   ├── CartTest.php
│   │   └── CartItemTest.php
│   ├── Checkout/
│   │   └── CheckoutTest.php
│   ├── Order/
│   │   ├── OrderTest.php
│   │   └── OrderStateMachineTest.php
│   └── Admin/
│       ├── DashboardTest.php
│       └── AdminMiddlewareTest.php
└── Browser/                 # Laravel Dusk (optional)
    └── ...
```

### Running Tests

```bash
# All tests
php artisan test

# Specific test
php artisan test --filter=ProductIndexTest

# With coverage
php artisan test --coverage
```

### Key Test Examples

```php
// tests/Feature/Order/OrderStateMachineTest.php
public function test_valid_status_transitions()
{
    $order = Order::factory()->create(['order_status' => 'pending', 'payment_status' => 'paid']);
    
    // pending -> processing
    $this->putJson("/api/admin/orders/{$order->id}/status", ['order_status' => 'processing'])
        ->assertStatus(200);
    
    // processing -> shipped
    $this->putJson("/api/admin/orders/{$order->id}/status", ['order_status' => 'shipped'])
        ->assertStatus(200);
    
    // shipped -> completed
    $this->putJson("/api/admin/orders/{$order->id}/status", ['order_status' => 'completed'])
        ->assertStatus(200);
}

public function test_invalid_transition_rejected()
{
    $order = Order::factory()->create(['order_status' => 'pending']);
    
    $this->putJson("/api/admin/orders/{$order->id}/status", ['order_status' => 'shipped'])
        ->assertStatus(422)
        ->assertJson(['success' => false, 'message' => 'Invalid status transition']);
}

public function test_cannot_process_unpaid_order()
{
    $order = Order::factory()->create(['order_status' => 'pending', 'payment_status' => 'pending']);
    
    $this->putJson("/api/admin/orders/{$order->id}/status", ['order_status' => 'processing'])
        ->assertStatus(422)
        ->assertJson(['success' => false, 'message' => 'Order has not been paid']);
}
```

```php
// tests/Feature/Checkout/CheckoutTest.php
public function test_checkout_creates_order_and_returns_snap_token()
{
    $user = User::factory()->create();
    $product = Product::factory()->create(['stock' => 10]);
    
    // Add to cart
    $this->actingAs($user, 'sanctum')
        ->postJson('/api/cart/items', ['product_id' => $product->id, 'quantity' => 2])
        ->assertStatus(201);
    
    // Checkout
    $response = $this->actingAs($user, 'sanctum')
        ->postJson('/api/checkout', [
            'recipient_name' => 'John',
            'phone' => '08123456789',
            'address' => 'Jl. Test',
            'city' => 'Jakarta',
            'postal_code' => '10000',
        ]);
    
    $response->assertStatus(201)
        ->assertJsonStructure(['success', 'message', 'data' => ['snap_token', 'redirect_url', 'order']]);
    
    // Stock decremented
    $this->assertEquals(8, $product->fresh()->stock);
}
```

```php
// tests/Unit/Services/MidtransServiceTest.php
public function test_verify_signature()
{
    Config::$serverKey = 'test-server-key';
    
    $notification = [
        'order_id' => 'ORD-123',
        'status_code' => '200',
        'gross_amount' => '100000',
        'signature_key' => hash('sha512', 'ORD-123200100000test-server-key'),
    ];
    
    $this->assertTrue(MidtransService::verifySignature($notification));
}

public function test_rejects_invalid_signature()
{
    $notification = [
        'order_id' => 'ORD-123',
        'status_code' => '200',
        'gross_amount' => '100000',
        'signature_key' => 'invalid',
    ];
    
    $this->assertFalse(MidtransService::verifySignature($notification));
}
```

---

## Frontend Testing

### Unit/Component Tests (Vitest + React Testing Library)

```bash
# Install
npm install -D vitest @testing-library/react @testing-library/jest-dom jsdom

# vitest.config.ts
import { defineConfig } from 'vitest/config';
import react from '@vitejs/plugin-react';

export default defineConfig({
  plugins: [react()],
  test: {
    environment: 'jsdom',
    setupFiles: ['./vitest.setup.ts'],
  },
});
```

```typescript
// src/components/product/ProductCard.test.tsx
import { render, screen, fireEvent } from '@testing-library/react';
import { ProductCard } from './ProductCard';
import { QueryClient, QueryClientProvider } from '@tanstack/react-query';

const product = {
  id: 1,
  name: 'Test Product',
  slug: 'test-product',
  price: '299000',
  image: 'https://example.com/image.jpg',
  stock: 10,
  category: { id: 1, name: 'T-Shirt', slug: 't-shirt' },
};

test('renders product info', () => {
  render(
    <QueryClientProvider client={new QueryClient()}>
      <ProductCard product={product} />
    </QueryClientProvider>
  );
  
  expect(screen.getByText('Test Product')).toBeInTheDocument();
  expect(screen.getByText('Rp299.000')).toBeInTheDocument();
});

test('adds to cart on button click', async () => {
  const addToCart = vi.fn();
  render(
    <QueryClientProvider client={new QueryClient()}>
      <ProductCard product={product} onAddToCart={addToCart} />
    </QueryClientProvider>
  );
  
  fireEvent.click(screen.getByRole('button', { name: /add to cart/i }));
  expect(addToCart).toHaveBeenCalledWith(product, 1);
});
```

### Integration Tests

```typescript
// src/lib/cart.test.ts
import { useCartStore } from '@/stores/useCartStore';

test('cart store adds and removes items', () => {
  const { addItem, removeItem, items } = useCartStore.getState();
  
  addItem({ id: 1, name: 'Product', price: '100000', stock: 5 }, 2);
  expect(useCartStore.getState().items).toHaveLength(1);
  expect(useCartStore.getState().items[0].quantity).toBe(2);
  
  removeItem(useCartStore.getState().items[0].id);
  expect(useCartStore.getState().items).toHaveLength(0);
});
```

### E2E Tests (Playwright)

```bash
# Install
npm install -D @playwright/test
npx playwright install
```

```typescript
// tests/e2e/purchase.spec.ts
import { test, expect } from '@playwright/test';

test('complete purchase flow', async ({ page }) => {
  // Register
  await page.goto('/register');
  await page.fill('[name="name"]', 'Test User');
  await page.fill('[name="email"]', 'test@example.com');
  await page.fill('[name="password"]', 'password123');
  await page.fill('[name="password_confirmation"]', 'password123');
  await page.click('button[type="submit"]');
  
  // Browse products
  await page.goto('/products');
  await expect(page.locator('text=Test Product')).toBeVisible();
  
  // Add to cart
  await page.click('button:has-text("Add to Cart")');
  await expect(page.locator('text=Cart (1)')).toBeVisible();
  
  // Checkout
  await page.goto('/cart');
  await page.click('button:has-text("Checkout")');
  await page.fill('[name="recipient_name"]', 'John Doe');
  await page.fill('[name="phone"]', '08123456789');
  await page.fill('[name="address"]', 'Jl. Test 123');
  await page.fill('[name="city"]', 'Jakarta');
  await page.fill('[name="postal_code"]', '10000');
  await page.click('button[type="submit"]');
  
  // Midtrans sandbox - complete payment
  // Note: Requires Midtrans sandbox test credentials
  await page.waitForURL('**/orders**');
  await expect(page.locator('text=Order created')).toBeVisible();
});
```

---

## CI/CD Pipeline

```yaml
# .github/workflows/test.yml
name: Tests

on: [push, pull_request]

jobs:
  backend-tests:
    runs-on: ubuntu-latest
    services:
      mysql:
        image: mysql:8.0
        env:
          MYSQL_ROOT_PASSWORD: password
          MYSQL_DATABASE: test
        ports: ['3306:3306']
    steps:
      - uses: actions/checkout@v4
      - uses: shivammathur/setup-php@v2
        with:
          php-version: '8.3'
          extensions: mbstring, pdo_mysql
      - run: composer install --no-interaction --prefer-dist
      - run: cp backend/.env.example backend/.env
      - run: php artisan key:generate
      - run: php artisan migrate --force
      - run: php artisan test

  frontend-tests:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v4
      - uses: actions/setup-node@v4
        with:
          node-version: '20'
      - run: cd frontend && npm ci
      - run: cd frontend && npm run test
      - run: cd frontend && npm run test:e2e
```

---

## Test Coverage Goals

| Layer | Target |
|-------|--------|
| Backend Unit | > 80% |
| Backend Feature | > 70% |
| Frontend Unit | > 60% |
| E2E Critical Paths | 100% (auth, cart, checkout) |

---

## Related Notes

- [[Backend Structure]] — Test file organization
- [[Frontend Structure]] — Component testing
- [[Order Management]] — State machine tests
- [[Checkout & Payment]] — Payment flow tests
- [[Deployment Guide]] — CI/CD integration