# TG-FileStream

TG-FileStream is a high-performance Telegram file streaming backend.

It allows efficient file delivery using:
- aiohttp
- Nginx reverse proxy
- Worker-based scaling
- Optional load balancing

---

## Features

- Large file support (30MB – 4GB)
- Worker scaling
- Nginx compatible
- Production-ready deployment

---

## Deployment Overview

Deployment flow:

1. Install dependencies
2. Configure environment
3. Test locally
4. Setup systemd
5. Setup Nginx
6. Configure DNS