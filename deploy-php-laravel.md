# Deploy PHP Laravel

Complete guide for deploying Laravel applications with PHP-FPM and Nginx.

---

## 1. Install PHP

```bash
sudo apt update
sudo apt install php8.3 php8.3-fpm php8.3-cli php8.3-common \
    php8.3-mysql php8.3-pgsql php8.3-sqlite3 php8.3-mbstring \
    php8.3-xml php8.3-curl php8.3-zip php8.3-bcmath php8.3-gd -y
```

---

## 2. Install Composer

```bash
curl -sS https://getcomposer.org/installer | php
sudo mv composer.phar /usr/local/bin/composer
composer --version
```

---

## 3. Project Setup

```bash
# Create directory
sudo mkdir -p /var/www/laravel
sudo chown -R $USER:$USER /var/www/laravel
cd /var/www/laravel

# Clone project
git clone https://github.com/user/repo.git .

# Install dependencies
composer install --no-dev --optimize-autoloader

# Permissions
sudo chown -R www-data:www-data storage bootstrap/cache
sudo chmod -R 775 storage bootstrap/cache
```

---

## 4. Environment Setup

```bash
cp .env.example .env
nano .env
```

```env
APP_NAME=MyApp
APP_ENV=production
APP_DEBUG=false
APP_URL=https://example.com

DB_CONNECTION=mysql
DB_HOST=127.0.0.1
DB_DATABASE=laravel
DB_USERNAME=laraveluser
DB_PASSWORD=password
```

```bash
# Generate key
php artisan key:generate

# Run migrations
php artisan migrate --force

# Cache config
php artisan config:cache
php artisan route:cache
php artisan view:cache
```

---

## 5. Nginx Configuration

```nginx
server {
    listen 80;
    server_name example.com;
    root /var/www/laravel/public;

    index index.php;

    location / {
        try_files $uri $uri/ /index.php?$query_string;
    }

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

```bash
sudo ln -s /etc/nginx/sites-available/laravel /etc/nginx/sites-enabled/
sudo nginx -t
sudo systemctl reload nginx
```

---

## 6. Queue Worker (Optional)

```bash
sudo nano /etc/supervisor/conf.d/laravel-worker.conf
```

```ini
[program:laravel-worker]
process_name=%(program_name)s_%(process_num)02d
command=php /var/www/laravel/artisan queue:work --sleep=3 --tries=3
autostart=true
autorestart=true
numprocs=2
user=www-data
```

```bash
sudo supervisorctl reread
sudo supervisorctl update
sudo supervisorctl start laravel-worker:*
```

---

## 7. Scheduler

```bash
# Add to crontab
crontab -e

# Add line:
* * * * * cd /var/www/laravel && php artisan schedule:run >> /dev/null 2>&1
```

---

## Quick Reference

| Task | Command |
|------|---------|
| Clear cache | `php artisan cache:clear` |
| Optimize | `php artisan optimize` |
| Migrate | `php artisan migrate` |
| Queue | `php artisan queue:work` |

---

✅ Laravel deployed!
