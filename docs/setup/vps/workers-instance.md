# Setup Worker Instances (Optional)

This step configures multiple TGFS worker instances for load balancing.

It allows:

- Running multiple server instances
- Distributing traffic using Nginx

⚠ This step is optional. You can skip it if a single instance is sufficient.

---

## 1. systemd Setup

### Service Architecture

TG-FileStream uses:

- `tgfs.service` → Main instance
- `tgfs-worker@.service` → Worker instances (templated)

Each worker:

- Runs on a different port
- Uses a separate session
- Can be load balanced via Nginx

---

### Create Worker Template Service

Create:

```bash
sudo nano /etc/systemd/system/tgfs-worker@.service
```

Paste:

```ini
[Unit]
Description=TG-FileStream Worker
After=network.target
Requires=tgfs.service

[Service]
Type=simple
User=<username>
Group=<group>

WorkingDirectory=<TG-FileStream Clone Path>

ExecStart=<TG-FileStream Clone Path>/<virtual-env-dir>/bin/python -m tgfs --port %i --no-update --session %i

Restart=always
RestartSec=3

NoNewPrivileges=true
PrivateTmp=true

StandardOutput=journal
StandardError=journal

[Install]
WantedBy=multi-user.target
```

Adjust:
- Replace `<username>` with your system username.
- Replace `<group>` with your group name (often same as username).
- Replace `<TG-FileStream Clone Path>` with the absolute path to your cloned TG-FileStream directory.
- Replace `<virtual-env-dir>` with the name of your virtual environment directory (usually `venv`).

---

??? info "Example"
    ```ini
    [Unit]
    Description=TG-FileStream Worker
    After=network.target
    Requires=tgfs.service

    [Service]
    Type=simple
    User=ubuntu
    Group=ubuntu

    WorkingDirectory=/home/ubuntu/tg-filestream

    ExecStart=/home/ubuntu/tg-filestream/venv/bin/python -m tgfs --port %i --no-update --session %i

    Restart=always
    RestartSec=3

    NoNewPrivileges=true
    PrivateTmp=true

    StandardOutput=journal
    StandardError=journal

    [Install]
    WantedBy=multi-user.target
    ```

---

### How the Worker Template Works

`%i` is replaced by the instance name.

Example:

```bash
sudo systemctl start tgfs-worker@8081
```

This runs:

```
python -m tgfs --port 8081 --no-update --session 8081
```

---

### Reload systemd

```bash
sudo systemctl daemon-reload
```

---

### Start Worker Instances

Example:

```bash
sudo systemctl start tgfs-worker@8081
sudo systemctl start tgfs-worker@8082
sudo systemctl start tgfs-worker@8083
```

Enable at boot:

```bash
sudo systemctl enable tgfs-worker@8081
sudo systemctl enable tgfs-worker@8082
sudo systemctl enable tgfs-worker@8083
```

---

### Check Status

```bash
sudo systemctl status tgfs-worker@8081
```

---

### View Logs

```bash
sudo journalctl -u tgfs-worker@8081 -f
```

---

### Restart & Stop Workers

Restart worker:

```bash
sudo systemctl restart tgfs-worker@<PORT>
```

or

```bash
sudo systemctl restart "tgfs-worker@*.service" --all
```

Stop worker:

```bash
sudo systemctl stop tgfs-worker@<PORT>
```

or

```bash
sudo systemctl stop "tgfs-worker@*.service" --all
```

---

## 2. Nginx Setup for Workers

Modify existing Nginx configuration:

```bash
sudo nano /etc/nginx/sites-available/tgfs
```

Update the upstream block:

```nginx
upstream aiohttp_backend {
    least_conn;
    keepalive 32;

    server <Bind Address>:<PORT> max_fails=3 fail_timeout=30s;
    server <Bind Address>:<PORT> max_fails=3 fail_timeout=30s;
    server <Bind Address>:<PORT> max_fails=3 fail_timeout=30s;
}
```

Replace:

- `<Bind Address>` with `HOST`
- `<PORT>` with worker ports (e.g., 8080, 8081, 8082)

---

??? info "Example"
    ```nginx
    upstream aiohttp_backend {
        least_conn;
        keepalive 32;

        server 127.0.0.1:8080 max_fails=3 fail_timeout=30s;
        server 127.0.0.1:8081 max_fails=3 fail_timeout=30s;
        server 127.0.0.1:8082 max_fails=3 fail_timeout=30s;
        server 127.0.0.1:8083 max_fails=3 fail_timeout=30s;
    }
    ```

---

### Test Configuration

```bash
sudo nginx -t
sudo systemctl reload nginx
```

---

### How Load Balancing Works

User → Nginx (Port 80) → `aiohttp_backend` (least_conn) → Workers → Telegram

Nginx distributes traffic based on active connections.

---

## 3. Scaling Workers

To add a new worker:

1. Start it:

```bash
sudo systemctl start tgfs-worker@<PORT>
```

!!! Info "Example"
    ```bash
    sudo systemctl start tgfs-worker@8084
    ```

2. Add it to the Nginx upstream block:

```
server <Bind Address>:<PORT> max_fails=3 fail_timeout=30s;
```

!!! Info "Example"
    ```
    server 127.0.0.1:8084 max_fails=3 fail_timeout=30s;
    ```

3. Reload Nginx:

```bash
sudo systemctl reload nginx
```
