# ⚛️ What is React Fiber?

## Simple Explanation

Before Fiber, React updated the entire DOM tree in one go — it couldn't pause or stop in the middle. If you had a huge component tree, it would block the main thread and your UI would freeze.

**Fiber is React's internal engine rewrite** that makes rendering interruptible.

```
Old React (before Fiber):
"I will update ALL components right now,
 nobody can interrupt me" → UI freezes 😤

React Fiber:
"I'll update components in small chunks,
 and pause if something more urgent comes up" → Smooth UI 😊
```

---

## How It Works — Two Phases

```
Phase 1 — Render Phase (can be paused/interrupted)
  → Figures out WHAT needs to change
  → Runs in background
  → Can be thrown away and restarted

Phase 2 — Commit Phase (cannot be interrupted)
  → Actually updates the real DOM
  → Runs all at once — must finish
```

---

## What Fiber Enables

| Feature | What It Does |
|---------|-------------|
| Concurrent Mode | React can work on multiple things at once |
| Suspense | Pause rendering while waiting for data |
| useTransition | Mark some updates as non-urgent |
| Time Slicing | Split work across multiple frames |

---

## 🔑 Key Points To Remember
- Fiber = React's internal reconciler rewrite (released in React 16)
- Breaks rendering into small units called "fibers" — one per component
- Render phase can be paused, restarted, or thrown away
- Commit phase always runs to completion — never interrupted
- Foundation for all React 18 concurrent features
