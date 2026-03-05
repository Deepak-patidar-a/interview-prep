# React Virtualization — How It Works Under the Hood

## Question
You have React-Virtualization on your resume. Explain how virtualization works under the hood, and when would you NOT use it?

---

## My Answer
React Virtualization is perfect for rendering thousands of products on a landing page. It shows 10-20 products in the visible window, fetches more from backend as you scroll, enhancing performance. When not to use it: when there aren't many products to show, or when pagination is a better fit. Also react-virtualized adds to bundle size. Scroll-based landing pages are the classic use case.

## What I Got Right
- Correct use case — large lists, scroll-based pages
- Good "when not to use" instinct — pagination as alternative, bundle size tradeoff
- Clearly used it in production

## What I Got Wrong
- Described infinite scroll / pagination, NOT virtualization. These are completely different things.
- Virtualization does NOT fetch from backend in chunks — the data can all already be in memory
- Missed the core mechanism: fixed-size DOM window, node reuse, calculated container height
- This is a critical gap because the question specifically asked "how it works under the hood"

---

## Model Answer
Virtualization works by maintaining a fixed-size DOM window — say 20 rows — regardless of how much data you have. As you scroll, the library continuously repositions and reuses those same 20 DOM nodes, swapping in new data values. So whether you have 100 or 100,000 rows, the browser only ever paints and manages 20 DOM nodes at any time.

The trick is an invisible container div with a calculated total height — if each row is 50px and you have 10,000 rows, the container is 500,000px tall. This makes the scrollbar look and behave correctly. Only the visible slice within this container is actually rendered in the DOM.

When NOT to use it: when the list is short enough that normal rendering is fine (under ~200 items). When items have highly variable heights — fixed height assumption breaks. When you need SEO indexing of list content — unrendered rows aren't in the DOM so crawlers miss them. When pagination is a better UX choice for the use case.

---

## Key Things to Remember

**The three things people confuse:**
| Technique | What it does | Data in memory? | DOM nodes |
|---|---|---|---|
| Pagination | Fetches less data per page | No — paginated | Only current page |
| Infinite scroll | Fetches more data as you scroll | Grows over time | Grows over time |
| Virtualization | Renders only visible rows | Yes — all loaded | Fixed window (e.g. 20) |

**How virtualization works — the mechanism:**
1. Calculate total container height = item count × row height
2. Set container to that height so scrollbar is correct
3. On scroll, calculate which items are in the visible viewport
4. Render only those items, positioned absolutely within the container
5. Reuse the same DOM nodes — just update their content and position

**Why this matters for performance:**
- 10,000 DOM nodes = browser layout/paint thrashing
- 20 DOM nodes = browser barely notices
- Memory stays lean, scroll stays smooth

**Libraries:**
- `react-window` — lightweight, recommended for most cases
- `react-virtualized` — more features, heavier
- TanStack Virtual — newest, framework agnostic

**When NOT to use:**
- Short lists (< 200 items)
- Variable row heights without measurement
- SEO-critical content
- When pagination is better UX

**Memory trick:** Virtualization = movie set. Only build the rooms the camera can see. Off-screen rooms are fake walls.

---

## Things to Improve
- Be very clear on the distinction: virtualization ≠ infinite scroll ≠ pagination
- Practice explaining the "invisible tall container" mechanism — that's what under the hood means
- Know react-window vs react-virtualized difference — react-window is the modern lighter choice
