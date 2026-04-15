# 🚀 Express.js — Interview Revision Notes

> Topic: Express.js fundamentals, middleware, routing, error handling, security  
> Express.js is the standard server framework for Node.js

---

## 🧠 What is Express.js?

Express.js is a **minimal and flexible Node.js web application framework** that provides tools for building web and mobile applications.

- Built on top of Node.js
- Simplifies routing, middleware, and HTTP utilities
- Used to build REST APIs, real-time apps, and web applications
- Default port by convention: **3000** (but can be any port)

```javascript
const express = require('express');
const app = express();

app.get('/', (req, res) => {
  res.send('Hello Express!');
});

app.listen(3000, () => {
  console.log('Server running on port 3000');
});
```

---

## 🗂️ Project Structure (MVC Pattern)

```
my-app/
├── models/          → User.js, Post.js (data + DB schema)
├── controllers/     → userController.js (business logic)
├── routes/          → userRoutes.js (URL definitions)
├── middleware/      → auth.js, errorHandler.js
├── config/          → db.js, env config
├── app.js           → express setup, middleware, routes
├── server.js        → http server, port binding
└── package.json
```

> **Why separate app.js and server.js?**  
> `app.js` handles the Express app (routes, middleware). `server.js` handles the HTTP server and port. This makes the app reusable and easier to test.

---

## ⚙️ Middleware — Most Important Topic

Middleware functions have access to **`req`**, **`res`**, and **`next`**. They run between request and response.

```
Request → Middleware 1 → Middleware 2 → Route Handler → Response
```

### Types of Middleware:

| Type | How to use | Example |
|---|---|---|
| Application-level | `app.use()` | Logging, body parsing |
| Router-level | `router.use()` | Route-specific auth |
| Error-handling | 4 args: `(err, req, res, next)` | Global error handler |
| Built-in | `express.json()`, `express.static()` | Body parsing, static files |
| Third-party | `cors()`, `morgan()`, `helmet()` | Security, logging |

### Custom Middleware Example:
```javascript
// Logging middleware
app.use((req, res, next) => {
  console.log(`${req.method} ${req.url} - ${new Date().toISOString()}`);
  next(); // MUST call next() or request hangs forever!
});

// Route-specific middleware
const authMiddleware = (req, res, next) => {
  const token = req.headers.authorization;
  if (!token) return res.status(401).json({ message: 'Unauthorized' });
  next();
};

app.get('/dashboard', authMiddleware, (req, res) => {
  res.send('Welcome to dashboard');
});
```

### ⚠️ Important: Always call `next()` in middleware or the request will hang!

---

## 📍 Routing

### Basic Routes:
```javascript
app.get('/users', getAllUsers);      // GET - fetch all
app.post('/users', createUser);     // POST - create
app.get('/users/:id', getUser);     // GET - fetch one
app.put('/users/:id', updateUser);  // PUT - replace
app.patch('/users/:id', patchUser); // PATCH - partial update
app.delete('/users/:id', deleteUser); // DELETE - remove
```

### app.use() vs app.get():
| Feature | `app.use()` | `app.get()` |
|---|---|---|
| HTTP Methods | All methods | Only GET |
| Path matching | Partial (`/api` matches `/api/users`) | Exact match |
| Use for | Middleware | Route handlers |

### Express Router (for modular code):
```javascript
// routes/userRoutes.js
const router = express.Router();

router.get('/', getAllUsers);
router.post('/', createUser);
router.get('/:id', getUserById);
router.put('/:id', updateUser);
router.delete('/:id', deleteUser);

module.exports = router;

// app.js
app.use('/api/users', require('./routes/userRoutes'));
```

---

## 📥 Request Data — req.params vs req.query vs req.body

```javascript
// URL: GET /users/123?page=2&limit=10
// Body: { "name": "John", "email": "john@example.com" }

app.use(express.json()); // required to parse req.body!

app.put('/users/:id', (req, res) => {
  const { id } = req.params;        // "123" — from URL path
  const { page, limit } = req.query; // "2", "10" — after the ?
  const { name, email } = req.body;  // from JSON body
});
```

| Source | Access via | Used for |
|---|---|---|
| URL path `/users/:id` | `req.params.id` | Resource identifiers |
| Query string `?page=2` | `req.query.page` | Filtering, pagination |
| Request body (JSON) | `req.body.name` | Creating/updating data |
| Headers | `req.headers.authorization` | Auth tokens |

---

## 🚨 Error Handling

### Error-handling middleware — 4 arguments, placed LAST:
```javascript
// Async route — must use try/catch and pass error to next()
app.get('/users', async (req, res, next) => {
  try {
    const users = await User.find();
    res.json(users);
  } catch (err) {
    next(err); // ← pass error to error handler
  }
});

// Global error handler — always at the END of all routes
app.use((err, req, res, next) => {
  console.error(err.stack);
  res.status(err.statusCode || 500).json({
    success: false,
    message: err.message || 'Internal Server Error'
  });
});
```

### Sync vs Async Errors:
- **Synchronous errors** are caught automatically by Express
- **Async errors** MUST be passed to `next(err)` manually or use `express-async-errors` package

---

## 🌐 CORS

CORS (Cross-Origin Resource Sharing) is a browser security feature blocking requests from a different domain.

```javascript
const cors = require('cors');

// Allow all origins (development only)
app.use(cors());

// Allow specific origin (production)
app.use(cors({
  origin: 'https://myfrontend.com',
  methods: ['GET', 'POST', 'PUT', 'DELETE'],
  allowedHeaders: ['Content-Type', 'Authorization']
}));
```

---

## 🔒 Security Best Practices

```javascript
const helmet = require('helmet');
const rateLimit = require('express-rate-limit');

// 1. Security headers
app.use(helmet());

// 2. Rate limiting
const limiter = rateLimit({
  windowMs: 15 * 60 * 1000, // 15 minutes
  max: 100,                  // 100 requests per window
  message: 'Too many requests, please try again later.'
});
app.use('/api/', limiter);

// 3. Parse JSON with size limit
app.use(express.json({ limit: '10kb' }));

// 4. CORS
app.use(cors({ origin: process.env.FRONTEND_URL }));
```

### Security Checklist:
- ✅ Use `helmet` for HTTP security headers
- ✅ Validate + sanitize all input (`express-validator`, `joi`)
- ✅ Rate limiting to prevent brute force
- ✅ HTTPS in production
- ✅ Store secrets in `.env`, never in code
- ✅ JWT with short expiry + refresh tokens
- ✅ Keep dependencies updated (`npm audit`)

---

## 📁 Static Files

```javascript
// Serve files from /public folder
app.use(express.static('public'));

// /public/image.png → http://localhost:3000/image.png
// /public/styles.css → http://localhost:3000/styles.css

// With custom path prefix
app.use('/static', express.static('public'));
// /public/image.png → http://localhost:3000/static/image.png
```

---

## 🔐 JWT Authentication Pattern

```javascript
const jwt = require('jsonwebtoken');

// Login — generate token
app.post('/api/login', async (req, res) => {
  const user = await User.findOne({ email: req.body.email });
  // verify password...
  const token = jwt.sign({ id: user._id }, process.env.JWT_SECRET, {
    expiresIn: '1h'
  });
  res.json({ token });
});

// Protect routes — middleware
const protect = (req, res, next) => {
  const token = req.headers.authorization?.split(' ')[1]; // Bearer <token>
  if (!token) return res.status(401).json({ message: 'Not authorized' });
  try {
    const decoded = jwt.verify(token, process.env.JWT_SECRET);
    req.user = decoded;
    next();
  } catch {
    res.status(401).json({ message: 'Token invalid' });
  }
};

app.get('/api/profile', protect, (req, res) => {
  res.json({ user: req.user });
});
```

---

## 🧩 Complete Express App Setup

```javascript
const express = require('express');
const cors = require('cors');
const helmet = require('helmet');
const morgan = require('morgan');
require('dotenv').config();

const app = express();

// Security & Utilities
app.use(helmet());
app.use(cors({ origin: process.env.FRONTEND_URL }));
app.use(morgan('dev')); // HTTP request logger

// Body Parsing
app.use(express.json({ limit: '10kb' }));
app.use(express.urlencoded({ extended: true }));

// Routes
app.use('/api/users', require('./routes/userRoutes'));
app.use('/api/posts', require('./routes/postRoutes'));

// 404 handler
app.use((req, res) => {
  res.status(404).json({ message: 'Route not found' });
});

// Global error handler (MUST be last)
app.use((err, req, res, next) => {
  res.status(err.statusCode || 500).json({
    success: false,
    message: err.message
  });
});

module.exports = app;
```

---

## ❓ Interview Questions — Express.js

| # | Question | Difficulty |
|---|---|---|
| 1 | What is middleware in Express? Explain with example | 🔴 Most Asked |
| 2 | What is the difference between app.use() and app.get()? | 🟡 Medium |
| 3 | How do you handle errors in Express? | 🔴 Most Asked |
| 4 | Difference between req.params, req.query, and req.body? | 🔴 Most Asked |
| 5 | What is Express Router and why use it? | 🟡 Medium |
| 6 | What is CORS and how do you handle it? | 🟡 Medium |
| 7 | How do you serve static files in Express? | 🟢 Easy |
| 8 | Why separate app.js from server.js? | 🟡 Medium |
| 9 | How do you secure an Express API? | 🔴 Hard |
| 10 | How do you implement JWT authentication in Express? | 🔴 Hard |
| 11 | What is helmet and why use it? | 🟡 Medium |
| 12 | How do you implement rate limiting? | 🟡 Medium |

---

## 💡 Key Interview Answers

### Middleware:
> *"Middleware functions have access to req, res, and next. They run between the request being received and the response being sent. You MUST call next() to pass control to the next middleware, otherwise the request hangs. Common uses: logging, authentication, body parsing, error handling."*

### Error Handling:
> *"Error-handling middleware takes 4 arguments: err, req, res, next. It must be defined after all routes. Synchronous errors are caught automatically. Async errors must be passed explicitly using next(err)."*

---

*Previous: [02_Event_Loop.md](./02_Event_Loop.md) | Next: [04_REST_API.md](./04_REST_API.md)*
