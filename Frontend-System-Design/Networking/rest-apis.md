# REST APIs

> **Module:** Frontend System Design  
> **Chapter:** 03 — REST APIs

---

## Table of Contents
1. [What is REST?](#1-what-is-rest)
2. [Benefits of REST](#2-benefits-of-rest)
3. [Building Blocks of REST](#3-building-blocks-of-rest)
4. [URL Anatomy](#4-url-anatomy)
5. [HTTP Methods](#5-http-methods)
6. [Request Structure](#6-request-structure)
7. [Response Structure](#7-response-structure)
8. [Request Headers](#8-request-headers)
9. [Response Headers](#9-response-headers)
10. [HTTP Status Codes](#10-http-status-codes)
11. [REST API Design Best Practices](#11-rest-api-design-best-practices)
12. [Frontend Engineer Cheatsheet](#12-frontend-engineer-cheatsheet)

---

## 1. What is REST?

**REST** = **Re**presentational **S**tate **T**ransfer

It is an **architectural style** (not a protocol) for designing networked applications. REST uses HTTP to communicate between client and server.

> REST is not a library or a tool — it's a **set of rules/conventions** for how APIs should behave.

**API** = **A**pplication **P**rogramming **I**nterface — a contract between client and server defining how they communicate.

```
Client (React App)          Server (Node/Express)
       │                           │
       │──── HTTP Request ────────▶│
       │     (GET /api/products)   │
       │                           │  Fetches from DB
       │◀─── HTTP Response ────────│
       │     (JSON data)           │
```

---

## 2. Benefits of REST

| # | Benefit | What it means |
|---|---|---|
| 1 | **Ease of Use** | Simple HTTP verbs — easy to understand and implement |
| 2 | **Stateless** ⭐ | Server stores NO previous request info — every request is independent |
| 3 | **Scalability** | Supports horizontal and vertical scaling easily |
| 4 | **Flexibility with Data** | Works with JSON, XML, plain text — JSON is standard today |
| 5 | **Uniform Interface** | Consistent URL patterns across all resources |
| 6 | **Caching** | HTTP caching works out of the box — improves performance |
| 7 | **Separation of Concerns** | Frontend and backend are completely decoupled |
| 8 | **Interoperability** | Language agnostic — any client (React, Android, iOS) can consume the same API |
| 9 | **Ease of Testing** | Simple to test with Postman, curl, or browser DevTools |
| 10 | **Security** | Works with HTTPS, JWT, OAuth, API keys |

### Stateless — The Most Important Constraint ⭐

```
❌ Stateful (bad):
Request 1: "Login as Deepak"  → Server remembers Deepak
Request 2: "Get my orders"    → Server uses memory of Request 1

✅ Stateless (REST):
Request 1: "Login as Deepak"  → Server issues JWT token
Request 2: "Get my orders"    → Client sends JWT token WITH the request
            Authorization: Bearer <token>
            Server reads token → knows who you are → responds
```

> **Why stateless matters for scale:** If the server remembers nothing, any server in a cluster can handle any request. No sticky sessions needed. This is why REST APIs scale horizontally so easily.

---

## 3. Building Blocks of REST

```
                    ┌─────────────────┐
                    │  Building Blocks │
                    └────────┬────────┘
          ┌─────────┬────────┼────────┬─────────┐
          ▼         ▼        ▼        ▼         ▼
        URL      Methods  Headers  Request   Response
                                             │
                                      ┌──────┴──────┐
                                      ▼             ▼
                                  Status Code    Body (JSON)
```

---

## 4. URL Anatomy

A URL has distinct parts, each with a specific role:

```
https://www.example.com/forum/questions/?tag=networking&order=newest
│       │   │           │    │           │
│       │   │           │    │           └── Query String (key=value pairs)
│       │   │           │    └── Subdirectory / Path
│       │   │           └── Path
│       │   └── Domain
│       └── Subdomain (www)
└── Scheme (https)

Also broken down:
┌──────┐  ┌───────────────────────────┐  ┌──────────────────────────────────────────┐
│scheme│  │           host            │  │                   path                   │
└──────┘  └───────────────────────────┘  └──────────────────────────────────────────┘
 https    :// www  . example . com        /forum/questions  ?tag=networking&order=newest
               ↑        ↑       ↑                ↑                    ↑
           subdomain  domain   TLD          subdirectory          query string
```

### URL Components

| Part | Example | Purpose |
|---|---|---|
| **Scheme** | `https://` | Protocol to use |
| **Subdomain** | `www.` | Sub-section of domain |
| **Domain** | `example` | The website name |
| **TLD** | `.com` | Top-level domain |
| **Path** | `/forum/questions` | Resource location |
| **Query String** | `?tag=networking` | Filter/search parameters |
| **Parameter** | `key=value` | Individual query param |

### Path Parameters vs Query Parameters

```
Path param   → /api/products/123          (identifies a specific resource)
Query param  → /api/products?category=shoes&sort=price  (filters/sorts)
```

```js
// Express.js
app.get('/api/products/:id', (req, res) => {
  const { id } = req.params;        // path param
  const { sort } = req.query;       // query param
});
```

---

## 5. HTTP Methods

### CRUD Mapping

| Method | CRUD | Action | Body? |
|---|---|---|---|
| `GET` | Read | Fetch resource(s) | No |
| `POST` | Create | Create new resource | Yes |
| `PUT` | Update | Replace entire resource | Yes |
| `PATCH` | Update | Partially update resource | Yes |
| `DELETE` | Delete | Remove resource | No |
| `HEAD` | Read | Like GET but returns headers only — no body (used to verify/check) | No |
| `OPTIONS` | — | Returns allowed methods — used in CORS preflight | No |
| `CONNECT` | — | Establishes a tunnel (used in HTTPS proxies) | No |
| `TRACE` | — | Diagnostic — echoes request back (rarely used) | No |

### PUT vs PATCH

```js
// PUT — replace the ENTIRE resource
PUT /api/users/1
{ "name": "Deepak", "email": "new@email.com", "age": 25 }
// Must send all fields — missing fields get wiped

// PATCH — update only specified fields
PATCH /api/users/1
{ "email": "new@email.com" }
// Only email changes, everything else stays the same
```

### Idempotency

| Method | Idempotent? | Safe? |
|---|---|---|
| GET | ✅ Yes | ✅ Yes |
| POST | ❌ No | ❌ No |
| PUT | ✅ Yes | ❌ No |
| PATCH | ❌ No | ❌ No |
| DELETE | ✅ Yes | ❌ No |

> **Idempotent** = calling it multiple times gives the same result. DELETE /users/1 ten times = same outcome as once (user is deleted).  
> **Safe** = doesn't modify server state. GET just reads.

---

## 6. Request Structure

Every HTTP request has three parts:

```
POST /api/products HTTP/1.1          ← Request Line (method + path + version)
Host: api.example.com                ← Headers start
Content-Type: application/json
Authorization: Bearer eyJhbGci...
Accept: application/json
                                     ← Blank line separates headers from body
{                                    ← Request Body (only POST/PUT/PATCH)
  "name": "Nike Air Max",
  "price": 4999,
  "category": "shoes"
}
```

### Three Parts:
1. **Request Line** — HTTP method + URL + HTTP version
2. **HTTP Headers** — metadata about the request
3. **HTTP Body** — data sent to server (POST, PUT, PATCH only)

---

## 7. Response Structure

Every HTTP response has three parts:

```
HTTP/1.1 200 OK                      ← Status Line (version + status code + message)
Content-Type: application/json       ← Response Headers start
Content-Length: 256
Date: Thu, 30 Nov 2023 08:00:00 GMT
Set-Cookie: session=abc123
                                     ← Blank line
{                                    ← Response Body
  "id": 1,
  "name": "Nike Air Max",
  "price": 4999
}
```

### Three Parts:
1. **Status Line** — HTTP version + status code + reason phrase
2. **Response Headers** — metadata about the response
3. **Response Body** — the actual data (usually JSON)

---

## 8. Request Headers

| Header | Purpose | Example |
|---|---|---|
| `Host` | Target host server | `host: www.example.com` |
| `Origin` | Origin host making the request | `origin: https://myapp.com` |
| `Referer` | Previous page that made the request | `referer: https://google.com/previous-page` |
| `User-Agent` | Client info sent to server — browser, OS, device | `Mozilla/5.0 (Macintosh)...` |
| `Accept` | Response content type client can handle | `application/json` |
| `Accept-Language` | Preferred response language | `en-US, en;q=0.9` |
| `Accept-Encoding` | Encoding algorithms client supports | `gzip, deflate, br` |
| `Connection` | Keep TCP connection open or close | `keep-alive` / `close` |
| `Authorization` | Send credentials / auth token | `Authorization: Bearer <jwt>` |
| `Cookie` | Previously set auth token / session data | `key=value; key2=value2` |
| `If-Modified-Since` | Conditional — only send if changed since this date | `Thu, 30 Nov 2023 08:00:00 GMT` |
| `Cache-Control` | Caching directives for request | `no-cache`, `max-age=3600` |
| `Content-Type` | Format of request body | `application/json` |
| `Content-Length` | Size of request body in bytes | `348` |

### Authorization Header Types

```
Authorization: Bearer <JWT token>        ← Most common for REST APIs
Authorization: Basic <base64(user:pass)> ← Basic auth (avoid in production)
Authorization: ApiKey <your-api-key>     ← API key auth
```

### Accept-Encoding — Why It Matters for Performance

When client sends `Accept-Encoding: gzip`, the server can compress the response body. A 1MB JSON response can become 100KB after gzip — **10x smaller**, much faster over the network.

---

## 9. Response Headers

| Header | Purpose | Example |
|---|---|---|
| `Date` | When the response was generated | `Thu, 30 Nov 2023 08:00:00 GMT` |
| `Server` | Server software info | `nginx`, `Apache` |
| `Content-Type` | Type of response content | `text/html`, `application/json` |
| `Content-Length` | Size of response body | `256` (KB) |
| `Set-Cookie` | Cookie to be stored for future requests | `Set-Cookie: session=abc; HttpOnly` |
| `Cache-Control` | Caching rules for response | `max-age=3600`, `no-store` |
| `Last-Modified` | When resource was last changed | `Thu, 30 Nov 2023 07:00:00 GMT` |
| `ETag` | Unique identifier for this version of resource | `"abc123xyz"` |
| `Expires` | When cached response expires | `Fri, 01 Dec 2023 00:00:00 GMT` |
| `Access-Control-Allow-Origin` | CORS — which origins can access | `*` or `https://myapp.com` |
| `Location` | Redirect URL (used with 301, 302, 201) | `https://example.com/new-url` |

### ETag — Smart Caching

```
1st Request:
Client  →  GET /api/products
Server  →  200 OK + ETag: "v1-abc123" + data

2nd Request (client sends ETag back):
Client  →  GET /api/products + If-None-Match: "v1-abc123"
Server  →  304 Not Modified (no body sent, use cache)
           OR
           200 OK + ETag: "v2-xyz789" + new data (if changed)
```

> ETag prevents sending the same data twice. Huge bandwidth saving for large responses.

### Cache-Control Values

| Value | Meaning |
|---|---|
| `max-age=3600` | Cache for 3600 seconds (1 hour) |
| `no-cache` | Must revalidate with server before using cache |
| `no-store` | Never cache (sensitive data) |
| `private` | Only browser can cache (not CDN) |
| `public` | CDN and browser can cache |
| `must-revalidate` | Expired cache must be revalidated before use |

---

## 10. HTTP Status Codes

### 2xx — Success
| Code | Meaning | When to use |
|---|---|---|
| `200` | OK | Successful GET, PUT, PATCH |
| `201` | Created | Successful POST — resource created |
| `204` | No Content | Successful DELETE — nothing to return |

### 4xx — Client Errors
| Code | Meaning | When to use |
|---|---|---|
| `400` | Bad Request | Invalid request syntax, missing required fields |
| `401` | Unauthorized | Not authenticated — missing or invalid token |
| `403` | Forbidden | Authenticated but not allowed to access this resource |
| `404` | Not Found | Resource doesn't exist |
| `405` | Method Not Allowed | HTTP method not supported on this endpoint |
| `429` | Too Many Requests | Rate limit exceeded |

### 5xx — Server Errors
| Code | Meaning | When to use |
|---|---|---|
| `500` | Internal Server Error | Unhandled server crash |
| `502` | Bad Gateway | Server received invalid response from upstream |
| `503` | Service Unavailable | Server down / overloaded |
| `504` | Gateway Timeout | Upstream server didn't respond in time |
| `507` | Insufficient Storage | Server ran out of storage |

### 401 vs 403 — The Classic Confusion

```
401 Unauthorized  → "Who are you? Show me your token."
                    (Not logged in / bad token)

403 Forbidden     → "I know who you are, but you can't do this."
                    (Logged in, but wrong role/permissions)
```

---

## 11. REST API Design Best Practices

### URL Naming Conventions

```
✅ Good — nouns, plural, lowercase, hyphens
GET    /api/products
GET    /api/products/123
GET    /api/products/123/reviews
POST   /api/products
PATCH  /api/products/123
DELETE /api/products/123

❌ Bad — verbs in URL, inconsistent
GET    /api/getProducts
POST   /api/createNewProduct
GET    /api/Product/GetById/123
```

### Versioning

```
/api/v1/products    ← version in URL (most common)
/api/v2/products    ← new version, old one still works
```

> Always version your APIs. Changing a v1 endpoint breaks existing clients.

### Filtering, Sorting, Pagination

```
# Filtering
GET /api/products?category=shoes&brand=nike

# Sorting
GET /api/products?sort=price&order=asc

# Pagination
GET /api/products?page=2&limit=20

# Search
GET /api/products?search=air+max
```

### Consistent Error Response Format

```json
{
  "status": 400,
  "error": "Bad Request",
  "message": "The 'price' field is required",
  "timestamp": "2023-11-30T08:00:00Z",
  "path": "/api/products"
}
```

---

## 12. Frontend Engineer Cheatsheet

### Fetch API — Complete Pattern

```js
const fetchProducts = async () => {
  try {
    const response = await fetch('/api/products', {
      method: 'GET',
      headers: {
        'Content-Type': 'application/json',
        'Authorization': `Bearer ${token}`,
      },
    });

    if (!response.ok) {
      throw new Error(`HTTP error: ${response.status}`);
    }

    const data = await response.json();
    return data;
  } catch (error) {
    console.error('API error:', error);
  }
};
```

### POST Request

```js
const createProduct = async (product) => {
  const response = await fetch('/api/products', {
    method: 'POST',
    headers: {
      'Content-Type': 'application/json',
      'Authorization': `Bearer ${token}`,
    },
    body: JSON.stringify(product),  // body only for POST/PUT/PATCH
  });

  if (response.status === 201) {
    return await response.json();
  }
};
```

### With React Query (Recommended)

```js
// GET
const { data, isLoading, error } = useQuery({
  queryKey: ['products'],
  queryFn: () => fetch('/api/products').then(r => r.json()),
});

// POST / mutation
const mutation = useMutation({
  mutationFn: (newProduct) =>
    fetch('/api/products', {
      method: 'POST',
      headers: { 'Content-Type': 'application/json' },
      body: JSON.stringify(newProduct),
    }).then(r => r.json()),
  onSuccess: () => queryClient.invalidateQueries(['products']),
});
```

### Quick Reference

| Scenario | Status Code | Action in frontend |
|---|---|---|
| Data loaded | 200 | Render data |
| Created successfully | 201 | Show success toast, redirect |
| Deleted successfully | 204 | Remove from UI |
| Validation error | 400 | Show field errors |
| Not logged in | 401 | Redirect to login |
| No permission | 403 | Show "Access Denied" |
| Not found | 404 | Show empty state |
| Rate limited | 429 | Show "Try again later" + backoff |
| Server crashed | 500 | Show generic error message |

### Key Terms

| Term | Meaning |
|---|---|
| **REST** | Architectural style for APIs using HTTP |
| **Stateless** | Server stores no session — client sends all context with every request |
| **Idempotent** | Same request called multiple times = same result |
| **CRUD** | Create, Read, Update, Delete |
| **ETag** | Version identifier for a resource — enables smart caching |
| **Query Param** | Filter/sort data → `?key=value` |
| **Path Param** | Identify specific resource → `/users/:id` |
| **CORS** | Cross-Origin Resource Sharing — controls which origins can call your API |
| **Preflight** | OPTIONS request browser sends before actual CORS request |

---

*Chapter 03 complete. Next → Chapter 04.*
