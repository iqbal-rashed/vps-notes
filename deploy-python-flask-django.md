# Deploy Python Flask or Django App

Python web apps use **Gunicorn** to run, and **Nginx** to receive web traffic. A systemd service keeps the app running 24/7.

---

## Step 1 — Install Python

```bash
sudo apt update
sudo apt install python3 python3-pip python3-venv -y
python3 --version
```

---

## Step 2 — Upload Your App

```bash
# Create the app directory
sudo mkdir -p /var/www/myapp
sudo chown -R $USER:$USER /var/www/myapp
cd /var/www/myapp

# Clone from GitHub
git clone https://github.com/yourusername/your-repo.git .
```

---

## Step 3 — Set Up Virtual Environment

A virtual environment keeps your app's dependencies separate from the system.

```bash
cd /var/www/myapp

# Create virtual environment
python3 -m venv venv

# Activate it
source venv/bin/activate

# Install your app's dependencies
pip install -r requirements.txt

# Install Gunicorn (the app server)
pip install gunicorn
```

---

## Step 4 — Environment Variables

```bash
nano /var/www/myapp/.env
```

```env
SECRET_KEY=your-very-long-random-secret
DEBUG=False
DATABASE_URL=postgresql://user:password@localhost:5432/myapp
ALLOWED_HOSTS=example.com,www.example.com
```

Secure it:
```bash
chmod 600 /var/www/myapp/.env
```

---

## Step 5 — Run Database Migrations

### Django:
```bash
source venv/bin/activate
python manage.py migrate
python manage.py collectstatic --noinput
```

### Flask:
```bash
source venv/bin/activate
flask db upgrade   # If using Flask-Migrate
```

---

## Step 6 — Test the App Works

```bash
# Flask
source venv/bin/activate
gunicorn --bind 127.0.0.1:5000 app:app

# Django
source venv/bin/activate
gunicorn --bind 127.0.0.1:8000 myproject.wsgi:application
```

Visit `http://YOUR_SERVER_IP:5000` (or 8000) to verify. Press Ctrl+C to stop.

---

## Step 7 — Create a Systemd Service

This makes your app start automatically on boot.

```bash
sudo nano /etc/systemd/system/myapp.service
```

### Flask:
```ini
[Unit]
Description=Flask app - myapp
After=network.target

[Service]
User=www-data
Group=www-data
WorkingDirectory=/var/www/myapp
EnvironmentFile=/var/www/myapp/.env
ExecStart=/var/www/myapp/venv/bin/gunicorn \
    --workers 3 \
    --bind 127.0.0.1:5000 \
    --access-logfile /var/log/gunicorn/myapp-access.log \
    --error-logfile /var/log/gunicorn/myapp-error.log \
    app:app
Restart=always

[Install]
WantedBy=multi-user.target
```

### Django:
```ini
[Unit]
Description=Django app - myapp
After=network.target

[Service]
User=www-data
Group=www-data
WorkingDirectory=/var/www/myapp
EnvironmentFile=/var/www/myapp/.env
ExecStart=/var/www/myapp/venv/bin/gunicorn \
    --workers 3 \
    --bind 127.0.0.1:8000 \
    --access-logfile /var/log/gunicorn/myapp-access.log \
    --error-logfile /var/log/gunicorn/myapp-error.log \
    myproject.wsgi:application
Restart=always

[Install]
WantedBy=multi-user.target
```

Create log directory and start:
```bash
sudo mkdir -p /var/log/gunicorn
sudo chown www-data:www-data /var/log/gunicorn

sudo systemctl daemon-reload
sudo systemctl start myapp
sudo systemctl enable myapp
sudo systemctl status myapp
```

---

## Step 8 — Configure Nginx

```bash
sudo nano /etc/nginx/sites-available/myapp
```

```nginx
server {
    listen 80;
    server_name example.com www.example.com;

    # Security headers
    add_header X-Content-Type-Options "nosniff" always;
    add_header X-Frame-Options "SAMEORIGIN" always;

    # For Django static/media files
    location /static/ {
        alias /var/www/myapp/staticfiles/;
        expires 30d;
    }

    location /media/ {
        alias /var/www/myapp/media/;
    }

    location / {
        proxy_pass http://127.0.0.1:8000;   # Change to 5000 for Flask
        proxy_set_header Host $host;
        proxy_set_header X-Real-IP $remote_addr;
        proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;
        proxy_set_header X-Forwarded-Proto $scheme;
    }
}
```

Enable it:
```bash
sudo ln -s /etc/nginx/sites-available/myapp /etc/nginx/sites-enabled/
sudo nginx -t
sudo systemctl reload nginx
```

---

## Step 9 — Add SSL

```bash
sudo certbot --nginx -d example.com -d www.example.com
```

---

## Deploy Updates

```bash
cd /var/www/myapp
git pull origin main
source venv/bin/activate
pip install -r requirements.txt
python manage.py migrate       # Django only
python manage.py collectstatic --noinput  # Django only
sudo systemctl restart myapp
```

---

## Troubleshooting

**App not starting:**
```bash
sudo systemctl status myapp
sudo journalctl -u myapp -n 50
sudo tail -f /var/log/gunicorn/myapp-error.log
```

**Static files not loading (Django):**
```bash
python manage.py collectstatic
sudo chown -R www-data:www-data /var/www/myapp/staticfiles
```

---

## Quick Reference

| Task | Command |
|------|---------|
| Start app | `sudo systemctl start myapp` |
| Check status | `sudo systemctl status myapp` |
| View logs | `sudo tail -f /var/log/gunicorn/myapp-error.log` |
| Reload after code change | `sudo systemctl restart myapp` |
