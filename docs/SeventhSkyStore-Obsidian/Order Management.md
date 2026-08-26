# Order Management — Seventh Sky Store

#backend #frontend #database

---

## Overview

Complete order lifecycle from creation to completion with state machine validation.

---

## Order State Machine

```
┌─────────┐
│ pending │──────► cancelled
└────┬────┘
     │
     ▼
┌───────────┐
│processing │
└────┬──────┘
     │
     ▼
┌────────┐
│ shipped│
└────┬───┘
     │
     ▼
┌──────────┐
│completed │
└──────────┘
```

### Valid Transitions

| Current Status | Allowed Next Status |
|----------------|---------------------|
| pending | processing, cancelled |
| processing | shipped |
| shipped | completed |
| completed | (none) |
| cancelled | (none) |

### Constraints

- Cannot transition to `processing`, `shipped`, or `completed` if `payment_status !== 'paid'`
- Returns 422 on invalid transition

---

## Backend

### OrderController

| Method | Endpoint | Access | Description |
|--------|----------|--------|-------------|
| `index` | GET `/api/orders` | Auth | Customer order history |
| `show` | GET `/api/orders/{id}` | Auth | Customer order detail |
| `adminIndex` | GET `/api/admin/orders` | Admin | All orders |
| `adminShow` | GET `/api/admin/orders/{id}` | Admin | Order detail + user |
| `updateStatus` | PUT `/api/admin/orders/{id}/status` | Admin | Update status |

### Update Status Logic

```php
// OrderController@updateStatus
public function updateStatus(Request $request, Order $order)
{
    $request->validate([
        'order_status' => ['required', 'in:pending,processing,shipped,completed,cancelled']
    ]);
    
    $currentStatus = $order->order_status;
    $newStatus = $request->order_status;
    
    $allowedTransitions = [
        'pending' => ['processing', 'cancelled'],
        'processing' => ['shipped'],
        'shipped' => ['completed'],
        'completed' => [],
        'cancelled' => [],
    ];
    
    // Prevent invalid transition
    if (!in_array($newStatus, $allowedTransitions[$currentStatus])) {
        return response()->json([
            'success' => false,
            'message' => 'Invalid status transition'
        ], 422);
    }
    
    // Prevent processing unpaid orders
    if ($order->payment_status !== 'paid' && in_array($newStatus, ['processing', 'shipped', 'completed'])) {
        return response()->json([
            'success' => false,
            'message' => 'Order has not been paid'
        ], 422);
    }
    
    $order->update(['order_status' => $newStatus]);
    
    return response()->json([
        'success' => true,
        'message' => 'Order status updated',
        'data' => $order
    ]);
}
```

---

## Frontend

### Customer Order History

```
app/(site)/orders/page.tsx
├── OrdersView component
│   ├── Order list (React Query)
│   ├── Status badges
│   └── Pagination

app/(site)/orders/[id]/page.tsx
├── OrderDetailView component
│   ├── Order info (number, date, status)
│   ├── Shipping address
│   ├── Order items (name, price, qty, subtotal)
│   ├── Payment info
│   └── Total breakdown
```

### Admin Order Management

```
app/(admin)/admin/orders/page.tsx
├── AdminOrders component
│   ├── Search/filter
│   ├── Paginated table
│   └── Status dropdown (per row)

app/(admin)/admin/orders/[id]/page.tsx
├── AdminOrderDetail component
│   ├── Full order details
│   ├── Customer info
│   ├── Items with product links
│   └── Status update form
```

---

## Order Model

```php
// app/Models/Order.php
protected $fillable = [
    'user_id', 'order_number', 'recipient_name', 'phone',
    'address', 'city', 'postal_code', 'total_price',
    'shipping_cost', 'grand_total', 'payment_status',
    'order_status', 'midtrans_order_id', 'midtrans_token',
    'transaction_id', 'payment_type', 'paid_at'
];

protected $casts = [
    'paid_at' => 'datetime',
];

public function user() { return $this->belongsTo(User::class); }
public function items() { return $this->hasMany(OrderItem::class); }
```

### OrderItem Model (Snapshot)

```php
// app/Models/OrderItem.php
protected $fillable = [
    'order_id', 'product_id', 'product_name', 'product_price',
    'quantity', 'subtotal'
];
```

**Note:** Stores product name/price at purchase time (immutable snapshot).

---

## Payment Statuses

| Status | Meaning | Source |
|--------|---------|--------|
| pending | Awaiting payment | Initial |
| paid | Payment confirmed | Webhook (settlement/capture) |
| failed | Payment denied | Webhook (deny/cancel) |
| expired | Payment timeout | Webhook (expire) |
| refunded | Refund issued | Webhook (refund) |

---

## Order Number Format

```
ORD-YYYYMMDDHHMMSS-XXXX
Example: ORD-20260826143052-A7K9
```

Generated in `CheckoutController`:
```php
'order_number' => 'ORD-' . now()->format('YmdHis') . '-' . Str::random(4),
```

---

## Admin Dashboard Stats

```php
// AdminDashboardController@index
return [
    'total_products' => Product::count(),
    'total_categories' => Category::count(),
    'total_orders' => Order::count(),
    'total_revenue' => Order::where('payment_status', 'paid')->sum('grand_total'),
];
```

---

## Related Notes

- [[Database Schema#orders]] — Order tables
- [[API Reference#Orders]] — Order endpoints
- [[Checkout & Payment]] — Order creation flow
- [[Admin Dashboard]] — Admin order management
- [[Database Schema#Order State Machine]] — State transitions