# NGINX quick reference – My Most Used Configs
==============================================

NGINX can be slightly overwhelming at first — blocks, directives, and all those semicolons. But once you learn the key parts, it becomes a powerful tool for reverse proxies, load balancing, static hosting, and more.

This is a quick reference of the **NGINX config snippets I use the most** when setting up websites, proxies, or microservices.

> 📌 This is a **quick reference**, not a deep-dive guide.

* * *

🔧 Basic Setup
--------------

### Install NGINX (Linux)
```bash
sudo apt update
sudo apt install nginx
```

### Start / Stop / Restart
```bash
sudo systemctl start nginx
sudo systemctl restart nginx
sudo systemctl stop nginx
```

### Check Status
```bash
sudo systemctl status nginx
```
* * *

🌐 Static Site Hosting
----------------------
```nginx.conf
server {
    listen 80;
    server_name example.com;

    root /var/www/html;
    index index.html;

    location / {
        try_files $uri $uri/ =404;
    }
}
```
Place your static files in `/var/www/html`.

* * *

🔁 Reverse Proxy
----------------
```nginx.conf
server {
    listen 80;
    server_name api.example.com;

    location / {
        proxy_pass http://localhost:3000;
        proxy_http_version 1.1;
        proxy_set_header Upgrade $http_upgrade;
        proxy_set_header Connection 'upgrade';
        proxy_set_header Host $host;
        proxy_cache_bypass $http_upgrade;
    }
}
```
This will forward all traffic from `api.example.com` to your backend server on port 3000.

* * *

🔒 SSL with Let's Encrypt (Certbot)
-----------------------------------
```bash
sudo apt install certbot python3-certbot-nginx
sudo certbot --nginx -d example.com -d www.example.com
```

This will automatically configure HTTPS for your domain.

* * *

📁 Serving Multiple Sites (Virtual Hosts)
-----------------------------------------

Create separate config files in `/etc/nginx/sites-available/`:

    /etc/nginx/sites-available/site1.com
    /etc/nginx/sites-available/site2.com

Link them to `sites-enabled`:
```bash
sudo ln -s /etc/nginx/sites-available/site1.com /etc/nginx/sites-enabled/
```

Then reload:
```bash
sudo nginx -t
sudo systemctl reload nginx
```

* * *

🔍 Debugging & Logs
-------------------

*   `/var/log/nginx/access.log`
*   `/var/log/nginx/error.log`

To test config syntax:
```bash
sudo nginx -t
```

* * *

⚙️ Rate Limiting Example
------------------------
```nginx.conf
    http {
        limit_req_zone $binary_remote_addr zone=one:10m rate=1r/s;
    
        server {
            location /api/ {
                limit_req zone=one burst=5 nodelay;
                proxy_pass http://localhost:3000;
            }
        }
    }
```

This limits each IP to 1 request/sec with a burst capacity of 5.

* * *

🔥 Caching Static Assets
------------------------
```nginx.conf
    location ~* \\.(jpg|jpeg|png|gif|ico|css|js)$ {
        expires 30d;
        access_log off;
    }
```

* * *

💡 Bonus Tips & Tricks
----------------------

*   `sudo nginx -s reload` → reload config without restarting
*   `try_files $uri $uri/ =404;` → safer fallback for SPAs
*   Use `include snippets/fastcgi-php.conf;` for quick PHP setup
*   Use `proxy_cache` and `proxy_cache_valid` for upstream caching

* * *

🏁 Final Tip
------------

Start small — one server block, one purpose. Test configs with `nginx -t`, and build confidence slowly. Once you master the basics, NGINX feels like a Swiss army knife for your web infrastructure.

> Happy serving!