# ⚛️ React Interview Revision — Programmer.io Round

---

## Q1. Authentication & Authorization (JWT + SSO + RBAC)

### Your Answer
- JWT access token (15 min) + refresh token (7 days) stored in cookies
- On login, check if refresh token valid → go to dashboard, else re-login
- SSO using OAuth2 + Azure Active Directory → same JWT rules after auth
- RBAC via an `<AuthComponent>` wrapper that matches role IDs from backend API

### ✅ Model Answer

**JWT Flow:**
1. User logs in → backend returns `accessToken` (15 min) + `refreshToken` (7 days) stored in **httpOnly cookies**
2. On app load → call auth API to check if refresh token is valid → redirect to dashboard or login
3. When access token expires → silently call refresh endpoint to get new access token

**SSO Flow:**
1. User clicks "Login with SSO" → redirected to Azure Active Directory
2. Azure authenticates → returns OAuth2 authorization code
3. Backend exchanges code for tokens → same JWT rules apply from here

**RBAC (Role Based Access Control):**
```jsx
// AuthComponent wraps protected routes
<AuthComponent requiredId="dashboard_view">
  <Dashboard />
</AuthComponent>

// Inside AuthComponent — checks role from backend
if (!userRoles.includes(requiredId)) {
  return <Navigate to="/home" />;
}
```

> 💡 Always mention **httpOnly cookies** — it shows security awareness (prevents XSS attacks from stealing tokens).

---

## Q2. useEffect — Dependency Array Variations + Cleanup

### Your Answer
- No array = runs every render
- Empty array = runs once on mount
- With deps = runs when deps change
- return = for cleanup on unmount, e.g. setTimeout, setInterval

### ✅ Model Answer

```jsx
// 1. No dependency array — runs after EVERY render
useEffect(() => {
  console.log("runs every render");
});

// 2. Empty array — runs ONCE on mount only
useEffect(() => {
  console.log("runs on mount");
}, []);

// 3. With dependencies — runs when dep changes
useEffect(() => {
  console.log("count changed");
}, [count]);

// 4. Cleanup function — return is called on UNMOUNT
useEffect(() => {
  const timer = setInterval(() => {}, 1000);
  const handler = () => {};
  const controller = new AbortController();

  window.addEventListener("resize", handler);

  return () => {
    clearInterval(timer);                          // ✅ prevent memory leak
    window.removeEventListener("resize", handler); // ✅ remove listener
    controller.abort();                            // ✅ cancel API call
  };
}, []);
```

**Why use cleanup (return)?**
To prevent **memory leaks** when the component unmounts. Common cases:
- `setInterval` / `setTimeout`
- Event listeners (`addEventListener`)
- API calls (using `AbortController`)

> ⚠️ You missed: **event listeners** and **AbortController** for API cancellation — mention all three next time.

---

## Q3. What is useMemo?

### Your Answer
- Stores return value of a function
- Used for performance optimization
- Re-runs only when dependency changes

### ✅ Model Answer
`useMemo` **memoizes the result** of an expensive calculation. It only recomputes when its dependencies change, preventing unnecessary recalculation on every render.

```jsx
// Without useMemo — recalculates on EVERY render (expensive!)
const sorted = heavySort(products);

// With useMemo — only recalculates when `products` changes
const sorted = useMemo(() => heavySort(products), [products]);
```

> 💡 Key distinction to remember:
> - `useMemo` → caches a **value**
> - `useCallback` → caches a **function**

---

## Q4. Preventing Child Re-render When Parent Re-renders

### Your Answer
- Use `React.memo` — shallow comparison of props
- Problem: functions as props change reference every render
- Solution: wrap function in `useCallback` to stabilize reference

### ✅ Model Answer
This was an excellent answer. Full picture:

```jsx
// React.memo — prevents re-render if props haven't changed (shallow compare)
const Child = React.memo(({ name }) => {
  return <div>{name}</div>;
});

// ❌ Problem — function reference changes on every parent render
// Child WILL re-render even with React.memo
<Child onClick={() => doSomething()} />

// ✅ Solution — useCallback stabilizes the function reference
const handleClick = useCallback(() => {
  doSomething();
}, []); // only recreated if deps change

<Child onClick={handleClick} /> // Now React.memo works correctly ✅
```

> 💡 The combo **React.memo + useCallback** is a very common deep-dive question. You nailed this one.

---

## Q5. Next.js vs React — Architecture + Folder Structure

### Your Answer
- Requirement gathering first ✅
- SEO needed → Next.js ✅
- TypeScript, shadcn UI, REST API, native fetch (no axios) ✅
- Stopped before folder structure

### ✅ Folder Structure — What You Should Have Said

For a learning platform like Udemy in Next.js (App Router):

```
/src
  /app
    /dashboard            → Dashboard route
    /courses              → Course listing page
    /courses/[id]         → Individual course (dynamic route)
    /auth                 → Login / Register
    layout.tsx            → Root layout (Navbar, Footer)
    page.tsx              → Home page

  /components
    /ui                   → Generic reusable UI (Button, Modal, Card)
    /shared               → Layout components (Navbar, Sidebar, Footer)
    /features             → Feature-specific (CourseCard, VideoPlayer, Rating)

  /hooks                  → Custom hooks (useFetch, useDebounce, useAuth)
  /lib                    → Utility functions, helpers, constants
  /services               → API call functions (courseService.ts, authService.ts)
  /types                  → TypeScript interfaces and types
  /context                → React context providers (AuthContext, ThemeContext)
  /middleware.ts           → Auth middleware for protected routes (Next.js)
```

> 💡 Key points to mention:
> - **`/features`** keeps feature-specific components separate from generic UI
> - **`/services`** centralizes all API calls — easy to maintain
> - **`middleware.ts`** in Next.js handles route protection at server level

---

## Q6. React 19 Features

### Your Answer
- `useId` hook (❌ this is React 18, not 19)
- React Compiler ✅

### ✅ Model Answer — React 19 Key Features

| Feature | What it does |
|---|---|
| **React Compiler** | Auto-memoizes components — no need to manually write useMemo/useCallback |
| **Actions** | Simplifies async form handling (replaces manual loading/error state) |
| **`useActionState`** | Manages state from form actions |
| **`useFormStatus`** | Gives pending/loading state of parent form submission |
| **`useOptimistic`** | Shows optimistic UI update before server confirms |
| **`use()` hook** | Read promises or context inside render directly |

```jsx
// useOptimistic example (React 19)
const [optimisticLikes, addOptimisticLike] = useOptimistic(likes);

// useActionState example (React 19)
const [state, formAction] = useActionState(submitForm, initialState);
```

> ⚠️ `useId` is **React 18**. For React 19, lead with **React Compiler** and **useOptimistic** — those impress interviewers most.

---

## Q7. Why `className` and not `class` in React?

### Your Answer
- Said it's a best practice ❌ (wrong reason)

### ✅ Model Answer
Because **JSX is JavaScript**, not HTML. The word `class` is a **reserved keyword** in JavaScript (used for defining JS classes like `class MyComponent {}`).

To avoid this conflict, React uses `className` instead.

```jsx
// ❌ Wrong — 'class' is reserved in JS
<div class="container">

// ✅ Correct — className is the JSX equivalent
<div className="container">
```

> 💡 One line answer: *"`class` is a reserved JS keyword, so JSX uses `className` to avoid the conflict."*

---

## Q8. Can We Write `<dashboard/>` with Lowercase?

### Your Answer
- Said no, because Pascal case is best practice ❌ (incomplete/wrong reason)

### ✅ Model Answer
It's not just a best practice — it's how React **distinguishes** between HTML elements and custom components.

- **Lowercase** (`<dashboard/>`) → React treats it as a **native HTML element** (like `<div>`, `<span>`). Since `dashboard` isn't a real HTML tag, React ignores or throws a warning.
- **Uppercase/PascalCase** (`<Dashboard/>`) → React treats it as a **custom component** and renders it correctly.

```jsx
// ❌ React thinks this is an HTML element — won't render your component
<dashboard />

// ✅ React knows this is a custom component
<Dashboard />
```

> 💡 One line answer: *"Lowercase = HTML element, Uppercase = React component. It's how React tells them apart, not just a naming convention."*

---

## 🔑 Quick Cheat Sheet — Programmer.io

| Topic | Key Point to Remember |
|---|---|
| JWT Security | Always say **httpOnly cookies** to show security awareness |
| useEffect cleanup | setTimeout + **event listeners** + **AbortController** |
| useMemo vs useCallback | useMemo = caches **value**, useCallback = caches **function** |
| React.memo + useCallback | Together they fully prevent unnecessary child re-renders |
| Folder structure | `/features`, `/services`, `/hooks`, `/types` are key folders |
| React 19 | **Compiler**, useOptimistic, useActionState — NOT useId (that's v18) |
| `className` | `class` is a **reserved JS keyword** — not just best practice |
| `<dashboard/>` | Lowercase = HTML element, Uppercase = React component |
