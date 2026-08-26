# Environment Config — Seventh Sky Store

#backend #frontend #deployment

---

## Backend (.env)

### Core

```env
APP_NAME="Seventh Sky Store"
APP_ENV=local
APP_KEY=base64:XXXXXXXXXXXXXXXXXXXXXXXXXXXX
APP_DEBUG=true
APP_URL=http://localhost:8000
```

### Database

```env
DB_CONNECTION=sqlite
DB_DATABASE=/var/www/html/database/database.sqlite
```

**MySQL alternative:**
```env
DB_CONNECTION=mysql
DB_HOST=127.0.0.1
DB_PORT=3306
DB_DATABASE=seventh_sky_store
DB_USERNAME=root
DB_PASSWORD=
```

**PostgreSQL alternative:**
```env
DB_CONNECTION=pgsql
DB_HOST=127.0.0.1
DB_PORT=5432
DB_DATABASE=seventh_sky_store
DB_USERNAME=postgres
DB_PASSWORD=
```

### Midtrans

```env
MIDTRANS_SERVER_KEY=SB-Mid-server-XXXXXXXXXXXXXXXXXXXX
MIDTRANS_CLIENT_KEY=SB-Mid-client-XXXXXXXXXXXXXXXXXXXX
MIDTRANS_IS_PRODUCTION=false
MIDTRANS_IS_SANITIZED=true
MIDTRANS_IS_3DS=true
```

### Cloudinary

```env
CLOUDINARY_CLOUD_NAME=your-cloud-name
CLOUDINARY_API_KEY=XXXXXXXXXXXXXXXXXXXX
CLOUDINARY_API_SECRET=XXXXXXXXXXXXXXXXXXXXXXXXXXXXXX
```

### Sanctum

```env
SANCTUM_STATEFUL_DOMAINS=localhost:3000
SESSION_DOMAIN=localhost
```

### Queue (Optional)

```env
QUEUE_CONNECTION=sync
```

---

## Frontend (.env.local)

```env
NEXT_PUBLIC_API_URL=http://localhost:8000/api
NEXT_PUBLIC_MIDTRANS_CLIENT_KEY=SB-Mid-client-XXXXXXXXXXXXXXXXXXXX
```

---

## Configuration Files

### config/midtrans.php

```php
<?php

return [
    'server_key' => env('MIDTRANS_SERVER_KEY'),
    'client_key' => env('MIDTRANS_CLIENT_KEY'),
    'is_production' => env('MIDTRANS_IS_PRODUCTION', false),
    'is_sanitized' => env('MIDTRANS_IS_SANITIZED', true),
    'is_3ds' => env('MIDTRANS_IS_3DS', true),
];
```

---

## Docker Compose (Optional)

```yaml
version: '3.8'
services:
  backend:
    build: ./backend
    ports:
      - "8000:8000"
    env_file: ./backend/.env
    volumes:
      - ./backend:/var/www/html
    depends_on:
      - db

  frontend:
    build: ./frontend
    ports:
      - "3000:3000"
    env_file: ./frontend/.env.local
    volumes:
      - ./frontend:/app

  db:
    image: mysql:8.0
    environment:
      MYSQL_ROOT_PASSWORD: secret
      MYSQL_DATABASE: seventh_sky_store
    ports:
      - "3306:3306"
    volumes:
      - db_data:/var/lib/mysql

volumes:
  db_data:
```

---

## Production Environment Variables

### Required by Backend

| Variable | Description |
|----------|-------------|
| `APP_KEY` | Laravel app key |
| `APP_URL` | Production URL |
| `DB_CONNECTION` | Database driver |
| `DB_*` | Database credentials |
| `MIDTRANS_SERVER_KEY` | Production server key |
| `MIDTRANS_CLIENT_KEY` | Production client key |
| `MIDTRANS_IS_PRODUCTION` | Set to `true` |
| `CLOUDINARY_*` | Cloudinary credentials |

### Required by Frontend

| Variable | Description |
|----------|-------------|
| `NEXT_PUBLIC_API_URL` | Production backend URL |
| `NEXT_PUBLIC_MIDTRANS_CLIENT_KEY` | Production client key |

---

## Deployment Scripts

### Backend

```bash
# Install
composer install --optimize-autoloader --no-dev

# Environment
cp .env.example .env
php artisan key:generate
php artisan config:cache
php artisan route:cache
php artisan view:cache

# Database
php artisan migrate --force

# Start (PHP built-in server for testing)
php artisan serve --host=0.0.0.0 --port=8000
```

### Frontend

```bash
# Install
npm ci

# Build
npm run build

# Start (production)
npm start

# Or deploy to Vercel
vercel --prod
```

---

## CI/CD Environment

| Stage | Backend Vars | Frontend Vars |
|-------|-------------|---------------|
| **Testing** | `APP_ENV=testing`, SQLite | `NEXT_PUBLIC_API_URL=http://localhost:8000/api`, sandbox Midtrans |
| **Staging** | `APP_ENV=production`, MySQL | `NEXT_PUBLIC_API_URL=https://staging-api.seventhsky.store/api`, sandbox Midtrans |
| **Production** | `APP_ENV=production`, PostgreSQL | `NEXT_PUBLIC_API_URL=https://api.seventhsky.store/api`, production Midtrans |

---

## Related Notes

- [[Deployment Guide]] — Deployment steps
- [[Architecture]] — Environment overview
- [[Checkout & Payment]] — Midtrans configuration
- [[Backend Structure]] — Backend config files