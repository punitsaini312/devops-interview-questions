# Nginx Interview Questions

## Beginner

### 1. What is Nginx?

Nginx is a web server and reverse proxy server. It can serve static
content, terminate TLS, load-balance traffic, and forward requests to
backend applications.

### 2. What is the difference between a web server and a reverse proxy?

A web server can directly serve content such as HTML, CSS, JavaScript,
and images.

A reverse proxy receives client requests and forwards them to backend
servers/applications.

### 3. What is `nginx.conf`?

It is the main Nginx configuration file.

Usually:

``` text
/etc/nginx/nginx.conf
```

### 4. What is a `server` block?

A `server` block defines configuration for a virtual server/website.

### 5. What is a `location` block?

A `location` block defines how Nginx handles requests matching a
particular URI path.

### 6. What is the purpose of `root`?

`root` tells Nginx where static files are located.

Example:

``` nginx
root /var/www/html/demo;
```

### 7. What is `server_name`?

It specifies the hostname/domain for which a server block should handle
requests.

Example:

``` nginx
server_name example.com;
```

### 8. What is `proxy_pass`?

`proxy_pass` tells Nginx to forward a request to another
server/application.

Example:

``` nginx
proxy_pass http://127.0.0.1:3000;
```

------------------------------------------------------------------------

## Intermediate

### 9. What is the difference between `reload` and `restart`?

`reload` loads the new configuration while keeping the existing Nginx
service running.

`restart` stops and starts the service again.

For normal configuration changes:

``` bash
sudo nginx -t
sudo systemctl reload nginx
```

is generally preferred.

### 10. What are `sites-available` and `sites-enabled`?

`sites-available` contains available site configurations.

`sites-enabled` contains configurations that are enabled and loaded by
Nginx.

On Debian/Ubuntu, `sites-enabled` commonly contains symbolic links to
`sites-available`.

### 11. How do you check whether an Nginx configuration is valid?

``` bash
sudo nginx -t
```

### 12. Where are Nginx logs?

Usually:

``` text
/var/log/nginx/access.log
/var/log/nginx/error.log
```

### 13. How can Nginx host multiple websites?

Using multiple `server` blocks with different `server_name` values.

Example:

``` nginx
server {
    listen 80;
    server_name site1.com;
    ...
}

server {
    listen 80;
    server_name site2.com;
    ...
}
```

### 14. What happens when a user requests `/about`?

Nginx first determines the appropriate `server` block and then selects
the matching `location` block.

If serving static files, the URL is mapped against the configured
`root`.

Example:

``` text
root: /var/www/html/demo
URL:  /about/
```

Result:

``` text
/var/www/html/demo/about/
```

### 15. Why do we run `nginx -t` before reload?

To verify that the configuration syntax is valid before applying it.
This helps avoid reloading a broken configuration.

------------------------------------------------------------------------

# 13. Quick Revision

``` text
Install
   ↓
apt install nginx
   ↓
nginx.conf
   ↓
events + http
   ↓
server
   ↓
location
```

### Static website

``` text
Browser
   ↓
Nginx
   ↓
root
   ↓
HTML/CSS/JS files
```

### Reverse proxy

``` text
Browser
   ↓
Nginx
   ↓
proxy_pass
   ↓
Backend application
```

### Important commands

``` bash
sudo systemctl start nginx
sudo systemctl stop nginx
sudo systemctl restart nginx
sudo systemctl reload nginx
sudo systemctl status nginx

sudo nginx -t
```

### Important paths

``` text
/etc/nginx/nginx.conf
/etc/nginx/sites-available/
/etc/nginx/sites-enabled/
/etc/nginx/conf.d/
/var/www/html/
/var/log/nginx/access.log
/var/log/nginx/error.log
```

### Most important mental model

``` text
Client
  ↓
Nginx
  ↓
server block
  ↓
location block
  ↓
 ┌──────────────────┐
 │                  │
root             proxy_pass
 │                  │
 ↓                  ↓
Static files      Backend
```

**Remember:**

> `server` decides **which website/server configuration** handles the
> request.

> `location` decides **how a particular URL/path is handled**.

> `root` tells Nginx **where static files are located**.

> `proxy_pass` tells Nginx **where to forward the request**.
