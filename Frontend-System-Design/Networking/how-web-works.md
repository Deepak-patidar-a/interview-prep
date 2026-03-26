# How the Web Works

> **Module:** Frontend System Design  
> **Chapter:** 01 — Networking Fundamentals

---

## Table of Contents
1. [The Big Picture](#1-the-big-picture)
2. [Client & Server](#2-client--server)
3. [IP Address & DNS](#3-ip-address--dns)
4. [DNS Hierarchy](#4-dns-hierarchy)
5. [ISP & How Internet Reaches You](#5-isp--how-internet-reaches-you)
6. [Complete Request Journey](#6-complete-request-journey)
7. [TCP Handshake](#7-tcp-handshake)
8. [SSL/TLS Handshake](#8-ssltls-handshake)
9. [What Happens in the Browser](#9-what-happens-in-the-browser)
10. [Browser Rendering Pipeline](#10-browser-rendering-pipeline)
11. [Caching Layers](#11-caching-layers)
12. [HTTP Status Codes](#12-http-status-codes)
13. [Frontend Engineer Cheatsheet](#13-frontend-engineer-cheatsheet)

---

## 1. The Big Picture

When you type `google.com` and hit Enter, this is roughly what happens:

```
You (Browser)
    → DNS Lookup (get IP address)
    → TCP Handshake (establish connection)
    → SSL Handshake (secure the connection)
    → HTTP GET Request (ask for the page)
    → Server sends back HTML
    → Browser parses HTML → fetches CSS, JS
    → Renders the page
```

We are all connected to the internet via **lines/wires** placed under the sea and ground (optical fibres). For mobile, it's via **cell towers** (towers → phone company → ISP → internet).

---

## 2. Client & Server

```
[Client (http://)]  ──── Request ────▶  [Server]
                    ◀─── Response ───
```

- **Client** — the browser or app making the request (your laptop, phone)
- **Server** — a computer/machine that receives requests and sends back responses
- Communication happens over **HTTP** (HyperText Transfer Protocol)

**What does a server send back?**
- `HTML` — structure of the page
- `.css` files — styles
- `JavaScript` files — behaviour and interactivity

> **HTTP vs HTTPS:**  
> HTTP = plain text, insecure.  
> HTTPS = encrypted via SSL/TLS. Always use HTTPS in production.

---

## 3. IP Address & DNS

To connect to any server (device on the internet), you need its **IP address**.

Example: `166.222.11.1`

**Problem:** We can't remember IP addresses for every website.  
**Solution:** DNS — Domain Name System.

```
You type: google.com
    ↓
DNS translates it to: 142.250.195.46
    ↓
Browser connects to that IP
```

**DNS = Phone book of the internet.**  
It maps human-readable domain names → machine-readable IP addresses.

### DNS Lookup Order (Important for Frontend Engineers)
When you visit a site, the browser doesn't immediately call DNS. It checks in this order:

```
1. Browser Cache
2. OS Cache (Operating System)
3. Router Cache
4. ISP DNS Server
5. Root DNS → TLD DNS → Authoritative DNS
```

> This is why after visiting a site once, it loads faster the second time — the IP is already cached.

---

## 4. DNS Hierarchy

DNS is structured in levels:

```
. (Root Domain)
│
├── Top-Level Domain (TLD)
│     .com, .org, .gov, .edu, .in, .au
│
├── Second-Level Domain
│     google.com, microsoft.com, openoffice.org
│
└── Third-Level Domain (Subdomain)
      www.google.com, download.microsoft.com, mail.google.com
```

**Real examples from notes:**
- `www.expedia.gov` → third level domain
- `download.microsoft.com` → third level domain

### How DNS Resolution Works Step by Step

```
Browser asks: "What is the IP of google.com?"
    ↓
Root DNS says: "I don't know, but ask .com TLD server"
    ↓
TLD (.com) server says: "I don't know, but ask Google's nameserver"
    ↓
Google's Authoritative DNS says: "IP is 142.250.195.46"
    ↓
Browser caches this IP and connects
```

> **TTL (Time To Live):** Every DNS record has a TTL — how long it stays cached before re-querying. Frontend engineers should know this when deploying to a new server — DNS changes can take time to propagate worldwide (up to 48 hours).

---

## 5. ISP & How Internet Reaches You

```
Internet
    ↓
ISP (Internet Service Provider) — BSNL, JIO, Airtel, Idea
    ↓
Telephone Line / Optical Fibre
    ↓
Modem  (converts digital ↔ analog signals)
    ↓
Router  (distributes connection to multiple devices in your home)
    ↓
Your Computer / Phone
```

**ISP = Your gateway to the internet.**

### ISP Has Power
- ISP can **block/enable** specific IPs or sites
- Example: In India, many Chinese websites are banned — ISPs block those IPs
- This is also how **VPNs** work — they route your traffic through a different ISP/server in another country, bypassing these blocks

### Mobile Internet Path
```
Your Phone → Cell Tower → Phone Company → ISP → Internet
```

---

## 6. Complete Request Journey

Full sequence from typing a URL to seeing a page:

```
[Browser]
    │
    ├─ 1. Check browser cache / service worker
    │        (if found → return cached response, skip DNS)
    │
    ├─ 2. DNS Lookup
    │        Browser → OS cache → Router → ISP → DNS servers
    │        Result: IP address of the server
    │
    ├─ 3. TCP Handshake
    │        Establish reliable connection with server
    │
    ├─ 4. SSL/TLS Handshake (for HTTPS)
    │        Verify certificate, establish encrypted tunnel
    │
    ├─ 5. HTTP GET Request
    │        "Give me the HTML for this page"
    │
    ├─ 6. Server responds with HTML (first ~14KB)
    │
    └─ 7. Browser parses HTML
             ├─ Finds <link> → GET CSS (render blocking)
             ├─ Finds <script> → GET JS (parser blocking)
             └─ Builds DOM → Renders page
```

---

## 7. TCP Handshake

Before any data is sent, a **TCP (Transmission Control Protocol) Handshake** establishes a reliable connection.

```
Client          Server
  │──── SYN ────▶│     "I want to connect"
  │◀─── SYN-ACK ─│     "OK, I'm ready"
  │──── ACK ────▶│     "Great, let's go"
  │              │
  │  Connection established  │
```

**SYN** = Synchronize  
**ACK** = Acknowledge

> This 3-step process adds latency before any real data is sent. This is why **HTTP/2** and **HTTP/3** (QUIC) were created — they reduce this overhead significantly.

---

## 8. SSL/TLS Handshake

After TCP, for HTTPS, an **SSL/TLS Handshake** happens to secure the connection.

```
Client                        Server
  │──── Client Hello ────────▶│   (supported encryption methods)
  │◀─── Server Hello ─────────│   (chosen encryption + certificate)
  │──── Verify Certificate ──▶│   (client checks cert is valid)
  │──── Key Exchange ────────▶│   (agree on session key)
  │◀─── Encrypted connection ─│
  │                            │
  │    All data now encrypted  │
```

**Certificate** = issued by a Certificate Authority (CA) like DigiCert, Let's Encrypt.  
It proves the server is who it claims to be — **security/certificate way**.

> **Why frontend engineers care:**  
> SSL certificates expire. If yours expires, users see a scary warning and bounce. Always set up **auto-renewal** (Let's Encrypt does this for free).

---

## 9. What Happens in the Browser

After the server responds, the browser has work to do. The lookup order before hitting network:

```
1. Browser Cache          → returns 304 (Not Modified) if found
2. Service Worker         → returns 200 (from ServiceWorker) if cached
3. Operating System Cache → OS-level DNS/IP cache
4. Router Cache           → router also caches IPs nowadays
5. ISP DNS Server         → last resort before full DNS resolution
```

### Status Codes You'll See in Network Tab
| Code | Meaning | Source |
|---|---|---|
| `200` | OK — fresh response from server | Server |
| `200 (ServiceWorker)` | Served from service worker cache | Service Worker |
| `304` | Not Modified — served from browser cache | Browser cache |
| `301` | Moved Permanently — redirect | Server |
| `404` | Not Found | Server |
| `500` | Internal Server Error | Server |

> **304 = no API call made.** The browser used its cached version.  
> **200 (ServiceWorker) = data came from service worker**, not the actual server. Useful for offline-first apps (PWAs).

---

## 10. Browser Rendering Pipeline

Once HTML arrives, the browser renders it in 5 steps:

```
1. Parsing
   └─ Parse HTML → build DOM Tree
   └─ Parse CSS  → build CSSOM Tree

2. Style Calculation
   └─ Compute final styles for each element (cascaded styles)

3. Layout
   └─ Merge DOM + CSSOM → Render Tree
   └─ Calculate position and size of every element on screen

4. Paint
   └─ Fill in pixels — colors, text, images, shadows, borders

5. Compositing
   └─ Layer the painted elements onto the screen in correct order
   └─ GPU accelerated layers (transform, opacity) are composited separately
```

### Render Blocking Resources

```
HTML parsing
    ↓
Finds <link rel="stylesheet">
    → GET CSS → Response          (render blocking — parser waits)
    → Build CSSOM

Finds <script src="...">
    → GET JS → Response           (parser blocking — everything pauses)
    → Execute JS
```

**Fix render blocking:**
- CSS: keep it lean, inline critical CSS
- JS: use `defer` or `async` attribute
  - `defer` — download in parallel, execute after HTML parsed
  - `async` — download in parallel, execute immediately when ready (can block)

### Reflow vs Repaint
| Term | Trigger | Cost |
|---|---|---|
| **Reflow (Layout)** | Size, position change — `width`, `margin`, `top` | Expensive — recalculates layout |
| **Repaint** | Visual change without layout — `color`, `background` | Moderate |
| **Composite only** | `transform`, `opacity` | Cheapest — GPU handles it |

> **Senior tip:** Animate using `transform` and `opacity` only. They skip reflow and repaint — pure compositing. This is why `transform: translateX()` is smoother than changing `left`.

---

## 11. Caching Layers

Modern browsers check multiple cache layers before hitting the network:

```
Request
    ↓
[Browser Cache] → found? → return (304)
    ↓ not found
[Service Worker] → found? → return (200 SW)
    ↓ not found
[Operating System]
    ↓ not found
[Router] (caches IPs now)
    ↓ not found
[ISP DNS]
    ↓ not found
[Full DNS Resolution + Server Request]
```

### Service Workers (PWA)
- A JavaScript file that runs in the background, separate from the main thread
- Acts as a **proxy between browser and network**
- Can intercept requests and serve from cache → enables **offline-first apps**
- Used by: Twitter Lite, Uber, Starbucks web app

```js
// Service worker intercepts fetch
self.addEventListener('fetch', (event) => {
  event.respondWith(
    caches.match(event.request) || fetch(event.request)
  );
});
```

### CDN (Content Delivery Network)
- ISPs are now starting to give **hosting/caching access** closer to users
- CDNs place servers worldwide — users get content from the **nearest server**
- Netflix videos, YouTube videos are served via CDN — not from one central server
- Reduces latency significantly for static assets

---

## 12. HTTP Status Codes

Quick reference for frontend engineers:

| Range | Category | Examples |
|---|---|---|
| `1xx` | Informational | `100 Continue` |
| `2xx` | Success | `200 OK`, `201 Created`, `204 No Content` |
| `3xx` | Redirect | `301 Moved Permanently`, `304 Not Modified` |
| `4xx` | Client Error | `400 Bad Request`, `401 Unauthorized`, `403 Forbidden`, `404 Not Found`, `429 Too Many Requests` |
| `5xx` | Server Error | `500 Internal Server Error`, `502 Bad Gateway`, `503 Service Unavailable` |

---

## 13. Frontend Engineer Cheatsheet

### Things That Affect Page Load Speed
```
DNS resolution time      → use DNS prefetch: <link rel="dns-prefetch" href="//api.example.com">
TCP + SSL handshake      → HTTP/2 reduces this (multiplexing, header compression)
Time to First Byte       → server response time
Render blocking CSS/JS   → defer JS, inline critical CSS
Large bundle size        → code splitting, lazy loading
No caching               → set Cache-Control headers, use service workers
No CDN                   → serve static assets from CDN
```

### Key Terms Summary
| Term | What it means |
|---|---|
| **DNS** | Maps domain names to IP addresses |
| **IP Address** | Unique address of a device on the internet |
| **ISP** | Your internet provider (JIO, BSNL, Airtel) |
| **TCP** | Protocol that ensures reliable data delivery |
| **SSL/TLS** | Encrypts data between client and server |
| **CDN** | Network of servers that serve content from nearest location |
| **Service Worker** | Background JS that can cache and intercept requests |
| **Reflow** | Browser recalculates layout — expensive |
| **Repaint** | Browser redraws pixels — moderate cost |
| **304** | Cache hit — no server request made |
| **TTL** | How long a DNS record stays cached |

---

*Chapter 01 complete. Next → Chapter 02.*
