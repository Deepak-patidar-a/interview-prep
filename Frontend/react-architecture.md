# React App Architecture

## Question
What is the overall structure of your React application? How do you organize folders, manage global state, handle protected routes, and how does the JWT token in cookies get sent with every API request?

---

## My Answer
We use React with TypeScript. Types are defined centrally or at component level depending on scope. Folder structure is src → Modules, commons, hooks, context. Inside Modules we have components, pages, and common components. For global state we use Context API. We have one ProtectedRoute function that contains all the auth logic and we wrap our routes inside it. JWT tokens stored in cookies are sent as base64 encoded in the JWT payload.

## What I Got Right
- Clean folder structure — feature-based architecture with Modules is a good pattern
- Centralized TypeScript types is a good practice
- Context API for global state — correct for mid-sized apps
- ProtectedRoute wrapper pattern — industry standard for auth guards
- Knowing JWT travels in cookies

## What I Got Wrong
- JWT payload is base64 encoded, not base26. More importantly — encoded is not the same as encrypted. Anyone can decode the JWT payload. Only the signature is hashed with a secret key.

---

## Model Answer
Our React app uses a feature-based folder structure — src contains Modules, commons, hooks, and context folders. Each module is self-contained with its own components and pages, which makes the codebase easier to scale and maintain. TypeScript interfaces are defined centrally for shared types and at component level for local types. Global state is managed with Context API. For route protection, we have a ProtectedRoute component that checks if a valid token exists in context or cookies — if yes it renders the child, if no it redirects to login using React Router's Navigate component. JWT tokens in HttpOnly cookies are automatically sent with every request by the browser — we don't manually attach them. For non-cookie approaches we would use an Axios interceptor to attach the Authorization header.

---

## Key Things to Remember

**Folder structure pattern — feature-based architecture:**
```
src/
├── modules/          — feature modules, each self-contained
│   ├── components/
│   ├── pages/
│   └── commonComponent/
├── commons/          — shared across modules
├── hooks/            — custom React hooks
└── context/          — global state via Context API
```

**ProtectedRoute pattern:**
```jsx
const ProtectedRoute = ({ children }) => {
  const { user } = useContext(AuthContext)
  if (!user) return <Navigate to="/login" />
  return children
}

// Usage
<Route path="/dashboard" element={
  <ProtectedRoute>
    <Dashboard />
  </ProtectedRoute>
} />
```

**JWT is encoded not encrypted:**
- Header + Payload = base64 encoded — anyone can decode and read
- Signature = hashed with secret key — prevents tampering
- Never store passwords, card numbers, or sensitive data in JWT payload
- Only store non-sensitive identifiers like userId, role, email

**Context API limitation:**
- Every time context value changes, every component consuming that context re-renders
- Fine for small-medium apps
- For large apps — split into smaller contexts or use Zustand/Redux for performance-critical state

---

## Things to Improve
- Add role-based access control logic to ProtectedRoute explanation
- Learn how Axios interceptor attaches Authorization header as alternative to cookies
