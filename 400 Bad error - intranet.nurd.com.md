# Redis Session Integration for `intranet.nurd.com`

## Purpose

This document describes the implementation of Redis-based session storage for `oauth2-proxy` used by `intranet.nurd.com`.

The purpose of this change is to reduce browser cookie size and avoid oversized HTTP request issues causing:

* HTTP 400 Bad Request
* Missing JS/CSS resources
* Authentication failures
* Nginx request parsing issues

---

# Root Cause

`oauth2-proxy` was storing complete session information in browser cookies.

Request structure:

```text
Request URL + Query Parameters
+ OAuth Cookies
+ Additional Headers
```

For some users, cookie size became large enough that the overall request exceeded safe request processing limits.

Result:

```text
Browser
   ↓
Nginx
   ↓
Request size exceeds processing limit
   ↓
HTTP 400 Bad Request
```

Increasing Nginx buffer settings alone is not a permanent fix because:

* It increases memory allocation only
* Does not remove internal parsing limitations
* High values can increase memory usage
* May create performance and security concerns

---

# Solution

Move session storage from browser cookies to Redis.

Before:

```text
Browser
   ↓
Large OAuth Cookie (~8KB+)
   ↓
Nginx
   ↓
400 Error
```

After:

```text
Browser
   ↓
Small Session ID (~100 bytes)
   ↓
oauth2-proxy
   ↓
Redis stores session information
   ↓
Nginx
   ↓
Application loads successfully
```

---

# Redis Installation

## Debian / Ubuntu

Install Redis:

```bash
apt update
apt install redis-server -y
```

Enable Redis service:

```bash
systemctl enable redis-server
systemctl start redis-server
```

Verify service:

```bash
systemctl status redis-server
```

Expected:

```text
active (running)
```

Test Redis:

```bash
redis-cli ping
```

Expected output:

```text
PONG
```

---

## RHEL / CentOS

Install Redis:

```bash
yum install redis -y
```

Enable and start:

```bash
systemctl enable redis
systemctl start redis
```

Verify:

```bash
redis-cli ping
```

Expected:

```text
PONG
```

---

# Update oauth2-proxy Configuration

Edit service file:

```bash
vi /etc/systemd/system/oauth2-proxy-intranet.nurd.com.service
```

Update:

```ini
[Unit]
Description=oauth2-proxy-intranet.nurd.com

[Service]
ExecStart=/usr/bin/oauth2-proxy \
  --alpha-config /etc/oauth2/intranet.nurd.com.yaml \
  --session-store-type=redis \
  --redis-connection-url=redis://127.0.0.1:6379

Environment=OAUTH2_PROXY_COOKIE_SECRET=xxxxxxxxxxxx
Environment=OAUTH2_PROXY_PROMPT=select_account
Environment=OAUTH2_PROXY_HTTP_ADDRESS=0.0.0.0:2214
Environment=OAUTH2_PROXY_EMAIL_DOMAINS=*

Restart=always
LimitNOFILE=10048576

[Install]
WantedBy=multi-user.target
```

---

# Apply Changes

Reload systemd:

```bash
systemctl daemon-reload
```

Restart Redis:

```bash
systemctl restart redis-server
```

Restart oauth2-proxy:

```bash
systemctl restart oauth2-proxy-intranet.nurd.com
```

---

# Validation

## Verify Redis activity

Monitor Redis:

```bash
redis-cli monitor
```

Login to:

```text
https://intranet.nurd.com
```

Expected activity:

```text
SET session:xxxxx
GET session:xxxxx
EXPIRE session:xxxxx
```

---

## Verify browser cookies

Open browser:

```text
F12 → Application → Cookies
```

Before:

```text
_oauth2_proxy_0 → several KB
_oauth2_proxy_1 → several KB
```

After:

```text
_oauth2_proxy → small session identifier
```

---

# Expected Result

* No HTTP 400 errors
* Reduced request size
* Stable authentication
* Smaller browser cookies
* Improved application stability

---

# Notes

Typical Redis session usage:

| Users | Approximate Memory |
| ----- | ------------------ |
| 100   | ~500 KB            |
| 1000  | ~5 MB              |
| 5000  | ~25 MB             |

Redis usage remains minimal because only session information is stored.

This approach is considered an industry-standard solution and is preferred over continuously increasing Nginx request limits.
