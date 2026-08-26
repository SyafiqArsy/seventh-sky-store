# Deployment Guide — Seventh Sky Store

#deployment #backend #frontend

---

## Overview

Production deployment guide for both backend (Laravel) and frontend (Next.js).

---

## Backend Deployment

### Requirements

- PHP 8.3+
- Composer 2.x
- Web server (Nginx/Apache) + PHP-FPM
- Database: MySQL 8.0+ / PostgreSQL 15+ / SQLite
- SSL certificate

### Server Setup (Nginx + PHP-FPM)

```nginx
server {
    listen 80;
    server_name api.seventhsky.store;
    root /var/www/seventh-sky-store/backend/public;
    index index.php;

    add_header X-Frame-Options "DENY";
    add_header X-Content-Type-Options "nosniff";

    location / {
        try_files $uri $uri/ /index.php?$query_string;
    }

    location = /favicon.ico { access_log off; log_not_found off; }
    location = /robots.txt  { access_log off; log_not_found off; }

    error_page 404 /index.php;

    location ~ \.php$ {
        fastcgi_pass unix:/var/run/php/php8.3-fpm.sock;
        fastcgi_param SCRIPT_FILENAME $realpath_root$fastcgi_script_name;
        include fastcgi_params;
    }

    location ~ /\.(?!well-known).* {
        deny all;
    }
}
```

### Deployment Steps

```bash
# 1. Clone repository
git clone https://github.com/your-username/seventh-sky-store.git
cd seventh-sky-store/backend

# 2. Install dependencies
composer install --optimize-autoloader --no-dev

# 3. Environment
cp .env.example .env
# Edit .env with production values

# 4. Generate app key
php artisan key:generate

# 5. Cache config
php artisan config:cache
php artisan route:cache
php artisan view:cache

# 6. Database
php artisan migrate --force

# 7. Storage link (if using local storage)
php artisan storage:link

# 8. Permissions
chown -R www-data:www-data storage bootstrap/cache
chmod -R 775 storage bootstrap/cache

# 9. Queue worker (systemd/supervisor)
# See: Supervisor config below
```

### Supervisor Config (Queue Worker)

```ini
[program:seventh-sky-store-worker]
process_name=%(program_name)s_%(process_num)02d
command=php /var/www/seventh-sky-store/backend/artisan queue:work --sleep=3 --tries=3 --max-time=3600
autostart=true
autorestart=true
user=www-data
numprocs=2
redirect_stderr=true
stdout_logfile=/var/www/seventh-sky-store/backend/storage/logs/worker.log
stopwaitsecs=3600
```

### Scheduler (Cron)

```bash
* * * * * cd /var/www/seventh-sky-store/backend && php artisan schedule:run >> /dev/null 2>&1
```

---

## Frontend Deployment

### Option 1: Vercel (Recommended)

```bash
# 1. Install Vercel CLI
npm i -g vercel

# 2. Deploy
cd frontend
vercel --prod

# 3. Set environment variables in Vercel dashboard:
# NEXT_PUBLIC_API_URL=https://api.seventhsky.store/api
# NEXT_PUBLIC_MIDTRANS_CLIENT_KEY=your_production_client_key
```

### Option 2: Static Export + CDN

```bash
# next.config.ts
output: 'export',

# Build
npm run build

# Output: frontend/out/
# Deploy to Netlify, Cloudflare Pages, AWS S3 + CloudFront
```

### Option 3: Node.js Server (VPS)

```bash
# Build
npm run build

# Start (uses standalone output)
npm start

# PM2 process manager
pm2 start ecosystem.config.js

# ecosystem.config.js
module.exports = {
  apps: [{
    name: 'seventh-sky-store-frontend',
    script: 'node_modules/.bin/next',
    args: 'start',
    cwd: '/var/www/seventh-sky-store/frontend',
    env: {
      NODE_ENV: 'production',
      PORT: 3000,
    },
  }],
};
```

---

## Database Migration

### SQLite → MySQL/PostgreSQL

```bash
# 1. Update .env
DB_CONNECTION=mysql
DB_HOST=127.0.0.1
DB_PORT=3306
DB_DATABASE=seventh_sky_store
DB_USERNAME=your_user
DB_PASSWORD=your_password

# 2. Run migrations
php artisan migrate --force

# 3. Seed (optional)
php artisan db:seed --force
```

### Backup Strategy

```bash
# MySQL
mysqldump -u root -p seventh_sky_store > backup_$(date +%F).sql

# PostgreSQL
pg_dump -U postgres seventh_sky_store > backup_$(date +%F).sql

# SQLite
cp database/database.sqlite database/backup_$(date +%F).sqlite
```

---

## SSL/TLS (Let's Encrypt)

```bash
# Certbot (Nginx)
certbot --nginx -d api.seventhsky.store -d seventhsky.store

# Auto-renewal
crontab -e
0 12 * * * /usr/bin/certbot renew --quiet
```

---

## Monitoring & Logging

### Laravel Telescope (Dev/Staging)

```bash
composer require laravel/telescope --dev
php artisan telescope:install
php artisan migrate
```

### Log Rotation

```nginx
# /etc/logrotate.d/laravel
/var/www/seventh-sky-store/backend/storage/logs/*.log {
    daily
    missingok
    rotate 14
    compress
    notifempty
    create 0640 www-data www-data
}
```

---

## Health Checks

### Backend

```bash
# Add to routes/api.php
Route::get('/health', fn() => response()->json(['status' => 'ok']));
```

### Frontend

```bash
# Next.js has /api/health by default with App Router
# Or add app/api/health/route.ts
```

---

## Rollback Procedure

### Backend

```bash
# 1. Revert code
git checkout previous-tag

# 2. Clear caches
php artisan config:clear
php artisan route:clear
php artisan view:clear

# 3. Rollback migration (if needed)
php artisan migrate:rollback --step=1

# 4. Rebuild caches
php artisan config:cache
php artisan route:cache
php artisan view:cache
```

### Frontend (Vercel)

```bash
vercel rollback [deployment-url]
```

---

## Related Notes

- [[Environment Config]] — Production variables
- [[Architecture#Deployment Architecture]] — Architecture overview
- [[Backend Structure]] — Laravel setup
- [[Frontend Structure]] — Next.js setup