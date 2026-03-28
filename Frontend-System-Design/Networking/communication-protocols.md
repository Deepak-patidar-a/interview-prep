# Communication Protocols

> **Module:** Frontend System Design  
> **Chapter:** 02 — Communication Protocols

---

## Table of Contents
1. [What is a Protocol?](#1-what-is-a-protocol)
2. [HTTP](#2-http)
3. [HTTPS](#3-https)
4. [HTTP/3 (QUIC)](#4-http3-quic)
5. [WebSocket](#5-websocket)
6. [TCP](#6-tcp)
7. [UDP](#7-udp)
8. [SMTP](#8-smtp)
9. [FTP](#9-ftp)
10. [Quick Comparison Table](#10-quick-comparison-table)
11. [TCP vs UDP — When to Use What](#11-tcp-vs-udp--when-to-use-what)
12. [Frontend Engineer Cheatsheet](#12-frontend-engineer-cheatsheet)

---

## 1. What is a Protocol?

A **protocol** is a set of rules that define how data is transmitted and received between devices.

Think of it like a language — both client and server must speak the same protocol to communicate. Different protocols exist because different use cases have different requirements: reliability, speed, security, real-time capability.

---

## 2. HTTP

**HyperText Transfer Protocol** — the foundation of data communication on the web.

### How it works
```
Client                        Server
  │──── TCP Connection ──────▶│
  │──── HTTP Request ────────▶│   GET /index.html
  │◀─── HTTP Response ────────│   200 OK + HTML
```

### Request / Response Structure

**Request:**
```
GET /api/products HTTP/1.1
Host: example.com
Accept: application/json
Authorization: Bearer <token>
```

**Response:**
```
HTTP/1.1 200 OK
Content-Type: application/json
Cache-Control: max-age=3600

{ "products": [...] }
```

### HTTP Versions
| Version | Connection | Key Feature |
|---|---|---|
| HTTP/1.0 | New TCP per request | Simple, slow |
| HTTP/1.1 | Keep-alive (reuse TCP) | Persistent connections, but HOL blocking |
| HTTP/2 | Multiplexing over 1 TCP | Multiple requests in parallel, header compression |
| HTTP/3 | UDP-based (QUIC) | No HOL blocking, faster on bad networks |

> **HOL Blocking (Head-of-Line):** In HTTP/1.1, if one request is slow, it blocks all requests behind it in that connection — like a slow car blocking a one-lane road.

### HTTP Methods
| Method | Purpose |
|---|---|
| `GET` | Fetch a resource |
| `POST` | Create a resource |
| `PUT` | Replace a resource entirely |
| `PATCH` | Partially update a resource |
| `DELETE` | Remove a resource |
| `OPTIONS` | Check what methods server supports (used in CORS preflight) |

### Use Cases
- Web browsing
- REST API calls
- Fetching HTML, CSS, JS files

---

## 3. HTTPS

**HTTP Secure** — HTTP with encryption via SSL/TLS on top.

### How it works
```
Client                              Server
  │──── TCP Connection ────────────▶│
  │◀─── Public Key (Certificate) ───│   Server sends its public key
  │──── Session Key (encrypted) ───▶│   Client encrypts session key with public key
  │◀─── Encrypted Data ─────────────│   All further data is encrypted
```

### Why It Matters
- **Encryption** — data in transit is unreadable to anyone intercepting
- **Authentication** — certificate proves the server is who it claims to be
- **Data Integrity** — data cannot be tampered with in transit

### Public Key vs Session Key
- **Public Key** — shared openly by server. Used to encrypt the session key.
- **Session Key (Private/Symmetric Key)** — generated per-connection. Used for actual data encryption. Much faster than asymmetric encryption.

> This is called **hybrid encryption** — asymmetric (slow, secure) for key exchange, symmetric (fast) for data transfer.

### Use Cases
- All modern web browsing
- Any page handling login, payments, personal data
- Required for HTTP/2, Service Workers, and PWAs

> **Senior tip:** Google penalizes non-HTTPS sites in search ranking. Browsers show "Not Secure" warnings. There is no valid reason to ship a production site without HTTPS in 2024. Use Let's Encrypt — it's free.

---

## 4. HTTP/3 (QUIC)

The latest version of HTTP, built on **UDP** instead of TCP.

### The Problem with HTTP/2
HTTP/2 solved HOL blocking at the HTTP layer, but TCP still has HOL blocking at the transport layer. If one TCP packet is lost, all streams wait.

### How HTTP/3 Solves It
```
HTTP/2 (TCP)                    HTTP/3 (QUIC / UDP)
──────────────                  ──────────────────────
Stream 1: ████░░░░              Stream 1: ████████  ✓
Stream 2: ████████  (waiting)   Stream 2: ████████  ✓
Stream 3: ████████  (waiting)   Stream 3: ████████  ✓
          ↑ packet loss                   independent
          blocks all
```

### QUIC Protocol
- Built on **UDP** — no handshake overhead
- **0-RTT connection** — returning users connect with zero round trips (connection resumed instantly)
- Each stream is **independent** — packet loss in stream 1 doesn't affect stream 2
- Built-in encryption (TLS 1.3 always included)

### Use Cases
- IoT (Internet of Things)
- Virtual Reality / AR applications
- YouTube video streaming
- Google Search (Google uses QUIC everywhere)
- Gaming

---

## 5. WebSocket

A protocol that provides **full-duplex** (two-way) communication over a single, persistent connection.

### How it works
```
Client                        Server
  │──── HTTP Upgrade ────────▶│   "I want to upgrade to WebSocket"
  │◀─── 101 Switching ────────│   "OK, switching protocols"
  │                            │
  │◀══════ Full Duplex ═══════▶│   Both can send at any time
  │◀══════ messages ══════════▶│   No need to request first
  │                            │
  │   (connection stays open)  │
```

### HTTP vs WebSocket
| | HTTP | WebSocket |
|---|---|---|
| Direction | Client requests, server responds | Both can send anytime |
| Connection | Opens and closes per request | Stays open |
| Overhead | Headers sent every request | Headers only on handshake |
| Use case | Fetch data on demand | Real-time, event-driven data |

### WebSocket in React
```js
const socket = new WebSocket('wss://example.com/chat');

socket.onopen = () => console.log('Connected');
socket.onmessage = (event) => console.log('Message:', event.data);
socket.onclose = () => console.log('Disconnected');

// Send message
socket.send(JSON.stringify({ type: 'chat', text: 'Hello!' }));
```

### Use Cases
- **Live chat** (WhatsApp Web, Slack)
- **Real-time collaboration** (Figma, Google Docs)
- **Live sports scores / stock prices**
- **Online multiplayer games**
- **Real-time data dashboards**
- **Notifications**

> **Socket.io** is a popular library built on top of WebSocket that adds fallbacks, rooms, namespaces, and auto-reconnection. Used in Deepak's Fasal Mitra project for expert chat rooms.

---

## 6. TCP

**Transmission Control Protocol** — a reliable, ordered, connection-based protocol.

### How it works — 3-Way Handshake
```
Client              Server
  │──── SYN ───────▶│    "I want to connect"
  │◀─── SYN + ACK ──│    "OK, I acknowledge. Ready?"
  │──── ACK ────────▶│    "Let's go"
  │                  │
  │   Data Transfer  │
  │                  │
  │──── FIN ────────▶│    "I'm done" (4-way close)
```

### TCP Guarantees
- ✅ **Delivery** — every packet is acknowledged; missing packets are retransmitted
- ✅ **Order** — packets arrive in the correct sequence
- ✅ **Error checking** — checksums validate data integrity
- ❌ **Speed** — all this reliability adds overhead

### Use Cases
- Web browsing (HTTP/HTTPS)
- Email (sending/receiving)
- File transfers
- Banking transactions
- Any use case where data accuracy matters more than speed

---

## 7. UDP

**User Datagram Protocol** — a fast, connectionless protocol. Fire and forget.

### How it works
```
Client              Server
  │──── Request ───▶│    (no handshake)
  │◀─── Response ───│    (no guarantee it arrived)
  │                  │
  (no connection maintained)
```

### UDP Characteristics
- ✅ **Fast** — no handshake, no acknowledgment overhead
- ✅ **Low latency** — packets go out immediately
- ❌ **No delivery guarantee** — packets can be lost
- ❌ **No order guarantee** — packets can arrive out of sequence
- ❌ **No error recovery** — lost packets are not retransmitted

### TCP vs UDP
| | TCP | UDP |
|---|---|---|
| Connection | Yes (handshake) | No |
| Reliable delivery | Yes | No |
| Ordered | Yes | No |
| Speed | Slower | Faster |
| Use when | Accuracy matters | Speed matters |

### Use Cases
- **Video conferencing** (Zoom, Google Meet)
- **Online gaming** — a dropped frame is better than freezing
- **DNS lookups** — small, fast, one request
- **Live video streaming**
- **IoT sensor data**

> **Why video calls use UDP:** If a video frame is lost, it's better to skip it and continue than to freeze and wait for retransmission. A 1-second freeze is worse than 1 blurry frame.

---

## 8. SMTP

**Simple Mail Transfer Protocol** — used for sending and receiving emails.

### How it works
```
[Sender] ──▶ [SMTP Server] ──▶ [Receiver's Mail Server] ──▶ [Receiver]
```

- Sender composes email → sent to their SMTP server
- SMTP server looks up the receiver's mail server (via DNS MX record)
- Delivers the email to receiver's server
- Receiver fetches it using **IMAP** or **POP3**

### SMTP vs IMAP vs POP3
| Protocol | Purpose |
|---|---|
| **SMTP** | Sending emails |
| **IMAP** | Reading emails (synced across devices) |
| **POP3** | Reading emails (downloaded, removed from server) |

### Use Cases
- Sending/receiving emails
- Transactional emails from apps (password reset, OTP, order confirmation)
- Used by services like SendGrid, AWS SES, Mailgun under the hood

> **Frontend relevance:** When you integrate email notifications in a web app (nodemailer, SendGrid SDK), you're using SMTP under the hood.

---

## 9. FTP

**File Transfer Protocol** — used for transferring files between client and server.

### How it works — Two Channels
```
Client                        Server
  │──── Control Channel ────▶│   Commands: LIST, RETR, STOR
  │◀─── Control Channel ─────│   Responses: 200 OK, 550 Error
  │                           │
  │──── Data Channel ───────▶│   Actual file transfer (separate connection)
  │◀─── Data Channel ─────────│
```

FTP uses **two separate connections:**
- **Control Channel** — sends commands (what to do)
- **Data Channel** — transfers actual file data

### FTP Variants
| Variant | Description |
|---|---|
| **FTP** | Plain, unencrypted — avoid in production |
| **FTPS** | FTP + SSL/TLS encryption |
| **SFTP** | SSH File Transfer Protocol — most secure, different protocol entirely |

### Use Cases
- Deploying files to a server (old-school hosting)
- Transferring large files between servers
- Legacy enterprise systems

> **Modern alternative:** Most teams now use **Git + CI/CD** (push to GitHub → auto-deploy) instead of manual FTP. FTP is largely legacy but still appears in enterprise environments.

---

## 10. Quick Comparison Table

| Protocol | Layer | Connection | Reliable | Speed | Use Case |
|---|---|---|---|---|---|
| **HTTP** | Application | TCP | Yes | Medium | Web browsing, REST APIs |
| **HTTPS** | Application | TCP + TLS | Yes + Encrypted | Medium | Secure web, all production |
| **HTTP/3** | Application | UDP (QUIC) | Yes | Fast | Streaming, IoT, modern web |
| **WebSocket** | Application | TCP (persistent) | Yes | Fast (real-time) | Chat, live data, games |
| **TCP** | Transport | Connection-based | Yes | Slower | Emails, file transfer, HTTP |
| **UDP** | Transport | Connectionless | No | Fastest | Video calls, gaming, DNS |
| **SMTP** | Application | TCP | Yes | Medium | Sending emails |
| **FTP** | Application | TCP (2 channels) | Yes | Medium | File transfers |

---

## 11. TCP vs UDP — When to Use What

```
Is data accuracy more important than speed?
├── YES → TCP
│         Web browsing, emails, file downloads, APIs
│
└── NO → UDP
          Is a dropped packet worse than a delay?
          ├── YES → TCP (e.g. banking transaction)
          └── NO  → UDP (e.g. video call, live game)
```

---

## 12. Frontend Engineer Cheatsheet

### Which protocol does what in a typical web app?

```
User opens browser         → HTTP/HTTPS (page load)
REST API calls             → HTTP/HTTPS
Live chat feature          → WebSocket
Real-time notifications    → WebSocket or SSE (Server-Sent Events)
Video call feature         → WebRTC (uses UDP under the hood)
File upload to server      → HTTP (multipart/form-data) or SFTP
Sending transactional email → SMTP (via SendGrid/Nodemailer)
DNS lookup                 → UDP
```

### SSE vs WebSocket (bonus — commonly asked)
| | Server-Sent Events (SSE) | WebSocket |
|---|---|---|
| Direction | Server → Client only | Full duplex (both ways) |
| Protocol | HTTP | WebSocket (ws://) |
| Reconnect | Auto | Manual |
| Use case | Live feed, notifications | Chat, collaboration |

```js
// SSE — one way, server pushes updates
const source = new EventSource('/api/live-feed');
source.onmessage = (e) => console.log(e.data);
```

### Key Terms
| Term | Meaning |
|---|---|
| **Full Duplex** | Both sides can send/receive simultaneously |
| **Half Duplex** | Only one side can send at a time |
| **Handshake** | Connection setup negotiation between client and server |
| **HOL Blocking** | Head-of-Line blocking — slow request blocks others behind it |
| **QUIC** | Google's protocol — UDP-based, used in HTTP/3 |
| **0-RTT** | Zero Round Trip Time — instant reconnection in HTTP/3 |
| **TLS** | Transport Layer Security — encryption layer in HTTPS |
| **MX Record** | DNS record that tells where to deliver email for a domain |

---

*Chapter 02 complete. Next → Chapter 03.*
