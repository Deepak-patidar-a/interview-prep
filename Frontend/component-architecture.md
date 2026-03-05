# Component Architecture — Reusable vs Specific

## Question
You mentioned building 20+ enterprise screens. How did you approach component architecture — how did you decide what to make reusable vs what to keep specific?

---

## My Answer
I divided into three parts: filter fields, form fields, and tables. We created dynamic filter fields where you can add product/vendor data and search to fetch details. We created dynamic column tables which can be used anywhere with different numbers of columns. For forms, we created dynamic input field components reusable anywhere. For specific needs that are totally different pages/components, we kept them specific. This gives reusability and scalability.

## What I Got Right
- Real answer from actual work — not theoretical
- Three-bucket approach (filters, tables, forms) shows systematic thinking
- Dynamic columns table and dynamic input fields — genuine reusability
- Knowing when NOT to reuse is actually a senior trait — "kept specific for unique pages"

## What I Got Wrong
- Didn't explain the abstraction boundary — at what point does a component become too generic and unmaintainable?
- No mention of props design — how did I structure the API of these reusable components? Render props? Config objects? Compound components?
- Didn't mention team adoption — did other engineers actually use these components? Impact matters.

---

## Model Answer
We identified three categories of UI that repeated across all 20+ screens — filters, data tables, and forms. For each, we built a config-driven reusable component. The table, for example, accepted a columns definition array — each entry describing the key, header label, render function, sort behavior, and width. This kept the component clean and dumb while giving consumers full flexibility without touching the component internals. Forms took a field config array with validation rules baked in. The hardest part was finding the right abstraction level. Too specific and you have duplication. Too generic and the component becomes a god component nobody wants to touch. Our rule: if a pattern appears 3+ times across screens, it's worth abstracting. If it's unique to one screen, keep it local. Once we had these primitives, new screens took a fraction of the time to build because engineers were composing, not rewriting.

---

## Key Things to Remember

**The abstraction rule of three:**
- Appears once → write it inline
- Appears twice → note it, might be a coincidence
- Appears three times → abstract it into a reusable component

**Component API design patterns:**
- **Config/data-driven** — pass a config array that describes behavior. Best for tables, forms, filters.
- **Render props** — pass a function as prop that returns JSX. Best for flexible layouts.
- **Compound components** — `<Table><Table.Header /><Table.Body /></Table>`. Best for complex components with related parts.
- **Composition** — wrap and extend rather than adding props endlessly.

**Signs a component is too generic:**
- It has 20+ props
- Reading the props list doesn't tell you what the component does
- Engineers avoid using it because it's hard to configure
- You need to read the source to understand how to use it

**Signs a component is too specific:**
- Copy-pasted with minor changes in 3+ places
- Small requirement change forces updating multiple components

**Memory trick:** Components are like tools. A hammer is reusable. A custom bracket for one specific shelf is specific. Build hammers, not brackets.

---

## Things to Improve
- Learn compound component pattern deeply — it's a common senior interview topic
- Be ready to describe the props API of your reusable table component in detail
- Mention team adoption and time saved when talking about reusable components — that's the impact
