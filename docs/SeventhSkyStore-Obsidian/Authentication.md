# Authentication — Seventh Sky Store

#auth #backend #frontend

---

## Overview

Seventh Sky Store uses **Laravel Sanctum** for token-based API authentication. The frontend manages auth state via React Context.

---

## Backend Implementation

### AuthController

| Method | Endpoint | Description |
|--------|----------|-------------|
| `register` | POST `/api/register` | Create account, return token |
| `login` | POST `/api/login` | Validate credentials, return token |
| `me` | GET `/api/me` | Get authenticated user |
| `logout` | POST `/api/logout` | Revoke current token |
| `logoutAll` | POST `/api/logout-all` | Revoke all user tokens |

### Token Management

```php
// On register/login
$token = $user->createToken('auth-token')->plainTextToken;

// On logout (current)
$request->user()->currentAccessToken()->delete();

// On logout all
$request->user()->tokens()->delete();
```

### Rate Limiting

```php
// Register: 5/min per IP
Route::post('/register')->middleware('throttle:5,1');

// Login: 10/min per IP
Route::post('/login')->middleware('throttle:10,1');

// Login attempts tracked via RateLimiter
$throttleKey = 'login:' . $request->ip();
if (RateLimiter::tooManyAttempts($throttleKey, 10)) { ... }
```

### Role-Based Access

```php
// User model
protected $fillable = [..., 'role']; // 'customer' | 'admin'

// AdminMiddleware
public function handle(Request $request, Closure $next)
{
    if ($request->user()->role !== 'admin') {
        return response()->json(['success' => false, 'message' => 'Forbidden'], 403);
    }
    return $next($request);
}
```

---

## Frontend Implementation

### AuthContext

```typescript
// src/context/AuthContext.tsx
interface AuthContextType {
  user: User | null;
  token: string | null;
  login: (email: string, password: string) => Promise<void>;
  register: (data: RegisterData) => Promise<void>;
  logout: () => Promise<void>;
  isAuthenticated: boolean;
  isAdmin: boolean;
}
```

**Persistence:** Token stored in `localStorage`, restored on app init.

### API Client (Axios)

```typescript
// src/lib/api.ts
const api = axios.create({
  baseURL: process.env.NEXT_PUBLIC_API_URL,
});

api.interceptors.request.use((config) => {
  const token = localStorage.getItem('auth_token');
  if (token) config.headers.Authorization = `Bearer ${token}`;
  return config;
});

api.interceptors.response.use(
  (res) => res,
  (err) => {
    if (err.response?.status === 401) {
      // Auto-logout on token expiry
      useAuthStore.getState().logout();
    }
    return Promise.reject(err);
  }
);
```

### Protected Routes

```typescript
// Admin route protection
// src/components/admin/AdminGuard.tsx
export function AdminGuard({ children }: { children: React.ReactNode }) {
  const { isAdmin, isAuthenticated } = useAuth();
  
  if (!isAuthenticated) return <Redirect to="/login" />;
  if (!isAdmin) return <Redirect to="/" />;
  
  return <>{children}</>;
}
```

---

## Flow Diagrams

### Registration Flow

```
User submits register form
        │
        ▼
POST /api/register { name, email, password }
        │
        ▼
Validate (RegisterRequest)
        │
        ▼
User::create([..., 'role' => 'customer'])
        │
        ▼
$user->createToken('auth-token')
        │
        ▼
Return { token, user }
        │
        ▼
Frontend: Store token in localStorage
        │
        ▼
Redirect to homepage
```

### Login Flow

```
User submits login form
        │
        ▼
POST /api/login { email, password }
        │
        ▼
RateLimiter check (10/min/IP)
        │
        ▼
Auth::attempt(credentials)
        │
        ├─► Fail: RateLimiter::hit() → 429/401
        │
        └─► Success: RateLimiter::clear()
                    │
                    ▼
            $user->createToken('auth-token')
                    │
                    ▼
            Return { token, user }
                    │
                    ▼
            Frontend: Store token, update context
                    │
                    ▼
            Redirect to intended page
```

---

## Security Considerations

| Threat | Mitigation |
|--------|------------|
| Brute force | Rate limiting (5/min register, 10/min login) |
| Token theft | HTTPS only, short token expiry (configurable) |
| CSRF | API is stateless, tokens in Authorization header |
| Session fixation | New token on each login |
| Privilege escalation | Admin middleware on all `/admin/*` routes |

---

## Related Notes

- [[API Reference#Authentication]] — Auth endpoints
- [[Architecture#Middleware Pipeline]] — Auth middleware flow
- [[Admin Dashboard]] — Admin access control
- [[Frontend Structure#Auth Context]] — Client-side auth