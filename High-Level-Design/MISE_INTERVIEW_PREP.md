# 🍽️ Mise — Interview Preparation Guide

> Everything you need to confidently explain this project in any interview.
> Read this the night before. Know it cold.

---

## ⚡ The One-Line Pitch

> *"Mise is a full-stack AI recipe SaaS I built from scratch — users can snap a photo of their fridge, the app identifies ingredients using Google Gemini Vision, and generates personalised step-by-step recipes using OpenAI. It's deployed on Vercel and scores 99 on Lighthouse mobile performance."*

---

## 🎯 Why I Built It

> *"I wanted to build something beyond a typical CRUD app that demonstrates production-level thinking. I chose a recipe platform because it forced me to solve real problems simultaneously — multiple data sources, expensive AI API calls, image processing, rate limiting, authentication, and performance. I also wanted to learn Next.js 15 App Router properly — not just follow a tutorial — so I made deliberate architectural decisions and understood the why behind each one."*

---

## 🔢 Numbers to Know Cold

| Metric | Number |
|---|---|
| Lighthouse Performance (Mobile) | **99** |
| Lighthouse Performance (Desktop) | **95** |
| Lighthouse SEO | **100** |
| Lighthouse Best Practices | **100** |
| AI models integrated | **2** (OpenAI GPT-4o-mini + Google Gemini Vision) |
| External APIs consumed | **4** (MealDB, Unsplash, OpenAI, Gemini) |
| Database tables | **4** (User, Recipe, PantryItem, SavedRecipe) |
| Rate limit tiers | **2** (Free 3/day, Pro unlimited) |

---

## 🏗️ Tech Stack — Know What Each Does

### Frontend
| Tech | Why I used it |
|---|---|
| Next.js 15 App Router | Server components, routing, ISR, server actions |
| TypeScript (strict) | Type safety from DB to UI — catches bugs at compile time |
| Tailwind CSS v4 | Custom warm theme with oklch color space |
| Shadcn UI | Pre-built accessible components (Dialog, Tabs, Badge, Card) |
| Zustand | Client state for optimistic UI (save/unsave) |
| React Query | Client-side data fetching with caching |

### Backend & Database
| Tech | Why I used it |
|---|---|
| Next.js Server Actions | Server-side mutations — API key never touches browser |
| PostgreSQL (Neon) | Serverless DB — scales to zero, perfect for this stage |
| Prisma v7 | Type-safe ORM — TypeScript types auto-generated from schema |
| Upstash Redis | Serverless rate limiting — sliding window algorithm |

### AI & External
| Tech | Why I used it |
|---|---|
| OpenAI GPT-4o-mini | Recipe generation and pantry suggestions |
| Google Gemini Vision | Image scanning — identifies ingredients from fridge photos |
| MealDB API | 50,000+ real recipes for the explore feature |
| Unsplash API | High-quality food photography for AI-generated recipes |
| Clerk | Authentication — handles OAuth, sessions, security |

---

## 🔑 Key Features — How to Explain Each

---

### 1. AI Pantry Scanner

**What it does:**
User uploads a fridge photo → Gemini Vision identifies up to 20 ingredients with confidence scores → user reviews and saves to their PostgreSQL pantry.

**How to explain it:**
> *"The image upload goes to a Next.js Server Action — it never touches the client side, so the Gemini API key is never exposed. I convert the image to base64, send it to Gemini Vision with a carefully written prompt that asks for structured JSON output, parse the response, and return the ingredient list to the UI for the user to review before saving."*

**What makes it interesting:**
> *"I implemented automatic model fallback — if Gemini's primary model is overloaded (which happens), it automatically retries with a secondary model. Only if all models fail does it return an error to the user."*

---

### 2. AI Recipe Generation — Cache-First Pattern

**What it does:**
User requests any recipe by name → check DB first → if found, return instantly → if not, call OpenAI → save to DB → return to user.

**How to explain it:**
> *"The most important design decision here was cache-first. OpenAI costs money. If 100 users all ask for Butter Chicken, I don't want to call OpenAI 100 times. So the first request generates and saves the recipe to PostgreSQL as public — shared across all users. Every subsequent request reads from DB in milliseconds at zero API cost. This is the same pattern CDNs use for static assets."*

**The prompt engineering:**
> *"I engineered the prompt to always return structured JSON with a specific schema — title, ingredients array with categories, step-by-step instructions with optional tips, substitutions, and nutrition. The system message says 'respond with valid JSON only' so I can reliably parse it. I also handle the case where the AI wraps JSON in markdown code blocks — strip those before parsing."*

---

### 3. Pantry-Based Recipe Suggestions

**What it does:**
Sends all user's pantry ingredients to OpenAI → returns 5 recipe suggestions ranked by match percentage → shows what ingredients are missing.

**The prompt design:**
> *"I prompt the AI to suggest recipes the user can realistically make with what they have, assume common staples like salt and oil are always available, return a match percentage from 70-100, and explicitly vary the cuisines so it doesn't suggest five pasta dishes when someone has pasta and tomatoes."*

---

### 4. World Recipe Explorer with ISR

**What it does:**
Explore page shows Recipe of the Day, category grid, and cuisine grid — all fetched from MealDB API with intelligent caching.

**How to explain it:**
> *"I used Next.js ISR — Incremental Static Regeneration. The explore page is pre-rendered at build time and cached. Recipe of the day revalidates every 24 hours, categories every week. This means the first request after cache expiry regenerates the page in the background — the user always gets a fast cached response. Category and cuisine pages are pre-generated at build time using generateStaticParams so they're instant."*

---

### 5. Authentication with Clerk-to-DB Sync

**The interesting engineering problem:**
> *"Clerk manages authentication but my PostgreSQL database manages app data. These two systems need to stay in sync. My solution — a syncUser server action that runs on every page load through the root layout. It reads the Clerk user ID from the session, checks if that user exists in my DB, and creates them if not. The clerkId field on my User model is the bridge between the two systems."*

---

### 6. Rate Limiting with Upstash Redis

**How to explain it:**
> *"Free users get 3 AI recipe generations per day. I enforce this server-side using Upstash Redis — client-side enforcement is trivially bypassable. I use sliding window rate limiting rather than fixed window. Fixed window has an edge case — a user can make 3 requests at 11:59pm and 3 more at 12:01am, doubling their effective limit. Sliding window tracks a rolling 24-hour window so this isn't possible. I have separate rate limiters with different prefixes for recipe generation, pantry scanning, and pantry suggestions — so they don't share quotas."*

---

## 🏛️ Architecture — The Big Picture

```
User visits page
      ↓
Next.js Server Component runs on server
      ↓
Direct DB query via Prisma (no API round trip)
      ↓
Fully rendered HTML sent to browser
      ↓
Browser receives content immediately (no loading state)
      ↓
Client components hydrate for interactivity only
```

### The Server/Client Boundary

```
Server Components (no JS sent to browser):
├── explore/page.tsx          — fetches MealDB data
├── explore/[id]/page.tsx     — fetches single recipe
├── category/[category]/page  — fetches meals list
└── cuisine/[cuisine]/page    — fetches meals list

Client Components (interactive only):
├── RecipeGrid.tsx            — search/filter state
├── AddToPantryModal.tsx      — form state, image upload
├── PantryPage.tsx            — edit/delete interactions
└── RecipePage.tsx            — save/unsave, useSearchParams
```

### Data Flow for AI Recipe Generation

```
1. User requests "Butter Chicken"
2. Server Action checks PostgreSQL
   → Found? Return instantly ✅ (no AI cost)
   → Not found? Continue...
3. Check rate limit via Upstash Redis
   → Limit reached? Return error ❌
   → Within limit? Continue...
4. Call OpenAI GPT-4o-mini with structured prompt
5. Parse JSON response
6. Fetch Unsplash image for the recipe
7. Save recipe to PostgreSQL (isPublic: true)
8. Return to client with isSaved, isPro status
```

---

## 🗄️ Database Schema — Know Your Models

```prisma
User
├── id          (cuid)
├── clerkId     (unique — bridge to Clerk)
├── email       (unique)
├── subscriptionTier (FREE | PRO)
└── relations → recipes[], pantryItems[], savedRecipes[]

Recipe
├── id, title, description
├── cuisine     (enum: INDIAN | ITALIAN | CHINESE | MEXICAN | OTHER)
├── category    (enum: VEG | NON_VEG | VEGAN | DESSERT | SNACK)
├── ingredients (Json — array of {item, amount, category})
├── steps       (Json — array of {step, title, instruction, tip})
├── nutrition   (Json — {calories, protein, carbs, fat})
├── tips        (Json — string[])
├── substitutions (Json — [{original, alternatives[]}])
├── isPublic    (shared cache across all users)
├── isAiGenerated
└── userId      (foreign key → User)

PantryItem
├── id, name, quantity, unit, imageUrl
└── userId      (foreign key → User)

SavedRecipe     (junction table)
├── userId      (foreign key → User)
├── recipeId    (foreign key → Recipe)
└── @@unique([userId, recipeId])  — prevents duplicates
```

**Why `@@unique([userId, recipeId])` on SavedRecipe?**
> *"This composite unique constraint means one user can't save the same recipe twice — the DB enforces it rather than me checking in application code. It's a data integrity guarantee at the database level."*

---

## 🚧 Challenges I Solved

---

### Challenge 1 — Gemini Model Overload

**Problem:** Gemini Vision returns 503 errors during peak times.

**Solution:**
```
Try primary model (gemini-2.0-flash)
      ↓ fails
Try fallback model (gemini-1.5-flash)
      ↓ fails
Return user-friendly error message
```

> *"I wrapped the Gemini call in a loop over an array of model names. Each failure catches the error, logs it, and tries the next model. The user never sees internal error details — just 'AI service is busy, try again in a few minutes' if all models fail."*

---

### Challenge 2 — TypeScript null vs undefined with Prisma

**Problem:** Build succeeded locally but type errors appeared in production build.

**Root cause:**
```ts
// My types used optional (undefined)
description?: string        // string | undefined

// Prisma returns null for optional DB fields
description: string | null  // NOT the same as undefined
```

**Solution:** Updated all Prisma-facing types to use `string | null` instead of optional properties. This is the correct pattern for any type that represents a database row.

> *"This was a subtle but important lesson — null and undefined are different in TypeScript strict mode. SQL uses NULL, TypeScript optionals use undefined. Your type definitions need to reflect the actual data source."*

---

### Challenge 3 — Server Functions Can't Be Props

**Problem:**
```
Error: Functions cannot be passed directly to Client Components
```

**What I tried:** Passing `getMealsByCategory` as a prop to `RecipeGrid` client component.

**Why it fails:** Functions can't cross the server/client boundary as props — they can't be serialized.

**Solution:** Fetch data in the server component, pass plain data arrays as props:
```ts
// Server component fetches
const result = await getMealsByCategory(category)
const meals  = result.meals

// Passes plain data to client component
return <RecipeGrid meals={meals} />
```

> *"This actually led to better architecture — the server component owns data fetching, the client component only handles rendering and interactivity. Cleaner separation of concerns."*

---

### Challenge 4 — Prisma v7 Breaking Changes

**Problem:** `url` and `directUrl` no longer allowed in `schema.prisma`.

**Solution:** Moved connection URLs to `prisma.config.ts` using the `PrismaNeon` driver adapter. The schema file only defines models, the config file handles connections.

> *"Prisma 7 was a significant version bump that moved connection configuration out of the schema. It took some debugging but once I understood the pattern it actually makes more sense — schema defines structure, config defines connections."*

---

## 🎤 Interview Questions You'll Get — With Answers

---

**Q: Why did you use server components instead of useEffect?**

> *"useEffect runs in the browser — user downloads JS, it executes, makes an API call, waits for a response, then renders. That's 3-4 sequential steps before content appears. Server components run on the server before HTML is sent — the browser receives fully rendered content immediately. No loading states for initial data, no waterfalls, no JavaScript bundle cost. That's why I score 99 on mobile Lighthouse — minimal JavaScript on the client. useEffect and React Query are still the right choice for interactive features like search filtering."*

---

**Q: How do you protect your API keys?**

> *"All AI API keys — OpenAI, Gemini, Upstash — are server-only environment variables without the NEXT_PUBLIC_ prefix. They're only accessed inside Server Actions which run exclusively on the server. The client never sees them. Even if someone inspects the network traffic they'll only see the response from my server, not the API key used to generate it."*

---

**Q: How does your rate limiting work?**

> *"I use Upstash Redis with a sliding window algorithm. When a user makes a request, I increment their counter in Redis with a 24-hour TTL. Before calling OpenAI or Gemini I check if they've exceeded their limit. Sliding window means I track a rolling 24-hour window — not a fixed midnight reset — so there's no edge case where users can double their limit by spanning a reset boundary. I use separate Redis key prefixes for each feature so recipe generation and pantry scanning don't share quotas."*

---

**Q: What's the difference between your free and pro tiers?**

> *"Free users get 3 AI recipe generations per day, 3 pantry scans per day, and basic recipe view. Pro users get unlimited generations, 50 pantry scans per day, and access to nutrition info, chef tips, and ingredient substitutions. The Pro gates are enforced server-side — the `isPro` flag comes from the database user record, not from the client."*

---

**Q: How do you handle the AI generating inconsistent output?**

> *"Prompt engineering. My system message explicitly says 'respond with valid JSON only — no markdown, no explanations'. The prompt specifies the exact JSON schema I expect. I still handle the failure case — if the AI wraps JSON in markdown code blocks I strip those before parsing. If parsing fails entirely I return a structured error rather than crashing. I also validate that the parsed result is an array and has at least one item before returning it."*

---

**Q: Why PostgreSQL over MongoDB?**

> *"Recipes have clear relational structure — users have many recipes, recipes have many saved-by users. That many-to-many relationship is exactly what relational databases are designed for. The `SavedRecipe` junction table with a composite unique constraint enforces data integrity at the database level. MongoDB would work but you'd be reimplementing relational guarantees in application code."*

---

**Q: What would you do differently if you built this again?**

> *"I'd add a job queue (BullMQ + Redis) for AI generation. Right now if a user navigates away during generation the request is lost. A queue would let generation happen asynchronously — user gets a notification when ready. I'd also add Cloudinary for image storage rather than relying on Unsplash URLs which could break if the API changes. And I'd implement the Clerk webhook for user sync rather than the check-on-every-request pattern — more reliable and avoids unnecessary DB reads."*

---

**Q: How did you achieve 99 Lighthouse on mobile?**

> *"It's an architecture result, not an optimization result. Server components mean the browser receives fully rendered HTML — almost no JavaScript to parse and execute. Next.js Image component handles lazy loading, correct sizing, and modern formats automatically. ISR means cached pages are served from Vercel's edge network — near-zero TTFB. I didn't spend time 'optimizing' — I made correct architectural decisions upfront and the score reflects that."*

---

## 🧠 Concepts to Understand Deeply

**ISR (Incremental Static Regeneration)**
Pre-renders pages at build time and caches them. After the `revalidate` period, the next request triggers a background regeneration while the user still gets the cached version. Best of static (fast) and dynamic (fresh).

**Sliding Window Rate Limiting**
Tracks requests in a rolling time window. Unlike fixed window (resets at midnight) it prevents gaming the reset boundary. Redis INCR + EXPIRE implements this efficiently.

**Composite Unique Constraint `@@unique([userId, recipeId])`**
Database-level guarantee that the combination of userId + recipeId is unique. Prevents duplicate saves without application-level checks.

**Server Action vs API Route**
Server Actions are functions marked with `"use server"` — they run on the server and can be called directly from client components without a fetch call. API Routes are explicit endpoints. Server Actions are better for mutations, API Routes are better for external consumers.

**cache-first pattern**
Check local cache (DB) before calling expensive external service (OpenAI). On miss — call API, store result, return. On hit — return immediately. Same principle as CDN caching for static assets.

---

## 📋 Quick Revision Checklist

Before the interview, make sure you can explain:

- [ ] What a server component is and why it improves performance
- [ ] How the cache-first AI generation pattern works
- [ ] Why sliding window > fixed window for rate limiting
- [ ] How Clerk syncs with your PostgreSQL database
- [ ] Why `null` ≠ `undefined` in TypeScript with Prisma
- [ ] What ISR is and when to use it vs static vs dynamic
- [ ] How Gemini Vision processes the pantry image
- [ ] Why `@@unique([userId, recipeId])` on SavedRecipe
- [ ] What happens when you exceed the rate limit
- [ ] Why 99 Lighthouse mobile — architecture not optimization

---

## 🔗 Links

- **Live App:** https://mise-psi-eight.vercel.app
- **GitHub:** https://github.com/Deepak-patidar-a/mise
- **Lighthouse Score:** 99 Mobile / 95 Desktop

---

*Good luck. You built something genuinely impressive — own it confidently.* 🔥
