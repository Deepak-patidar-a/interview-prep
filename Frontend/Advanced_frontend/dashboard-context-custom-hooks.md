# ⚛️ Scenario — Dashboard with Context + Custom Hooks

## The Problem
Charts, notifications, and data fetches are completely independent. If they share one Context — when any one updates, everything re-renders together.

---

## ❌ Wrong Approach — One Big Context

```javascript
// All 3 concerns in one context
// Chart update → notifications re-render too (unnecessary)
const DashboardContext = createContext({
  charts: [],
  notifications: [],
  userData: {}
});
```

---

## ✅ Right Approach — Separate Contexts + Custom Hooks

### Step 1 — Separate context per concern

```javascript
// Each context is independent — updating one doesn't affect others
const ChartContext  = createContext();
const NotifContext  = createContext();
const UserContext   = createContext();
```

### Step 2 — Custom hook per concern

```javascript
// Charts — polls every 5 seconds
function useChartData() {
  const [charts, setCharts] = useState([]);

  useEffect(() => {
    const interval = setInterval(async () => {
      const data = await fetchChartData();
      setCharts(data);
    }, 5000);

    return () => clearInterval(interval);
  }, []);

  return { charts };
}

// Notifications — real-time via WebSocket
function useNotifications() {
  const [notifications, setNotifications] = useState([]);

  useEffect(() => {
    const socket = connectWebSocket();

    socket.on('notification', (notif) => {
      setNotifications(prev => [...prev, notif]);
    });

    return () => socket.disconnect();
  }, []);

  return { notifications };
}

// User data — fetches once on mount
function useUserData() {
  const [userData, setUserData] = useState(null);

  useEffect(() => {
    fetchUserData().then(setUserData);
  }, []);

  return { userData };
}
```

### Step 3 — Dashboard uses all 3 independently

```javascript
function Dashboard() {
  const { charts }        = useChartData();        // re-renders only when charts update
  const { notifications } = useNotifications();    // re-renders only when notif arrives
  const { userData }      = useUserData();         // re-renders only when user changes

  return (
    <div>
      <Charts data={charts} />
      <Notifications data={notifications} />
      <UserProfile data={userData} />
    </div>
  );
}
```

---

## Result

```
Chart updates every 5s    → only Charts component re-renders ✅
New notification arrives  → only Notifications re-renders ✅
User data loads once      → only UserProfile re-renders ✅
No unnecessary re-renders across concerns ✅
```

---

## 🔑 Key Points To Remember
- One big context = one update re-renders everything — avoid for independent concerns
- Split contexts by concern — each updates independently
- Custom hook per concern = clean, reusable, testable logic
- Each component only re-renders when its own data changes
- This pattern is called **separation of concerns** — mention this term in interview
