# Architecture

This page explains how TG-FileStream works internally.

---

## High-Level Architecture

User → Cloudflare (Optional) → Nginx Reverse Proxy → TGFS Workers → Telegram API

---

## Core Components

### 1. Telegram Client Layer

Handles:

- Receiving files
- Sending files to bin channel
- Generating file metadata
- Managing flood waits

---

### 2. HTTP Streaming Layer

Built using:

- aiohttp

Responsible for:

- Handling incoming HTTP requests
- Serving file streams
- Managing chunked downloads

---

### 3. Worker Layer

Each worker:

- Runs on a separate port
- Maintains its own session
- Handles HTTP requests independently

Nginx distributes traffic across workers.

---

### 4. Reverse Proxy (Nginx)

Responsible for:

- Load balancing
- HTTPS termination
- Client buffering control
- Connection management

---

### 5. Patch System

During startup:

- TGFS loads patches from `tgfs/patches/`
- Extensions modify behavior
- Translation overrides are registered

This enables safe customization.

---

## Data Flow Example

1. User sends file to bot
2. File and Metadata is stored in Database
3. Download link generated
4. User accesses link
5. Nginx forwards request to worker
6. Worker fetches file from Telegram
7. File streamed to user

---

## Scaling Model

Scaling is achieved by:

- Running multiple worker services
- Using Nginx upstream with `least_conn`
- Optionally using multiple bot tokens
