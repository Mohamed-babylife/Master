
# Secure Nginx with Let's Encrypt on Ubuntu (Wildcard SSL for All Domains and Subdomains)

## 🚀 Step 1: Install Nginx Web Server
Update packages and install Nginx:
```bash
sudo apt update
sudo apt install nginx
```

Enable Nginx to start on boot and check its status:
```bash
sudo systemctl enable nginx
sudo systemctl start nginx
sudo systemctl status nginx
```

Allow HTTP and HTTPS through the firewall:
```bash
sudo ufw allow 'Nginx Full'
```

---

## 🔒 Step 2: Install Certbot for SSL
Install Certbot and the Nginx plugin:
```bash
sudo apt install certbot python3-certbot-nginx
```

---

## 📜 Step 3: Configure Nginx for Your Domain
Create a new Nginx config for your domain:
```bash
sudo nano /etc/nginx/sites-available/primepacc.com
```

Add the following basic configuration:
```nginx
upstream backend {
    server 127.0.0.1:8069;
}

server {
    listen 80;
    server_name primepacc.com *.primepacc.com;

    access_log /var/log/nginx/access.log;
    error_log /var/log/nginx/error.log;
    client_max_body_size 100M;

    location / {
        proxy_pass http://127.0.0.1:8069;
        proxy_set_header Host $host;
    }

    location /longpolling {
        proxy_pass http://127.0.0.1:8072;
    }

    location ~* /web/static/ {
        proxy_cache_valid 200 90m;
        expires 864000;
        proxy_pass http://127.0.0.1:8069;
    }
}
```

Enable the site and test the config:
```bash
sudo ln -s /etc/nginx/sites-available/primepacc.com /etc/nginx/sites-enabled/
sudo nginx -t
sudo systemctl reload nginx
```

---

## 🌐 Step 4: Get a Wildcard SSL Certificate
Request a wildcard SSL certificate:
```bash
sudo certbot certonly --manual -d primepacc.com -d *.primepacc.com --preferred-challenges dns-01
```

Certbot will ask you to create a DNS TXT record — go to your domain registrar’s DNS settings and add the following record:

- **Name:** `_acme-challenge`
- **Type:** `TXT`
- **Value:** (Certbot will give you the value)

Once the DNS record propagates, press **Enter** to continue.

---

## 🔐 Step 5: Secure Nginx with SSL
Edit your Nginx configuration again:
```bash
sudo nano /etc/nginx/sites-available/primepacc.com
```

Update it for HTTPS:
```nginx
upstream backend {
    server 127.0.0.1:8069;
}

server {
    listen 80;
    server_name primepacc.com *.primepacc.com;
    return 301 https://$host$request_uri;
}

server {
    listen 443 ssl;
    server_name primepacc.com *.primepacc.com;

    ssl_certificate /etc/letsencrypt/live/primepacc.com/fullchain.pem;
    ssl_certificate_key /etc/letsencrypt/live/primepacc.com/privkey.pem;
    ssl_protocols TLSv1.2 TLSv1.3;
    ssl_ciphers 'ECDHE-ECDSA-AES128-GCM-SHA256:ECDHE-RSA-AES128-GCM-SHA256';
    ssl_prefer_server_ciphers on;

    access_log /var/log/nginx/access.log;
    error_log /var/log/nginx/error.log;
    client_max_body_size 100M;

    location / {
        proxy_pass http://127.0.0.1:8069;
        proxy_set_header Host $host;
    }

    location /longpolling {
        proxy_pass http://127.0.0.1:8072;
    }

    location ~* /web/static/ {
        proxy_cache_valid 200 90m;
        expires 864000;
        proxy_pass http://127.0.0.1:8069;
    }
}
```

Test and reload Nginx:
```bash
sudo nginx -t
sudo systemctl reload nginx
```

---

## 🔄 Step 6: Enable Auto-Renewal for SSL
Test auto-renewal:
```bash
sudo certbot renew --dry-run
```

If successful, Certbot will auto-renew certificates every 90 days.

---

## ✅ Step 7: Verify HTTPS
Visit your domain:
- `https://primepacc.com`
- `https://subdomain.primepacc.com`

Check the SSL lock icon in the browser to confirm it's secure.

---

Would you like to add extra security layers like **HSTS** or **SSL hardening**? Let me know, and we can fine-tune the setup! 🚀
