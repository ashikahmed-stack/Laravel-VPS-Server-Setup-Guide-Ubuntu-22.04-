# Laravel VPS Server Setup Guide (Ubuntu 22.04)

This guide walks through a full Laravel VPS deployment with PHP 8.2, MySQL, Redis, NGINX, SSL (Let's Encrypt), Mail server, and queue processing.

---

## Prerequisites

- VPS with **Ubuntu 22.04**
- Root SSH access
- A **domain** pointed to your VPS (e.g., `example.com`)

---

## 1. Connect to VPS

```bash
ssh root@your-vps-ip
```

---

## 2. System Setup

```bash
apt update && apt upgrade -y
apt install -y curl git unzip zip ufw software-properties-common
add-apt-repository ppa:ondrej/php -y
apt update
```

---

## 3. Install PHP 8.2 + Extensions

```bash
apt install -y php8.2 php8.2-fpm php8.2-cli php8.2-mysql php8.2-mbstring php8.2-xml \
php8.2-curl php8.2-bcmath php8.2-zip php8.2-gd php8.2-readline php8.2-tokenizer php8.2-common php8.2-opcache
```

---

## 4. Install MySQL

```bash
apt install -y mysql-server
mysql_secure_installation
```

```sql
CREATE DATABASE laravel;
CREATE USER 'laravel'@'localhost' IDENTIFIED BY 'secret';
GRANT ALL PRIVILEGES ON laravel.* TO 'laravel'@'localhost';
FLUSH PRIVILEGES;
```

---

## 5. Install Redis

```bash
apt install -y redis-server
systemctl enable redis
systemctl start redis
```

Update `.env`:

```env
CACHE_DRIVER=redis
QUEUE_CONNECTION=redis
```

---

## 6. Install Composer

```bash
curl -sS https://getcomposer.org/installer | php
mv composer.phar /usr/local/bin/composer
```

---

## 7. Laravel Setup

```bash
cd /var/www
git clone https://github.com/your-user/your-laravel-project.git example.com
cd example.com
composer install --no-dev --optimize-autoloader
cp .env.example .env
php artisan key:generate
php artisan config:cache
php artisan route:cache
php artisan view:cache
```

Permissions:

```bash
chown -R www-data:www-data /var/www/example.com
chmod -R 775 storage bootstrap/cache
```

---

## 8. NGINX Setup

```bash
apt install -y nginx
nano /etc/nginx/sites-available/example.com
```

Paste config:

```nginx
server {
    listen 80;
    server_name example.com www.example.com;
    root /var/www/example.com/public;

    index index.php;
    charset utf-8;

    add_header X-Frame-Options "SAMEORIGIN";
    add_header X-Content-Type-Options "nosniff";

    location / {
        try_files $uri $uri/ /index.php?$query_string;
    }

    location ~ \.php$ {
        include snippets/fastcgi-php.conf;
        fastcgi_pass unix:/run/php/php8.2-fpm.sock;
        fastcgi_param SCRIPT_FILENAME $realpath_root$fastcgi_script_name;
        include fastcgi_params;
    }

    location ~ /\. {
        deny all;
    }

    error_page 404 /index.php;
}
```

Enable site:

```bash
ln -s /etc/nginx/sites-available/example.com /etc/nginx/sites-enabled/
nginx -t && systemctl reload nginx
```

---

## 9. SSL with Let's Encrypt

```bash
apt install -y certbot python3-certbot-nginx
certbot --nginx -d example.com -d www.example.com
```

Auto-renewal:

```bash
echo "0 3 * * * root certbot renew --quiet" >> /etc/crontab
```

---

## 10. Supervisor for Laravel Queues

```bash
apt install -y supervisor
nano /etc/supervisor/conf.d/laravel-worker.conf
```

Config:

```ini
[program:laravel-worker]
process_name=%(program_name)s_%(process_num)02d
command=php /var/www/example.com/artisan queue:work redis --sleep=3 --tries=3
autostart=true
autorestart=true
user=www-data
numprocs=1
redirect_stderr=true
stdout_logfile=/var/www/example.com/storage/logs/worker.log
```

Enable:

```bash
supervisorctl reread
supervisorctl update
supervisorctl start laravel-worker:*
```

---

## 11. Mail Server Options

### Option A: Mailpit (for testing)

```bash
docker run -d -p 8025:8025 -p 1025:1025 --name mailpit axllent/mailpit
```

.env:

```env
MAIL_MAILER=smtp
MAIL_HOST=localhost
MAIL_PORT=1025
MAIL_USERNAME=null
MAIL_PASSWORD=null
MAIL_ENCRYPTION=null
```

Access: `http://your-server-ip:8025`

---

### Option B: Gmail SMTP

```env
MAIL_MAILER=smtp
MAIL_HOST=smtp.gmail.com
MAIL_PORT=587
MAIL_USERNAME=your@gmail.com
MAIL_PASSWORD=your-app-password
MAIL_ENCRYPTION=tls
```

---

## 12. Firewall Setup

```bash
ufw allow OpenSSH
ufw allow 'Nginx Full'
ufw enable
```

---

## Done

Laravel is now deployed and running at:

- `http://example.com`
- `https://example.com` (SSL secured)
- Mail: Mailpit or SMTP
- Queues: Running under Supervisor

