# HTML & CSS Interview Questions — For 6 Years Experience

At this level, interviewers rarely ask "what is a div." They ask *why* things work the way they do, how the browser handles them, and how you'd debug/optimize them. Below are the most common ones, grouped by theme, with concise answers.

---

## 1. HTML — Semantics & Accessibility

**Q1. Why does semantic HTML matter, and give examples beyond `div`/`span`?**
Semantic tags (`<header>`, `<nav>`, `<main>`, `<article>`, `<section>`, `<aside>`, `<footer>`, `<figure>`, `<time>`) tell the browser and assistive tech what content *means*, not just how it looks. Benefits: better SEO (crawlers weight semantic structure), better accessibility (screen readers use landmarks to let users jump around), and more maintainable code for other devs.

**Q2. Difference between `<section>`, `<article>`, and `<div>`?**
- `<article>` — self-contained content that could stand alone (a blog post, a card in a feed).
- `<section>` — a thematic grouping of content, usually with a heading, part of a larger document.
- `<div>` — no semantic meaning, purely a styling/layout hook.

**Q3. How do you make a custom component (like a dropdown) accessible?**
Use proper ARIA roles/attributes (`role="listbox"`, `aria-expanded`, `aria-haspopup`), ensure keyboard operability (Tab, Enter, Escape, Arrow keys), manage focus programmatically, and make sure it works with screen readers (test with VoiceOver/NVDA). Prefer native elements (`<select>`, `<button>`) over custom-built ones whenever possible — "no ARIA is better than bad ARIA."

**Q4. What's the difference between `id` and `class`, semantically and in terms of ARIA/accessibility use?**
`id` is unique per page and used for direct targeting (`aria-labelledby`, `aria-describedby`, fragment links, JS hooks). `class` applies to multiple elements, mainly for styling and grouping. Overusing IDs for styling causes specificity issues (see CSS section).

**Q5. What are `defer` and `async` on `<script>` tags, and when do you use each?**
- No attribute: HTML parsing blocks until script downloads and executes.
- `async`: downloads in parallel, executes as soon as ready (order not guaranteed) — good for independent scripts like analytics.
- `defer`: downloads in parallel, executes after HTML parsing completes, in order — good for scripts that depend on the DOM or on each other.

**Q6. What is the Shadow DOM and when would you use it?**
An encapsulated DOM subtree attached to an element, whose styles and markup don't leak in or out. Used in Web Components to build reusable, style-isolated widgets (e.g. a `<video>` player's native controls are Shadow DOM under the hood).

---

## 2. CSS — Core Mechanics

**Q7. Explain the CSS box model, including `box-sizing`.**
Every element is a box: `content` → `padding` → `border` → `margin`.
- `box-sizing: content-box` (default): width/height apply only to content; padding/border add on top.
- `box-sizing: border-box`: width/height include padding and border — makes sizing predictable, which is why most teams set `* { box-sizing: border-box; }` globally.

**Q8. Explain CSS specificity and how conflicts are resolved.**
Specificity is calculated as a tuple: (inline styles, IDs, classes/attributes/pseudo-classes, elements/pseudo-elements). Higher tuple wins; if equal, the later rule in source order wins. `!important` overrides normal specificity (use sparingly — it breaks the cascade and is hard to override later).
Order (low → high): element selectors → classes/attributes/pseudo-classes → IDs → inline styles → `!important`.

**Q9. What is the difference between `visibility: hidden`, `display: none`, and `opacity: 0`?**
- `display: none` — removed from layout entirely, no space taken, not accessible.
- `visibility: hidden` — invisible but still takes up space in layout; children can override with `visibility: visible`.
- `opacity: 0` — invisible but still takes space *and* is still interactive/clickable and in the accessibility tree (needs `pointer-events: none` + `aria-hidden` if you want it fully hidden).

**Q10. Explain stacking contexts and `z-index`.**
`z-index` only works on positioned elements (`position` not `static`) or flex/grid items, and it's relative *within* a stacking context, not globally. New stacking contexts are created by things like `position: fixed/sticky`, `opacity < 1`, `transform`, `filter`, `will-change`, or `isolation: isolate`. This is why a child with `z-index: 9999` can still appear behind another element — its parent's stacking context caps it.

**Q11. What causes reflow and repaint, and how do you minimize them?**
- **Reflow (layout)**: geometry changes (width, height, position, adding/removing DOM nodes) force the browser to recalculate layout — expensive.
- **Repaint**: visual-only changes (color, background) without layout changes — cheaper.
- **Composite-only** (cheapest): `transform` and `opacity` changes can be handled purely on the GPU compositor thread without touching layout or paint.
Optimization: animate `transform`/`opacity` instead of `top`/`left`/`width`, batch DOM reads/writes, use `will-change` sparingly, avoid layout thrashing (reading layout properties like `offsetHeight` right after writing styles in a loop).

---

## 3. CSS — Layout

**Q12. Flexbox vs Grid — when do you use which?**
- **Flexbox**: one-dimensional (row *or* column) — great for nav bars, toolbars, aligning items within a container.
- **Grid**: two-dimensional (rows *and* columns simultaneously) — great for full page layouts, card grids, complex alignment where both axes matter.
They're often combined: Grid for overall page structure, Flexbox for aligning content inside individual grid areas.

**Q13. Explain `position: sticky` and common pitfalls.**
`sticky` behaves like `relative` until the scroll position hits a threshold (`top`, `left`, etc.), then behaves like `fixed` within its containing block. Pitfalls: it won't work if a parent has `overflow: hidden/auto/scroll`, or if the parent's height equals the sticky element's height (no room to "stick").

**Q14. How does `float` work, and why is it rarely used for layout today?**
Floated elements are taken out of normal flow horizontally, letting text/inline content wrap around them; the parent doesn't grow to contain them ("collapsing" — historically fixed with clearfix hacks). Modern layout uses Flexbox/Grid instead since they're purpose-built and don't need clearfix workarounds; `float` is now mainly used for its original purpose — wrapping text around images.

**Q15. What's the difference between `auto`, `fr`, `%`, and `minmax()` in CSS Grid?**
- `auto` — sizes to content.
- `fr` — a fraction of remaining free space after fixed-size tracks are accounted for.
- `%` — relative to the grid container.
- `minmax(min, max)` — sets a range, e.g. `minmax(200px, 1fr)` for responsive columns without media queries.

**Q16. How would you center a div, and what are the different ways?**
- Flexbox: `display:flex; justify-content:center; align-items:center;`
- Grid: `display:grid; place-items:center;`
- Absolute positioning: `position:absolute; top:50%; left:50%; transform:translate(-50%,-50%);`
- Margin auto (horizontal only, for block elements with defined width): `margin: 0 auto;`

---

## 4. CSS — Responsive & Architecture

**Q17. Mobile-first vs desktop-first — which do you prefer and why?**
Mobile-first (`min-width` media queries, base styles for small screens) generally leads to leaner CSS since you're progressively *enhancing* rather than overriding desktop styles for smaller screens. It also matches how most traffic and Google's indexing prioritize mobile today.

**Q18. Explain CSS custom properties (variables) and how they differ from Sass variables.**
`--main-color: #333;` used via `var(--main-color)`. Unlike Sass variables (compile-time, static), CSS custom properties are live in the browser — they can be changed at runtime with JS, cascade and inherit like normal CSS properties, and can be scoped/overridden per component (e.g. theming with `[data-theme="dark"] { --bg: #000; }`).

**Q19. What is BEM and why use a naming methodology at all?**
Block__Element--Modifier (`.card__title--highlighted`). It keeps specificity flat and predictable (mostly single-class selectors), avoids naming collisions, and makes it obvious from the class name alone what a piece of markup relates to — helpful at scale and in team codebases.

**Q20. What are container queries and how are they different from media queries?**
Media queries respond to the *viewport* size; container queries (`@container`) respond to the size of a containing element itself, letting a component adapt based on the space it's actually given (e.g. a card that's narrow in a sidebar but wide in the main content) — much better for truly reusable components.

---

## 5. CSS — Performance & Modern Features

**Q21. What's the critical rendering path, and how does CSS block it?**
Browser builds the DOM (from HTML) and CSSOM (from CSS), combines them into the render tree, then does layout and paint. CSS is render-blocking by default — the browser won't paint until it has the full CSSOM, so large/unoptimized stylesheets delay first paint. Mitigations: inline critical above-the-fold CSS, defer non-critical CSS, minimize/split stylesheets.

**Q22. How do you approach optimizing a slow CSS animation?**
Animate only `transform` and `opacity` (compositor-only, GPU-accelerated), avoid animating `width`/`height`/`top`/`left`/`box-shadow` which trigger layout/paint, use `will-change` to hint the browser (but remove it after, since it consumes memory), and check DevTools' Performance/Rendering tab for layout thrashing or excessive paint areas.

**Q23. What's the difference between `rem`, `em`, `%`, `vw/vh`, and `px`?**
- `px` — fixed, absolute.
- `em` — relative to the *parent's* font size (compounds when nested).
- `rem` — relative to the *root* (`html`) font size — predictable, doesn't compound.
- `%` — relative to the parent's corresponding property.
- `vw`/`vh` — relative to 1% of viewport width/height — useful for full-bleed/responsive typography.

**Q24. What is the difference between pseudo-classes and pseudo-elements?**
- Pseudo-class (single colon, `:hover`, `:nth-child()`, `:focus`) — targets an element in a particular *state*.
- Pseudo-element (double colon, `::before`, `::after`, `::first-line`) — targets a *sub-part* of an element, or generates content that isn't in the DOM.

**Q25. How would you implement a dark mode toggle purely with CSS?**
Use CSS custom properties for all colors, then flip them with `prefers-color-scheme: dark` media query for automatic OS-based switching, and/or a `[data-theme="dark"]` attribute toggled via JS for a manual user override.

---

### Quick tips for the interview itself
- When asked "what is X," add "and here's when I'd actually use it / avoid it" — that's the senior-level signal.
- Be ready to whiteboard or live-code: centering a div, a responsive card grid, a sticky header, a CSS-only tooltip.
- Have 1–2 real war stories ready: a specificity bug you debugged, a layout shift (CLS) you fixed, an accessibility issue you caught in review.
