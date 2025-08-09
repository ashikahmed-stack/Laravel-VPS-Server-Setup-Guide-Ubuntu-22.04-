# Nuxt 3 Deployment on VPS with PM2 & CyberPanel

# OpenLiteSpeed Admin Password Reset Guide

## Purpose
This guide explains how to reset the OpenLiteSpeed WebAdmin Console password.

---

## Steps to Reset OpenLiteSpeed Admin Password

### 1. Login to your server via SSH

Use your preferred SSH client to connect to your VPS or dedicated server.

---

### 2. Run the password reset script

Execute the following command in the terminal:

```bash
/usr/local/lsws/admin/misc/admpass.sh


---

## 1. Clean up existing PM2 processes
```bash
pm2 delete grameen
pm2 delete grameenschool


# 2. Install Node.js (Latest LTS)

# Update system
apt update && apt upgrade -y

# Install Node.js LTS (e.g., 20.x)
curl -fsSL https://deb.nodesource.com/setup_20.x | bash -
apt install -y nodejs

# Verify versions
node -v
npm -v

# 3. Install PM2 globally
npm install -g pm2
pm2 -v


# 4. Navigate to your Nuxt project directory
cd /home/grameenschool.com/public_html


# 5. Start your Nuxt app with PM2
For a typical Nuxt 3 production server start:

pm2 start .output/server/index.mjs --name "grameenschool" -- --hostname 0.0.0.0 --port 3000

## If you want to use npm start script (if defined):
pm2 start npm --name "nuxt-app" -- run start

# 6. Save PM2 process list & enable startup on boot
pm2 save
pm2 startup

# 7. Configure Reverse Proxy with Nginx (Port 80 → Port 3000)
Create Nginx config:

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


Enable and restart Nginx:

ln -s /etc/nginx/sites-available/yourdomain.com /etc/nginx/sites-enabled/
nginx -t
systemctl restart nginx

# 8. Optional: Enable HTTPS with Let's Encrypt

apt install certbot python3-certbot-nginx -y
certbot --nginx -d yourdomain.com -d www.yourdomain.com

# 9. PM2 Useful Commands
pm2 list                  # List running processes
pm2 stop nuxt-app         # Stop the app
pm2 restart nuxt-app      # Restart the app
pm2 delete nuxt-app       # Remove from PM2
pm2 logs nuxt-app         # View logs
pm2 monit                 # Monitor live

Troubleshooting nghttpx Conflict (Port 3000 Already in Use)
১. nghttpx কি?
nghttpx হল HTTP/2 proxy server, যা হয়তো CyberPanel বা OpenLiteSpeed-এ চলছে এবং 3000 পোর্ট ব্যবহার করছে।

২. nghttpx বন্ধ বা অন্য পোর্টে সরান
sudo systemctl stop nghttpx
sudo systemctl disable nghttpx
তারপর নিশ্চিত হওয়া যাবে 3000 পোর্ট ফাঁকা কিনা:

sudo lsof -i :3000
৩. Nuxt app পোর্ট পরিবর্তন (যদি nghttpx বন্ধ করতে না পারো)
nuxt.config.ts এ অথবা স্টার্ট কমান্ডে পোর্ট পরিবর্তন করো:

server: {
  host: '0.0.0.0',
  port: 3001,
},
আর OpenLiteSpeed বা Nginx reverse proxy সেটিংসে সেই নতুন পোর্ট ব্যবহার করো:
http://127.0.0.1:3001

৪. Nuxt অ্যাপ পুনরায় চালানো
pm2 delete grameenschool
pm2 start .output/server/index.mjs --name "grameenschool" -- --hostname 0.0.0.0 --port 3001
pm2 save
৫. OpenLiteSpeed / Nginx রিস্টার্ট দিন

systemctl restart lsws    # OpenLiteSpeed restart
systemctl restart nginx   # Nginx restart (যদি nginx ব্যবহার হয়)
এই স্টেপগুলো ফলো করলে Nuxt.js অ্যাপ CyberPanel VPS-এ PM2 দিয়ে সঠিকভাবে চালু হবে এবং 3000 পোর্ট কনফ্লিক্ট সমস্যাও ফিক্স হবে।


