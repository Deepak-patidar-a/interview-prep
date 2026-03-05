# React Performance Optimization — Real Example

## Question
You mentioned improving rendering speed by 50% at Blue Yonder. Walk me through exactly what you did — what was the problem, what did you try, and what actually worked?

---

## My Answer
On a product page where we calculate cost based on 4-5 parameters, we were using useEffect with those parameters as dependencies. When any field changed it fired a state change and triggered the useEffect. I implemented useMemo with the same parameters, and instead of state I used useRef. Using these two I improved rendering speed by over 50%.

## What I Got Right
- Real, specific scenario — not a textbook answer
- Correctly identified the problem: unnecessary re-renders on every field change
- Named the right tools: useMemo and useRef
- Had an actual number to back it up (50%)

## What I Got Wrong
- Didn't explain WHY I chose useRef over state — interviewers want the reasoning, not just the decision
- Didn't mention how I measured the 50% improvement — without a profiler reference, numbers sound made up
- Didn't mention other options I considered first — seniors evaluate tradeoffs before picking a solution

---

## Model Answer
We had a cost calculation component with 4-5 derived values all sitting in state. Every keystroke triggered re-renders across the whole component tree. I opened React DevTools Profiler and saw render times of 200ms+ on each keystroke. The root cause was derived values being recalculated and stored as state — triggering re-renders even when the visual output hadn't changed. I replaced derived state with useMemo so values only recalculate when actual dependencies change. For values I needed to track across renders without triggering re-renders, I switched to useRef. After these changes, the profiler showed render times dropped to under 80ms — roughly 50-60% improvement. I also considered moving the calculation outside the component entirely, but useMemo was the right tradeoff since the values were tightly coupled to component state.

---

## Key Things to Remember

**useMemo vs useRef vs useState:**
- `useState` — value change triggers re-render. Use for UI-visible values.
- `useMemo` — caches derived/computed values, only recalculates when deps change. No re-render on its own.
- `useRef` — persists value across renders without causing re-render. Use for tracking, timers, DOM refs.

**How to measure performance:**
- React DevTools Profiler → flamegraph → look for long bars (slow commits)
- Before/after render times in ms is the gold standard
- Lighthouse for page-level metrics
- `console.time()` for quick local checks

**The tradeoff seniors mention:**
- useMemo has a cost too — memory + comparison overhead
- Only use it when the calculation is genuinely expensive or re-renders are visually problematic
- Don't memoize everything blindly

**Memory trick:** State = triggers paint. Memo = skips recalculation. Ref = remembers without repainting.

---

## Things to Improve
- Always open React Profiler before and after any optimization — get the actual numbers
- Practice articulating the "why" before the "what" when explaining technical decisions
- Mention at least one alternative considered, even if you didn't go with it
