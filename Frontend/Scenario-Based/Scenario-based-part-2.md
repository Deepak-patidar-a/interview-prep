# 🔄 React Interview Questions & Answers — Part 2
### For 5+ Years Experience | State Management, Networking & Security

---

## 🔄 Section 3: State Management

---

### Q10. Your app uses Redux but developers complain that adding a simple feature requires touching 5 files. How do you restructure this?

**The Problem — Classic Redux (too much boilerplate):**
```
actions/userActions.js       ← Action constants + creators
reducers/userReducer.js      ← Reducer
selectors/userSelectors.js   ← Selectors
sagas/userSagas.js           ← Side effects
store/index.js               ← Combine reducers
```
Every single feature touches all 5 files. That's why devs hate it.

---

**Solution 1: Redux Toolkit (RTK) — Feature Slice Pattern**
```js
// ✅ ONE file handles everything for a feature
// features/user/userSlice.js
import { createSlice, createAsyncThunk } from '@reduxjs/toolkit';

// Async thunk replaces sagas for most cases
export const fetchUser = createAsyncThunk('user/fetch', async (id) => {
  const res = await api.getUser(id);
  return res.data;
});

const userSlice = createSlice({
  name: 'user',
  initialState: { data: null, loading: false, error: null },
  reducers: {
    // Sync actions — immer lets you "mutate" directly
    clearUser: (state) => { state.data = null; },
    updateName: (state, action) => { state.data.name = action.payload; },
  },
  extraReducers: (builder) => {
    builder
      .addCase(fetchUser.pending, (state) => { state.loading = true; })
      .addCase(fetchUser.fulfilled, (state, action) => {
        state.loading = false;
        state.data = action.payload;
      })
      .addCase(fetchUser.rejected, (state, action) => {
        state.loading = false;
        state.error = action.error.message;
      });
  }
});

export const { clearUser, updateName } = userSlice.actions;
export default userSlice.reducer;
```

**Feature-based folder structure:**
```
src/
├── features/
│   ├── user/
│   │   ├── userSlice.js      ← State + actions + reducers
│   │   ├── UserCard.jsx      ← Component
│   │   └── userSelectors.js  ← Memoized selectors (optional)
│   └── orders/
│       └── ordersSlice.js
└── store/
    └── index.js              ← Just combines slices
```

---

### Q11. You have a deeply nested form (5 levels deep) with conditional fields, validation, and autosave. What state management approach do you use?

**Best Approach: React Hook Form + Zod + Custom Autosave Hook**

```jsx
// 1. React Hook Form handles form state efficiently
// (doesn't re-render on every keystroke like controlled inputs)
import { useForm, useFieldArray } from 'react-hook-form';
import { zodResolver } from '@hookform/resolvers/zod';
import { z } from 'zod';

const schema = z.object({
  personalInfo: z.object({
    name: z.string().min(1, 'Required'),
    email: z.string().email(),
  }),
  addresses: z.array(z.object({
    street: z.string(),
    city: z.string(),
    // Conditional: zipCode only required if country is 'US'
    zipCode: z.string().optional(),
  }))
});

const ComplexForm = () => {
  const { register, handleSubmit, watch, formState: { errors } } = useForm({
    resolver: zodResolver(schema),
    defaultValues: { personalInfo: {}, addresses: [{}] }
  });

  // 2. Conditional fields
  const country = watch('addresses.0.country');

  // 3. Autosave with debounce
  const formValues = watch();
  useAutosave(formValues, async (data) => {
    await api.saveDraft(data);
  }, 2000); // Save 2 seconds after user stops typing

  return (
    <form onSubmit={handleSubmit(onSubmit)}>
      <input {...register('personalInfo.name')} />
      {errors.personalInfo?.name && <span>{errors.personalInfo.name.message}</span>}

      {/* Conditional field */}
      {country === 'US' && <input {...register('addresses.0.zipCode')} />}
    </form>
  );
};

// 4. Autosave hook
const useAutosave = (data, saveFn, delay = 1000) => {
  useEffect(() => {
    const timer = setTimeout(() => saveFn(data), delay);
    return () => clearTimeout(timer);
  }, [JSON.stringify(data)]); // Only trigger when data actually changes
};
```

**Why NOT useState for this:**
- 5 levels deep = prop drilling nightmare
- Every field change triggers re-render of the whole form
- Validation logic gets messy

---

### Q12. Two sibling components need to share real-time data updating every 500ms. What's your approach — Context, Redux, Zustand, or something else?

**Walk through the trade-offs:**

| Option | Good For | Problem with 500ms updates |
|---|---|---|
| Context API | Simple shared state | Re-renders ALL consumers on every update |
| Redux | Complex global state | Too much boilerplate for this use case |
| Zustand | Simple + performant | ✅ Best option here |
| React Query | Server state | Good if data comes from API |

**Winner: Zustand with selectors**

```js
// store/liveDataStore.js
import { create } from 'zustand';

const useLiveStore = create((set) => ({
  price: 0,
  volume: 0,
  updateData: (data) => set(data),
}));

// Component A — only subscribes to price
const PriceDisplay = () => {
  const price = useLiveStore(state => state.price); // ← Selector!
  // Only re-renders when price changes, NOT when volume changes
  return <div>${price}</div>;
};

// Component B — only subscribes to volume
const VolumeDisplay = () => {
  const volume = useLiveStore(state => state.volume);
  return <div>{volume}</div>;
};

// WebSocket feeding the store
useEffect(() => {
  const ws = new WebSocket('wss://realtime.api.com');
  ws.onmessage = (e) => {
    useLiveStore.getState().updateData(JSON.parse(e.data));
  };
  return () => ws.close();
}, []);
```

**Why NOT Context here:**
```jsx
// ❌ Context re-renders ALL consumers every 500ms
const LiveContext = createContext();
const LiveProvider = ({ children }) => {
  const [data, setData] = useState({});
  // Every update → ALL consumers re-render, even if their slice didn't change
  return <LiveContext.Provider value={data}>{children}</LiveContext.Provider>;
};
```

---

## 🌐 Section 4: Networking & Data Fetching

---

### Q13. Your app makes 15 API calls on page load. The backend can't create a BFF. How do you reduce perceived load time?

**Strategy 1: Parallel fetching (most impactful)**
```js
// ❌ Sequential - each waits for previous
const user = await fetchUser();
const orders = await fetchOrders();
const notifications = await fetchNotifications();
// Total time = time1 + time2 + time3

// ✅ Parallel
const [user, orders, notifications] = await Promise.all([
  fetchUser(),
  fetchOrders(),
  fetchNotifications()
]);
// Total time = max(time1, time2, time3)
```

**Strategy 2: Prioritize above-the-fold data**
```jsx
// Load critical data first, defer the rest
const Page = () => {
  const { data: heroData } = useQuery('hero', fetchHero, { priority: 'high' });

  return (
    <>
      <HeroSection data={heroData} />
      {/* Below fold widgets load lazily */}
      <Suspense fallback={<Skeleton />}>
        <LazyWidget />
      </Suspense>
    </>
  );
};
```

**Strategy 3: Prefetch on route change**
```js
// React Query — prefetch on hover before user even clicks
const queryClient = useQueryClient();

<Link
  onMouseEnter={() => queryClient.prefetchQuery(['dashboard'], fetchDashboard)}
  to="/dashboard"
>
  Dashboard
</Link>
```

**Strategy 4: Skeleton screens + optimistic UI**
```jsx
// Show skeleton immediately — user perceives faster load
const OrdersList = () => {
  const { data, isLoading } = useQuery('orders', fetchOrders);
  if (isLoading) return <OrdersSkeleton />;
  return data.map(order => <OrderRow key={order.id} order={order} />);
};
```

---

### Q14. A user fills a long form and submits, but internet drops mid-request. How do you handle this gracefully end-to-end?

**Full Handling Strategy:**

```jsx
const LongForm = () => {
  const [submitStatus, setSubmitStatus] = useState('idle'); // idle|saving|offline|error|success

  const handleSubmit = async (data) => {
    setSubmitStatus('saving');

    // 1. Check connectivity before submitting
    if (!navigator.onLine) {
      saveToLocalQueue(data);       // Queue for later
      setSubmitStatus('offline');
      return;
    }

    try {
      // 2. Use AbortController for timeout
      const controller = new AbortController();
      const timeoutId = setTimeout(() => controller.abort(), 10000); // 10s timeout

      await api.submitForm(data, { signal: controller.signal });
      clearTimeout(timeoutId);
      setSubmitStatus('success');

    } catch (err) {
      if (err.name === 'AbortError') {
        // Request timed out
        saveToLocalQueue(data);
        setSubmitStatus('offline');
      } else {
        setSubmitStatus('error');
      }
    }
  };

  // 3. Auto-retry when connection restores
  useEffect(() => {
    const handleOnline = async () => {
      const queue = getLocalQueue();
      if (queue.length > 0) {
        for (const item of queue) {
          await api.submitForm(item); // Drain the queue
          removeFromQueue(item.id);
        }
      }
    };
    window.addEventListener('online', handleOnline);
    return () => window.removeEventListener('online', handleOnline);
  }, []);

  return (
    <form onSubmit={handleSubmit(onSubmit)}>
      {/* ... form fields ... */}
      {submitStatus === 'offline' && (
        <Alert>You're offline. Your form is saved and will submit when you reconnect.</Alert>
      )}
      {submitStatus === 'saving' && <Spinner />}
    </form>
  );
};

// 4. Autosave draft to prevent data loss
const saveToLocalQueue = (data) => {
  const queue = JSON.parse(localStorage.getItem('submitQueue') || '[]');
  queue.push({ ...data, id: Date.now() });
  localStorage.setItem('submitQueue', JSON.stringify(queue));
};
```

---

### Q15. You're using React Query and stale data shows for 10 seconds after a mutation. How do you fix this without setting staleTime to 0?

**The Problem:**
```js
// Data is cached and not refreshed after mutation
const { mutate } = useMutation(updateUser);
mutate({ name: 'New Name' }); // Updates server but UI still shows old name
```

**Fix 1: Invalidate queries after mutation (recommended)**
```js
const queryClient = useQueryClient();

const { mutate } = useMutation(updateUser, {
  onSuccess: () => {
    // Tell React Query this cache is now stale → triggers refetch
    queryClient.invalidateQueries(['user', userId]);
  }
});
```

**Fix 2: Optimistic updates (best UX)**
```js
const { mutate } = useMutation(updateUser, {
  onMutate: async (newUser) => {
    // Cancel in-flight queries to avoid overwriting our optimistic update
    await queryClient.cancelQueries(['user', userId]);

    // Save previous data in case we need to roll back
    const previousUser = queryClient.getQueryData(['user', userId]);

    // Immediately update the UI (optimistic!)
    queryClient.setQueryData(['user', userId], newUser);

    return { previousUser };
  },
  onError: (err, newUser, context) => {
    // Mutation failed → roll back to previous data
    queryClient.setQueryData(['user', userId], context.previousUser);
  },
  onSettled: () => {
    // Always refetch after success or error to ensure consistency
    queryClient.invalidateQueries(['user', userId]);
  }
});
```

---

### Q16. Your app needs to support offline mode for field agents with no connectivity. How do you architect this on the frontend?

**Architecture: Offline-First with Service Worker + IndexedDB**

```
Browser
├── Service Worker (intercepts all network requests)
│   ├── Cache API → Static assets (HTML, CSS, JS)
│   └── Background Sync → Queue failed API calls
├── IndexedDB (via Dexie.js)
│   └── Local data store (survives page refresh)
└── React App
    └── Reads from IndexedDB first, syncs to server when online
```

```js
// 1. Service Worker — cache static assets
self.addEventListener('install', (event) => {
  event.waitUntil(
    caches.open('v1').then(cache => cache.addAll([
      '/', '/index.html', '/main.js', '/styles.css'
    ]))
  );
});

// 2. Intercept fetch — serve from cache when offline
self.addEventListener('fetch', (event) => {
  event.respondWith(
    fetch(event.request).catch(() => caches.match(event.request))
  );
});

// 3. IndexedDB for data (using Dexie.js)
import Dexie from 'dexie';
const db = new Dexie('FieldApp');
db.version(1).stores({
  reports: '++id, status, syncedAt'
});

// 4. React hook that reads local-first
const useReports = () => {
  const [reports, setReports] = useState([]);

  useEffect(() => {
    // Always read from IndexedDB first (instant)
    db.reports.toArray().then(setReports);

    // Then sync with server if online
    if (navigator.onLine) {
      fetchReportsFromServer().then(serverReports => {
        db.reports.bulkPut(serverReports); // Update local DB
        setReports(serverReports);
      });
    }
  }, []);

  return reports;
};

// 5. Saving while offline
const saveReport = async (report) => {
  await db.reports.add({ ...report, status: 'pending' });

  if (navigator.onLine) {
    await api.saveReport(report);
    await db.reports.update(report.id, { status: 'synced' });
  }
  // If offline, Background Sync will handle it when connection returns
};
```

---

## 🔐 Section 5: Security

---

### Q17. A security audit flags your React app is vulnerable to XSS. Where are the common React XSS entry points and how do you patch them?

**Entry Point 1: `dangerouslySetInnerHTML`**
```jsx
// ❌ DANGEROUS — never use with user input
<div dangerouslySetInnerHTML={{ __html: userProvidedContent }} />

// ✅ Sanitize with DOMPurify before rendering
import DOMPurify from 'dompurify';
const clean = DOMPurify.sanitize(userProvidedContent);
<div dangerouslySetInnerHTML={{ __html: clean }} />

// ✅ Better: use a markdown renderer with sanitization built in
import ReactMarkdown from 'react-markdown';
<ReactMarkdown>{userContent}</ReactMarkdown>
```

**Entry Point 2: `href` with `javascript:` protocol**
```jsx
// ❌ Allows javascript: URLs
<a href={userProvidedUrl}>Click me</a>
// User sets href to: javascript:alert(document.cookie)

// ✅ Validate URL protocol
const SafeLink = ({ href, children }) => {
  const isSafe = href.startsWith('http://') || href.startsWith('https://');
  return isSafe ? <a href={href}>{children}</a> : <span>{children}</span>;
};
```

**Entry Point 3: `eval()` and `new Function()`**
```js
// ❌ Never evaluate user input
eval(userInput);
new Function(userInput)();

// ✅ Use JSON.parse for data, never eval
const data = JSON.parse(userInput); // Safe — only parses data, not code
```

**Entry Point 4: DOM manipulation outside React**
```js
// ❌ Bypasses React's escaping
document.getElementById('output').innerHTML = userInput;

// ✅ Use React state instead
const [content, setContent] = useState('');
<div>{content}</div> {/* React escapes this automatically */}
```

**Security Headers (add via server/nginx):**
```
Content-Security-Policy: default-src 'self'; script-src 'self'
X-Content-Type-Options: nosniff
X-Frame-Options: DENY
```

---

### Q18. Your app stores JWT in localStorage. The security team raises a concern. What's the trade-off and what would you change?

**The Problem with localStorage:**
- Accessible via JavaScript → XSS attack can steal tokens
- `document.cookie` and `localStorage` both readable by any script on the page
- Once stolen, attacker has full access until token expires

**Trade-offs:**

| Storage | XSS Risk | CSRF Risk | Best For |
|---|---|---|---|
| localStorage | ❌ High | ✅ Safe | Never JWT |
| sessionStorage | ❌ High | ✅ Safe | Never JWT |
| httpOnly Cookie | ✅ Safe | ❌ Needs protection | ✅ JWT tokens |
| Memory (React state) | ✅ Safe | ✅ Safe | Short-lived tokens |

**Recommended Approach:**
```
1. Store JWT in httpOnly cookie (server sets it, JS can't read it)
2. Add CSRF protection (SameSite=Strict or CSRF token)
3. Keep a non-sensitive "logged-in" flag in localStorage for UI
```

```js
// Server sets token in httpOnly cookie (Node.js/Express example)
res.cookie('auth_token', jwt, {
  httpOnly: true,      // JS can't read it
  secure: true,        // HTTPS only
  sameSite: 'Strict',  // CSRF protection
  maxAge: 7 * 24 * 60 * 60 * 1000 // 7 days
});

// Client never touches the token
// Browser automatically sends it with every request
fetch('/api/user', { credentials: 'include' }); // Cookie sent automatically

// For UI state (is user logged in?), store non-sensitive flag
localStorage.setItem('isLoggedIn', 'true'); // Not a secret, just for UX
```

---

### Q19. How do you prevent an authenticated user from accessing another user's data through frontend route manipulation?

**Important Principle:** The frontend is NOT a security boundary. All authorization must happen on the backend.

**Frontend protections (defense in depth):**

```jsx
// 1. Route guard — check auth before rendering protected routes
const ProtectedRoute = ({ children }) => {
  const { user, isLoading } = useAuth();
  if (isLoading) return <Spinner />;
  if (!user) return <Navigate to="/login" replace />;
  return children;
};

// 2. Check ownership before showing edit UI
const EditProfile = ({ profileId }) => {
  const { user } = useAuth();

  // Frontend check — prevents accidental navigation, NOT security
  if (user.id !== profileId) {
    return <Navigate to="/forbidden" />;
  }
  return <ProfileForm />;
};

// 3. Strip sensitive data from API responses on the backend
// ❌ Bad: returning full user object including password hash
// ✅ Good: return only what the client needs
```

**Backend must enforce (the real security):**
```js
// Node.js/Express example
app.get('/api/users/:id/data', authMiddleware, (req, res) => {
  // Always check: does the authenticated user own this resource?
  if (req.user.id !== req.params.id && !req.user.isAdmin) {
    return res.status(403).json({ error: 'Forbidden' });
  }
  // Proceed only if authorized
  const data = await getUserData(req.params.id);
  res.json(data);
});
```

**The key insight:** Even if a user manually changes the URL or intercepts a request, the backend returns 403. Frontend checks only improve UX (no flashes of forbidden content) — they are NOT a substitute for backend authorization.

---

*📄 Continue in Part 3 → Testing, CI/CD & Leadership Questions*
