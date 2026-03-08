# 08 - Authentication & Authorization in React

## ❓ Interview Question
> "How would you design authentication and authorization in a React application? Consider protected routes, redirecting unauthenticated users, storing tokens, and managing different user roles and permissions."

---

## 🙋 My Answer (What I Said)
- On app load, check if refresh token exists in cookie
- If expired or missing → user must login or register
- On register/login → get access token (15-30 min) and refresh token (2-7 days)
- When access token expires → use refresh token to silently get new access token
- If refresh token expires → force re-login
- Store both tokens in HTTPOnly cookies (not localStorage — XSS safe)
- Use Context to store user details, roles, and permissions
- Create a `ProtectedRoute` wrapper component to check roles/permissions
- Wrap protected pages in `ProtectedRoute` — redirect to home if unauthorized
- Assign roles (student, teacher, admin) at registration

---

## ✅ What I Got Right
- Checking refresh token on app load
- Access + Refresh token concept explained correctly
- Short access token expiry for XSS safety
- Auto refresh using refresh token
- HTTPOnly cookies — excellent security choice
- Context for user roles/permissions
- ProtectedRoute wrapper component
- Redirecting unauthorized users
- Role-based access at registration

## ❌ What Was Missing
- Logout handling — clearing cookies, resetting context
- Persisting auth state on page refresh via useEffect
- CSRF protection (important when using cookies)
- Axios interceptor to globally handle 401 and auto-refresh

---

## 💡 Model Answer

### Auth Context

```jsx
import { createContext, useContext, useState, useEffect } from "react";

const AuthContext = createContext(null);

export const AuthProvider = ({ children }) => {
  const [user, setUser] = useState(null);
  const [loading, setLoading] = useState(true);

  // Check auth status on app load
  useEffect(() => {
    const verifyToken = async () => {
      try {
        const res = await fetch("/api/auth/verify", {
          credentials: "include", // send cookies
        });
        if (res.ok) {
          const data = await res.json();
          setUser(data.user); // { id, name, role, permissions }
        }
      } catch {
        setUser(null);
      } finally {
        setLoading(false);
      }
    };
    verifyToken();
  }, []);

  const login = async (credentials) => {
    const res = await fetch("/api/auth/login", {
      method: "POST",
      headers: { "Content-Type": "application/json" },
      credentials: "include", // receive HTTPOnly cookies
      body: JSON.stringify(credentials),
    });
    const data = await res.json();
    setUser(data.user);
  };

  const logout = async () => {
    await fetch("/api/auth/logout", {
      method: "POST",
      credentials: "include",
    });
    setUser(null); // clear context
  };

  return (
    <AuthContext.Provider value={{ user, login, logout, loading }}>
      {children}
    </AuthContext.Provider>
  );
};

export const useAuth = () => useContext(AuthContext);
```

### Protected Route Component

```jsx
import { Navigate } from "react-router-dom";
import { useAuth } from "./AuthContext";

const ProtectedRoute = ({ children, allowedRoles = [] }) => {
  const { user, loading } = useAuth();

  if (loading) return <div>Loading...</div>;

  // Not logged in → redirect to login
  if (!user) return <Navigate to="/login" replace />;

  // Logged in but wrong role → redirect to home
  if (allowedRoles.length > 0 && !allowedRoles.includes(user.role)) {
    return <Navigate to="/home" replace />;
  }

  return children;
};

export default ProtectedRoute;
```

### Route Setup with Role-Based Protection

```jsx
import { BrowserRouter, Routes, Route } from "react-router-dom";

const App = () => (
  <AuthProvider>
    <BrowserRouter>
      <Routes>
        {/* Public routes */}
        <Route path="/login" element={<Login />} />
        <Route path="/register" element={<Register />} />

        {/* Any logged-in user */}
        <Route
          path="/dashboard"
          element={
            <ProtectedRoute>
              <Dashboard />
            </ProtectedRoute>
          }
        />

        {/* Admin only */}
        <Route
          path="/admin"
          element={
            <ProtectedRoute allowedRoles={["admin"]}>
              <AdminPanel />
            </ProtectedRoute>
          }
        />

        {/* Teacher or Admin */}
        <Route
          path="/courses"
          element={
            <ProtectedRoute allowedRoles={["teacher", "admin"]}>
              <CoursesPage />
            </ProtectedRoute>
          }
        />
      </Routes>
    </BrowserRouter>
  </AuthProvider>
);
```

### Axios Interceptor for Auto Token Refresh

```js
import axios from "axios";

const api = axios.create({
  baseURL: "/api",
  withCredentials: true, // send cookies with every request
});

// Response interceptor — catch 401 and refresh token
api.interceptors.response.use(
  (response) => response,
  async (error) => {
    const original = error.config;

    if (error.response?.status === 401 && !original._retry) {
      original._retry = true;
      try {
        // Try to get new access token using refresh token
        await axios.post("/api/auth/refresh", {}, { withCredentials: true });
        return api(original); // retry original request
      } catch {
        // Refresh failed — redirect to login
        window.location.href = "/login";
      }
    }
    return Promise.reject(error);
  }
);

export default api;
```

### Token Strategy Summary

```
User logs in
    ↓
Server sets:
  - Access Token  → HTTPOnly Cookie → expires in 15-30 mins
  - Refresh Token → HTTPOnly Cookie → expires in 2-7 days
    ↓
Every API request sends cookies automatically
    ↓
Access token expires → 401 response
    ↓
Interceptor calls /auth/refresh
    ↓
Server checks refresh token → issues new access token
    ↓
Refresh token expires → user redirected to /login
```

---

## 🔑 Key Concepts to Remember
| Concept | Detail |
|---|---|
| Token storage | HTTPOnly cookies — safe from XSS (never localStorage) |
| Access token | Short lived (15-30 min) — used for API calls |
| Refresh token | Long lived (2-7 days) — used to get new access token |
| Auto refresh | Axios interceptor catches 401 and silently refreshes |
| Auth Context | Stores user, roles, login(), logout() — wrap whole app |
| ProtectedRoute | Checks user + role — redirects if unauthorized |
| CSRF | Use CSRF tokens as extra protection when using cookies |
| On app load | Call verify endpoint in useEffect to restore auth state |
| Logout | Clear cookies server-side AND reset user state in context |
