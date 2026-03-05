# Handling Technical Disagreement with Team

## Question
Tell me about a time you disagreed with a technical decision made by your team or lead. What did you do?

---

## My Answer
There was a discussion about implementing Redux vs Context API in our already developed app. Our app has 100s of screens and is a retail product with lots of fields and tables per page. I knew Redux was better for this scale because Context API has a re-rendering problem — when context changes, it re-renders all components using that context, even unnecessarily. The disagreement was that fully migrating from Context to Redux in a large existing app wasn't practical. So instead we split into multiple smaller contexts to reduce the re-render scope.

## What I Got Right
- Real scenario — not a made-up textbook story
- Showed technical depth — knew WHY Redux was better and could articulate the re-render problem
- The compromise solution (splitting contexts) shows pragmatism — a senior trait
- Understood legacy codebase reality — refactoring 100 screens is not always the right call

## What I Got Wrong
- Didn't explain HOW I raised the disagreement — did I bring data? Write a doc? Raise it in a standup?
- No outcome mentioned — did the split-context approach actually work? Did the team agree?
- Didn't mention the relationship — did it stay professional? Was the other person my lead or peer?

---

## Model Answer
At Blue Yonder, there was a discussion about whether to introduce Redux into our existing app that was already built on Context API. The app had 100+ screens and the Context re-render issue was causing visible performance problems. I disagreed with just accepting it but also knew a full Redux migration across 100 screens wasn't realistic mid-product. Instead of just voicing an opinion in standup, I put together a quick profiler recording showing a single context update triggering 40+ component re-renders. I brought it to the next technical discussion with data, not just a feeling. I proposed a middle path — split our single monolithic context into domain-specific contexts: pricing, user, filters. Smaller context = fewer consumers = fewer unnecessary re-renders. The team agreed, we implemented it over two sprints, and the re-render count dropped significantly. The key was coming with data, not just an opinion. The disagreement stayed fully professional — it was about the problem, not the people.

---

## Key Things to Remember

**How seniors handle technical disagreements:**
1. Validate your concern with data before raising it
2. Come with a solution, not just a complaint
3. Frame it as "here's what I observed" not "you're wrong"
4. Propose a middle ground when full replacement isn't feasible
5. Keep it about the problem, not the person

**The STAR format for behavioral answers:**
- **S**ituation — what was the context?
- **T**ask — what was your role / what needed to happen?
- **A**ction — exactly what did YOU do?
- **R**esult — what was the outcome?

**What interviewers are actually testing:**
- Can you advocate for technical quality without being difficult?
- Do you have good judgment about when to push and when to compromise?
- Can you influence without authority?
- Do you bring data or just opinions?

**The key line to remember:**
> "Coming with data, not just an opinion" — this is what makes a senior engineer's disagreement credible.

**Memory trick:** Disagree with evidence. Propose a middle path. Stay professional. DPS.

---

## Things to Improve
- Always have a specific outcome to close the story — "and the result was X"
- Mention the profiler/data angle — it makes the disagreement feel senior, not emotional
- Practice the full STAR structure so the story flows naturally in 60-90 seconds
