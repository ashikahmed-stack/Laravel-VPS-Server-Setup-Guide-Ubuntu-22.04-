pm2 delete grameen
pm2 delete grameenschool



rewrite {
    reverse_proxy http://127.0.0.1:3000
}


cat: /home/grameenschool.com/public_html/.htaccess: No such file or directory


#Server root এ PM2
npm install -g pm2

# root
cd /home/grameenschool.com/public_html
pm2 start npm --name "nuxt-app" -- run start


# 🚀 Nuxt 3 Deployment on VPS with PM2

## 1. Install Node.js (Latest LTS)
```bash
# Update system
apt update && apt upgrade -y

# Install Node.js LTS (e.g., 20.x)
curl -fsSL https://deb.nodesource.com/setup_20.x | bash -
apt install -y nodejs

# Check versions
node -v
npm -v


# 2. Install PM2 Globally
npm install -g pm2
pm2 -v

# 3. Upload / Clone Nuxt Project

cd /home/yourdomain.com/public_html
# If using git
git clone https://github.com/username/your-nuxt-app.git .

# 4. Install Dependencies & Build

npm install
npm run build

# 5. Start with PM2
pm2 start npm --name "nuxt-app" -- run start

# 6. Save PM2 Process
pm2 save
pm2 startup

# 7. Reverse Proxy with Nginx (Port 80 → Port 3000)
nano /etc/nginx/sites-available/yourdomain.com
server {
    listen 80;
    server_name yourdomain.com www.yourdomain.com;

    location / {
        proxy_pass http://127.0.0.1:3000;
        proxy_http_version 1.1;
        proxy_set_header Upgrade $http_upgrade;
        proxy_set_header Connection 'upgrade';
        proxy_set_header Host $host;
        proxy_cache_bypass $http_upgrade;
    }
}

Enable site & restart Nginx:
ln -s /etc/nginx/sites-available/yourdomain.com /etc/nginx/sites-enabled/
nginx -t
systemctl restart nginx

# 8. Optional: Enable HTTPS (Let's Encrypt)
apt install certbot python3-certbot-nginx -y
certbot --nginx -d yourdomain.com -d www.yourdomain.com

# 9. PM2 Commands Quick Reference
pm2 list                  # Running processes দেখাবে
pm2 stop nuxt-app         # Stop
pm2 restart nuxt-app      # Restart
pm2 delete nuxt-app       # Remove
pm2 logs nuxt-app         # View logs
pm2 monit                 # Live monitor
