# ⚛️ Next.js — Code Splitting and SSR

## Code Splitting

### Normal React vs Next.js
```
Normal React:
  One huge bundle.js → user downloads everything upfront → slow first load

Next.js — automatic splitting by page:
  /home     → home.js      (loads only when visiting home)
  /products → products.js  (loads only when visiting products)
  /cart     → cart.js      (loads only when visiting cart)
```

### Manual Code Splitting with dynamic()
```javascript
import dynamic from 'next/dynamic';

// Only loads HeavyChart component when it's needed
const HeavyChart = dynamic(() => import('./HeavyChart'), {
  loading: () => <p>Loading chart...</p>,
  ssr: false   // skip server rendering — client only
});
```

---

## SSR — Server Side Rendering

### Without SSR vs With SSR
```
Without SSR (normal React):
  User requests page
  → Server sends empty HTML + JS bundle
  → Browser downloads JS
  → JS runs → page renders
  → User sees content   ← slow, bad for SEO

With Next.js SSR:
  User requests page
  → Server runs React → generates full HTML
  → Server sends full HTML
  → User sees content immediately  ← fast, great for SEO
  → JS loads in background (hydration)
```

---

## Three Data Fetching Methods

### SSR — getServerSideProps (runs on every request)
```javascript
export async function getServerSideProps() {
  const res = await fetch('https://api.example.com/products');
  const products = await res.json();

  return {
    props: { products }  // passed to component as props
  };
}
```
**Use when:** Data changes frequently, must be fresh on every visit.

### SSG — getStaticProps (runs at build time)
```javascript
export async function getStaticProps() {
  const res = await fetch('https://api.example.com/products');
  const products = await res.json();

  return {
    props: { products },
    revalidate: 60   // ISR — rebuild page every 60 seconds
  };
}
```
**Use when:** Data doesn't change often — blog posts, product pages.

### ISR — Incremental Static Regeneration
Same as SSG but with `revalidate` — rebuilds the page in background after N seconds without full redeploy.

---

## When To Use What

| Method | When To Use |
|--------|------------|
| SSG | Static content — blogs, docs, marketing pages |
| SSR | User-specific or frequently changing data |
| ISR | Semi-static — product listings, news articles |
| Client-side fetch | User-specific data after login, dashboards |

---

## 🔑 Key Points To Remember
- Next.js splits bundle automatically per page — no config needed
- `dynamic()` = manual code splitting for heavy components
- SSR = HTML generated on server per request — fast first paint, good SEO
- SSG = HTML generated at build time — fastest, but data can be stale
- ISR = SSG + background regeneration — best of both worlds
- Hydration = React attaches JS event handlers to server-rendered HTML
