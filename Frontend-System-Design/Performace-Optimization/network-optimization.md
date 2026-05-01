# 🌐 Network Optimization in Frontend System Design

> **Series**: Frontend Performance & Optimization  
> **Chapter**: Network Optimization  
> **Goal**: Understand how browsers fetch and render resources — and how to make it blazing fast.

---

## Table of Contents

1. [Critical Rendering Path](#1-critical-rendering-path)
2. [Minimize Network Requests](#2-minimize-network-requests)
3. [Async JavaScript](#3-async-javascript)
4. [Avoid Redirections](#4-avoid-redirections)
5. [Resource Hinting](#5-resource-hinting)
6. [Resource Hint Priority](#6-resource-hint-priority)
7. [Early Hints (103)](#7-early-hints-103)
8. [HTTP Upgrades](#8-http-upgrades-http11-vs-http2-vs-http3)
9. [Compression](#9-compression)
10. [Caching](#10-caching)

---

## 1. Critical Rendering Path

### What is it?
The **Critical Rendering Path (CRP)** is the sequence of steps the browser takes to convert HTML, CSS, and JS into pixels on the screen.

```
Browser  <----->  Server
         (data travels in chunks)
```

### The 14KB Rule 🎯
- The **first TCP packet** from the server carries approximately **14KB** of data.
- This is due to TCP slow start — the initial congestion window size.
- **Goal**: Fit your critical HTML + CSS + above-the-fold content within this 14KB.
- This ensures the fastest possible **First Contentful Paint (FCP)**.

### Rendering Pipeline
```
HTML Parsing
    │
    ▼
DOM Tree ──────────────────────────────────────────┐
    │                                               │
    ▼                                               │
CSS Parsing → CSSOM Tree                           │
    │               │                               │
    └───────────────┘                               │
            │                                       │
            ▼                                       │
      Render Tree ◄──────────────────────────────────┘
            │
            ▼
         Layout (Reflow)
            │
            ▼
         Paint
            │
            ▼
        Composite
```

### Key Metrics
| Metric | Description | Target |
|--------|-------------|--------|
| **FCP** | First Contentful Paint | < 1.8s |
| **LCP** | Largest Contentful Paint | < 2.5s |
| **TTI** | Time to Interactive | < 3.8s |
| **TBT** | Total Blocking Time | < 200ms |

### 💡 Tips
- Minify and inline critical CSS
- Defer non-critical JS
- Server-side render (SSR) above-the-fold HTML
- Use tools like [PageSpeed Insights](https://pagespeed.web.dev/) to analyze CRP

---

## 2. Minimize Network Requests

### The Problem
Every network request has overhead:
- **DNS Lookup** — resolving domain to IP
- **TCP Handshake** — establishing connection (3-way handshake)
- **TLS/SSL Handshake** — secure connection setup (adds 1-2 RTTs)
- **HTTP Request/Response** — actual data transfer

### Browser Parallel Request Limit
```
Per Domain Limit: 6–10 parallel connections (HTTP/1.1)
```
More requests = queue waiting = slower page load.

### Solutions

#### ✅ Inline Critical CSS
```html
<head>
  <style>
    /* Only above-the-fold, critical styles */
    body { margin: 0; font-family: sans-serif; }
    .hero { background: #0a0a0a; color: #fff; padding: 4rem; }
  </style>
</head>
```

#### ✅ Inline Small JS
```html
<!-- Inline only tiny scripts; avoid for large bundles -->
<script>
  // e.g., theme initialization to prevent flash
  const theme = localStorage.getItem('theme') || 'light';
  document.documentElement.setAttribute('data-theme', theme);
</script>
```

#### ✅ Base64 for Small Images
```html
<!-- Eliminates a network round trip for small icons -->
<img src="data:image/png;base64,iVBORw0KGgoAAAANSUhEUgAA..." alt="icon" />
```
> ⚠️ Use only for very small images (< 5KB). Base64 encoding increases size by ~33%.

#### ✅ SVG Instead of Raster Images
```html
<!-- Inline SVG: zero network request, fully scalable -->
<svg xmlns="http://www.w3.org/2000/svg" viewBox="0 0 24 24" width="24" height="24">
  <path d="M12 2L2 7l10 5 10-5-10-5z" fill="currentColor"/>
</svg>
```

### Summary Table
| Technique | Use Case | Saves |
|-----------|----------|-------|
| Inline CSS | Critical above-fold styles | 1 HTTP request |
| Inline JS | Tiny scripts (< 1KB) | 1 HTTP request + RTT |
| Base64 Image | Small icons/thumbnails | 1 HTTP request |
| SVG | Icons, logos, illustrations | 1 HTTP request + scalable |

---

## 3. Async JavaScript

JavaScript is **parser-blocking** by default. Understanding the three ways to load scripts is crucial.

### Default `<script>`
```html
<script src="app.js"></script>
```
```
HTML Parsing ──────────►STOP◄────────── Resume ──────────► Done
                          │
                   Fetch + Execute JS
```
- ❌ **Blocks HTML parsing** entirely
- Fetches and executes immediately when encountered
- Page appears frozen while JS loads

---

### `<script async>`
```html
<script async src="analytics.js"></script>
```
```
HTML Parsing ──────────────────────────────────► Done
         │                              STOP◄───┤
         └──── Fetch JS (parallel) ────► Execute
```
- ✅ HTML parsing continues **while JS is fetched**
- ❌ **Execution still blocks** parsing (but only briefly)
- ⚠️ Execution order is **NOT guaranteed** — runs as soon as downloaded
- **Best for**: Independent scripts (analytics, ads) that don't depend on DOM

---

### `<script defer>`
```html
<script defer src="main.js"></script>
```
```
HTML Parsing ──────────────────────────────────► Done
         │                                         │
         └──── Fetch JS (parallel) ─────────────► Execute (after DOM ready)
```
- ✅ HTML parsing continues **while JS is fetched**
- ✅ Executes **only after HTML is fully parsed**
- ✅ **Order is guaranteed** — multiple deferred scripts run in order
- **Best for**: Scripts that need the full DOM (most app scripts)

### Quick Comparison

| Feature | `<script>` | `<script async>` | `<script defer>` |
|---------|-----------|-----------------|-----------------|
| Blocks HTML parsing | ✅ Yes | ❌ No | ❌ No |
| Parallel fetch | ❌ No | ✅ Yes | ✅ Yes |
| Execution order guaranteed | ✅ Yes | ❌ No | ✅ Yes |
| Executes after DOM ready | ❌ No | ❌ No | ✅ Yes |
| Best for | Critical inline | Analytics, ads | App scripts |

---

## 4. Avoid Redirections

### What is a Redirect?
A redirect is an extra HTTP round trip before your page loads.
```
Browser → http://flipkart.com
             ↓ 301 Moved Permanently
Browser → https://www.flipkart.com  (wasted RTT!)
             ↓ 200 OK
         Page Loads
```

Each redirect adds **100–300ms** of latency.

### Common Redirect Chains to Avoid
- `http://` → `https://` (very common)
- `www.` → non-www (or vice versa)
- Trailing slash redirects: `/about` → `/about/`

### Solution: HSTS (HTTP Strict Transport Security)

**HSTS** tells browsers to **always use HTTPS** — no redirect needed.

```
# Server response header
Strict-Transport-Security: max-age=31536000; includeSubDomains; preload
```

Once a browser sees this header, it internally redirects — no round trip to the server.

### HSTS Preload
- Visit [hstspreload.org](https://hstspreload.org) to submit your domain
- Your domain gets hardcoded into browsers' HSTS preload list
- **Result**: Browser uses HTTPS from the very first request, even before it ever visits your site
- Supported by Chrome, Firefox, Safari, Edge

```
Without HSTS Preload:
  First visit: http → redirect → https (extra round trip)

With HSTS Preload:
  First visit: browser uses https directly ✅
```

---

## 5. Resource Hinting

Resource hints tell the browser to **prepare resources in advance** before they are needed.

### `preconnect` — Connect Early to External Servers
```html
<!-- Connect to Google Fonts server before the CSS even asks for it -->
<link rel="preconnect" href="https://fonts.googleapis.com" />
<link rel="preconnect" href="https://fonts.gstatic.com" crossorigin />
```
- Performs DNS lookup + TCP handshake + TLS negotiation **in advance**
- Use for: Cross-origin resources you'll definitely need (fonts, CDN, API servers)

---

### `dns-prefetch` — DNS Lookup Only
```html
<!-- Cheaper than preconnect; just resolves the IP address -->
<link rel="dns-prefetch" href="https://api.example.com" />
<link rel="dns-prefetch" href="https://cdn.example.com" />
```
- Only does **DNS resolution** (not TCP/TLS)
- Fallback for browsers that don't support `preconnect`
- Use for: Many third-party domains where full preconnect is too expensive

---

### `preload` — Load Critical Resources Early
```html
<!-- Preload a critical font -->
<link rel="preload" href="/fonts/Inter.woff2" as="font" type="font/woff2" crossorigin />

<!-- Preload the hero image -->
<link rel="preload" href="/images/hero.webp" as="image" />

<!-- Preload a critical JS file -->
<link rel="preload" href="/js/critical.js" as="script" />

<!-- Preload critical CSS -->
<link rel="preload" href="/css/critical.css" as="style" />
```
- Tells the browser: **"I will definitely need this resource — fetch it NOW"**
- High priority fetch — happens alongside HTML parsing
- Use for: Fonts, hero images, critical CSS/JS

---

### `prefetch` — Load Future Resources (Low Priority)
```html
<!-- Prefetch the next page's bundle when user is on current page -->
<link rel="prefetch" href="/js/checkout-page.bundle.js" as="script" />

<!-- Prefetch an image that appears after user interaction -->
<link rel="prefetch" href="/images/product-detail.webp" as="image" />
```
- Loads resources **for future navigation** with low priority
- Downloaded and cached during idle browser time
- Use for: Resources needed on the next likely page

---

### `prerender` — Render Entire Page in Background
```html
<!-- Pre-render the checkout page while user is on cart page -->
<link rel="prerender" href="https://example.com/checkout" />
```
- Loads the **entire page + all its dependencies** in a hidden background tab
- When user navigates to it, it appears **instantly**
- ⚠️ Expensive: Uses memory and bandwidth. Use sparingly.
- Use for: Near-certain next navigation (e.g., next step in a checkout flow)

---

### Summary of Resource Hints

| Hint | Does | Priority | Use When |
|------|------|----------|----------|
| `preconnect` | DNS + TCP + TLS | High | Cross-origin resources you'll need soon |
| `dns-prefetch` | DNS only | Low | Many third-party domains |
| `preload` | Fetch resource | High | Critical resources for current page |
| `prefetch` | Fetch resource | Low | Resources for next page/interaction |
| `prerender` | Fetch + render page | Low | Near-certain next navigation |

---

### Real-World Example: Google Fonts Optimization
```html
<head>
  <!-- Step 1: Preconnect to font servers -->
  <link rel="preconnect" href="https://fonts.googleapis.com" />
  <link rel="preconnect" href="https://fonts.gstatic.com" crossorigin />

  <!-- Step 2: Load the CSS (which references the font files) -->
  <link href="https://fonts.googleapis.com/css2?family=Playfair+Display:wght@400;700&display=swap" rel="stylesheet" />

  <!-- Step 3: Preload the actual font file (once you know the URL) -->
  <link rel="preload" href="https://fonts.gstatic.com/s/playfairdisplay/..." as="font" type="font/woff2" crossorigin />
</head>
```

---

## 6. Resource Hint Priority

You can fine-tune fetch priority using the `fetchpriority` attribute.

### `fetchpriority` Values
- **`high`** — Fetch this ASAP (important resource)
- **`low`** — Fetch when idle (non-critical)
- **`auto`** — Let browser decide (default)

### Examples

```html
<!-- Hero image: high priority (LCP element) -->
<img src="/images/hero.webp" fetchpriority="high" alt="Hero" />

<!-- Below-fold images: low priority -->
<img src="/images/footer-banner.webp" fetchpriority="low" alt="Footer" />

<!-- Non-critical JS: low priority -->
<script src="/js/chat-widget.js" fetchpriority="low" defer></script>
```

### Load CSS Without Blocking Render
```html
<!--
  Trick: Load CSS asynchronously, then apply it once loaded.
  - rel="preload" + as="style" = fetch at high priority but don't apply yet
  - onload switches it to a real stylesheet after load
  - <noscript> is fallback for no-JS
-->
<link
  rel="preload"
  href="/css/non-critical.css"
  as="style"
  fetchpriority="low"
  onload="this.rel='stylesheet'"
/>
<noscript>
  <link rel="stylesheet" href="/css/non-critical.css" />
</noscript>
```

---

## 7. Early Hints (103)

### What is HTTP 103?
**103 Early Hints** is an HTTP status code that allows the server to send resource hints to the browser **before** the main response (200 OK) is ready.

```
Browser                          Server
  │                                 │
  │──── GET /page ─────────────────►│
  │                                 │  (server processing...)
  │◄─── 103 Early Hints ────────────│  ← Browser starts preloading NOW
  │     Link: </css/main.css>; rel=preload
  │     Link: </fonts/Inter.woff2>; rel=preload
  │                                 │  (server finishes)
  │◄─── 200 OK + HTML ─────────────│
```

### Why It Matters
- Without 103: Browser waits for 200 OK before it sees any hints
- With 103: Browser gets a head start — resources load **in parallel** with server processing
- Can save **100–200ms** on server-side rendered pages

### Server Config Example (Nginx)
```nginx
location / {
    add_header Link "</css/main.css>; rel=preload; as=style";
    add_header Link "</fonts/Inter.woff2>; rel=preload; as=font; crossorigin";
    # ... serve the page
}
```

---

## 8. HTTP Upgrades: HTTP/1.1 vs HTTP/2 vs HTTP/3

### HTTP/1.1
```
Request 1: ──────────► Response 1
Request 2:              ──────────► Response 2   (must wait)
Request 3:                          ──────────► Response 3
```
- **Sequential** — one request at a time per connection
- Browser opens 6 parallel TCP connections per domain as a workaround
- **Head-of-line (HOL) blocking**: slow response blocks everything behind it

---

### HTTP/2
```
Request 1 ─────────────────────► Response 1
Request 2 ─────────────────────► Response 2   (all parallel!)
Request 3 ─────────────────────► Response 3
```
- **Multiplexing** — many requests over a single TCP connection
- **Header compression** (HPACK) — reduces overhead
- **Server Push** — server can proactively send resources
- Still has HOL blocking at the **TCP layer**

---

### HTTP/3
```
Uses QUIC protocol (UDP-based, not TCP)

Request 1 ─────────────────────► Response 1
Request 2 ─────────────────────► Response 2   (truly independent streams)
Request 3 ─────────────────────► Response 3
```
- Built on **QUIC** (UDP) — no TCP handshake overhead
- **0-RTT** reconnection — faster repeated connections
- **No HOL blocking** — each stream is independent
- **Built-in encryption** (TLS 1.3)
- Excellent on **mobile and poor networks** (connection migration)

### Comparison Table

| Feature | HTTP/1.1 | HTTP/2 | HTTP/3 |
|---------|----------|--------|--------|
| Protocol | TCP | TCP | UDP (QUIC) |
| Multiplexing | ❌ | ✅ | ✅ |
| Header Compression | ❌ | ✅ (HPACK) | ✅ (QPACK) |
| HOL Blocking | ❌ TCP + HTTP | ❌ HTTP level | ✅ None |
| 0-RTT reconnect | ❌ | ❌ | ✅ |
| Server Push | ❌ | ✅ | ✅ |
| Encryption | Optional | Practical | Required |

> **Practical tip**: Most modern CDNs (Cloudflare, Fastly) support HTTP/2 and HTTP/3. Enable it on your server and you get multiplexing for free — no code changes needed.

---

## 9. Compression

Compressing responses reduces the bytes transferred over the network.

### Gzip
- **Universal support** — works everywhere
- Compression ratio: ~70% reduction on text files
- Server config (Nginx):
```nginx
gzip on;
gzip_types text/plain text/css application/javascript application/json;
gzip_min_length 1024;
```

### Brotli
- Developed by Google
- **20–26% better** compression than gzip
- Supported in all modern browsers (requires HTTPS)
- Server config (Nginx):
```nginx
brotli on;
brotli_comp_level 6;
brotli_types text/plain text/css application/javascript application/json;
```

### What Gets Compressed?
| Resource | Compress? | Notes |
|----------|-----------|-------|
| HTML | ✅ Yes | Large gains |
| CSS | ✅ Yes | Large gains |
| JavaScript | ✅ Yes | Largest gains |
| JSON/XML | ✅ Yes | APIs benefit a lot |
| JPEG/PNG/WebP | ❌ No | Already compressed |
| WOFF2 fonts | ❌ No | Already compressed |

### Bundle-Level Compression
```bash
# During build — pre-compress assets
npm install --save-dev compression-webpack-plugin

# webpack.config.js
const CompressionPlugin = require('compression-webpack-plugin');
module.exports = {
  plugins: [
    new CompressionPlugin({ algorithm: 'brotliCompress' })
  ]
}
```

---

## 10. Caching

Caching avoids repeated network requests by storing responses locally.

### Cache-Control Header
The most important caching header. Tells browser and CDN how long to cache.

```
Cache-Control: max-age=31536000, immutable
```

| Directive | Meaning |
|-----------|---------|
| `max-age=N` | Cache for N seconds |
| `no-cache` | Revalidate with server before using cache |
| `no-store` | Never cache (sensitive data) |
| `public` | Can be cached by CDN/proxy |
| `private` | Only browser cache (not CDN) |
| `immutable` | Content will never change (safe to cache forever) |
| `stale-while-revalidate=N` | Serve stale while fetching fresh in background |

### Caching Strategy by Resource Type

```
Static assets (JS, CSS with hash in filename)
  Cache-Control: public, max-age=31536000, immutable

HTML pages
  Cache-Control: no-cache  (always revalidate)

API responses
  Cache-Control: private, max-age=60

Sensitive data
  Cache-Control: no-store
```

### ETag & Last-Modified (Conditional Requests)
```
First request:
  Browser ──GET /api/data──────────────────────► Server
  Browser ◄──200 OK + ETag: "abc123"───────────── Server

Second request:
  Browser ──GET /api/data + If-None-Match: "abc123"──► Server
  Browser ◄──304 Not Modified (no body sent!)────────── Server
```
- **ETag**: A hash of the response content
- **Last-Modified**: Timestamp of last modification
- Both allow **304 Not Modified** — saves bandwidth (no body transferred)

### Service Worker Caching
```javascript
// service-worker.js
const CACHE_NAME = 'app-v1';
const STATIC_ASSETS = ['/index.html', '/css/main.css', '/js/app.js'];

// Install: cache static assets
self.addEventListener('install', event => {
  event.waitUntil(
    caches.open(CACHE_NAME).then(cache => cache.addAll(STATIC_ASSETS))
  );
});

// Fetch: cache-first strategy
self.addEventListener('fetch', event => {
  event.respondWith(
    caches.match(event.request).then(cached => {
      // Return cached version, or fetch from network
      return cached || fetch(event.request);
    })
  );
});
```

### Caching Strategies
| Strategy | How | Use Case |
|----------|-----|----------|
| **Cache First** | Cache → Network | Static assets, fonts |
| **Network First** | Network → Cache | API data, HTML |
| **Stale While Revalidate** | Cache immediately, update in bg | News, feeds |
| **Cache Only** | Cache only | Offline-first apps |
| **Network Only** | Network only | Real-time data |

---

## Quick Reference Cheatsheet

```
📦 Critical Rendering Path
  └── Fit critical content in first 14KB
  └── Minimize render-blocking resources

🔗 Minimize Requests
  └── Inline critical CSS/JS
  └── Use SVG for icons
  └── Base64 only for tiny images

⚡ Async JS
  └── <script>        → blocking
  └── <script async>  → parallel fetch, immediate execute
  └── <script defer>  → parallel fetch, execute after DOM

↩️ Redirects
  └── Use HSTS + hstspreload.org
  └── Serve https directly

🔍 Resource Hints
  └── preconnect   → cross-origin early connection
  └── dns-prefetch → DNS only
  └── preload      → current page critical resources
  └── prefetch     → next page resources
  └── prerender    → next page full render

🎯 Priority
  └── fetchpriority="high" for LCP images
  └── fetchpriority="low" for below-fold

⚡ HTTP/2 & HTTP/3
  └── Multiplexing = parallel requests
  └── HTTP/3 = best on mobile/lossy networks

🗜️ Compression
  └── Brotli > Gzip
  └── Compress: HTML, CSS, JS, JSON
  └── Skip: images, fonts (already compressed)

📦 Caching
  └── Static assets: max-age=31536000, immutable
  └── HTML: no-cache
  └── Service Worker for offline support
```

---

## Tools for Analysis

| Tool | Purpose |
|------|---------|
| [PageSpeed Insights](https://pagespeed.web.dev/) | Full CRP + performance audit |
| [WebPageTest](https://www.webpagetest.org/) | Waterfall charts, network analysis |
| [hstspreload.org](https://hstspreload.org) | HSTS preload submission |
| Chrome DevTools → Network tab | Inspect all requests, caching headers |
| Chrome DevTools → Coverage tab | Find unused CSS/JS |
| [Bundlephobia](https://bundlephobia.com/) | npm package size analysis |

---

*Part of the Frontend Performance & Optimization series.*
