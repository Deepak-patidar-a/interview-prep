# TypeORM + PL/SQL Business Logic

## Question
Why put business logic in PL/SQL stored procedures instead of the Node.js layer? What are the advantages and disadvantages? How does your Express API call a PL/SQL procedure?

---

## My Answer
We inherited PL/SQL procedures from a legacy system. Instead of rewriting working logic, we call them directly from TypeORM. Main advantage is performance — PL/SQL can read, write, and update large numbers of records in one go without slowing down. Disadvantage is developer experience — it is an old language and new developers don't know it. In TypeORM we call PL/SQL using raw query syntax like `BEGIN procedure_name... END`.

## What I Got Right
- Correct reason — legacy reuse is a valid engineering decision
- Performance advantage explained correctly — PL/SQL runs inside the DB engine, bulk operations are fast
- Developer experience as a disadvantage — mature observation
- Knowing the TypeORM raw query syntax for calling procedures

---

## Model Answer
We inherited PL/SQL stored procedures from a legacy system, and rather than rewriting working logic we call them directly from TypeORM using raw query syntax like `BEGIN procedure_name(:param) END`. The main advantage is performance — PL/SQL executes inside the database engine itself so bulk read/write operations on large datasets are extremely fast with no network round trips. The disadvantages are tight coupling to the database, difficulty testing, and knowledge gap risk since fewer developers know PL/SQL today.

---

## Key Things to Remember

**Advantages of PL/SQL stored procedures:**
- Runs inside the database engine — no network round trips
- Extremely fast for bulk operations on millions of records
- Already written and tested — no rewrite risk
- Reusable across multiple services

**Disadvantages of PL/SQL stored procedures:**
- Tight coupling to database — if you switch databases, all PL/SQL must be rewritten
- Hard to unit test — cannot write Jest tests for stored procedures
- Debugging is painful — errors surface as vague database exceptions in Express
- Knowledge gap — fewer developers know PL/SQL today, hiring becomes harder
- Business logic scattered across two layers — Node.js and database

**TypeORM raw query to call PL/SQL:**
```typescript
await dataSource.query(`BEGIN procedure_name(:param1, :param2); END;`, [value1, value2])
```

**When to keep logic in PL/SQL vs move to Node.js:**
- Keep in PL/SQL — bulk data operations, complex joins, batch processing
- Move to Node.js — business rules, validations, orchestration logic

**Long term consideration:**
Keeping business logic in Node.js makes the application database-agnostic. If the team plans to scale or switch databases, migrating PL/SQL logic to Node.js should be on the roadmap.

---

## Things to Improve
- Ask team: is there a plan to gradually migrate PL/SQL logic to Node.js layer?
- Learn how to handle PL/SQL exceptions that bubble up to Express
