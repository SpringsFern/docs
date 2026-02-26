# Setup Nginx Reverse Proxy (IP Based Setup)

In this step, we configure Nginx to:

- Reverse proxy requests to TG-FileStream workers
- Load balance multiple worker instances
- Handle large file streaming properly

⚠ At this stage, we are NOT using a domain yet.
We will configure using the server IP address.

---

# 1. Install Nginx

```bash
sudo apt update
sudo apt install nginx -y
```

Enable and start:

```bash
sudo systemctl enable nginx
sudo systemctl start nginx
```

---

# 2. Ensure Workers Are Running

Make sure these services are active:

```bash
sudo systemctl status tgfs
sudo systemctl status tgfs-worker@8081
sudo systemctl status tgfs-worker@8082
sudo systemctl status tgfs-worker@8083
```

Workers should be running on:

- 127.0.0.1:8080
- 127.0.0.1:8081
- 127.0.0.1:8082
- 127.0.0.1:8083

---

# 3. Create Nginx Configuration

Create a new site configuration:

```bash
sudo nano /etc/nginx/sites-available/tgfs
```

Paste the following:

```nginx
upstream aiohttp_backend {
    least_conn;
    keepalive 32;

    server 127.0.0.1:8080 max_fails=3 fail_timeout=30s;
    server 127.0.0.1:8081 max_fails=3 fail_timeout=30s;
    server 127.0.0.1:8082 max_fails=3 fail_timeout=30s;
    server 127.0.0.1:8083 max_fails=3 fail_timeout=30s;
}

server {
    listen 80;
    server_name _;

    location / {
        proxy_pass http://aiohttp_backend;

        proxy_http_version 1.1;
        proxy_buffering off;
        proxy_redirect off;

        proxy_set_header Host $host;
        proxy_set_header X-Real-IP $remote_addr;
        proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;
        proxy_set_header X-Forwarded-Proto $scheme;
    }
}
```

Explanation:

- `least_conn` → Sends traffic to worker with least active connections
- `keepalive 32` → Keeps upstream connections alive
- `proxy_buffering off` → Prevents buffering large streams

---

# 4. Enable Site

```bash
sudo ln -s /etc/nginx/sites-available/tgfs /etc/nginx/sites-enabled/
```

Remove default site (recommended):

```bash
sudo rm /etc/nginx/sites-enabled/default
```

---

# 5. Test Configuration

```bash
sudo nginx -t
```

If successful:

```
syntax is ok
test is successful
```

Reload Nginx:

```bash
sudo systemctl reload nginx
```

---

# 6. Test Using Server IP

Find your server IP:

```bash
curl ifconfig.me
```

Now open in browser:

```
http://<your-server-ip>
```

Or test:

```bash
curl http://<your-server-ip>
```

If configured correctly, requests should be forwarded to your workers.

---

# How Load Balancing Works Here

User
  ↓
Nginx (Port 80)
  ↓
aiohttp_backend (least_conn)
  ↓
Workers 8080–8083
  ↓
Telegram

Nginx distributes traffic based on active connections.

---

# Scaling Workers

To add more workers:

1. Start new worker:

```bash
sudo systemctl start tgfs-worker@8084
```

2. Add to upstream block:

```
server 127.0.0.1:8084 max_fails=3 fail_timeout=30s;
```

3. Reload Nginx:

```bash
sudo systemctl reload nginx
```

---

# Troubleshooting

### 502 Bad Gateway

Check worker status:

```bash
sudo systemctl status tgfs-worker@8081
```

Check logs:

```bash
sudo journalctl -u tgfs-worker@8081 -f
```

### Nginx Errors

```bash
sudo nginx -t
sudo journalctl -u nginx -f
```
