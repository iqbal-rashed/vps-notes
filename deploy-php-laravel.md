# Deploy PHP / Laravel App

Laravel uses PHP-FPM (a PHP process manager) and Nginx. No PM2 needed — PHP runs differently from Node.js.

---

## Step 1 — Install PHP

```bash
sudo apt update
sudo apt install php8.3 php8.3-fpm php8.3-cli php8.3-common \
    php8.3-mysql php8.3-mbstring php8.3-xml \
    php8.3-curl php8.3-zip php8.3-bcmath php8.3-gd -y

# Check PHP is installed
php --version
```

---

## Step 2 — Install Composer (PHP Package Manager)

```bash
curl -sS https://getcomposer.org/installer | php
sudo mv composer.phar /usr/local/bin/composer
composer --version
```

---

## Step 3 — Upload Your App

```bash
# Create directory
sudo mkdir -p /var/www/laravel
sudo chown -R $USER:$USER /var/www/laravel
cd /var/www/laravel

# Clone from GitHub
git clone https://github.com/yourusername/your-repo.git .

# Install dependencies (no dev packages in production)
composer install --no-dev --optimize-autoloader
```

---

## Step 4 — Configure Environment

```bash
cp .env.example .env
nano .env
```

```env
APP_NAME=MyApp
APP_ENV=production
APP_KEY=                    # Will be generated below
APP_DEBUG=false
APP_URL=https://example.com

DB_CONNECTION=mysql
DB_HOST=127.0.0.1
DB_PORT=3306
DB_DATABASE=myapp
DB_USERNAME=appuser
DB_PASSWORD=YourDBPassword!
```

Generate the app key and run migrations:
```bash
php artisan key:generate
php artisan migrate --force

# Speed up the app by caching config and routes
php artisan config:cache
php artisan route:cache
php artisan view:cache
```

---

## Step 5 — Set File Permissions

Laravel needs to write to `storage` and `bootstrap/cache`:

```bash
sudo chown -R www-data:www-data storage bootstrap/cache
sudo chmod -R 775 storage bootstrap/cache
```

---

## Step 6 — Configure Nginx

```bash
sudo nano /etc/nginx/sites-available/laravel
```

```nginx
server {
    listen 80;
    server_name example.com www.example.com;
    root /var/www/laravel/public;
    index index.php;

    # Security headers
    add_header X-Content-Type-Options "nosniff" always;
    add_header X-Frame-Options "SAMEORIGIN" always;

    location / {
        try_files $uri $uri/ /index.php?$query_string;
    }

    location ~ \.php$ {
        fastcgi_pass unix:/var/run/php/php8.3-fpm.sock;
        fastcgi_param SCRIPT_FILENAME $realpath_root$fastcgi_script_name;
        include fastcgi_params;
    }

    # Block hidden files and sensitive files
    location ~ /\.(?!well-known).* {
        deny all;
    }

    location ~ /\.(env|git|htaccess) {
        deny all;
    }
}
```

Enable it:
```bash
sudo ln -s /etc/nginx/sites-available/laravel /etc/nginx/sites-enabled/
sudo nginx -t
sudo systemctl reload nginx
```

---

## Step 7 — Add SSL

```bash
sudo certbot --nginx -d example.com -d www.example.com
```

---

## Optional — Queue Worker

If your app uses queues (sending emails, processing jobs):

```bash
sudo apt install supervisor -y
sudo nano /etc/supervisor/conf.d/laravel-worker.conf
```

```ini
[program:laravel-worker]
command=php /var/www/laravel/artisan queue:work --sleep=3 --tries=3
directory=/var/www/laravel
user=www-data
autostart=true
autorestart=true
numprocs=2
```

```bash
sudo supervisorctl reread
sudo supervisorctl update
sudo supervisorctl start laravel-worker:*
```

---

## Optional — Cron Scheduler

If your app uses scheduled tasks:

```bash
crontab -e
```

Add:
```
* * * * * cd /var/www/laravel && php artisan schedule:run >> /dev/null 2>&1
```

---

## Deploy Updates

```bash
cd /var/www/laravel
git pull origin main
composer install --no-dev --optimize-autoloader
php artisan migrate --force
php artisan config:cache
php artisan route:cache
php artisan view:cache
sudo chown -R www-data:www-data storage bootstrap/cache
sudo systemctl reload php8.3-fpm
```

---

## Troubleshooting

**500 error / permissions issue:**
```bash
sudo chown -R www-data:www-data /var/www/laravel
sudo chmod -R 775 storage bootstrap/cache
```

**Check Laravel logs:**
```bash
tail -f /var/www/laravel/storage/logs/laravel.log
```

**Check PHP-FPM:**
```bash
sudo systemctl status php8.3-fpm
sudo tail -f /var/log/php8.3-fpm.log
```

---

## Quick Reference

| Task | Command |
|------|---------|
| Clear all cache | `php artisan optimize:clear` |
| Cache config/routes | `php artisan optimize` |
| Run migrations | `php artisan migrate` |
| Check PHP-FPM | `sudo systemctl status php8.3-fpm` |
| View app logs | `tail -f storage/logs/laravel.log` |
