# NTT Second Round — Interview Q&A (Node.js / React)

A revision-ready set of answers for the technical round questions.

---

## 1. How do you integrate 3rd party APIs in Node.js? How do you write a proxy and handle certificates?

**Basic integration**
- Use `axios` or native `fetch`/`https` module to call third-party REST APIs.
- Keep API keys/secrets in `.env` (never hardcode) and load with `dotenv`.
- Wrap calls in a service/client module (`services/paymentApi.js`) so the rest of the app doesn't know API details.
- Add timeout, retry (e.g. `axios-retry`), and centralized error handling.

**Proxy setup**
- If the frontend can't call the third-party API directly (CORS, hidden keys), create an Express route that acts as a **proxy**:
  ```js
  app.use('/api/proxy', createProxyMiddleware({
    target: 'https://thirdparty.com',
    changeOrigin: true,
    pathRewrite: { '^/api/proxy': '' },
  }));
  ```
- `http-proxy-middleware` is commonly used for this.
- Benefits: hides API keys from the client, lets you add auth/rate-limiting/logging in one place, avoids CORS issues.

**Certificates (mTLS / HTTPS)**
- Some enterprise APIs require **mutual TLS** — client certificate + key.
- In Node, pass a custom `https.Agent` with the cert:
  ```js
  const agent = new https.Agent({
    cert: fs.readFileSync('client-cert.pem'),
    key: fs.readFileSync('client-key.pem'),
    ca: fs.readFileSync('ca-cert.pem'),
  });
  axios.get(url, { httpsAgent: agent });
  ```
- Store certs securely (secrets manager / vault, not in repo).
- For self-signed certs in non-prod, set `rejectUnauthorized: false` **only in dev**, never in production.

---

## 2. How would you debug and fix issues on frontend as well as backend/server side?

**Frontend**
- Reproduce the issue, check browser **Console** and **Network tab** (status codes, payload, response).
- Use React DevTools to inspect component tree, props/state, and re-render causes.
- Add breakpoints via browser DevTools (`debugger;` statement or Sources tab).
- Check for console warnings (key props, deprecated lifecycle, hook rule violations).
- Use `console.log`/`console.table` strategically, or React error boundaries to catch render crashes.

**Backend**
- Check server logs (structured logging with `winston`/`pino`).
- Use Node's built-in debugger (`node --inspect`) or VS Code debugger with breakpoints.
- Test endpoints in isolation with Postman/Insomnia/curl to rule out frontend vs backend.
- Check DB queries, middleware order, and error stack traces.
- Add try/catch + centralized error-handling middleware in Express.
- Use monitoring/APM tools (New Relic, Datadog, Sentry) in production for real-time error tracking.

**General approach**
1. Reproduce reliably.
2. Isolate — is it frontend, backend, network, or data issue? (Check Network tab first — this instantly tells you which side.)
3. Narrow down with logs/breakpoints.
4. Fix, then write a test to prevent regression.
5. Verify in the environment closest to production.

---

## 3. What things would you consider before writing code for a React application?

- **Project structure**: feature-based or atomic folder structure (components, hooks, services, utils, pages).
- **State management**: local state vs Context vs Redux/Zustand/React Query — decide based on scope and how much is server vs client state.
- **Component design**: keep components small, reusable, and presentational vs container separation.
- **Performance**: plan for memoization (`useMemo`, `useCallback`, `React.memo`), code-splitting (`React.lazy` + `Suspense`), avoiding unnecessary re-renders.
- **Routing**: React Router structure, protected/private routes.
- **API layer**: centralized API service (axios instance with interceptors), error handling, loading states.
- **Styling approach**: CSS modules, Tailwind, styled-components — pick one and stay consistent.
- **Accessibility (a11y)**: semantic HTML, ARIA attributes, keyboard navigation.
- **Error handling**: Error boundaries for UI crashes, fallback UIs.
- **Environment config**: `.env` files for different environments (dev/stage/prod).
- **Testing strategy**: unit tests (Jest/RTL), and what needs coverage.
- **Linting/formatting**: ESLint + Prettier set up from day one.
- **Security**: sanitize user input, avoid `dangerouslySetInnerHTML` unless necessary.

---

## 4. How would you fix ESLint errors and other security concerns on the frontend?

**ESLint errors**
- Run `eslint . --fix` for auto-fixable issues (spacing, quotes, unused imports).
- For issues that aren't auto-fixable (e.g., missing dependency in `useEffect`, unused variables), fix manually based on the rule's intent rather than disabling it.
- Only use `// eslint-disable-next-line` as a last resort, with a comment explaining why.
- Keep a shared `.eslintrc` with rules like `eslint-plugin-react-hooks`, `eslint-plugin-react`, `eslint-plugin-jsx-a11y`.

**Security concerns**
- Run `npm audit` / `yarn audit` regularly and update vulnerable dependencies.
- Use `eslint-plugin-security` (mainly for Node, but good practice) and `eslint-plugin-react/no-danger` to catch risky patterns.
- Avoid XSS: never inject raw HTML/user input via `dangerouslySetInnerHTML` without sanitizing (e.g., `DOMPurify`).
- Store no secrets/API keys in frontend code — only public/safe config.
- Set proper **CSP (Content-Security-Policy)** headers.
- Validate and sanitize all user inputs on both client and server (client-side validation is UX, not security).
- Use HTTPS everywhere, set `Secure`/`HttpOnly`/`SameSite` flags on cookies.
- Add Husky + lint-staged so lint/security checks run before every commit.

---

## 5. How would you check if the JWT access token sent by the client is valid? Do you generate it or is it generated by default?

**Who generates it**
- JWTs are **not generated automatically** — your backend (auth server/login endpoint) generates them explicitly after verifying user credentials, using a library like `jsonwebtoken`:
  ```js
  const token = jwt.sign({ userId: user.id, role: user.role }, SECRET_KEY, { expiresIn: '15m' });
  ```

**How to validate an incoming token**
- On protected routes, use middleware that calls `jwt.verify()`:
  ```js
  const decoded = jwt.verify(token, SECRET_KEY); // throws if invalid/expired
  ```
- This checks:
  - **Signature** — was it signed with your secret/private key (not tampered with)?
  - **Expiry (`exp`)** — is it still valid?
  - **Issuer/Audience (`iss`/`aud`)** — optional extra claims to confirm it came from the right source.
- If verification fails (bad signature, expired, malformed), respond with `401 Unauthorized`.
- Extract the token from the `Authorization: Bearer <token>` header (or an HttpOnly cookie for better XSS protection).
- Typical flow: short-lived **access token** + long-lived **refresh token**, so when access token expires, the client silently gets a new one via the refresh token without re-login.

---

## 6. When migrating React 18 → React 19 (or any major migration), what issues can occur?

- **Removed/changed APIs**: `ReactDOM.render` is removed in favor of `createRoot` (this actually started in 18, but fully enforced by 19) — old code using legacy render APIs breaks.
- **PropTypes and legacy context**: `propTypes` and `defaultProps` on function components are deprecated/removed — need to migrate to TypeScript or other validation.
- **New JSX transform requirements**: build tooling (Babel/webpack config) may need updates.
- **`ref` as a regular prop**: React 19 allows passing `ref` directly without `forwardRef` — old `forwardRef`-heavy code still works but new patterns emerge, causing inconsistency if not standardized.
- **Third-party library incompatibility**: libraries not yet updated for the new version (state management, UI kits) may throw errors or warnings.
- **Stricter Suspense/concurrent rendering behavior**: components not designed for concurrent rendering may show flickering or inconsistent state.
- **Double-invoked effects in Strict Mode** (dev only): `useEffect` may run twice, exposing bugs in code that assumed single execution — needs proper cleanup functions.
- **New hooks introduced** (`use`, `useFormStatus`, `useOptimistic` in 19): existing custom hooks with similar names could conflict.
- **Testing library versions**: `@testing-library/react` and `react-test-renderer` may need version bumps to remain compatible.
- **General migration approach**: read the official upgrade/codemod guide, run official codemods where available, upgrade in a branch, run full test suite, fix warnings before they become errors, upgrade dependencies incrementally rather than all at once.

---

## 7. Can we use `useParams` and path variables together? What are they and when do you use them?

- **Path variable**: a dynamic segment defined in a route path, e.g.
  ```jsx
  <Route path="/user/:userId" element={<UserProfile />} />
  ```
  Here `:userId` is the path variable/parameter.

- **`useParams`**: a React Router hook used **inside the component rendered by that route** to read the values of those path variables:
  ```jsx
  import { useParams } from 'react-router-dom';

  function UserProfile() {
    const { userId } = useParams();
    // fetch user data using userId
  }
  ```

- **Yes, they're used together by design** — the path variable is *defined* in the route config, and `useParams` is *how you read* it in the component. One doesn't work meaningfully without the other.

- **When to use**: whenever a component needs data from the URL itself — e.g., `/product/:productId`, `/blog/:slug`, `/order/:orderId/invoice` — typically for detail pages, edit pages, or any view scoped to a specific resource identified in the URL (as opposed to query params like `?sort=asc`, which are for filters/optional state and read via `useSearchParams`).

---
