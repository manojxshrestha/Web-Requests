# 🌐 Web Requests Mastery Cheatsheet

> **Your ultimate reference for HTTP/HTTPS requests, methods, headers, and API interactions — built from HTB Academy's Web Requests module. Perfect for penetration testers, bug bounty hunters, and developers.**

[![Made for Hackers](https://img.shields.io/badge/Made%20for-Hackers-9cf.svg)](https://hackthebox.com)
[![HTTP Mastery](https://img.shields.io/badge/HTTP-Mastery-brightgreen.svg)](https://developer.mozilla.org/en-US/docs/Web/HTTP)
[![cURL Power](https://img.shields.io/badge/cURL-Power-blue.svg)](https://curl.se/)
[![GitHub](https://img.shields.io/badge/GitHub-manojxshrestha-181717?logo=github)](https://github.com/manojxshrestha)

---

## 📋 Table of Contents
- [URL Anatomy](#-url-anatomy)
- [HTTP vs HTTPS](#-http-vs-https)
- [HTTP Flow & DNS](#-http-flow--dns)
- [HTTP Headers](#-http-headers)
- [Request Methods](#-request-methods)
- [Status Codes](#-status-codes)
- [cURL Essentials](#-curl-essentials)
- [Authentication & Cookies](#-authentication--cookies)
- [GET Requests](#-get-requests)
- [POST Requests](#-post-requests)
- [CRUD API Operations](#-crud-api-operations)
- [Browser DevTools Tricks](#-browser-devtools-tricks)
- [cURL Flags Cheatsheet](#-curl-flags-cheatsheet)
- [Pro Tips](#-pro-tips)
- [Resources](#-resources)

---

## 🔗 URL Anatomy

```
scheme://user:pass@host:port/path?query#fragment
```
| Component   | Example                          | Description |
|-------------|----------------------------------|-------------|
| **Scheme**  | `http://` `https://`             | Protocol (always ends with `://`) |
| **User Info** | `admin:password@`              | Optional credentials (basic auth) |
| **Host**    | `inlanefreight.com`              | Domain name or IP address |
| **Port**    | `:80` `:443`                     | Default: 80 (HTTP), 443 (HTTPS) |
| **Path**    | `/dashboard.php`                 | File or folder on server |
| **Query**   | `?login=true&page=1`             | Parameters (`key=value` separated by `&`) |
| **Fragment**| `#status`                        | Client-side section (not sent to server) |

> 💡 **Note:** The fragment is processed by the browser and never sent in the HTTP request.

---

## 🔒 HTTP vs HTTPS

| Feature          | HTTP                          | HTTPS                                   |
|------------------|-------------------------------|-----------------------------------------|
| Encryption       | None (plain text)             | TLS/SSL (encrypted)                     |
| Port             | 80                            | 443                                     |
| Security         | Vulnerable to MITM            | Protects against eavesdropping           |
| cURL skip cert   | N/A                           | `-k` (ignore invalid SSL)                |
| Browser warning  | "Not secure"                  | Padlock icon (secure)                    |

**cURL with invalid SSL certificate:**
```bash
curl -k https://inlanefreight.com
```

**HTTP → HTTPS redirect:**
```bash
curl -L http://inlanefreight.com   # Follows 301 redirect automatically
```

---

## 🔄 HTTP Flow & DNS

```mermaid
graph LR
    A[Browser] -->|1. DNS Query| B[DNS Server]
    B -->|2. IP Address| A
    A -->|3. HTTP Request| C[Web Server]
    C -->|4. HTTP Response| A
    A -->|5. Render HTML| A
```

**Key points:**
- Browser first checks local `/etc/hosts` for manual entries.
- If not found, it queries external DNS servers.
- Default port for HTTP is 80; for HTTPS it's 443 (unless specified otherwise).

---

## 📮 HTTP Headers

### General Headers (request + response)
```http
Date: Wed, 16 Feb 2022 10:38:44 GMT
Connection: close | keep-alive
```

### Entity Headers (content description)
```http
Content-Type: text/html; charset=UTF-8
Content-Length: 385
Content-Encoding: gzip
Media-Type: application/pdf
Boundary: --b4e4fbd93540
```

### Request Headers (sent by client)
```http
Host: www.inlanefreight.com
User-Agent: Mozilla/5.0 (Windows NT 10.0; Win64; x64)
Referer: https://google.com
Accept: */*
Cookie: PHPSESSID=b4e4fbd93540
Authorization: Basic YWRtaW46YWRtaW4=        # base64(admin:admin)
```

### Response Headers (sent by server)
```http
Server: Apache/2.2.14 (Win32)
Set-Cookie: PHPSESSID=b4e4fbd93540; path=/
WWW-Authenticate: BASIC realm="localhost"
Location: /dashboard.php
```

### Security Headers (protect against attacks)
```http
Content-Security-Policy: script-src 'self'
Strict-Transport-Security: max-age=31536000
Referrer-Policy: origin
X-Frame-Options: DENY
X-Content-Type-Options: nosniff
```

> 💡 **Tip:** Use `curl -I` to quickly inspect response headers only.

---

## ⚡ Request Methods

| Method   | Description                                      | Idempotent | Safe |
|----------|--------------------------------------------------|------------|------|
| **GET**  | Retrieve a resource                              | ✅         | ✅   |
| **POST** | Submit data (forms, file upload)                 | ❌         | ❌   |
| **HEAD** | Like GET but only returns headers                | ✅         | ✅   |
| **PUT**  | Create or replace a resource                     | ✅         | ❌   |
| **DELETE** | Remove a resource                              | ✅         | ❌   |
| **PATCH** | Partial modification of a resource               | ❌         | ❌   |
| **OPTIONS** | List allowed methods for a resource            | ✅         | ✅   |

**Check allowed methods with OPTIONS:**
```bash
curl -X OPTIONS http://example.com/api -i | grep -i allow
```

---

## 📊 Status Codes

### 1xx – Informational
| Code | Meaning            |
|------|--------------------|
| 100  | Continue           |
| 101  | Switching Protocols|

### 2xx – Success
| Code | Meaning            |
|------|--------------------|
| 200  | OK                 |
| 201  | Created            |
| 204  | No Content         |

### 3xx – Redirection
| Code | Meaning            |
|------|--------------------|
| 301  | Moved Permanently  |
| 302  | Found (temporary)  |
| 304  | Not Modified       |

### 4xx – Client Errors
| Code | Meaning            |
|------|--------------------|
| 400  | Bad Request        |
| 401  | Unauthorized       |
| 403  | Forbidden          |
| 404  | Not Found          |
| 405  | Method Not Allowed |
| 429  | Too Many Requests  |

### 5xx – Server Errors
| Code | Meaning            |
|------|--------------------|
| 500  | Internal Server Error|
| 502  | Bad Gateway        |
| 503  | Service Unavailable|
| 504  | Gateway Timeout    |

---

## 🚀 cURL Essentials

### Basic Commands
```bash
curl http://example.com                     # Simple GET
curl -O http://example.com/file.html        # Save with remote filename
curl -o custom.html http://example.com      # Save with custom name
curl -s http://example.com                   # Silent (no progress)
curl -v http://example.com                   # Verbose (full request/response)
curl -i http://example.com                   # Include headers in output
curl -I http://example.com                   # HEAD request (headers only)
```

### Headers & User-Agent
```bash
curl -A "Mozilla/5.0 (X11; Linux x86_64)" http://example.com
curl -H "X-Custom-Header: value" http://example.com
curl -H "Accept: application/json" http://example.com
```

### Follow Redirects & Timeouts
```bash
curl -L http://example.com                   # Follow redirects
curl --max-time 10 http://example.com        # Max 10 seconds
curl --connect-timeout 5 http://example.com  # Connection timeout
```

### Data Transfer
```bash
curl -d "param1=value1&param2=value2" http://example.com/post
curl --data @file.txt http://example.com/upload
curl -F "file=@/path/to/image.jpg" http://example.com/upload
```

### Cookies
```bash
curl -b "name=value" http://example.com
curl -b cookies.txt -c cookies.txt http://example.com   # Save & load
```

### Authentication
```bash
curl -u username:password http://example.com
curl -H "Authorization: Basic $(echo -n user:pass | base64)" http://example.com
```

---

## 🔐 Authentication & Cookies

### Basic Auth
```bash
# Header format: Authorization: Basic base64(user:pass)
echo -n "admin:admin" | base64
# YWRtaW46YWRtaW4=

# Using -u
curl -u admin:admin http://example.com

# Manual header
curl -H "Authorization: Basic YWRtaW46YWRtaW4=" http://example.com

# In URL (works in cURL and browsers)
curl http://admin:admin@example.com
```

### Cookie Auth (after login)
```bash
# Get cookie (login)
curl -i -X POST -d "username=admin&password=admin" http://example.com/login
# Look for Set-Cookie: PHPSESSID=abc123

# Use cookie for subsequent requests
curl -b "PHPSESSID=abc123" http://example.com/dashboard
curl -H "Cookie: PHPSESSID=abc123" http://example.com/dashboard
```

### Token Auth (Bearer / JWT)
```bash
curl -H "Authorization: Bearer eyJhbGciOiJIUzI1NiIs..." http://example.com/api
```

---

## 📥 GET Requests

### Simple GET with parameters
```bash
curl "http://example.com/search.php?search=leeds&page=2"
```

### GET with Basic Auth (as seen in HTB Academy)
```bash
# With -u
curl -u admin:admin "http://<SERVER_IP>:<PORT>/search.php?search=le"

# With Authorization header
curl -H "Authorization: Basic YWRtaW46YWRtaW4=" "http://<SERVER_IP>:<PORT>/search.php?search=le"
```

**Browser DevTools copy trick:**  
Right-click on request → Copy → Copy as cURL → paste in terminal.

---

## 📤 POST Requests

### URL-encoded form (login)
```bash
curl -X POST -d "username=admin&password=admin" http://example.com/login
```

### With cookie after authentication
```bash
curl -X POST -d "username=admin&password=admin" http://example.com/login -i
# Capture Set-Cookie, then use it
curl -b "PHPSESSID=abc123" http://example.com/dashboard
```

### JSON data
```bash
curl -X POST -d '{"search":"london"}' \
     -H "Content-Type: application/json" \
     -b "PHPSESSID=abc123" \
     http://example.com/search.php
```

### File upload
```bash
curl -F "file=@exploit.php" -F "name=test" http://example.com/upload
```

---

## 🔁 CRUD API Operations

Assume REST API endpoint: `http://<SERVER_IP>:<PORT>/api.php/city/{name}`

### Read (GET)
```bash
curl -s http://example.com/api.php/city/london | jq .
curl -s http://example.com/api.php/city/       # All cities
```

### Create (POST)
```bash
curl -X POST http://example.com/api.php/city/ \
     -d '{"city_name":"HTB_City","country_name":"HTB"}' \
     -H "Content-Type: application/json"
```

### Update (PUT)
```bash
curl -X PUT http://example.com/api.php/city/london \
     -d '{"city_name":"New_HTB_City","country_name":"HTB"}' \
     -H "Content-Type: application/json"
```

### Delete (DELETE)
```bash
curl -X DELETE http://example.com/api.php/city/New_HTB_City
```

### OPTIONS (check allowed methods)
```bash
curl -X OPTIONS http://example.com/api.php/city/ -v 2>&1 | grep -i allow
```

> 💡 **Pro tip:** Pipe JSON responses through `jq` for pretty printing and filtering:
> ```bash
> curl -s http://example.com/api.php/city/ | jq '.[].city_name'
> ```

---

## 🛠️ Browser DevTools Tricks

| Shortcut | Action |
|----------|--------|
| `F12` / `Ctrl+Shift+I` | Open DevTools |
| `Ctrl+Shift+E` | Network tab |
| `Ctrl+Shift+J` | Console |
| `Shift+F9` | Storage (cookies) |

### Network Tab Features
- **Preserve log** – Keep requests across page loads
- **Disable cache** – Simulate first visit
- **Filter** – By type (XHR, JS, CSS, etc.)
- **Search** – Find specific request

### Copy as cURL / Fetch
Right-click on a request → Copy →  
- **Copy as cURL** – Get ready-to-use terminal command  
- **Copy as Fetch** – Get JavaScript `fetch` code

### Modify and Resend
Right-click → Edit and Resend → Modify method, headers, body → Send.

### Set Cookie Manually
1. Storage tab → Cookies → Select domain
2. Double-click to edit value
3. Refresh page to test authenticated session

---

## 📌 cURL Flags Cheatsheet

| Flag | Description |
|------|-------------|
| `-X METHOD` | HTTP method (GET, POST, PUT, DELETE, etc.) |
| `-d 'data'` | Send data in body (for POST/PUT) |
| `-H 'Header: val'` | Add custom header (multiple allowed) |
| `-b 'cookies'` | Send cookies (e.g., `-b 'name=value'`) |
| `-c file` | Save cookies to file |
| `-u user:pass` | Basic authentication |
| `-A 'agent'` | Set User-Agent |
| `-L` | Follow redirects |
| `-k` | Allow insecure SSL connections |
| `-i` | Include headers in output |
| `-I` | Send HEAD request (headers only) |
| `-v` | Verbose (full request/response) |
| `-s` | Silent (no progress meter) |
| `-O` | Save output with remote filename |
| `-o file` | Write output to file |
| `-F 'name=@file'` | Multipart form upload |
| `--max-time N` | Max time in seconds for request |
| `--limit-rate 100K` | Limit transfer rate |

---

## 🎯 Pro Tips

1. **Always use `-v` during testing** – see exact request/response.
2. **Use `-i` to spot cookies** (e.g., `Set-Cookie`).
3. **Copy as cURL from DevTools** – saves hours of typing.
4. **Test with/without cookies** to understand auth requirements.
5. **Check `OPTIONS` first** to see allowed methods.
6. **Use `-L` for redirects** when testing login flows.
7. **Rate limit your requests** – `--limit-rate` to avoid being blocked.
8. **Compare authenticated vs unauthenticated responses** to identify missing access controls.
9. **Pipe JSON to `jq`** – `curl -s ... | jq .`
10. **Inspect security headers** – CSP, HSTS, etc. for misconfigurations.

---

## 📚 Resources

- [MDN HTTP Documentation](https://developer.mozilla.org/en-US/docs/Web/HTTP)
- [cURL Manual](https://curl.se/docs/manual.html)
- [HTTP Status Codes](https://httpstatuses.com/)
- [JQ Manual](https://stedolan.github.io/jq/manual/)
- [Hack The Box Academy – Web Requests](https://academy.hackthebox.com/course/preview/web-requests)

---

<div align="center">
  
**by ~/. [manojxshrestha](https://github.com/manojxshrestha)**  

[Report Issue](https://github.com/manojxshrestha/web-requests-cheatsheet/issues) • [Suggest Improvement](https://github.com/manojxshrestha/web-requests-cheatsheet/pulls)

</div>
