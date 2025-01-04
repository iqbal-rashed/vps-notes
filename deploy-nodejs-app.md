## Install nodejs

```bash
curl -fsSL https://deb.nodesource.com/setup_18.x | sudo -E bash - &&\
sudo apt-get install -y nodejs
```

## Install yarn and pm2

```bash
sudo npm i -g yarn pm2
```

## Install nginx and start

```bash
sudo apt install nginx
sudo systemctl start nginx
```

## Setup nginx for project

```bash
sudo nano /etc/nginx/sites-available/example

server {

  listen 80;
  listen [::]:80;

  server_name example.com www.example.com;

  location / {
    proxy_pass http://localhost:3000;
    proxy_http_version 1.1;
    proxy_set_header Upgrade $http_upgrade;
    proxy_set_header Connection 'upgrade';
    proxy_set_header Host $host;
    proxy_cache_bypass $http_upgrade;
  }
}

sudo ln -s /etc/nginx/sites-available/example /etc/nginx/sites-enabled/

sudo systemctl restart nginx
```
