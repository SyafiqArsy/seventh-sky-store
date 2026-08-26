# Checkout & Payment — Seventh Sky Store

#payment #backend #frontend #midtrans

---

## Overview

Complete checkout flow with Midtrans Snap payment integration, webhook handling, and stock management.

---

## Flow Diagram

```
User fills shipping form
        │
        ▼
POST /api/checkout { shipping_details }
        │
        ▼
Backend: CheckoutController@store
        │
        ├─► Validate request
        ├─► DB Transaction
        │     ├─► Create Order (pending, unpaid)
        │     ├─► Create OrderItems (snapshot prices)
        │     ├─► Decrement product stock
        │     └─► Clear user cart
        │
        ├─► MidtransService::createSnapToken(order)
        │     └─► Snap::getSnapToken(params)
        │
        ▼
Return { snap_token, redirect_url, order }
        │
        ▼
Frontend: loadMidtransScript()
        │
        ▼
snap.pay(snap_token, { onSuccess, onClose })
        │
        ▼
Midtrans Popup → User pays
        │
        ├─► Success → onSuccess → redirect to /orders
        ├─► Pending → onPending
        └─► Close/Error → onClose/onError
        │
        ▼
Midtrans → Webhook POST /api/midtrans/notification
        │
        ▼
Backend: MidtransController@notification
        │
        ├─► Verify SHA-512 signature
        ├─► Parse transaction_status
        ├─► Update Order payment_status
        ├─► If paid → order_status = processing
        └─► If failed/expired → restore stock
```

---

## Backend

### CheckoutController

```php
// POST /api/checkout
public function store(CheckoutRequest $request)
{
    $user = $request->user();
    $cart = $user->cart;
    
    // Validate cart not empty
    if ($cart->items->isEmpty()) {
        return response()->json(['success' => false, 'message' => 'Cart is empty'], 422);
    }
    
    DB::transaction(function () use ($request, $user, $cart) {
        // Calculate totals
        $totalPrice = $cart->items->sum(fn($i) => $i->product->price * $i->quantity);
        $shippingCost = $request->shipping_cost ?? 15000;
        $grandTotal = $totalPrice + $shippingCost;
        
        // Create Order
        $order = Order::create([
            'user_id' => $user->id,
            'order_number' => 'ORD-' . now()->format('YmdHis') . '-' . Str::random(4),
            'recipient_name' => $request->recipient_name,
            'phone' => $request->phone,
            'address' => $request->address,
            'city' => $request->city,
            'postal_code' => $request->postal_code,
            'total_price' => $totalPrice,
            'shipping_cost' => $shippingCost,
            'grand_total' => $grandTotal,
            'payment_status' => 'pending',
            'order_status' => 'pending',
        ]);
        
        // Create OrderItems (snapshot)
        foreach ($cart->items as $item) {
            OrderItem::create([
                'order_id' => $order->id,
                'product_id' => $item->product_id,
                'product_name' => $item->product->name,
                'product_price' => $item->product->price,
                'quantity' => $item->quantity,
                'subtotal' => $item->product->price * $item->quantity,
            ]);
            
            // Decrement stock
            $item->product->decrement('stock', $item->quantity);
        }
        
        // Clear cart
        $cart->items()->delete();
        
        // Get Snap token
        $snapToken = MidtransService::createSnapToken([
            'transaction_details' => [
                'order_id' => $order->order_number,
                'gross_amount' => (int) $grandTotal,
            ],
            'customer_details' => [
                'first_name' => $request->recipient_name,
                'phone' => $request->phone,
                'address' => $request->address,
                'city' => $request->city,
                'postal_code' => $request->postal_code,
            ],
            'item_details' => $order->items->map(fn($i) => [
                'id' => $i->product_id,
                'price' => (int) $i->product_price,
                'quantity' => $i->quantity,
                'name' => $i->product_name,
            ])->toArray(),
        ]);
        
        $order->update([
            'midtrans_token' => $snapToken,
            'midtrans_order_id' => $order->order_number,
        ]);
        
        return response()->json([
            'success' => true,
            'message' => 'Order created. Complete payment to confirm.',
            'data' => [
                'snap_token' => $snapToken,
                'redirect_url' => config('midtrans.is_production') 
                    ? 'https://app.midtrans.com/snap/v2/vtweb/' . $snapToken
                    : 'https://app.sandbox.midtrans.com/snap/v2/vtweb/' . $snapToken,
                'order' => $order->load('items'),
            ],
        ], 201);
    });
}
```

### MidtransService

```php
// app/Services/MidtransService.php
class MidtransService
{
    public static function init()
    {
        Config::$serverKey = config('midtrans.server_key');
        Config::$clientKey = config('midtrans.client_key');
        Config::$isProduction = config('midtrans.is_production');
        Config::$isSanitized = config('midtrans.is_sanitized');
        Config::$is3ds = config('midtrans.is_3ds');
    }
    
    public static function createSnapToken(array $params): string
    {
        self::init();
        return Snap::getSnapToken($params);
    }
    
    public static function verifySignature(array $notification): bool
    {
        self::init();
        
        $signatureKey = hash('sha512', 
            $notification['order_id'] . 
            $notification['status_code'] . 
            $notification['gross_amount'] . 
            Config::$serverKey
        );
        
        return hash_equals($signatureKey, $notification['signature_key']);
    }
}
```

### MidtransController (Webhook)

```php
// POST /api/midtrans/notification
public function notification(Request $request)
{
    $notification = $request->all();
    
    // Verify signature
    if (!MidtransService::verifySignature($notification)) {
        return response()->json(['success' => false, 'message' => 'Invalid signature'], 400);
    }
    
    $order = Order::where('order_number', $notification['order_id'])->first();
    if (!$order) {
        return response()->json(['success' => false, 'message' => 'Order not found'], 404);
    }
    
    $status = $notification['transaction_status'];
    $paymentType = $notification['payment_type'];
    $transactionId = $notification['transaction_id'];
    
    // Map Midtrans status to our status
    $paymentStatusMap = [
        'capture' => 'paid',
        'settlement' => 'paid',
        'pending' => 'pending',
        'deny' => 'failed',
        'cancel' => 'failed',
        'expire' => 'expired',
        'refund' => 'refunded',
    ];
    
    $paymentStatus = $paymentStatusMap[$status] ?? 'pending';
    
    // Idempotent: only process if status changed
    if ($order->payment_status !== $paymentStatus) {
        $updateData = [
            'payment_status' => $paymentStatus,
            'midtrans_transaction_id' => $transactionId,
            'payment_type' => $paymentType,
        ];
        
        if ($paymentStatus === 'paid') {
            $updateData['paid_at'] = now();
            $updateData['order_status'] = 'processing';
        }
        
        $order->update($updateData);
        
        // Restore stock on failure/expiry
        if (in_array($paymentStatus, ['failed', 'expired', 'cancelled'])) {
            foreach ($order->items as $item) {
                $item->product->increment('stock', $item->quantity);
            }
        }
    }
    
    return response()->json(['success' => true]);
}
```

---

## Frontend

### CheckoutForm Component

```
src/components/checkout/CheckoutForm.tsx
├── Shipping Form
│   ├── Recipient Name
│   ├── Phone
│   ├── Address
│   ├── City
│   └── Postal Code
├── Order Summary (from cart)
├── Submit Button
│   ├── Disabled while loading
│   └── Shows "Processing..."
└── Midtrans Integration
    ├── loadMidtransScript()
    └── snap.pay(token, callbacks)
```

### Midtrans Loading

```typescript
// src/lib/loadMidtrans.ts
export async function loadMidtransScript(): Promise<void> {
  if (window.Midtrans) return;
  
  return new Promise((resolve, reject) => {
    const script = document.createElement('script');
    script.src = `https://app.sandbox.midtrans.com/snap/snap.js`;
    script.setAttribute('data-client-key', process.env.NEXT_PUBLIC_MIDTRANS_CLIENT_KEY!);
    script.onload = () => resolve();
    script.onerror = () => reject(new Error('Failed to load Midtrans'));
    document.body.appendChild(script);
  });
}

// Usage in CheckoutForm
const handleSubmit = async (data) => {
  const { snap_token } = await checkout(data);
  await loadMidtransScript();
  
  window.MidtransNewWindow = false; // Use popup
  window.snap.pay(snap_token, {
    onSuccess: () => router.push('/orders'),
    onPending: () => toast('Payment pending'),
    onError: () => toast('Payment failed'),
    onClose: () => toast('Payment cancelled'),
  });
};
```

---

## Midtrans Configuration

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

### Environment Variables

```env
# Backend .env
MIDTRANS_SERVER_KEY=SB-Mid-server-xxxx
MIDTRANS_CLIENT_KEY=SB-Mid-client-xxxx
MIDTRANS_IS_PRODUCTION=false

# Frontend .env.local
NEXT_PUBLIC_MIDTRANS_CLIENT_KEY=SB-Mid-client-xxxx
```

---

## Payment Methods Supported

| Type | Methods |
|------|---------|
| **Credit/Debit Card** | Visa, Mastercard, JCB |
| **Bank Transfer** | BCA, BNI, BRI, Mandiri, Permata (VA) |
| **E-Wallet** | GoPay, ShopeePay, OVO, Dana, LinkAja |
| **Convenience Store** | Alfamart, Indomaret |
| **QRIS** | Scan to pay |

---

## Stock Management

| Event | Action |
|-------|--------|
| Checkout | Decrement stock, create order |
| Payment Success | Confirm order_status = processing |
| Payment Failed | Restore stock, payment_status = failed |
| Payment Expired | Restore stock, payment_status = expired |
| Refund | Restore stock, payment_status = refunded |

---

## Error Handling

| Scenario | Frontend | Backend |
|----------|----------|---------|
| Empty cart | Disable checkout | 422 Cart is empty |
| Insufficient stock | Show error toast | 422 Insufficient stock |
| Midtrans load fail | Retry button | N/A |
| Payment cancelled | Return to cart | Webhook: expired |
| Webhook invalid sig | N/A | 400 Invalid signature |
| Order not found | N/A | 404 Order not found |

---

## Related Notes

- [[API Reference#Checkout]] — Checkout endpoint
- [[API Reference#Midtrans Webhook]] — Webhook endpoint
- [[Shopping Cart]] — Cart → Checkout
- [[Order Management]] — Post-payment order flow
- [[Database Schema#orders]] — Order tables
- [[Authentication]] — Authenticated checkout