# Shopping Cart — Seventh Sky Store

#frontend #backend #database

---

## Overview

Shopping cart with server-side persistence (authenticated) and client-side localStorage sync for guest users.

---

## Database

### carts
- `id` (PK)
- `user_id` (FK → users, Unique) — One cart per user

### cart_items
- `id` (PK)
- `cart_id` (FK → carts)
- `product_id` (FK → products)
- `quantity` (Default: 1)
- **Unique constraint:** `cart_id + product_id`

---

## Backend

### CartController

| Method | Endpoint | Description |
|--------|----------|-------------|
| `index` | GET `/api/cart` | Get user's cart with items |
| `store` | POST `/api/cart/items` | Add item to cart |
| `update` | PUT `/api/cart/items/{id}` | Update quantity |
| `destroy` | DELETE `/api/cart/items/{id}` | Remove item |

### Add to Cart Logic

```php
// CartController@store
public function store(AddToCartRequest $request)
{
    $user = $request->user();
    $cart = $user->cart()->firstOrCreate([]);
    
    $cartItem = $cart->items()->where('product_id', $request->product_id)->first();
    
    if ($cartItem) {
        $newQty = $cartItem->quantity + $request->quantity;
        // Check stock
        if ($newQty > $cartItem->product->stock) {
            return response()->json(['success' => false, 'message' => 'Insufficient stock'], 422);
        }
        $cartItem->update(['quantity' => $newQty]);
    } else {
        // Check stock
        if ($request->quantity > $product->stock) { ... }
        $cartItem = $cart->items()->create([
            'product_id' => $request->product_id,
            'quantity' => $request->quantity,
        ]);
    }
    
    return response()->json(['success' => true, 'data' => $cartItem->load('product')], 201);
}
```

---

## Frontend

### CartContext (Zustand)

```typescript
// src/stores/cartStore.ts
interface CartState {
  items: CartItem[];
  isOpen: boolean;
  addItem: (product: Product, quantity: number) => Promise<void>;
  updateQuantity: (id: number, quantity: number) => Promise<void>;
  removeItem: (id: number) => Promise<void>;
  clearCart: () => void;
  syncFromServer: () => Promise<void>;
  totalItems: number;
  subtotal: number;
}
```

### Persistence Strategy

```
Authenticated User:
  ├── Server is source of truth
  ├── localStorage cache for instant UI
  └── Sync on login/app init

Guest User:
  ├── localStorage only
  └── Merge to server on login
```

### Sync Flow

```typescript
// On app init
useEffect(() => {
  const token = localStorage.getItem('auth_token');
  if (token) {
    fetchCartFromServer(); // Sync server → client
  } else {
    loadFromLocalStorage(); // Guest cart
  }
}, []);

// On login
const handleLogin = async () => {
  await login(email, password);
  await mergeGuestCartToServer(); // POST each localStorage item
};
```

### CartView Component

```
src/components/cart/CartView.tsx
├── CartItem[] (ProductCard + QuantitySelector)
├── Subtotal calculation
├── Checkout button → /checkout
└── Empty state
```

---

## API Requests

### Get Cart
```http
GET /api/cart
Authorization: Bearer <token>

Response: { success: true, data: { id, user_id, items: [...], total } }
```

### Add Item
```http
POST /api/cart/items
Authorization: Bearer <token>
{ "product_id": 1, "quantity": 2 }

Response: 201 { success: true, data: CartItem }
```

### Update Quantity
```http
PUT /api/cart/items/5
Authorization: Bearer <token>
{ "quantity": 3 }

Response: 200 { success: true, data: CartItem }
```

### Remove Item
```http
DELETE /api/cart/items/5
Authorization: Bearer <token>

Response: 200 { success: true, message: "Item removed" }
```

---

## Validation Rules

| Field | Rule |
|-------|------|
| product_id | Required, exists:products,id |
| quantity | Required, integer, min:1, max:99 |
| stock check | quantity ≤ product.stock |

---

## Related Notes

- [[Database Schema#carts]] — Cart tables
- [[API Reference#Cart]] — Cart endpoints
- [[Checkout & Payment]] — Cart → Checkout flow
- [[Authentication]] — Guest vs authenticated cart
- [[Frontend Structure#Cart Context]] — Zustand store