# ⚛️ Next.js — Environment Variables

## File Types and When They Load

```
.env              → all environments (lowest priority — fallback)
.env.local        → local dev only, never committed to Git (highest priority)
.env.development  → when running npm run dev
.env.production   → when running npm run build / production
```

---

## Priority Order (highest wins)
```
.env.local          ← always wins — your personal overrides
.env.production     ← when NODE_ENV=production
.env.development    ← when NODE_ENV=development
.env                ← fallback for all environments
```

---

## Critical Rule — NEXT_PUBLIC Prefix

```bash
# ❌ Server only — NOT available in browser
API_SECRET_KEY=abc123
DATABASE_URL=postgres://...

# ✅ Available in browser — must prefix with NEXT_PUBLIC
NEXT_PUBLIC_API_URL=https://api.example.com
NEXT_PUBLIC_APP_NAME=MyApp
```

```javascript
// Usage in code
const apiUrl = process.env.NEXT_PUBLIC_API_URL;  // ✅ works in browser + server
const secret = process.env.API_SECRET_KEY;        // ✅ server only — undefined in browser
```

---

## Practical Example

```bash
# .env.local (local dev — not committed to Git)
NEXT_PUBLIC_API_URL=http://localhost:3001
DATABASE_URL=postgres://localhost:5432/mydb
JWT_SECRET=my-local-secret

# .env.production (production values)
NEXT_PUBLIC_API_URL=https://api.myapp.com
DATABASE_URL=postgres://prod-server:5432/mydb
JWT_SECRET=super-secure-prod-secret
```

---

## Common Mistakes

```javascript
// ❌ Mistake 1 — trying to use server var in browser component
export default function Header() {
  return <div>{process.env.DATABASE_URL}</div>  // undefined in browser!
}

// ❌ Mistake 2 — exposing secrets with NEXT_PUBLIC
NEXT_PUBLIC_DATABASE_URL=...    // anyone can see this in browser devtools!
NEXT_PUBLIC_JWT_SECRET=...      // never do this!

// ✅ Rule — NEXT_PUBLIC only for non-sensitive public config
NEXT_PUBLIC_API_URL=...         // fine — public API endpoint
NEXT_PUBLIC_GOOGLE_MAPS_KEY=... // fine — public key
```

---

## 🔑 Key Points To Remember
- `NEXT_PUBLIC_` prefix = exposed to browser (public)
- No prefix = server only (private — secrets, DB URLs, API keys)
- `.env.local` = never commit to Git — add to `.gitignore`
- `.env.local` always overrides other env files
- Variables are replaced at build time — not dynamic at runtime
