# Setup systemd Service (Production Deployment)

This section explains how to configure TG-FileStream using systemd for:

- Automatic startup on boot
- Automatic restart on crash
- Multi-worker support
- Proper process isolation

This guide is based on the recommended production setup.

---

## Service Architecture

TG-FileStream uses:

- `tgfs.service` → Main instance
- `tgfs-worker@.service` → Worker instances (templated) (optional)

Workers run on different ports and sessions.

---

## 1. Main Service Configuration

Create:

```bash
sudo nano /etc/systemd/system/tgfs.service
```

Paste:

```ini
[Unit]
Description=TG-FileStream
After=network.target

[Service]
Type=simple
User=ubuntu
Group=ubuntu

WorkingDirectory=/home/ubuntu/tg-filestream

ExecStart=/home/ubuntu/tg-filestream/venv/bin/python -m tgfs

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

- `User`
- `WorkingDirectory`
- Project path

---

## 2. Worker Template Service
> [!NOTE]  
> It is optional and you can skip if you don't want

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

### How Worker Template Works

`%i` is replaced by the instance name.

Example:

```bash
sudo systemctl start tgfs-worker@8081
```

This runs:

```
python -m tgfs --port 8081 --no-update --session 8081
```

Each worker:

- Runs on a different port
- Uses separate session
- Can be load balanced by Nginx

---

## 3. Reload systemd

```bash
sudo systemctl daemon-reload
```

---

## 4. Enable Services

Enable main service:

```bash
sudo systemctl enable tgfs
```

Start main service:

```bash
sudo systemctl start tgfs
```

---

## 5. Start Worker Instances (If any)

Example:

```bash
sudo systemctl start tgfs-worker@8081
sudo systemctl start tgfs-worker@8082
sudo systemctl start tgfs-worker@8083
```

Enable workers at boot:

```bash
sudo systemctl enable tgfs-worker@8081
sudo systemctl enable tgfs-worker@8082
sudo systemctl enable tgfs-worker@8083
```

---

## 6. Check Status

Main service:

```bash
sudo systemctl status tgfs
```

Worker (if any):

```bash
sudo systemctl status tgfs-worker@8081
```

---

## 7. View Logs

Main:

```bash
sudo journalctl -u tgfs -f
```

Worker (if any):

```bash
sudo journalctl -u tgfs-worker@8081 -f
```

---

# Recommended Production Setup

Example architecture:

- tgfs.service → Main bot instance
- tgfs-worker@8081
- tgfs-worker@8082
- tgfs-worker@8083
- Nginx load balancing across workers

---

# Restart & Stop Commands

Restart main service:

```bash
sudo systemctl restart tgfs
```

Restart worker:

```bash
sudo systemctl restart tgfs-worker@8081
```

Stop worker:

```bash
sudo systemctl stop tgfs-worker@8081
```
