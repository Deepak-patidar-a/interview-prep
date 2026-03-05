# Diagnosing React Performance Problems — Step by Step

## Question
If I open your enterprise app and it feels slow — how do you diagnose and fix a React performance problem? Walk me through your exact process step by step.

---

## My Answer
First use React Profiler and DevTools to check which component is causing slowness. Check if we lazy loaded components not visible at first load, and if we used code splitting. Check bundle size using bundle analyzer — run build locally to see size per library. fetch gives less bundle size than axios. framer-motion increases bundle size. For multilingual apps, move language files to the public folder and serve from backend.

## What I Got Right
- Started with profiler first — not guessing, diagnosing. Correct approach.
- Bundle size analysis — most mid-level engineers never think about this
- Lazy loading + code splitting shows thinking about initial load performance
- fetch vs axios bundle tradeoff — practical, real experience
- Language files in public folder — specific, clearly something I've actually done

## What I Got Wrong
- After opening profiler, didn't explain what to look for — flamegraph, wasted renders, commit phase vs render phase
- Missed network tab — waterfall analysis, API response times, large payloads
- Missed virtualization — React-Virtualization is on my resume, and for large tables it's the biggest win
- No mention of Lighthouse for overall page score
- No structured layer-by-layer process

---

## Model Answer
I follow three diagnostic layers — render, bundle, and network.

**Layer 1 — Render:** Open React DevTools Profiler. Record an interaction. Look at the flamegraph for long commit bars and wasted renders — components highlighted in yellow that re-rendered without any visible change. Fix with memoization, useMemo, useCallback. For long lists with thousands of rows, virtualization is the first thing I reach for — render only visible DOM nodes, not all 10,000.

**Layer 2 — Bundle:** Run webpack-bundle-analyzer. Look for unexpectedly large libraries. Check if heavy libraries have lighter alternatives. Ensure lazy loading and code splitting are in place for routes and non-critical components. Check if tree shaking is working.

**Layer 3 — Network:** Open DevTools Network tab. Check the waterfall — are requests sequential when they could be parallel? Are API payloads bloated? Is the initial HTML arriving fast? Run Lighthouse for an overall score with specific recommendations.

Fix in order of highest impact. Usually render issues give the most immediate UX improvement.

---

## Key Things to Remember

**Three diagnostic layers:**
1. **Render** → React Profiler, wasted re-renders, virtualization
2. **Bundle** → webpack-bundle-analyzer, lazy loading, code splitting
3. **Network** → DevTools Network tab, Lighthouse, API payload size

**React Profiler — what to look for:**
- Flamegraph: wide bars = slow components
- "Why did this render?" feature in Profiler
- Commit phase vs render phase — commit phase is the expensive one
- Grey components = did not re-render (good). Colored = re-rendered.

**Virtualization vs Pagination vs Infinite Scroll:**
- **Pagination** = fetch less data from server per page
- **Infinite scroll** = fetch more data as user scrolls
- **Virtualization** = ALL data in memory, but only visible rows in DOM
- These are three different solutions to three different problems

**Bundle size quick wins:**
- Replace moment.js → date-fns (10x smaller)
- Replace axios → fetch for simple cases
- Dynamic import heavy components: `const Chart = lazy(() => import('./Chart'))`
- Move language/config files to public folder — not bundled

**Memory trick:** Render → Bundle → Network. RBN. "Really Bad Network" — ironically it's often the render that's the problem.

---

## Things to Improve
- Practice opening React Profiler and reading a flamegraph — do it on a real project
- Always mention virtualization for large data tables — it's on the resume, own it
- Learn to read a webpack-bundle-analyzer output and identify what to cut
