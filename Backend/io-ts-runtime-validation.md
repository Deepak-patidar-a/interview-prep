# Runtime Validation with io-ts

## Question
You've worked with io-ts for runtime validation of 60+ APIs. Most engineers just use TypeScript types and call it a day — why did you go with io-ts, and what problem does it actually solve that TypeScript alone doesn't?

---

## My Answer
TypeScript types check only at compile time. But when we fetch an API response at runtime, TypeScript is already gone. We need to validate that we received a valid response as per our types. This is where io-ts comes in — it can check types at runtime and gives a solution to runtime errors.

## What I Got Right
- Nailed the core concept: compile time vs runtime validation
- Clear and concise — no fluff
- Showed I understood WHY we used it, not just THAT we used it

## What I Got Wrong
- Didn't explain what happens when validation fails — io-ts returns an Either type, not a thrown error
- No mention of the tradeoff — io-ts has a steep learning curve and verbose codec definitions
- Missed a real example to seal the answer — "API said price was a number, returned null, io-ts caught it before it crashed the UI"

---

## Model Answer
TypeScript disappears at runtime — all type information is erased after compilation. So when our app receives an API response in the browser, there's nothing preventing a broken payload from silently corrupting UI state. io-ts solves this with runtime codecs — you define the shape of expected data once, and io-ts validates incoming data against it at runtime. The key design is the Either monad pattern: on failure it returns a Left with detailed decode errors showing exactly which field failed and why. On success it returns a Right with the fully typed value. This let us handle bad API responses gracefully — log the error, show a fallback — instead of letting null values crash the UI downstream. The tradeoff is verbosity. Codec definitions can get complex for deeply nested types. But for enterprise APIs where backend contracts change frequently, that safety net was worth the overhead.

---

## Key Things to Remember

**Why TypeScript alone isn't enough:**
- TypeScript is a compile-time tool only
- At runtime, your app is just JavaScript — no type guarantees
- API responses can change without your frontend knowing

**io-ts core pattern:**
```ts
import * as t from 'io-ts'

const Product = t.type({
  id: t.number,
  name: t.string,
  price: t.number,
})

const result = Product.decode(apiResponse)
// result is Either<Errors, Product>
// Left = validation failed with detailed errors
// Right = valid, typed data
```

**Either monad:**
- `Left` = failure path — contains decode errors
- `Right` = success path — contains valid typed value
- Forces you to handle both cases explicitly

**When to use io-ts:**
- Enterprise apps with many API integrations
- When backend contracts change often
- When silent data corruption is more dangerous than a loud error

**When NOT to use it:**
- Small apps with stable, trusted APIs
- When the learning curve cost outweighs the safety benefit

**Memory trick:** TypeScript = bouncer who checks IDs at the door (compile time). io-ts = bouncer who checks IDs again inside the club (runtime).

---

## Things to Improve
- Learn the Either monad pattern deeply — it appears in functional programming broadly
- Be ready to write a simple io-ts codec from memory in an interview
- Always have one real bug-caught-by-io-ts story ready
