# 18 - JavaScript Design Patterns

## ❓ Interview Question
> "What are JavaScript design patterns? Explain at least 3 patterns you have actually used in real projects — walk me through what problem each solves and show a code example."

---

## 💡 Full Answer

### What are Design Patterns?
> Reusable solutions to commonly occurring problems in software design. They are templates, not specific code.

---

### 1. Singleton Pattern
**Problem:** Ensure only ONE instance of something exists globally

```js
// Real world use: config manager, database connection, store

class ConfigManager {
  constructor() {
    if (ConfigManager.instance) {
      return ConfigManager.instance; // return existing instance
    }
    this.config = {
      apiUrl: "https://api.example.com",
      theme: "dark",
      language: "en"
    };
    ConfigManager.instance = this; // save instance
  }

  get(key) {
    return this.config[key];
  }

  set(key, value) {
    this.config[key] = value;
  }
}

const config1 = new ConfigManager();
const config2 = new ConfigManager();

console.log(config1 === config2); // true — same instance!
config1.set("theme", "light");
console.log(config2.get("theme")); // "light" — same object
```

---

### 2. Observer Pattern (Pub/Sub)
**Problem:** One object changes → automatically notify all dependents

```js
// Real world use: event systems, Redux store, React Context, WebSockets

class EventEmitter {
  constructor() {
    this.listeners = {};
  }

  // Subscribe
  on(event, callback) {
    if (!this.listeners[event]) {
      this.listeners[event] = [];
    }
    this.listeners[event].push(callback);
    return this; // for chaining
  }

  // Unsubscribe
  off(event, callback) {
    if (this.listeners[event]) {
      this.listeners[event] = this.listeners[event]
        .filter(cb => cb !== callback);
    }
    return this;
  }

  // Publish
  emit(event, data) {
    if (this.listeners[event]) {
      this.listeners[event].forEach(cb => cb(data));
    }
    return this;
  }
}

// Usage
const emitter = new EventEmitter();

const onLogin = (user) => console.log(`Welcome ${user.name}!`);
const logLogin = (user) => console.log(`Log: user ${user.id} logged in`);

emitter.on("login", onLogin);
emitter.on("login", logLogin);
emitter.emit("login", { id: 1, name: "Deepak" });
// Welcome Deepak!
// Log: user 1 logged in

emitter.off("login", logLogin); // unsubscribe logger
emitter.emit("login", { id: 2, name: "John" });
// Welcome John! (only one listener now)
```

---

### 3. Factory Pattern
**Problem:** Create objects without specifying exact class — flexible, centralized creation

```js
// Real world use: creating different user types, UI components

class AdminUser {
  constructor(name) {
    this.name = name;
    this.role = "admin";
    this.permissions = ["read", "write", "delete", "manage_users"];
  }
  canDelete() { return true; }
}

class RegularUser {
  constructor(name) {
    this.name = name;
    this.role = "user";
    this.permissions = ["read", "write"];
  }
  canDelete() { return false; }
}

class GuestUser {
  constructor(name) {
    this.name = name;
    this.role = "guest";
    this.permissions = ["read"];
  }
  canDelete() { return false; }
}

// Factory — single place to create all types
class UserFactory {
  static create(type, name) {
    const types = {
      admin: AdminUser,
      user: RegularUser,
      guest: GuestUser
    };

    const UserClass = types[type];
    if (!UserClass) throw new Error(`Unknown user type: ${type}`);
    return new UserClass(name);
  }
}

const admin = UserFactory.create("admin", "Deepak");
const guest = UserFactory.create("guest", "John");

console.log(admin.permissions); // ["read", "write", "delete", "manage_users"]
console.log(guest.canDelete()); // false
```

---

### 4. Module Pattern
**Problem:** Encapsulation — keep private things private, expose only what's needed

```js
// Real world use: utility libraries, shopping cart, auth module

const AuthModule = (() => {
  // Private
  let currentUser = null;
  let token = null;

  function validateToken(t) {
    return t && t.length > 10; // private validation
  }

  // Public API
  return {
    login(user, receivedToken) {
      if (!validateToken(receivedToken)) {
        throw new Error("Invalid token");
      }
      currentUser = user;
      token = receivedToken;
      console.log(`${user.name} logged in`);
    },

    logout() {
      currentUser = null;
      token = null;
    },

    getUser() {
      return currentUser ? { ...currentUser } : null; // return copy
    },

    isLoggedIn() {
      return currentUser !== null;
    }
  };
})();

AuthModule.login({ name: "Deepak", id: 1 }, "valid-token-here");
console.log(AuthModule.isLoggedIn()); // true
console.log(AuthModule.currentUser);  // undefined — private!
```

---

### 5. Strategy Pattern
**Problem:** Switch algorithms/behavior at runtime without changing the client

```js
// Real world use: payment methods, sorting algorithms, validation

// Strategies
const paymentStrategies = {
  creditCard: (amount, details) => ({
    method: "Credit Card",
    amount,
    last4: details.cardNumber.slice(-4),
    status: "processed"
  }),

  upi: (amount, details) => ({
    method: "UPI",
    amount,
    upiId: details.upiId,
    status: "processed"
  }),

  netBanking: (amount, details) => ({
    method: "Net Banking",
    amount,
    bank: details.bank,
    status: "processed"
  })
};

class PaymentProcessor {
  constructor(strategy) {
    this.strategy = strategy;
  }

  setStrategy(strategy) {
    this.strategy = strategy;
  }

  pay(amount, details) {
    return this.strategy(amount, details);
  }
}

// Usage
const processor = new PaymentProcessor(paymentStrategies.upi);
processor.pay(999, { upiId: "deepak@upi" });
// { method: "UPI", amount: 999, ... }

processor.setStrategy(paymentStrategies.creditCard);
processor.pay(1499, { cardNumber: "1234567890123456" });
// { method: "Credit Card", amount: 1499, last4: "3456", ... }
```

---

### 6. Decorator Pattern
**Problem:** Add behavior to objects without modifying their class

```js
// Real world use: logging, caching, authentication middleware

function withLogging(fn) {
  return function(...args) {
    console.log(`Calling ${fn.name} with`, args);
    const result = fn(...args);
    console.log(`${fn.name} returned`, result);
    return result;
  };
}

function withCache(fn) {
  const cache = new Map();
  return function(...args) {
    const key = JSON.stringify(args);
    if (cache.has(key)) {
      console.log("cache hit!");
      return cache.get(key);
    }
    const result = fn(...args);
    cache.set(key, result);
    return result;
  };
}

// Original function
function expensiveCalculation(n) {
  return n * n;
}

// Decorated — add logging AND caching
const enhanced = withLogging(withCache(expensiveCalculation));
enhanced(5);  // calculates, logs
enhanced(5);  // cache hit!, logs
```

---

## 🔑 Key Concepts to Remember
| Pattern | Problem Solved | Real World Example |
|---|---|---|
| Singleton | One instance only | Config, DB connection, Redux store |
| Observer | Notify on change | Events, Redux, Pub/Sub, WebSocket |
| Factory | Flexible object creation | User types, UI components |
| Module | Private encapsulation | Libraries, utilities |
| Strategy | Swap behavior at runtime | Payment methods, sorting |
| Decorator | Add behavior without modifying | Logging, caching, middleware |
