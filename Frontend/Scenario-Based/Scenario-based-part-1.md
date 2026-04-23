# 🚀 React Interview Questions & Answers — Part 1
### For 5+ Years Experience | Performance, Architecture & Deloitte Special

---

## 🏢 Deloitte Interview Question

### Q1. We are rendering 1000 rows on a landing page with editable rows. I'm editing one record, but the whole list is re-rendering. What might be the reason?

**Root Causes & Fixes:**

---

#### ❌ Reason 1: Missing or Wrong `key` Prop
React uses `key` to identify which items changed. Without a stable unique key, it re-renders everything.

```jsx
// ❌ Wrong - index as key
{rows.map((row, index) => <Row key={index} data={row} />)}

// ✅ Correct - unique stable ID
{rows.map((row) => <Row key={row.id} data={row} />)}
```

---

#### ❌ Reason 2: Edit State Lifted Too High (Most Common)
If `editingRow` state lives in the parent/page, any change triggers re-render of ALL children.

```jsx
// ❌ Parent holds edit state → all 1000 rows re-render on every keystroke
const [editingRow, setEditingRow] = useState(null);
return rows.map(row => <Row data={row} editing={editingRow === row.id} />);

// ✅ Move edit state INTO each Row component
const Row = ({ data }) => {
  const [isEditing, setIsEditing] = useState(false); // State is local now
  return isEditing ? <EditForm data={data} /> : <DisplayRow data={data} />;
};
```

---

#### ❌ Reason 3: Not Using `React.memo()`
Even with correct keys, child components without memoization re-render every time the parent re-renders.

```jsx
// ❌ Plain component - re-renders every time parent does
const Row = ({ data }) => { return <div>{data.name}</div>; };

// ✅ Memoized - only re-renders if `data` prop actually changes
const Row = React.memo(({ data }) => { return <div>{data.name}</div>; });
```

---

#### ❌ Reason 4: Inline Functions / Objects as Props
Inline functions create a **new reference on every render**, which breaks `React.memo`.

```jsx
// ❌ New function reference on every render → React.memo thinks prop changed
<Row onEdit={() => handleEdit(row.id)} data={row} />

// ✅ Stable reference with useCallback
const handleEdit = useCallback((id) => { ... }, []);
<Row onEdit={handleEdit} data={row} />
```

---

#### ❌ Reason 5: All Rows in One State Object
Updating one row replaces the entire array, triggering a full diff.

```jsx
// ❌ Replacing whole array
setRows(rows.map(r => r.id === id ? updatedRow : r));

// ✅ Use normalized state (object keyed by ID)
const [rowsMap, setRowsMap] = useState({ "1": {...}, "2": {...} });
setRowsMap(prev => ({ ...prev, [id]: updatedRow })); // Only one key changes
```

---

**Interview Answer Summary:**
> *"The most common reason is that edit state is held in the parent component, causing full re-renders. Combined with missing `React.memo()`, unstable prop references, and wrong keys — React re-renders all 1000 rows. Fix by pushing state into row components, memoizing with `React.memo`, using `useCallback` for handlers, and ensuring stable unique keys."*

---

---

## ⚡ Section 1: Performance & Optimization

---

### Q2. Your React dashboard has 10+ widgets, each fetching their own data. Users complain the page feels slow on first load. How do you diagnose and fix this?

**Diagnosis Steps:**
- Open Chrome DevTools → Network tab → check for **waterfall of API calls** (sequential instead of parallel)
- Check the Performance tab for **long tasks** blocking the main thread
- Use React DevTools Profiler to spot **slow renders**

**Fixes:**

```jsx
// 1. Fetch all data in parallel using Promise.all
const [users, orders, analytics] = await Promise.all([
  fetchUsers(),
  fetchOrders(),
  fetchAnalytics()
]);

// 2. Use React Query for smart caching & deduplication
const { data } = useQuery(['widget-data'], fetchWidgetData, {
  staleTime: 5 * 60 * 1000, // Don't refetch for 5 mins
});

// 3. Lazy load widgets below the fold
const HeavyWidget = React.lazy(() => import('./HeavyWidget'));

<Suspense fallback={<Skeleton />}>
  <HeavyWidget />
</Suspense>

// 4. Show skeleton screens for perceived performance
const Widget = () => {
  const { data, isLoading } = useQuery(...);
  if (isLoading) return <SkeletonCard />;
  return <ActualWidget data={data} />;
};
```

**Other strategies:**
- Use **HTTP/2** to parallelize requests
- Move shared data fetching to a **layout/page level** and pass down as props
- Use **React Query's prefetching** on hover/focus before user clicks

---

### Q3. You have a virtualized list of 10,000 rows with drag-and-drop. After every drop, the list jitters and repaints. What's causing this and how do you fix it?

**Root Causes:**
- DOM mutation during drag triggers a **full layout reflow**
- `key` prop changes after reorder causing React to **unmount/remount** components
- CSS transitions conflicting with drag library's positioning

**Fixes:**

```jsx
// 1. Use react-window or react-virtual for virtualization
import { FixedSizeList } from 'react-window';

<FixedSizeList height={600} itemCount={10000} itemSize={50}>
  {({ index, style }) => <Row style={style} data={items[index]} />}
</FixedSizeList>

// 2. For drag-and-drop, use dnd-kit which is optimized for virtualized lists
import { DndContext, closestCenter } from '@dnd-kit/core';
import { SortableContext } from '@dnd-kit/sortable';

// 3. Stabilize keys — use item IDs, NOT index
{items.map(item => <SortableRow key={item.id} id={item.id} />)}

// 4. Use CSS will-change to hint GPU compositing
.row-dragging {
  will-change: transform;
  transform: translateZ(0); /* Force GPU layer */
}
```

**Key Insight:** During drag, update a **draft state** (not the real list), only committing on drop. This prevents React from re-rendering the full list during drag.

---

### Q4. A React app works fine in dev but is slow in production. Bundle size is 4MB. Walk me through your entire optimization strategy.

**Step 1 — Analyze the bundle:**
```bash
# Using webpack-bundle-analyzer
npm install --save-dev webpack-bundle-analyzer
npx source-map-explorer build/static/js/*.js
```

**Step 2 — Code splitting:**
```jsx
// Route-level splitting
const Dashboard = React.lazy(() => import('./pages/Dashboard'));
const Settings = React.lazy(() => import('./pages/Settings'));

// Component-level splitting for heavy libs
const RichEditor = React.lazy(() => import('./components/RichEditor'));
```

**Step 3 — Tree shaking (remove unused code):**
```jsx
// ❌ Imports entire lodash (~70KB)
import _ from 'lodash';

// ✅ Import only what you need (~2KB)
import debounce from 'lodash/debounce';
```

**Step 4 — Replace heavy libraries:**
| Heavy Library | Lightweight Alternative |
|---|---|
| moment.js (66KB) | date-fns (13KB) or dayjs (7KB) |
| lodash (entire) | Individual imports or native JS |
| axios | native fetch |

**Step 5 — Other optimizations:**
```jsx
// Image optimization
<img src={image} loading="lazy" width={300} height={200} />

// Preload critical assets
<link rel="preload" href="/critical.css" as="style" />

// Enable Brotli/gzip compression on server
// Use CDN for static assets
```

---

### Q5. Your app re-renders 300+ components on a single keypress in a search input. How do you trace and fix this?

**Tracing:**
```jsx
// 1. React DevTools Profiler → record → type → see what re-rendered
// 2. Add why-did-you-render library in dev
import whyDidYouRender from '@welldone-software/why-did-you-render';
whyDidYouRender(React, { trackAllPureComponents: true });
```

**Fixing:**

```jsx
// ❌ Search state in parent causes 300 children to re-render
const Page = () => {
  const [search, setSearch] = useState('');
  return (
    <>
      <input onChange={e => setSearch(e.target.value)} />
      <HugeList data={filteredData} /> {/* Re-renders every keypress */}
    </>
  );
};

// ✅ Fix 1: Debounce the search
const [search, setSearch] = useState('');
const debouncedSearch = useDebounce(search, 300);
// Only filter/re-render when user stops typing

// ✅ Fix 2: Use useDeferredValue (React 18)
const deferredSearch = useDeferredValue(search);
// List renders with stale value while user types — feels instant

// ✅ Fix 3: Move search state into its own isolated component
const SearchInput = ({ onSearch }) => {
  const [local, setLocal] = useState('');
  return <input value={local} onChange={e => {
    setLocal(e.target.value);
    debounce(onSearch, 300)(e.target.value); // Parent only updates after debounce
  }} />;
};

// ✅ Fix 4: Memoize the filtered list
const filteredData = useMemo(() => 
  data.filter(item => item.name.includes(debouncedSearch)),
  [data, debouncedSearch]
);
```

---

## 🏗️ Section 2: Architecture & Design

---

### Q6. You're building a micro-frontend architecture where 3 teams own different parts of the same page. How do you handle shared state, routing, and CSS conflicts?

**Architecture Overview:**

```
Shell App (Host)
├── Team A → Header MFE
├── Team B → Dashboard MFE  
└── Team C → Settings MFE
```

**Shared State:**
```js
// Use a shared event bus (CustomEvents) for loose coupling
// Team A dispatches:
window.dispatchEvent(new CustomEvent('user:updated', { detail: { userId: 123 } }));

// Team B listens:
window.addEventListener('user:updated', (e) => {
  setUser(e.detail);
});

// OR use a shared store exposed on window (last resort)
window.__sharedStore = createStore();
```

**Routing:**
```js
// Shell app owns top-level routing
// Each MFE owns its sub-routes internally
// Use Module Federation (Webpack 5) or Single-SPA framework

// webpack.config.js (Shell)
new ModuleFederationPlugin({
  remotes: {
    teamA: 'teamA@https://team-a.cdn.com/remoteEntry.js',
    teamB: 'teamB@https://team-b.cdn.com/remoteEntry.js',
  }
})
```

**CSS Conflicts:**
```css
/* Option 1: CSS Modules - scoped automatically */
/* Option 2: Shadow DOM for full isolation */
/* Option 3: CSS-in-JS (styled-components) */
/* Option 4: BEM prefix per team */
.teamA__button { }
.teamB__button { }
```

---

### Q7. A junior dev asks why not put all API calls inside components. How do you architect the data layer for a large-scale React app?

**Problems with API calls in components:**
- Hard to test (need to mock fetch inside component)
- Can't reuse across components
- Business logic mixed with UI logic
- No central error handling

**Proper Architecture:**

```
src/
├── api/           ← Raw API calls (axios/fetch)
│   └── userApi.ts
├── services/      ← Business logic layer
│   └── userService.ts
├── hooks/         ← React-specific data fetching
│   └── useUser.ts
└── components/    ← Pure UI, no API calls
    └── UserCard.tsx
```

```ts
// api/userApi.ts — Only HTTP concerns
export const userApi = {
  getById: (id: string) => axios.get(`/users/${id}`),
  update: (id: string, data: User) => axios.put(`/users/${id}`, data),
};

// services/userService.ts — Business logic
export const userService = {
  getUser: async (id: string) => {
    const res = await userApi.getById(id);
    return transformUser(res.data); // transform/validate data here
  }
};

// hooks/useUser.ts — React integration
export const useUser = (id: string) => {
  return useQuery(['user', id], () => userService.getUser(id));
};

// components/UserCard.tsx — Pure UI, no API
const UserCard = ({ userId }) => {
  const { data, isLoading } = useUser(userId); // Clean!
  return isLoading ? <Skeleton /> : <div>{data.name}</div>;
};
```

---

### Q8. Your team is migrating a 5-year-old class-based React codebase to functional components. What's your strategy to avoid breaking production?

**Strategy: Strangler Fig Pattern** — migrate incrementally, never rewrite all at once.

```
Phase 1: Setup (Week 1)
  ├── Add TypeScript gradually
  ├── Set up ESLint rules for hooks
  └── Identify component priority (leaf → parent)

Phase 2: Migrate leaf components first
  ├── Simple display components with no state → easiest
  ├── Then add useState, useEffect equivalents
  └── Each migration gets its own PR + review

Phase 3: Complex components
  ├── componentDidMount → useEffect(() => {}, [])
  ├── componentDidUpdate → useEffect(() => {}, [deps])
  └── componentWillUnmount → useEffect(() => { return () => cleanup() }, [])

Phase 4: State management migration
  └── Move from this.state to useState / useReducer
```

```jsx
// Class component
class UserCard extends React.Component {
  state = { user: null };
  componentDidMount() { fetchUser(this.props.id).then(u => this.setState({user: u})); }
  render() { return <div>{this.state.user?.name}</div>; }
}

// ✅ Equivalent functional component
const UserCard = ({ id }) => {
  const [user, setUser] = useState(null);
  useEffect(() => {
    fetchUser(id).then(setUser);
  }, [id]);
  return <div>{user?.name}</div>;
};
```

**Important rules:**
- Never migrate and add features in the same PR
- Keep a feature flag to roll back if needed
- Write tests BEFORE migrating so you can verify behavior is identical

---

### Q9. You need to build a design system used by 6 different React apps. How do you structure, version, and distribute it?

**Structure:**
```
design-system/
├── packages/
│   ├── tokens/        ← Colors, spacing, typography (CSS vars + JS)
│   ├── components/    ← React components
│   └── icons/         ← SVG icon library
├── apps/
│   └── storybook/     ← Documentation + visual testing
└── package.json       ← Monorepo with Turborepo/Nx
```

**Versioning & Distribution:**
```json
// Use semantic versioning strictly
// Publish to npm (private registry or public)
{
  "name": "@company/design-system",
  "version": "2.3.1"  // MAJOR.MINOR.PATCH
}
```

```bash
# Consuming apps install it like any package
npm install @company/design-system

# Each app pins to a version — no surprise breakage
"@company/design-system": "^2.3.0"
```

**Key Decisions:**
- Use **Storybook** for documentation and visual regression testing
- Use **Changesets** for changelog management
- Ship **CSS variables for tokens** so theming works without JS
- Test with **Chromatic** for visual diffs on every PR

---

*📄 Continue in Part 2 → State Management, Networking & Security*
