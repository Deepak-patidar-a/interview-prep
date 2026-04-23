# 🧪 React Interview Questions & Answers — Part 3
### For 5+ Years Experience | Testing, CI/CD & Leadership

---

## 🧪 Section 6: Testing

---

### Q20. You join a team with 0% test coverage on a large production app. How do you introduce testing without slowing down feature delivery?

**The Wrong Approach:** Stopping all feature work to write tests. Team resists, management pushes back, nothing gets done.

**The Right Approach: "Test as you go" strategy**

---

**Step 1: Establish the baseline (Week 1)**
```bash
# Understand what you're working with
npx jest --coverage
# Set minimum thresholds in jest.config.js
```

```js
// jest.config.js
module.exports = {
  coverageThreshold: {
    global: {
      branches: 10,    // Start low
      functions: 10,
      lines: 10,
    }
  }
};
// Increase this threshold by 5% every sprint
```

**Step 2: The "Boy Scout Rule" — tests on every PR**
```
Team Agreement:
- If you touch a file → write/update tests for it
- New feature → tests are part of the Definition of Done
- Bug fix → write a failing test first, then fix it (regression test)
```

**Step 3: Start with the highest-value tests first**
```
Priority order:
1. Critical user flows (checkout, login, signup) → E2E tests
2. Utility functions and business logic → Unit tests (easy + high ROI)
3. API integration points → Integration tests
4. Complex UI components → Component tests
5. Simple display components → Last priority
```

**Step 4: The Testing Pyramid**

```
         /\
        /E2E\          ← Few (5-10) — Cypress/Playwright
       /------\
      /Integr. \       ← Some (20-30) — Testing Library
     /----------\
    /  Unit Tests \    ← Many (100+) — Jest
   /--------------\
```

**Step 5: Practical examples of where to start**

```jsx
// Start with pure utility functions — easiest to test
// utils/formatCurrency.js
export const formatCurrency = (amount, currency = 'USD') => {
  return new Intl.NumberFormat('en-US', { style: 'currency', currency })
    .format(amount);
};

// utils/formatCurrency.test.js
describe('formatCurrency', () => {
  it('formats positive amounts', () => {
    expect(formatCurrency(1234.5)).toBe('$1,234.50');
  });
  it('handles zero', () => {
    expect(formatCurrency(0)).toBe('$0.00');
  });
  it('handles negative amounts', () => {
    expect(formatCurrency(-50)).toBe('-$50.00');
  });
});
```

```jsx
// Then critical components with React Testing Library
import { render, screen, fireEvent } from '@testing-library/react';
import LoginForm from './LoginForm';

describe('LoginForm', () => {
  it('shows error when submitting empty form', async () => {
    render(<LoginForm />);
    fireEvent.click(screen.getByRole('button', { name: /login/i }));
    expect(await screen.findByText(/email is required/i)).toBeInTheDocument();
  });

  it('calls onSubmit with correct data', async () => {
    const mockSubmit = jest.fn();
    render(<LoginForm onSubmit={mockSubmit} />);

    fireEvent.change(screen.getByLabelText(/email/i), {
      target: { value: 'user@test.com' }
    });
    fireEvent.change(screen.getByLabelText(/password/i), {
      target: { value: 'password123' }
    });
    fireEvent.click(screen.getByRole('button', { name: /login/i }));

    expect(mockSubmit).toHaveBeenCalledWith({
      email: 'user@test.com',
      password: 'password123'
    });
  });
});
```

---

### Q21. Your end-to-end Cypress tests pass locally but randomly fail in CI. How do you investigate and stabilize them?

**Common Causes of Flaky E2E Tests:**

| Cause | Symptom | Fix |
|---|---|---|
| Race conditions | Test clicks before element is ready | Use `cy.findByRole()` with retry |
| Hardcoded waits | `cy.wait(3000)` breaks on slow CI | Replace with `cy.intercept()` |
| Test pollution | Previous test leaves dirty state | Reset DB + clear cookies before each test |
| Viewport differences | Works on 1440px, fails on CI 1024px | Set explicit viewport |
| Date/time sensitive | Tests fail on certain days/times | Mock the system clock |

**Debugging Steps:**

```bash
# 1. Run tests with video recording in CI
cypress run --record --key YOUR_KEY

# 2. Check Cypress Dashboard for failure screenshots/videos

# 3. Run locally with CI environment variables
CI=true npm test
```

**Stabilization Techniques:**

```js
// ❌ Flaky — hardcoded wait
cy.wait(2000);
cy.get('.submit-btn').click();

// ✅ Stable — wait for network request
cy.intercept('POST', '/api/login').as('loginRequest');
cy.get('.submit-btn').click();
cy.wait('@loginRequest'); // Waits for actual API call, not time

// ❌ Flaky — element might not exist yet
cy.get('.user-name').should('have.text', 'John');

// ✅ Stable — retry-able assertion with timeout
cy.findByText('John', { timeout: 10000 }); // Retry for 10 seconds

// ❌ Flaky — tests share state
describe('User tests', () => {
  it('test 1 creates a user');
  it('test 2 assumes that user exists'); // Fails if test 1 is skipped!
});

// ✅ Stable — each test is independent
beforeEach(() => {
  cy.request('POST', '/api/test/reset-db'); // Reset state
  cy.clearCookies();
  cy.clearLocalStorage();
});

// ✅ Mock time-dependent behavior
cy.clock(new Date('2025-01-15').getTime()); // Freeze time
```

---

## 🚀 Section 7: CI/CD & Deployment

---

### Q22. Your React app needs to support feature flags so marketing can toggle features without a deployment. How do you build this?

**Architecture:**

```
Feature Flag Sources:
├── Option A: LaunchDarkly / Flagsmith (paid, managed)
├── Option B: Environment variables (simple, deploy needed to change)
└── Option C: Remote config from your API (free, flexible) ← recommended
```

**Implementation (Remote Config approach):**

```js
// 1. API endpoint returns feature flags for the user
GET /api/features
Response: {
  "newCheckoutFlow": true,
  "darkMode": false,
  "betaDashboard": true
}

// 2. Custom hook to fetch and cache flags
const useFeatureFlags = () => {
  const { data: flags } = useQuery(
    'feature-flags',
    () => api.getFeatureFlags(),
    {
      staleTime: 5 * 60 * 1000, // Cache for 5 minutes
      initialData: { newCheckoutFlow: false } // Safe defaults
    }
  );
  return flags;
};

// 3. FeatureFlag component for clean JSX usage
const FeatureFlag = ({ name, children, fallback = null }) => {
  const flags = useFeatureFlags();
  return flags[name] ? children : fallback;
};

// 4. Usage in components
const CheckoutPage = () => (
  <FeatureFlag name="newCheckoutFlow" fallback={<OldCheckout />}>
    <NewCheckout />
  </FeatureFlag>
);

// 5. Usage in JS logic
const useCheckoutFlow = () => {
  const flags = useFeatureFlags();
  return flags.newCheckoutFlow ? newCheckoutLogic : oldCheckoutLogic;
};
```

**Backend flag targeting (user segments):**
```js
// Flags can be per-user, per-role, per-region
const getFeatureFlags = (user) => ({
  newCheckoutFlow: user.betaTester || user.country === 'US',
  darkMode: user.preferences.darkMode,
  betaDashboard: user.role === 'admin',
});
```

---

### Q23. After a deployment, 2% of users are seeing a white screen. Error monitoring shows `ChunkLoadError`. What happened and how do you fix and prevent it?

**What Happened:**
When you deploy a new version, old JS chunk filenames change (hashed). Users with the old page open try to load new chunks that no longer exist → `ChunkLoadError`.

```
Old deployment: main.abc123.js, vendor.xyz789.js
New deployment: main.def456.js, vendor.qrs012.js
↑ User with OLD page tries to import main.abc123.js → 404 → White screen
```

**Immediate Fix:**
```js
// Catch ChunkLoadErrors globally and force a page refresh
window.addEventListener('error', (event) => {
  if (event.message?.includes('ChunkLoadError') ||
      event.message?.includes('Loading chunk')) {
    window.location.reload(); // Forces fresh page with new chunks
  }
});

// OR with React Error Boundary
class AppErrorBoundary extends React.Component {
  componentDidCatch(error) {
    if (error.name === 'ChunkLoadError') {
      window.location.reload();
    }
  }
}
```

**Prevent via graceful reload prompts:**
```js
// Detect new deployment and prompt user to refresh
const checkForUpdate = async () => {
  const res = await fetch('/version.json'); // Deploy a version.json file
  const { version } = await res.json();

  if (version !== APP_VERSION) {
    // Show non-disruptive toast
    toast('A new version is available. Refresh for the best experience.', {
      action: { label: 'Refresh', onClick: () => window.location.reload() }
    });
  }
};

// Check every 5 minutes while user is on page
useEffect(() => {
  const interval = setInterval(checkForUpdate, 5 * 60 * 1000);
  return () => clearInterval(interval);
}, []);
```

**Long-term Prevention:**
```nginx
# Keep old chunk files for at least 1 hour after deployment
# In nginx — serve old files from /prev/ directory
location /static/js/ {
  try_files $uri /prev/$uri =404;
  expires 1h;
}
```

---

### Q24. You need to deploy the same React app for 10 different clients, each with different themes, features, and API endpoints. How do you manage this?

**Architecture: White-Label Multi-Tenant System**

```
Single codebase → 10 different builds
```

**Step 1: Client config file**
```json
// clients/acme-corp/config.json
{
  "clientId": "acme-corp",
  "apiBaseUrl": "https://api.acme-corp.com",
  "theme": {
    "primaryColor": "#E74C3C",
    "logo": "/assets/acme-logo.png",
    "fontFamily": "Roboto"
  },
  "features": {
    "darkMode": true,
    "advancedReporting": false,
    "ssoEnabled": true
  }
}
```

**Step 2: Build script per client**
```bash
# package.json scripts
{
  "scripts": {
    "build:acme": "CLIENT=acme-corp react-scripts build",
    "build:globex": "CLIENT=globex react-scripts build",
    "build:all": "npm run build:acme && npm run build:globex"
  }
}
```

**Step 3: Config injection**
```js
// src/config.js
const clientConfig = require(`../clients/${process.env.CLIENT}/config.json`);
export default clientConfig;

// Usage anywhere in the app
import config from '@/config';

const App = () => (
  <ThemeProvider theme={config.theme}>
    <ApiProvider baseUrl={config.apiBaseUrl}>
      {/* App content */}
    </ApiProvider>
  </ThemeProvider>
);
```

**Step 4: Feature flags per client**
```jsx
import config from '@/config';

const ReportingPage = () => {
  if (!config.features.advancedReporting) {
    return <BasicReporting />;
  }
  return <AdvancedReporting />;
};
```

**Step 5: CI/CD pipeline**
```yaml
# GitHub Actions — build all clients in parallel
jobs:
  build:
    strategy:
      matrix:
        client: [acme-corp, globex, initech, umbrella] # All 10 clients
    steps:
      - run: npm run build:${{ matrix.client }}
      - run: aws s3 sync build/ s3://${{ matrix.client }}-app/
```

---

## 🤝 Section 8: Leadership & Collaboration

---

### Q25. A backend API your team depends on is poorly designed and causing frontend hacks everywhere. How do you handle this without creating team conflict?

**The Wrong Approach:** Complaining in Slack or going to management immediately. This damages relationships and rarely fixes the problem.

**The Right Approach:**

**Step 1: Document the impact (data, not opinions)**
```
Create a short report:
- List 5 specific examples of frontend workarounds
- Estimate dev hours wasted per sprint (e.g., "3 hours/sprint on data transformation")
- Show concrete examples of technical debt created
```

**Step 2: Short-term mitigation — API adapter layer**
```js
// Instead of spreading hacks everywhere, centralize them
// api/adapters/userAdapter.js
export const adaptUserFromApi = (apiUser) => ({
  // API returns snake_case → frontend expects camelCase
  id: apiUser.user_id,
  fullName: `${apiUser.first_name} ${apiUser.last_name}`,
  // API sends nested address, we need flat
  city: apiUser.address?.city,
  // API sends 0/1, we need boolean
  isActive: apiUser.is_active === 1,
});

// Now only ONE place has the hack, not 50 components
const user = adaptUserFromApi(apiResponse.data);
```

**Step 3: Collaborative conversation with backend team**
```
Frame it as: "How can we work better together?" not "Your API is bad"

Propose:
- A shared API design session (invite both teams)
- Agree on a contract first (OpenAPI spec) before building
- Establish a frontend/backend liaison role
- API versioning so frontend isn't broken on backend changes
```

**Step 4: Escalate only if needed, with data**
```
If direct conversation doesn't work:
- Present the documented impact to the tech lead / engineering manager
- Frame as a business impact, not a blame issue
- Propose solutions, not just problems
```

---

### Q26. Two senior developers strongly disagree on Context API vs Zustand for a new project. How do you resolve this as a lead?

**The Wrong Approach:** Picking a side or letting the debate drag on for weeks.

**The Right Approach: Structured decision process**

**Step 1: Set a time-box**
> "We'll discuss this for 30 minutes and make a decision. After that, we commit and move forward."

**Step 2: Define the criteria first (before arguing solutions)**
```
For this project, we care about:
1. Bundle size impact
2. Learning curve for the team
3. DevTools support
4. Long-term maintainability
5. Performance at our data update frequency
```

**Step 3: Evaluate against criteria (not vibes)**

| Criteria | Context API | Zustand |
|---|---|---|
| Bundle size | 0KB (built-in) | 1.1KB |
| Learning curve | Low (team knows React) | Low (simple API) |
| DevTools | Basic (React DevTools) | Redux DevTools support |
| Re-render control | ❌ All consumers re-render | ✅ Selector-based |
| Async support | Manual | Built-in middleware |

**Step 4: Apply to the specific project**
```
Questions to ask:
- How often does state update? (If rarely → Context fine. If every 500ms → Zustand)
- How large is the team? (Larger teams → Zustand's structure helps)
- What's the complexity of state? (Simple → Context. Complex → Zustand/Redux)
```

**Step 5: Make the call**
```
As a lead:
"Based on our criteria, we'll go with [X] because [specific reasons].
Both options are valid. We're choosing this one to unblock progress.
We'll revisit in 3 months if pain points arise."
```

**Key Leadership Principle:** The best decision is often not the technically perfect one — it's the one the team commits to and executes well.

---

### Q27. A product manager wants a feature delivered in 2 days that you estimate needs 2 weeks. How do you handle this?

**Never say just "No" or "It takes 2 weeks, end of story".**

**Step 1: Understand the real deadline**
```
Ask: "What's driving the 2-day timeline?"
- Hard: Client demo, conference, contractual deadline
- Soft: Internal preference, optimism

This changes everything. Hard deadlines require scope cuts, not estimation magic.
```

**Step 2: Break down what 2 weeks contains**
```
Full feature (2 weeks):
├── Core functionality: 3 days
├── Edge cases & error handling: 2 days
├── Responsive design: 2 days
├── Accessibility (a11y): 1 day
├── Tests: 2 days
├── Code review + fixes: 1 day
└── QA + bug fixes: 3 days
```

**Step 3: Offer a negotiation with options**
```
"I can deliver X in 2 days if we agree to:
Option A (MVP): Core flow only, happy path, no mobile, no error handling
Option B (Partial): 70% of the feature with best quality in 1 week
Option C (Full): Complete feature with tests and edge cases in 2 weeks

The MVP gets you something to demo. We follow up with the rest in the next sprint."
```

**Step 4: Document the trade-offs**
```
Send a short written summary after the conversation:
"As discussed, we're going with Option A. This means we are consciously
skipping: mobile layout, error states, accessibility. These are tracked
in JIRA as tech debt tickets [link] to be completed in Sprint 12."

This protects you AND aligns everyone on expectations.
```

**Key Principle:** Scope is negotiable. Quality and estimation honesty are not.

---

## 📚 Quick Revision Summary

| Topic | Key Concepts |
|---|---|
| Performance | React.memo, useCallback, useMemo, virtualization, code splitting |
| Architecture | Feature slices, data layers, micro-frontends, design systems |
| State | RTK, React Hook Form, Zustand selectors, normalized state |
| Networking | Parallel fetching, React Query invalidation, offline-first, service workers |
| Security | httpOnly cookies, CSP headers, DOMPurify, backend authorization |
| Testing | Testing pyramid, Testing Library, Cypress best practices |
| CI/CD | Feature flags, ChunkLoadError, white-label builds |
| Leadership | Data-driven decisions, structured disagreement, scope negotiation |

---

*✅ All 27 questions covered across 3 files.*
*📄 Part 1 → Performance & Architecture | Part 2 → State & Networking | Part 3 → Testing, CI/CD & Leadership*
