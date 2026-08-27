# Nginx Notes --- Revision Handbook

## 1. Nginx Installation

### Ubuntu / Debian

``` bash
sudo apt update
sudo apt install nginx
```

Check installation:

``` bash
nginx -v
```

Check where Nginx is installed:

``` bash
which nginx
```

------------------------------------------------------------------------

## 2. Nginx Service Management

Nginx is normally managed using `systemctl`.

### Start Nginx

``` bash
sudo systemctl start nginx
```

### Stop Nginx

``` bash
sudo systemctl stop nginx
```

### Restart Nginx

``` bash
sudo systemctl restart nginx
```

This stops and starts Nginx again.

### Reload Nginx

``` bash
sudo systemctl reload nginx
```

Reload applies configuration changes without completely stopping Nginx.

**Important:** After changing an Nginx configuration, prefer:

``` bash
sudo nginx -t
sudo systemctl reload nginx
```

### Check status

``` bash
sudo systemctl status nginx
```

### Enable Nginx at boot

``` bash
sudo systemctl enable nginx
```

### Disable Nginx at boot

``` bash
sudo systemctl disable nginx
```

### Check configuration syntax

``` bash
sudo nginx -t
```

Typical successful output:

``` text
syntax is ok
test is successful
```

------------------------------------------------------------------------

# 3. Basic Structure of `nginx.conf`

Main configuration file:

``` text
/etc/nginx/nginx.conf
```

A simplified structure looks like:

``` nginx
events {
    # Connection-related settings
}

http {

    # HTTP-level settings

    server {

        # Website/server configuration

        listen 80;
        server_name example.com;

        location / {
            # Configuration for /
        }

        location /about {
            # Configuration for /about
        }
    }

    server {
        # Another website
    }
}
```

## Important hierarchy

Remember:

``` text
nginx
│
├── events
│
└── http
    │
    ├── server
    │   ├── location /
    │   └── location /about
    │
    └── server
        └── location /
```

### `events`

Contains settings related to how Nginx handles connections.

Example:

``` nginx
events {
    worker_connections 1024;
}
```

### `http`

Contains HTTP/web-server configuration.

For example:

``` nginx
http {
    include /etc/nginx/mime.types;

    server {
        ...
    }
}
```

### `server`

A `server` block represents a virtual server / website.

Example:

``` nginx
server {
    listen 80;
    server_name example.com;

    root /var/www/html/demo;

    location / {
        try_files $uri $uri/ =404;
    }
}
```

Multiple websites can have multiple `server` blocks.

### `location`

Controls how Nginx handles a particular URL path.

Example:

``` nginx
location / {
    ...
}

location /about {
    ...
}
```

Then:

``` text
http://example.com/
        ↓
location /

http://example.com/about
        ↓
location /about
```

------------------------------------------------------------------------

# 4. Important Nginx Files and Directories

## Main configuration

``` text
/etc/nginx/nginx.conf
```

This is the main Nginx configuration file.

## Sites available

``` text
/etc/nginx/sites-available/
```

Contains server configurations that are available to use.

Example:

``` text
/etc/nginx/sites-available/default
```

## Sites enabled

``` text
/etc/nginx/sites-enabled/
```

Contains configurations that Nginx actually loads.

Usually, files here are symbolic links to files in `sites-available`.

Check:

``` bash
ls -l /etc/nginx/sites-enabled/
```

Typical relationship:

``` text
sites-available/default
        ↑
        │ symbolic link
        │
sites-enabled/default
```

## Default web root

On Ubuntu/Debian, a common default web root is:

``` text
/var/www/html/
```

Example:

``` text
/var/www/html/index.html
```

## Nginx logs

Access log:

``` text
/var/log/nginx/access.log
```

Error log:

``` text
/var/log/nginx/error.log
```

Useful commands:

``` bash
sudo tail -f /var/log/nginx/access.log
```

``` bash
sudo tail -f /var/log/nginx/error.log
```

## Other useful directory

``` text
/etc/nginx/conf.d/
```

Can contain additional `.conf` configuration files.

------------------------------------------------------------------------

# 5. My Practical Nginx Exercise

## Step 1 --- Create a demo website

Create a directory:

``` bash
sudo mkdir -p /var/www/html/demo
```

Create an HTML file:

``` bash
sudo nano /var/www/html/demo/index.html
```

Example:

``` html
<h1>This is my Demo Website</h1>
```

------------------------------------------------------------------------

## Step 2 --- Remove the default `index.html`

The default Nginx page is normally:

``` text
/var/www/html/index.html
```

Remove it:

``` bash
sudo rm /var/www/html/index.html
```

Now the default website content is removed.

------------------------------------------------------------------------

## Step 3 --- Change the Nginx default configuration

Edit:

``` bash
sudo nano /etc/nginx/sites-available/default
```

On Ubuntu/Debian, the enabled configuration is usually linked as:

``` text
/etc/nginx/sites-enabled/default
```

A basic configuration:

``` nginx
server {
    listen 80;
    listen [::]:80;

    root /var/www/html/demo;
    index index.html;

    server_name _;

    location / {
        try_files $uri $uri/ =404;
    }
}
```

The important change is:

``` nginx
root /var/www/html/demo;
```

Now when the browser requests:

``` text
http://SERVER_IP/
```

Nginx looks inside:

``` text
/var/www/html/demo/
```

and serves:

``` text
/var/www/html/demo/index.html
```

------------------------------------------------------------------------

## Step 4 --- Test and reload

Always test the configuration first:

``` bash
sudo nginx -t
```

Then:

``` bash
sudo systemctl reload nginx
```

------------------------------------------------------------------------

# 6. Practising URL Paths with `/about`

Now create an `about` directory:

``` bash
sudo mkdir /var/www/html/demo/about
```

Create:

``` bash
sudo nano /var/www/html/demo/about/index.html
```

Example:

``` html
<h1>About Page</h1>
```

Now the directory structure is:

``` text
/var/www/html/demo/
│
├── index.html
│
└── about/
    └── index.html
```

Because the Nginx root is:

``` nginx
root /var/www/html/demo;
```

these URLs map to:

``` text
http://SERVER_IP/
        ↓
/var/www/html/demo/index.html
```

and:

``` text
http://SERVER_IP/about/
        ↓
/var/www/html/demo/about/index.html
```

### Important concept

Nginx combines:

``` text
root + URL path
```

For:

``` text
root = /var/www/html/demo
URL  = /about/
```

Nginx looks approximately at:

``` text
/var/www/html/demo/about/
```

------------------------------------------------------------------------

# 7. Hosting a Second Website

Now create another website directory:

``` bash
sudo mkdir -p /var/www/html/demo/mysecondwebsite
```

Create its HTML file:

``` bash
sudo nano /var/www/html/demo/mysecondwebsite/index.html
```

Example:

``` html
<h1>My Second Website</h1>
```

Directory structure:

``` text
/var/www/html/demo/
│
├── index.html
│
├── about/
│   └── index.html
│
└── mysecondwebsite/
    └── index.html
```

There are two different concepts here:

### Case 1 --- Second website under a URL path

If you simply use the same `root`:

``` nginx
root /var/www/html/demo;

location / {
    try_files $uri $uri/ =404;
}
```

then:

``` text
http://SERVER_IP/mysecondwebsite/
```

serves:

``` text
/var/www/html/demo/mysecondwebsite/index.html
```

This is a **different URL path**, but technically it is still inside the
same `server` block.

------------------------------------------------------------------------

# 8. Adding a New `server` Block

To host a genuinely separate website, create another `server` block.

Example:

``` nginx
server {
    listen 80;
    server_name mysecondwebsite.com;

    root /var/www/html/demo/mysecondwebsite;
    index index.html;

    location / {
        try_files $uri $uri/ =404;
    }
}
```

Now you can have:

``` nginx
server {
    listen 80;
    server_name example.com;

    root /var/www/html/demo;

    location / {
        try_files $uri $uri/ =404;
    }
}

server {
    listen 80;
    server_name mysecondwebsite.com;

    root /var/www/html/demo/mysecondwebsite;

    location / {
        try_files $uri $uri/ =404;
    }
}
```

### How does Nginx decide which server block to use?

The browser sends the hostname:

``` text
Host: example.com
```

or:

``` text
Host: mysecondwebsite.com
```

Nginx uses the `server_name` to select the appropriate `server` block.

Conceptually:

``` text
example.com
      ↓
server_name example.com
      ↓
root /var/www/html/demo


mysecondwebsite.com
      ↓
server_name mysecondwebsite.com
      ↓
root /var/www/html/demo/mysecondwebsite
```

------------------------------------------------------------------------

# 9. Reverse Proxy

A reverse proxy is different from serving static HTML files.

Instead of:

``` text
Browser → Nginx → HTML file
```

Nginx can work like:

``` text
Browser
   │
   ▼
 Nginx
   │
   │ proxy_pass
   ▼
Application
```

For example, suppose your application is running on:

``` text
127.0.0.1:3000
```

You want users to access:

``` text
http://example.com/
```

through Nginx.

Configuration:

``` nginx
server {
    listen 80;
    server_name example.com;

    location / {
        proxy_pass http://127.0.0.1:3000;

        proxy_set_header Host $host;
        proxy_set_header X-Real-IP $remote_addr;
        proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;
        proxy_set_header X-Forwarded-Proto $scheme;
    }
}
```

Now:

``` text
Browser
   │
   │ http://example.com
   ▼
Nginx :80
   │
   │ proxy_pass
   ▼
Application :3000
```

## What changed?

For static website hosting:

``` nginx
root /var/www/html/demo;
```

Nginx reads files from the filesystem.

For reverse proxy:

``` nginx
proxy_pass http://127.0.0.1:3000;
```

Nginx forwards the request to another application/server.

### Example with `/api`

Suppose:

``` text
Frontend → Nginx
             │
             ├── /       → frontend/static files
             │
             └── /api/   → backend :8000
```

Configuration:

``` nginx
server {
    listen 80;
    server_name example.com;

    root /var/www/html/demo;

    location / {
        try_files $uri $uri/ =404;
    }

    location /api/ {
        proxy_pass http://127.0.0.1:8000;

        proxy_set_header Host $host;
        proxy_set_header X-Real-IP $remote_addr;
        proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;
        proxy_set_header X-Forwarded-Proto $scheme;
    }
}
```

Request:

``` text
GET /
```

→ Nginx serves the frontend.

Request:

``` text
GET /api/users
```

→ Nginx forwards it to the backend.

------------------------------------------------------------------------

# 10. Very Important Nginx Concepts

## Process vs Request

A request does **not** normally create a new Nginx process.

Nginx has long-running processes:

``` text
Nginx
│
├── Master process
│
├── Worker 1
├── Worker 2
├── Worker 3
└── Worker 4
```

Workers handle many requests.

Therefore:

``` text
1000 HTTP requests
        ≠
1000 Nginx processes
```

Every request can consume CPU and memory, but the existing Nginx workers
handle the requests.

------------------------------------------------------------------------

# 11. Useful Commands for Practice

Check Nginx version:

``` bash
nginx -v
```

Test configuration:

``` bash
sudo nginx -t
```

See running processes:

``` bash
ps aux | grep nginx
```

Check listening ports:

``` bash
sudo ss -tulpn | grep nginx
```

Check service:

``` bash
sudo systemctl status nginx
```

Reload configuration:

``` bash
sudo systemctl reload nginx
```

View access logs:

``` bash
sudo tail -f /var/log/nginx/access.log
```

View error logs:

``` bash
sudo tail -f /var/log/nginx/error.log
```

13. Using a Domain Name with Nginx

Instead of accessing the server using:

http://SERVER_IP

you can use a domain such as:

http://example.com
Step 1 — Point the domain to your server

In your domain/DNS provider, create an A record:

Type: A
Name: @
Value: YOUR_SERVER_PUBLIC_IP

For example:

example.com → 203.0.113.10

For www:

Type: A
Name: www
Value: YOUR_SERVER_PUBLIC_IP

Then:

www.example.com → 203.0.113.10

You can verify DNS resolution:

nslookup example.com

or:

dig example.com
Step 2 — Configure server_name

Edit your site configuration:

sudo nano /etc/nginx/sites-available/default

Instead of:

server_name _;

use:

server_name example.com www.example.com;

Example:

server {
    listen 80;
    listen [::]:80;

    server_name example.com www.example.com;

    root /var/www/html/demo;
    index index.html;

    location / {
        try_files $uri $uri/ =404;
    }
}

Now:

http://example.com/

will be handled by this server block.

Test:

sudo nginx -t

Then:

sudo systemctl reload nginx
14. Secure the Domain with Let's Encrypt

HTTP:

http://example.com

is not encrypted.

We want:

https://example.com

with a valid TLS certificate.

Let's Encrypt provides free TLS certificates.

A common way to obtain and configure them with Nginx is Certbot.

Step 1 — Install Certbot

Ubuntu/Debian:

sudo apt update
sudo apt install certbot python3-certbot-nginx

Check:

certbot --version
Step 2 — Make sure HTTP is working first

Before requesting the certificate, make sure:

http://example.com

works correctly.

Also make sure port 80 and 443 are allowed through your firewall/security group.

Step 3 — Request the certificate

Run:

sudo certbot --nginx -d example.com -d www.example.com

Certbot will:

Verify that you control the domain.
Obtain the Let's Encrypt certificate.
Configure Nginx for HTTPS.
Usually create an HTTP → HTTPS redirect if you choose that option.

Afterward:

http://example.com
       ↓
   redirect
       ↓
https://example.com
       ↓
     Nginx
15. Where are the Let's Encrypt Certificates?

Certbot normally stores them under:

/etc/letsencrypt/live/example.com/

Important files:

fullchain.pem
privkey.pem
fullchain.pem

This is the public certificate chain.

privkey.pem

This is the private key.

Never share privkey.pem.

You can check the files:

sudo ls -l /etc/letsencrypt/live/example.com/
16. Configure Certificate Paths in Nginx

After Certbot configuration, the HTTPS server block will contain something similar to:

server {
    listen 443 ssl;
    listen [::]:443 ssl;

    server_name example.com www.example.com;

    root /var/www/html/demo;
    index index.html;

    ssl_certificate /etc/letsencrypt/live/example.com/fullchain.pem;
    ssl_certificate_key /etc/letsencrypt/live/example.com/privkey.pem;

    location / {
        try_files $uri $uri/ =404;
    }
}

The important lines are:

ssl_certificate /etc/letsencrypt/live/example.com/fullchain.pem;

ssl_certificate_key /etc/letsencrypt/live/example.com/privkey.pem;

You can put these in the relevant HTTPS server block. They don't normally need to go into the global nginx.conf.

17. HTTP → HTTPS Redirect

Usually you don't want users to continue using:

http://example.com

Instead, redirect HTTP to HTTPS:

server {
    listen 80;
    listen [::]:80;

    server_name example.com www.example.com;

    return 301 https://$host$request_uri;
}

Then your HTTPS server:

server {
    listen 443 ssl;
    listen [::]:443 ssl;

    server_name example.com www.example.com;

    root /var/www/html/demo;
    index index.html;

    ssl_certificate /etc/letsencrypt/live/example.com/fullchain.pem;
    ssl_certificate_key /etc/letsencrypt/live/example.com/privkey.pem;

    location / {
        try_files $uri $uri/ =404;
    }
}

The complete flow becomes:

                    DNS
                     │
example.com ─────────┘
                     │
                     ▼
                Server :80
                     │
                     │ 301 Redirect
                     ▼
                Server :443
                     │
              TLS Certificate
                     │
                     ▼
                   Nginx
                     │
                     ▼
              /var/www/html/demo
18. Certificate Renewal

Let's Encrypt certificates are short-lived, so automatic renewal is important.

Check Certbot's renewal configuration:

sudo systemctl status certbot.timer

You can test renewal without actually renewing:

sudo certbot renew --dry-run

If the dry run succeeds, automatic renewal should be working.
------------------------------------------------------------------------

