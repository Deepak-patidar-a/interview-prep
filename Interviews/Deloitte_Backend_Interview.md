# 🎯 Deloitte Backend Interview — Questions & Model Answers

> Round 1 — Node.js, Security, Architecture, Coding
> Use this to revise. Every answer is written in clean "interview language" + explained in plain English below it.

---

## Table of Contents

1. [How RazorPay payment integration works](#1-how-razorpay-payment-integration-works)
2. [If Node.js is single-threaded, how does it handle too many requests?](#2-if-nodejs-is-single-threaded-how-does-it-handle-too-many-requests)
3. [Why not just use an access token for signup?](#3-why-not-just-use-an-access-token-for-signup)
4. [After signup, what happens behind the scenes?](#4-after-signup-what-happens-behind-the-scenes)
5. [How do we store JWT tokens, and how do we prevent security threats?](#5-how-do-we-store-jwt-tokens-and-how-do-we-prevent-security-threats)
6. [How do we store passwords in the backend database?](#6-how-do-we-store-passwords-in-the-backend-database)
7. [How do we compare a login password with a hashed DB password?](#7-how-do-we-compare-a-login-password-with-a-hashed-db-password)
8. [When do we choose Java vs Node.js vs Python for backend?](#8-when-do-we-choose-java-vs-nodejs-vs-python-for-backend)
9. [What other security concerns exist in the backend?](#9-what-other-security-concerns-exist-in-the-backend)
10. [What is an ORM like TypeORM, and why use it?](#10-what-is-an-orm-like-typeorm-and-why-use-it)
11. [Where do we store access token and refresh token, and why cookies?](#11-where-do-we-store-access-token-and-refresh-token-and-why-cookies)
12. [Coding — Remove duplicates + frequency count](#12-coding--remove-duplicates--frequency-count)

---

## 1. How RazorPay Payment Integration Works

### 🎤 Model Interview Answer

> "RazorPay uses a **server-side order creation + client-side checkout + server-side verification** flow. The frontend never directly talks to RazorPay for creating a payment — that goes through our backend to keep the API secret safe.
>
> The flow is: our server creates an order using RazorPay's API and returns the `order_id` to the frontend. The frontend loads the RazorPay checkout SDK and completes the payment. RazorPay then returns a `payment_id`, `order_id`, and a `signature`. We send these three to our backend, and we verify the signature using HMAC-SHA256. Only if the signature matches do we mark the order as paid in our database."

### 📖 Explained Simply

```
Step 1 — User clicks "Pay"
    → Frontend calls YOUR backend: POST /create-order

Step 2 — Your backend calls RazorPay API
    → Creates an order with amount + currency
    → Gets back an order_id
    → Sends order_id to frontend

Step 3 — Frontend opens RazorPay checkout popup
    → User enters card/UPI details
    → RazorPay processes the payment
    → On success, RazorPay sends back:
        { razorpay_payment_id, razorpay_order_id, razorpay_signature }

Step 4 — Frontend sends these 3 values to YOUR backend

Step 5 — Your backend VERIFIES the signature
    const body = order_id + "|" + payment_id;
    const expectedSig = crypto
        .createHmac("sha256", RAZORPAY_KEY_SECRET)
        .update(body)
        .digest("hex");

    if (expectedSig === razorpay_signature) → Payment is genuine ✅
    else → Reject, possible fraud ❌

Step 6 — Mark order as PAID in your DB
```

### 💡 Why this design?
- Your **API secret never goes to the frontend** — if it did, anyone could fake payments
- The **signature verification** ensures RazorPay actually processed the payment, not a fake request from an attacker

---

## 2. If Node.js Is Single-Threaded, How Does It Handle Too Many Requests?

### 🎤 Model Interview Answer

> "Node.js is single-threaded for JavaScript execution, but it handles concurrency through its **event loop** and **libuv's thread pool**. When a request comes in that involves I/O — like a DB query or file read — Node offloads that work to the OS or libuv's background threads. The JS thread is free to accept the next request immediately. When the I/O finishes, the callback is queued and the event loop picks it up.
>
> So Node handles thousands of concurrent requests not by creating threads for each one, but by never blocking on I/O. This is called **non-blocking I/O**. For CPU-heavy work, we use **Worker Threads** or offload to a separate service."

### 📖 Explained Simply

```
Traditional server (e.g., Java/PHP):
  Request 1 → Thread 1 (busy waiting for DB) 🔴
  Request 2 → Thread 2 (busy waiting for DB) 🔴
  Request 3 → Thread 3 (busy waiting for DB) 🔴
  → 1000 requests = 1000 threads = crashes

Node.js server:
  Request 1 → "Hey OS, get me data from DB" → JS thread free immediately ✅
  Request 2 → "Hey OS, read this file"       → JS thread free immediately ✅
  Request 3 → "Hey OS, call this API"        → JS thread free immediately ✅
  → When DB responds → callback runs → response sent
  → 1000 requests = 1 JS thread + libuv pool = handles it fine ✅
```

### Key Terms to Use
- **Non-blocking I/O** — JS doesn't wait for I/O to complete
- **Event Loop** — keeps checking if any async work is done
- **libuv** — C library with 4 background threads (default) for file/crypto/DNS
- **Worker Threads** — for CPU-intensive JS tasks (image processing, ML)

---

## 3. Why Not Just Use an Access Token for Signup?

### 🎤 Model Interview Answer

> "An access token is a **short-lived** credential — typically 15 minutes to 1 hour — used to authenticate API requests. We can't use it for signup because during signup, the user doesn't exist yet. There's no identity to generate a token for.
>
> What I think you're asking is: why not skip the signup flow and just generate an access token directly without a refresh token? The reason is **security**. If an access token is stolen, a short expiry limits the damage window. If we made it long-lived, a stolen token gives an attacker months of access. The **refresh token** lets us silently issue new short-lived access tokens without re-login, while still being able to revoke access at any time by deleting the refresh token from the DB."

### 📖 Explained Simply

```
Access Token  → Short-lived (15 min–1 hr), stateless, stored in memory
Refresh Token → Long-lived (7–30 days), stored in DB + httpOnly cookie

Flow:
  Login → Server gives BOTH tokens
  
  API Call → Send access token in Authorization header
  
  Access token expires → Frontend uses refresh token to get a new access token
  
  Logout / suspicious activity → Delete refresh token from DB
                               → All future refresh attempts fail
                               → User must re-login ✅

Why not just one long-lived access token?
  → If stolen, attacker has access for months
  → Can't revoke it (it's stateless/JWT)
  → No way to force re-login
```

---

## 4. After Signup, What Happens Behind the Scenes?

### 🎤 Model Interview Answer

> "When a user submits a signup form, the backend:
> 1. **Validates** the input — checks required fields, email format, password strength
> 2. **Checks for duplicates** — queries the DB to ensure the email doesn't already exist
> 3. **Hashes the password** using bcrypt with a salt round of 10–12
> 4. **Saves the user** to the database with the hashed password
> 5. **Sends a verification email** (if email verification is required) with a signed token
> 6. **Generates JWT tokens** — access token and refresh token
> 7. **Stores the refresh token** in the DB against the user
> 8. **Sets the refresh token in an httpOnly cookie** and returns the access token in the response body"

### 📖 Code Flow

```javascript
// POST /auth/signup
async function signup(req, res) {
    const { name, email, password } = req.body;

    // Step 1: Validate
    if (!email || !password) return res.status(400).json({ error: "Missing fields" });

    // Step 2: Check duplicate
    const existing = await User.findOne({ where: { email } });
    if (existing) return res.status(409).json({ error: "Email already registered" });

    // Step 3: Hash password
    const hashedPassword = await bcrypt.hash(password, 12);

    // Step 4: Save user
    const user = await User.create({ name, email, password: hashedPassword });

    // Step 5: Send verification email (async, don't block response)
    sendVerificationEmail(user.email, user.id);

    // Step 6 & 7: Generate tokens
    const accessToken  = jwt.sign({ userId: user.id }, ACCESS_SECRET,  { expiresIn: "15m" });
    const refreshToken = jwt.sign({ userId: user.id }, REFRESH_SECRET, { expiresIn: "7d" });
    await RefreshToken.create({ token: refreshToken, userId: user.id });

    // Step 8: Set cookie + respond
    res.cookie("refreshToken", refreshToken, { httpOnly: true, secure: true, sameSite: "strict" });
    res.status(201).json({ accessToken });
}
```

---

## 5. How Do We Store JWT Tokens, and How Do We Prevent Security Threats?

### 🎤 Model Interview Answer

> "There are two common storage options — **localStorage** and **httpOnly cookies**. localStorage is accessible via JavaScript, which makes it vulnerable to **XSS (Cross-Site Scripting)** attacks — if an attacker injects malicious JS into the page, they can steal the token.
>
> The safer approach is to store the **refresh token in an httpOnly cookie** — the browser sends it automatically but JavaScript cannot read it, neutralising XSS. The **access token is stored in memory** (JS variable or React state), not in any persistent storage, so it's lost on page refresh — that's intentional.
>
> For **CSRF protection** when using cookies, we set `SameSite=Strict` or use CSRF tokens."

### 📖 Attack vs Defence Table

| Threat | Description | Prevention |
|--------|-------------|------------|
| **XSS** | Malicious script reads token from localStorage | Store refresh token in `httpOnly` cookie — JS cannot read it |
| **CSRF** | Attacker tricks browser into sending cookie | `SameSite=Strict` cookie flag + CSRF tokens |
| **Token theft (network)** | Token intercepted in transit | Always use HTTPS (`Secure` cookie flag) |
| **Stolen refresh token** | Attacker gets long-lived token | Refresh token rotation — issue new one + invalidate old on each use |
| **Token replay** | Reuse a valid token after logout | Maintain a token blacklist or delete from DB on logout |

### Storage Summary

```
Access Token  → In-memory (React state / JS variable)
              → Short-lived (15 min), safe to lose on refresh

Refresh Token → httpOnly cookie (browser auto-sends, JS cannot read)
              → Also stored in DB (so we can revoke it)
```

---

## 6. How Do We Store Passwords in the Backend Database?

### 🎤 Model Interview Answer

> "We **never store plain text passwords**. We use **bcrypt** to hash them. bcrypt is a one-way hashing algorithm that automatically generates a **salt** (random value) and embeds it in the hash. This means even if two users have the same password, their hashes will be different.
>
> The salt rounds parameter (typically 10–12) controls how computationally expensive the hashing is — higher rounds = slower hashing = harder to brute force. We store only the resulting hash string in the DB. bcrypt's hash output looks like: `$2b$12$...` where `$12$` is the salt rounds and everything after is the salted hash."

### 📖 Why Not MD5 / SHA256?

```
MD5/SHA256:  Fast ❌ — can brute-force millions of guesses per second
bcrypt:      Slow by design ✅ — ~100ms per hash makes brute force impractical

Plain text:  "password123"           ← NEVER do this
MD5:         "482c811da5d5b4bc..."   ← Rainbow table attack possible
bcrypt:      "$2b$12$eImiTXuWVxfM..." ← Safe ✅ (salt + slow)

// Code:
const hash = await bcrypt.hash(plainPassword, 12);
// Store 'hash' in DB. Never store 'plainPassword'.
```

---

## 7. How Do We Compare a Login Password With a Hashed DB Password?

### 🎤 Model Interview Answer

> "We use `bcrypt.compare()`. We never decrypt the stored hash — that's not possible because bcrypt is one-way. Instead, bcrypt takes the plain text password the user just entered, extracts the salt from the stored hash, re-hashes the input using that same salt, and compares the two hashes. If they match, the password is correct. If not, reject it."

### 📖 Code Example

```javascript
// POST /auth/login
async function login(req, res) {
    const { email, password } = req.body;

    // Find user by email
    const user = await User.findOne({ where: { email } });
    if (!user) return res.status(401).json({ error: "Invalid credentials" });

    // bcrypt.compare does NOT decrypt — it re-hashes and compares
    const isMatch = await bcrypt.compare(password, user.password);
    //                              ↑ plain text    ↑ hash from DB

    if (!isMatch) return res.status(401).json({ error: "Invalid credentials" });

    // Password matched — generate tokens
    const accessToken = jwt.sign({ userId: user.id }, ACCESS_SECRET, { expiresIn: "15m" });
    res.json({ accessToken });
}
```

### 💡 Why the same error message for "user not found" AND "wrong password"?
Returning different errors ("User not found" vs "Wrong password") allows **user enumeration** — attackers can discover valid emails. Always use a generic `"Invalid credentials"` for both.

---

## 8. When Do We Choose Java vs Node.js vs Python for Backend?

### 🎤 Model Interview Answer

> "The choice depends on the use case, team, and performance requirements."

### 📖 Decision Table

| Factor | Node.js | Java (Spring Boot) | Python (Django/FastAPI) |
|---|---|---|---|
| **Best for** | Real-time apps, APIs, microservices | Enterprise, high-load, complex business logic | Data science, ML pipelines, rapid prototyping |
| **Concurrency model** | Non-blocking I/O (event loop) | Multi-threaded (can block but scales with threads) | Async (FastAPI) or sync (Django) |
| **Performance** | High for I/O bound tasks | Very high, mature JVM optimisations | Moderate (GIL limits CPU tasks) |
| **Type safety** | TypeScript adds it | Strong (compiled, strict) | Optional (type hints) |
| **Startup time** | Fast | Slow (JVM warmup) | Fast |
| **Ecosystem** | npm — vast | Maven/Gradle — mature, enterprise-grade | pip — strong in data/ML |
| **Use in practice** | Chat apps, REST APIs, BFF layers | Banking, insurance, ERP systems | ML APIs, data pipelines, scrapers |

### When Deloitte Projects Might Choose Each

```
Node.js   → Customer portal APIs, CRM integrations, real-time dashboards
Java      → Core banking systems, legacy enterprise migrations, high-compliance apps
Python    → AI/ML features (Agentforce/GenAI mentioned in the JD!), analytics backends
```

---

## 9. What Other Security Concerns Exist in the Backend?

### 🎤 Model Interview Answer

> "Beyond JWT and password hashing, the major backend security concerns are:
> **SQL Injection, XSS, CSRF, Rate Limiting, CORS misconfiguration, insecure direct object references, and dependency vulnerabilities.**"

### 📖 The Full List

| Threat | What It Is | How to Prevent |
|---|---|---|
| **SQL Injection** | Attacker injects SQL via inputs | Use ORM / parameterised queries — never string-concatenate SQL |
| **XSS** | Inject malicious JS into pages | Sanitise/escape all user input, CSP headers |
| **CSRF** | Trick logged-in user's browser into a request | `SameSite=Strict` cookies, CSRF tokens |
| **Rate Limiting** | Brute force login / DDoS | express-rate-limit, Cloudflare, token bucket |
| **IDOR** | Access another user's data by changing an ID | Always check `req.user.id === resource.userId` |
| **CORS misconfiguration** | Allow any origin = data theft | Whitelist specific origins only |
| **Sensitive data exposure** | Returning passwords/keys in API responses | Explicitly select fields; never `SELECT *` and return raw |
| **Dependency vulnerabilities** | Outdated packages with CVEs | `npm audit`, Dependabot, lock file |
| **Environment secrets** | Hardcoded API keys in code | `.env` files, secrets manager (AWS Secrets Manager) |
| **HTTPS only** | Data in transit unencrypted | Force HTTPS, HSTS headers |

---

## 10. What Is an ORM Like TypeORM, and Why Use It?

### 🎤 Model Interview Answer

> "ORM stands for **Object-Relational Mapper**. It maps your database tables to JavaScript/TypeScript classes (entities), so you can query and manipulate data using code instead of raw SQL. TypeORM, for example, lets you define a `User` class with decorators, and it handles creating the table, migrations, and queries automatically.
>
> The benefits are: **type safety**, **SQL injection prevention** (queries are parameterised), **database abstraction** (switch from Postgres to MySQL with minimal change), and **migration management**."

### 📖 Without ORM vs With ORM

```javascript
// ❌ Without ORM — raw SQL, error-prone, injection risk if not careful
const result = await db.query(
    `SELECT * FROM users WHERE email = '${email}'`  // SQL injection risk!
);

// ✅ With TypeORM — clean, safe, typed
const user = await userRepository.findOne({ where: { email } });

// ✅ TypeORM Entity definition
@Entity()
class User {
    @PrimaryGeneratedColumn()     id: number;
    @Column({ unique: true })     email: string;
    @Column()                     password: string;
    @CreateDateColumn()           createdAt: Date;
}
```

### Alternatives

| ORM/Query Builder | Best For |
|---|---|
| **TypeORM** | TypeScript-first, decorators, great for Node/NestJS |
| **Prisma** | Modern, type-safe, auto-generates client (mentioned in JD!) |
| **Sequelize** | Older, JS-based, widely used |
| **Knex.js** | Query builder (not full ORM) — more control over SQL |
| **Drizzle** | Lightweight, TypeScript, gaining popularity |

> **Note:** The job description specifically mentions **Prisma ORM with PostgreSQL** — mention Prisma by name in interviews!

---

## 11. Where Do We Store Access Token and Refresh Token, and Why Cookies?

### 🎤 Model Interview Answer

> "The **access token** is stored in-memory — a JS variable or React state. It's intentionally lost on page refresh; the app silently fetches a new one using the refresh token. The **refresh token** is stored in an **httpOnly, Secure, SameSite=Strict cookie**.
>
> We use httpOnly cookies for the refresh token because: JavaScript cannot read httpOnly cookies, so XSS attacks can't steal it. The browser sends it automatically with requests to our domain. Combined with `SameSite=Strict`, CSRF is also mitigated."

### 📖 Storage Comparison

```
Option 1: localStorage
  ✅ Simple, persists across tabs
  ❌ Readable by ANY JavaScript on the page
  ❌ XSS attack steals the token instantly
  → Never use for sensitive tokens

Option 2: sessionStorage
  ✅ Cleared on tab close
  ❌ Still readable by JavaScript
  → Same XSS problem

Option 3: httpOnly Cookie  ← Best for refresh token
  ✅ JavaScript CANNOT read it (document.cookie won't show it)
  ✅ Browser sends it automatically
  ✅ Secure flag = only sent over HTTPS
  ✅ SameSite=Strict = not sent on cross-site requests (CSRF protection)
  ❌ Slightly more complex setup

Option 4: In-memory (JS variable)  ← Best for access token
  ✅ Never touches disk or cookie storage
  ✅ XSS can't persist it
  ❌ Lost on page refresh (intentional — re-fetch from refresh token)
```

### Cookie Flags Explained

```javascript
res.cookie("refreshToken", token, {
    httpOnly: true,   // JS cannot access via document.cookie
    secure:   true,   // Only sent over HTTPS
    sameSite: "strict", // Not sent on cross-site requests (CSRF protection)
    maxAge:   7 * 24 * 60 * 60 * 1000  // 7 days
});
```

---

## 12. Coding — Remove Duplicates + Frequency Count

### The Code From Interview

```javascript
const arr = [0, 0, 1, 1, 2, 2, 3, 4, 3, 5];

// Method 1: Using Set (cleanest)
// [...new Set(arr)]

// Method 2: Using filter + indexOf (what was asked)
const unique = arr.filter((item, index) => arr.indexOf(item) === index);
console.log(unique); // [0, 1, 2, 3, 4, 5]

// Frequency count
const freqCheck = (arr) => {
    let freq = {};
    for (let i = 0; i < arr.length; i++) {
        freq[arr[i]] = (freq[arr[i]] || 0) + 1;
    }
    return freq;
};
console.log(freqCheck(arr));
// { '0': 2, '1': 2, '2': 2, '3': 2, '4': 1, '5': 1 }
```

### 🎤 How to Explain the filter Logic

> "For each element, `arr.indexOf(item)` returns the **first** index where that item appears in the array. We compare it to the current `index`. If they're equal, this IS the first occurrence — keep it. If they're not equal, this is a duplicate — filter it out."

```
arr = [0, 0, 1, 1, 2, ...]
         ↑  ↑
index:   0  1

For item=0 at index=0: arr.indexOf(0) = 0, matches index 0 → KEEP ✅
For item=0 at index=1: arr.indexOf(0) = 0, doesn't match index 1 → REMOVE ❌
For item=1 at index=2: arr.indexOf(1) = 2, matches index 2 → KEEP ✅
```

### 🎤 How to Explain the freqCheck Logic

> "We use an object as a hashmap. For each element, we either initialise it to 1 or increment by 1. The `(freq[arr[i]] || 0)` handles the first occurrence — if the key doesn't exist yet, it's `undefined`, so `undefined || 0` gives us 0, and we add 1 to get 1."

### Time Complexity

| Operation | Complexity | Why |
|---|---|---|
| `filter + indexOf` unique | O(n²) | For each element, indexOf scans the whole array |
| `Set` unique | O(n) | Hash-based lookup is O(1) |
| `freqCheck` | O(n) | Single pass through array |

> **Mention in interview:** "The `filter + indexOf` approach is O(n²). In a production scenario with large arrays, I'd prefer `[...new Set(arr)]` which is O(n) using a hash set."

---

## Quick Revision Cheatsheet

| Topic | Key Phrase to Remember |
|---|---|
| RazorPay | "Server creates order → Client pays → Server verifies HMAC signature" |
| Node concurrency | "Non-blocking I/O + event loop + libuv thread pool" |
| Access vs refresh token | "Access = short-lived in memory, Refresh = long-lived in httpOnly cookie" |
| Signup flow | "Validate → Check duplicate → Hash password → Save → Generate tokens" |
| Password storage | "bcrypt with salt rounds 10–12, one-way hash, never decrypt" |
| Password compare | "bcrypt.compare() re-hashes with extracted salt, compares — never decrypts" |
| Language choice | "Node = I/O heavy APIs, Java = enterprise/compliance, Python = ML/data" |
| Security | "Injection, XSS, CSRF, Rate limit, IDOR, CORS, HTTPS" |
| ORM | "Type-safe DB queries, prevents SQL injection, manages migrations" |
| Token storage | "Refresh in httpOnly cookie (XSS-safe), Access in memory" |
| filter+indexOf | "indexOf returns first occurrence — if it doesn't match current index, it's a duplicate" |

---

*Created after Deloitte Round 1 interview — April 2026*
