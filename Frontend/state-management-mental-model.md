# State Management — Redux vs React Query vs Context vs Local State

## Question
How do you decide when to use Redux vs React Query vs local state vs useContext? Walk me through your mental model.

---

## My Answer
For big enterprise applications where we want to control data from a central state used everywhere — Redux. For values that stay local to a component — local state. useContext is another centralized state, good for things like multilanguage or auth that the whole app needs.

## What I Got Right
- Correct understanding of Redux for global/central state
- Good real-world examples for useContext — multilanguage, auth
- Clean explanation of local state

## What I Got Wrong
- Completely missed React Query — it's on my resume. Big red flag.
- No tradeoffs mentioned — Redux has boilerplate, Context has re-render issues at scale
- React Query is specifically for server state, not UI state — this distinction is the core of the mental model
- No decision framework — seniors have a clear mental model, not just "it depends"

---

## Model Answer
I think of state in three buckets — server state, global UI state, and local UI state.

For server state — anything fetched from an API — React Query is the right tool. It handles caching, background refetching, loading/error states, and stale data management out of the box. There's no reason to put API data into Redux anymore.

For global UI state that's purely client-side — user preferences, complex shared UI logic, things that multiple unrelated components need — Redux. It gives predictable updates, great DevTools, and middleware support for side effects.

For component-level things that don't need to leave the component — local useState or useReducer.

For Context — I use it sparingly. Only for truly static or very slow-changing values like theme or auth user object. The reason: any Context value change re-renders every single consumer, even components that don't use the changed value. At scale this becomes a performance problem.

---

## Key Things to Remember

**The three state buckets:**
| Type | Tool | Example |
|---|---|---|
| Server state | React Query | Products list, user profile from API |
| Global UI state | Redux | Sidebar open, selected filters, cart |
| Local UI state | useState / useReducer | Form input, modal open/closed |
| Slow-changing globals | Context | Theme, auth user, language |

**Why Context re-renders are dangerous at scale:**
- Every component wrapped in a Context consumer re-renders when the context value changes
- Even if that component doesn't use the specific value that changed
- Solution: split into multiple small contexts, or use Redux/Zustand for high-frequency updates

**React Query vs Redux for API data:**
- Redux for API data = you manage loading, error, cache, refetch manually
- React Query = all of that handled for you + background sync + stale-while-revalidate
- Rule of thumb: if data lives on a server, React Query owns it

**Memory trick:**
- "Does it come from an API?" → React Query
- "Do 5+ unrelated components need it?" → Redux
- "Does only this component need it?" → useState
- "Does it rarely change and whole app needs it?" → Context

---

## Things to Improve
- Never leave React Query out of state management answers — it's the modern default for server state
- Practice explaining the Context re-render problem clearly — it's a senior signal
- Learn Zustand as a lighter Redux alternative — increasingly common in interviews
