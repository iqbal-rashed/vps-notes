# Deploy Python Flask/Django

Complete guide for deploying Python web applications with Gunicorn.

---

## 1. Install Python

```bash
sudo apt update
sudo apt install python3 python3-pip python3-venv -y
python3 --version
```

---

## 2. Project Setup

```bash
# Create directory
sudo mkdir -p /var/www/myapp
sudo chown -R $USER:$USER /var/www/myapp
cd /var/www/myapp

# Clone project
git clone https://github.com/user/repo.git .

# Create virtual environment
python3 -m venv venv
source venv/bin/activate

# Install dependencies
pip install -r requirements.txt
pip install gunicorn
```

---

## 3. Gunicorn Setup

### Flask

```bash
gunicorn --bind 0.0.0.0:5000 app:app
```

### Django

```bash
gunicorn --bind 0.0.0.0:8000 myproject.wsgi:application
```

### Production Config

```bash
# /var/www/myapp/gunicorn.conf.py
bind = "127.0.0.1:5000"
workers = 3
threads = 2
worker_class = "sync"
timeout = 30
accesslog = "/var/log/gunicorn/access.log"
errorlog = "/var/log/gunicorn/error.log"
```

---

## 4. Systemd Service

```bash
sudo nano /etc/systemd/system/myapp.service
```

```ini
[Unit]
Description=Gunicorn instance for myapp
After=network.target

[Service]
User=www-data
Group=www-data
WorkingDirectory=/var/www/myapp
Environment="PATH=/var/www/myapp/venv/bin"
ExecStart=/var/www/myapp/venv/bin/gunicorn --config gunicorn.conf.py app:app

[Install]
WantedBy=multi-user.target
```

```bash
sudo systemctl daemon-reload
sudo systemctl start myapp
sudo systemctl enable myapp
sudo systemctl status myapp
```

---

## 5. Nginx Configuration

```nginx
server {
    listen 80;
    server_name example.com;

    location / {
        proxy_pass http://127.0.0.1:5000;
        proxy_set_header Host $host;
        proxy_set_header X-Real-IP $remote_addr;
        proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;
    }

    location /static {
        alias /var/www/myapp/static;
        expires 30d;
    }
}
```

```bash
sudo ln -s /etc/nginx/sites-available/myapp /etc/nginx/sites-enabled/
sudo nginx -t
sudo systemctl reload nginx
```

---

## 6. Environment Variables

```bash
# .env file
SECRET_KEY=your-secret-key
DATABASE_URL=postgresql://user:pass@localhost/db
DEBUG=False
```

```python
# Load in Python
from dotenv import load_dotenv
load_dotenv()
```

---

## 7. Django Specific

```bash
# Collect static files
python manage.py collectstatic

# Run migrations
python manage.py migrate

# Create superuser
python manage.py createsuperuser
```

---

## Quick Reference

| Task | Command |
|------|---------|
| Activate venv | `source venv/bin/activate` |
| Run dev | `flask run` or `python manage.py runserver` |
| Run prod | `gunicorn app:app` |
| Restart | `sudo systemctl restart myapp` |
| Logs | `sudo journalctl -u myapp -f` |

---

✅ Python app deployed!
