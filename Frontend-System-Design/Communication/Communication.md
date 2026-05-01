# 📡 Frontend System Design — Communication

> **Chapter:** Communication Patterns between Client & Server  
> **Topics Covered:** Short Polling · Long Polling · WebSockets · Server-Sent Events · WebHooks

---

## Table of Contents
1. [Short Polling](#1-short-polling)
2. [Long Polling](#2-long-polling)
3. [WebSockets](#3-websockets)
4. [Server-Sent Events (SSE)](#4-server-sent-events-sse)
5. [WebHooks](#5-webhooks)
6. [Quick Comparison Table](#-quick-comparison-table)

---

## 1. Short Polling

### How it Works
The client sends a request to the server at **fixed intervals** (e.g., every 5 seconds), regardless of whether new data is available. The server responds immediately and closes the connection.

```
Client ──── Request A ────► Server
Client ◄─── Response A ─── Server

(after 5s...)

Client ──── Request B ────► Server
Client ◄─── Response B ─── Server
```

### Characteristics
- ✅ Short-lived connection (no persistent connection)
- ✅ Less resource usage per request
- ✅ Simple to implement
- ❌ Problem with scale — too many redundant requests
- ❌ Not truly real-time (delay = polling interval)

### Code Example (JavaScript)
```js
let intervalId;

function startShortPolling() {
  intervalId = setInterval(() => {
    getData(); // fetch latest data from server
  }, 5000); // every 5 seconds
}

function stopShortPolling() {
  return () => clearInterval(intervalId);
}

async function getData() {
  const res = await fetch('/api/data');
  const data = await res.json();
  console.log(data);
}
```

### Real-World Use Cases
- Real-time dashboards (low-frequency updates)
- Notification badges
- Cricinfo (score updates)
- Analytics dashboards
- App version update checks

---

## 2. Long Polling

### How it Works
The client sends a request, but the server **holds the connection open** until new data is available or a timeout occurs. Once the server responds, the client immediately sends another request.

```
Client ──── Request ────────────────► Server
                          (server waits...)
Client ◄─── Response (new data/timeout) ── Server
Client ──── Request (immediately) ──► Server
```

### Characteristics
- ✅ Single long-lived connection (more efficient than short polling)
- ✅ Near real-time — response is sent as soon as data is ready
- ❌ Cons: Large number of connections can increase server load
- ❌ More complex to implement than short polling
- ❌ Timeout handling adds complexity

### How it differs from Short Polling
| | Short Polling | Long Polling |
|---|---|---|
| Connection | Opens & closes repeatedly | Stays open until data/timeout |
| Delay | Fixed interval delay | Near-instant when data arrives |
| Server Load | High (constant requests) | Medium (fewer but longer connections) |

### Code Example (JavaScript)
```js
async function longPoll() {
  try {
    const res = await fetch('/api/long-poll'); // server holds this
    const data = await res.json();
    console.log('New data:', data);
  } catch (err) {
    console.error('Polling error:', err);
  } finally {
    longPoll(); // immediately reconnect
  }
}

longPoll(); // start
```

### Real-World Use Cases
- Chat applications (before WebSockets were common)
- Real-time notifications
- Live sports score updates

---

## 3. WebSockets

### How it Works
A **persistent, full-duplex** (two-way) TCP connection is established via an HTTP Upgrade handshake. Both client and server can send messages to each other at any time.

```
Client ──── Handshake (HTTP Upgrade) ────► Server
Client ◄─── Connection Opened ─────────── Server

Client ◄───► Bidirectional Messages ◄───► Server

Client ──── Connection Closed ───────────► Server
```

### Characteristics
1. **Full Duplex** — both sides can send/receive simultaneously
2. **Single long-lived TCP connection** — no repeated handshakes
3. **Continuous bi-directional communication**
4. Protocol: `ws://` (or `wss://` for secure)

### Code Example (JavaScript)
```js
// Client side
const socket = new WebSocket('ws://localhost:3000');

socket.onopen = () => {
  console.log('Connected!');
  socket.send(JSON.stringify({ type: 'hello' }));
};

socket.onmessage = (event) => {
  const data = JSON.parse(event.data);
  console.log('Received:', data);
};

socket.onclose = () => console.log('Disconnected');
socket.onerror = (err) => console.error('Error:', err);
```

```js
// Server side (Node.js with 'ws' library)
const WebSocket = require('ws');
const wss = new WebSocket.Server({ port: 3000 });

wss.on('connection', (ws) => {
  ws.on('message', (message) => {
    console.log('Received:', message);
    ws.send(JSON.stringify({ reply: 'Got it!' }));
  });
});
```

### Libraries
- **`socket.io`** — abstraction over WebSockets with fallbacks, rooms, namespaces
- **`ws`** — lightweight, raw WebSocket implementation for Node.js

### Real-World Use Cases
- Analytics & financial trading platforms
- Online gaming (real-time state sync)
- Collaborative tools (Google Sheets, Figma)
- Live chat applications

### Challenges
| Challenge | Description |
|---|---|
| Resource Usage | Each open connection consumes memory |
| Connection Limits | Servers have max connection caps |
| Sticky Sessions | Load balancer must route same client to same server |
| Authentication | Tokens must be validated on connection |
| Resource Cleanup | Must handle disconnect and cleanup gracefully |
| Firewall/Proxy | Some proxies block `ws://` protocol |
| Scaling | Horizontal scaling requires pub/sub (e.g., Redis) |
| Testing/Debugging | Harder than HTTP — needs special tools |

---

## 4. Server-Sent Events (SSE)

### How it Works
A **unidirectional** stream from server to client over a single HTTP connection. The client opens a connection and the server pushes events whenever they're available.

```
Client ──── Connection Opened ───────────► Server
Client ◄─── event ──────────────────────── Server
Client ◄─── event ──────────────────────── Server
Client ◄─── event ──────────────────────── Server
Client ◄─── Connection Closed ──────────── Server
```

### Characteristics
- ✅ Long-lived **unidirectional** (Server → Client only)
- ✅ Single HTTP connection (uses standard HTTP, no special protocol)
- ✅ Auto-reconnect built into the browser
- ✅ Works well for feeds, notifications, monitoring dashboards
- ✅ `Connection: keep-alive` header keeps it alive
- ✅ Uses **event-stream** format — if data is large, it comes in frames
- ❌ Browser compatibility (older browsers may not support)
- ❌ Connection limits (browsers allow ~6 per domain)
- ❌ Proxy/firewall issues
- ❌ Connection timeout needs handling

### SSE vs WebSockets
| | SSE | WebSockets |
|---|---|---|
| Direction | One-way (Server → Client) | Two-way |
| Protocol | HTTP | WS (TCP upgrade) |
| Auto-reconnect | ✅ Built-in | ❌ Manual |
| Complexity | Simple | More complex |
| Use Case | Live feeds, notifications | Chat, gaming |

### Code Example (JavaScript)
```js
// Client side
const eventSource = new EventSource('/api/events');

eventSource.onmessage = (event) => {
  console.log('New event:', event.data);
};

eventSource.onerror = () => {
  console.error('Connection error');
  eventSource.close();
};
```

```js
// Server side (Node.js / Express)
app.get('/api/events', (req, res) => {
  res.setHeader('Content-Type', 'text/event-stream');
  res.setHeader('Cache-Control', 'no-cache');
  res.setHeader('Connection', 'keep-alive');

  const sendEvent = (data) => {
    res.write(`data: ${JSON.stringify(data)}\n\n`);
  };

  const interval = setInterval(() => {
    sendEvent({ time: new Date().toISOString() });
  }, 1000);

  req.on('close', () => clearInterval(interval)); // cleanup
});
```

### Real-World Use Cases
- Live news/social media feeds
- Notification systems
- Monitoring dashboards
- Stock price tickers (read-only)

### Challenges
- Browser compatibility
- Connection limits (concurrent per domain)
- Connection timeout management
- Resource utilization on server
- Proxy / firewalls may buffer SSE responses

---

## 5. WebHooks

### How it Works
Instead of the client polling the server, the **external service calls your server** when an event happens. You register a callback URL; the service POSTs data to it.

**Polling approach (inefficient):**
```
Polling ──► API Gateway ──► Order Service ──► Payment Service
                                               ↑↓ (keep asking)
                                    External Payment Processor (Stripe)
```

**WebHook approach (event-driven):**
```
WebHook ──► API Gateway ──► Order Service ──► Payment Service
                                               ↓
                                    Register: callback at
                                    https://myweb.com/payment/stripe
                                               ↓
                                    External Payment Processor (Stripe)
                                    (calls back with result when done)
```

### Characteristics
- ✅ **Real-time communication** — triggered by events, not polling
- ✅ **Event-driven** architecture
- ✅ Uses **POST REST API** with payload data + authorization headers
- ✅ **Retry mechanism** — if your server is down, the service retries
- ✅ **Verification/Acknowledgement** — server must return `200 OK` to confirm receipt
- ❌ Your server must be publicly accessible (no localhost)
- ❌ Security — must verify webhook signatures
- ❌ No control over when events arrive

### Implementation Checklist
```
1. Register your callback URL with the service (e.g., Stripe Dashboard)
2. Expose a POST endpoint on your server
3. Validate the webhook signature (HMAC)
4. Return 200 OK immediately (process async)
5. Handle retries idempotently
```

### Code Example (Node.js + Express + Stripe)
```js
const express = require('express');
const stripe = require('stripe')(process.env.STRIPE_SECRET);
const app = express();

app.post('/payment/stripe', express.raw({ type: 'application/json' }), (req, res) => {
  const sig = req.headers['stripe-signature'];

  let event;
  try {
    // Verify signature to ensure request is from Stripe
    event = stripe.webhooks.constructEvent(req.body, sig, process.env.WEBHOOK_SECRET);
  } catch (err) {
    return res.status(400).send(`Webhook Error: ${err.message}`);
  }

  // Handle different event types
  switch (event.type) {
    case 'payment_intent.succeeded':
      console.log('Payment successful!', event.data.object);
      break;
    case 'payment_intent.payment_failed':
      console.log('Payment failed!', event.data.object);
      break;
  }

  res.json({ received: true }); // Must return 200
});
```

### Real-World Use Cases
- **Notification systems** — send alerts when events happen
- **Data synchronization** — Instagram ↔ Facebook cross-posting
- **CI/CD pipelines** — GitHub webhooks trigger builds on push, merge, PR
- **Payment processing** — Stripe notifies on payment success/failure

---

## 🔍 Quick Comparison Table

| Feature | Short Polling | Long Polling | WebSockets | SSE | WebHooks |
|---|---|---|---|---|---|
| **Direction** | Client→Server | Client→Server | Bidirectional | Server→Client | Server→Server |
| **Connection** | Short-lived | Medium-lived | Persistent | Persistent | Stateless |
| **Latency** | High (interval) | Low | Very Low | Low | Near-instant |
| **Complexity** | Low | Medium | High | Medium | Medium |
| **Server Load** | High | Medium | Medium | Medium | Low |
| **Use Case** | Simple updates | Notifications | Chat/Gaming | Live feeds | Event triggers |
| **Protocol** | HTTP | HTTP | WS (TCP) | HTTP | HTTP POST |

---

## 🧠 When to Use What?

```
Need two-way real-time?          → WebSockets
Need server-push only?           → SSE
Need event from external system? → WebHooks
Need simple periodic updates?    → Short Polling
Need near-real-time, simpler?    → Long Polling
```

---

## 📚 Further Reading

- [MDN WebSockets API](https://developer.mozilla.org/en-US/docs/Web/API/WebSockets_API)
- [MDN Server-Sent Events](https://developer.mozilla.org/en-US/docs/Web/API/Server-sent_events)
- [Stripe Webhooks Guide](https://stripe.com/docs/webhooks)
- [Socket.io Documentation](https://socket.io/docs/v4/)
- [WebSocket vs SSE vs Long Polling](https://ably.com/blog/websockets-vs-long-polling)
