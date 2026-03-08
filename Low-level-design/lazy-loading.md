# 06 - Lazy Loading Components in React

## ❓ Interview Question
> "How would you implement lazy loading of components in React? Explain how and why you would use it, and walk me through how `React.lazy` and `Suspense` work together — especially in the context of a route-based application."

---

## 🙋 My Answer (What I Said)
- Lazy loading improves performance — components only load when needed, not on initial page load
- Use `React.lazy` to import component
- Wrap with `<Suspense fallback={<Fallback/>}>` to show UI while loading
- Same approach works inside `BrowserRouter` for route-based lazy loading

---

## ✅ What I Got Right
- Correct motivation (performance, not loading everything upfront)
- `React.lazy` with dynamic import — correct concept
- `Suspense` with fallback — correct pattern
- Fallback UI explanation
- Mentioned route-based usage

## ❌ What Was Wrong / Missing
- Import syntax was wrong — `lazy(() => import('../Dashboard'))` not `import Dashboard from`
- Didn't show route-based code example
- Missing: why it helps (separate JS chunks, smaller bundle)
- Missing: Error Boundary for handling chunk load failures
- Missing: `React.lazy` only supports default exports

---

## 💡 Model Answer

### Basic Lazy Loading

```jsx
import { lazy, Suspense } from "react";

// ✅ Correct syntax — dynamic import
const Dashboard = lazy(() => import("./Dashboard"));
const Profile = lazy(() => import("./Profile"));

const App = () => (
  <Suspense fallback={<div>Loading...</div>}>
    <Dashboard />
  </Suspense>
);
```

### Route-Based Lazy Loading (Most Common Use Case)

```jsx
import { lazy, Suspense } from "react";
import { BrowserRouter, Routes, Route } from "react-router-dom";

// Each page becomes a separate JS chunk
const Home = lazy(() => import("./pages/Home"));
const Dashboard = lazy(() => import("./pages/Dashboard"));
const Settings = lazy(() => import("./pages/Settings"));
const Profile = lazy(() => import("./pages/Profile"));

const App = () => (
  <BrowserRouter>
    <Suspense fallback={<PageLoader />}>
      <Routes>
        <Route path="/" element={<Home />} />
        <Route path="/dashboard" element={<Dashboard />} />
        <Route path="/settings" element={<Settings />} />
        <Route path="/profile" element={<Profile />} />
      </Routes>
    </Suspense>
  </BrowserRouter>
);

// Nice loading spinner component
const PageLoader = () => (
  <div style={{ display: "flex", justifyContent: "center", marginTop: "20%" }}>
    <p>Loading page...</p>
  </div>
);
```

### With Error Boundary (Production Best Practice)

```jsx
import { lazy, Suspense, Component } from "react";

// Error boundary to catch chunk load failures (e.g. network issues)
class ErrorBoundary extends Component {
  state = { hasError: false };

  static getDerivedStateFromError() {
    return { hasError: true };
  }

  render() {
    if (this.state.hasError) {
      return <div>Failed to load page. Please refresh.</div>;
    }
    return this.props.children;
  }
}

const Dashboard = lazy(() => import("./Dashboard"));

const App = () => (
  <ErrorBoundary>
    <Suspense fallback={<div>Loading...</div>}>
      <Dashboard />
    </Suspense>
  </ErrorBoundary>
);
```

### Named Export Workaround

```jsx
// React.lazy only supports default exports
// If your component uses named export, wrap it:

// ❌ This won't work directly
const { MyComponent } = lazy(() => import("./MyComponent"));

// ✅ Workaround
const MyComponent = lazy(() =>
  import("./MyComponent").then((module) => ({ default: module.MyComponent }))
);
```

---

## 🔑 Key Concepts to Remember
| Concept | Detail |
|---|---|
| Why lazy load | Reduces initial JS bundle size — faster first load |
| Syntax | `lazy(() => import('./Component'))` — no `from`, no destructuring |
| Suspense | Required wrapper — shows `fallback` until component loads |
| Route-based | Most impactful use — each route = separate chunk |
| Error Boundary | Catches chunk load failures (network errors) |
| Default exports | `React.lazy` only works with default exports |
| Multiple lazy | One `<Suspense>` can wrap many lazy components |
