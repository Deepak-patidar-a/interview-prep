# 🏥 Optum — Final Managerial Round Preparation
> **Interview Day: Monday | Prepared: May 2026**

---

## 📋 Table of Contents
1. [Project 1 — HTS Code Integration](#project-1--hts-code-integration)
2. [Project 2 — Data Integration Pipeline](#project-2--data-integration-pipeline)
3. [Project 3 — AI Invoice Automation](#project-3--ai-invoice-automation-pipeline)
4. [Scenario Q1 — Unclear Requirements](#scenario-q1--unclear-requirements)
5. [Scenario Q2 — Tough Situation / Deadline Pressure](#scenario-q2--tough-situation--deadline-pressure)
6. [Scenario Q3 — Disagreement / Conflict](#scenario-q3--disagreement--conflict)
7. [Scenario Q4 — Career Goals](#scenario-q4--career-goals)
8. [Quick Cheat Sheet](#-quick-cheat-sheet)
9. [Last Minute Tips](#-last-minute-tips)

---

## 🗂️ Project 1 — HTS Code Integration

### What to Say
> One of my key projects was integrating HTS — Harmonized Tariff Schedule — codes into our existing supply chain application. HTS codes define import/export duty rates for products, and our business needed this to handle international trade more accurately.
>
> As a full-stack developer, I owned the entire implementation — from UI to backend to database. We integrated HTS across five core modules: Vendor, Product, Organization, Transfer, and Purchase Orders.
>
> For example — when a vendor is marked as an Import Vendor, the system automatically triggers HTS-related fields during product creation. This flows all the way into the PO module, where import shipping days are factored in, and the final product price is calculated per line item using a complex duty-based formula.

### 🔧 Tech Stack
- **Frontend:** React JS, Context API, Material UI
- **Backend:** REST APIs, TypeORM
- **Database:** SQL

### 🔥 Biggest Challenge — Complex Price Calculation
> The most complex part was building the per-line-item price calculation. Each product could have a different HTS code, different duty rate, different import days affecting logistics cost — and all of this had to reflect dynamically on the PO. I worked through the business logic with the functional team, modeled it in SQL, and exposed it via a clean REST API that the frontend consumed in real time.

### 💡 Technical Benefits
- Reusable integration across 5 modules using React Context API — no redundant API calls
- Dynamic price calculation engine built on the backend using TypeORM + SQL — accurate to each line item
- REST APIs designed to be modular, so future modules can plug in HTS easily

### 💼 Functional / Business Benefits
- Gave the business full visibility into import costs before raising a PO
- Reduced manual errors in duty calculation — previously done on spreadsheets
- Helped procurement teams make smarter vendor decisions — knowing true landed cost upfront

---

## 🗂️ Project 2 — Data Integration Pipeline

### What to Say
> My second project was a data integration initiative for a client who was using an upstream system called Simplain — also called The HUB. They had all their master data sitting there — products, vendors, locations — and they needed it to flow into our main application seamlessly.
>
> I was the sole architect and developer for the API layer. I designed and built custom POST APIs from scratch for each data entity. The flow was — the HUB triggers our API with a JSON payload, we first insert that data into staging tables after basic validation, then we call existing PL/SQL procedures that handle all the business validations. If data passes, it goes into the final destination tables. If it fails, we send the rejected records back to the HUB with proper error details.

### 🔧 Tech Stack
- **Backend:** Node.js, REST APIs, TypeORM
- **Database:** SQL, PL/SQL Procedures

### 🔥 Biggest Challenges

**Challenge 1 — Large Payload Crashing the API**
> When the client started sending bulk data, the JSON payloads were too large and the API was timing out. I tackled this in two steps — first I increased the JSON parsing limit on the Node/Express configuration, then more importantly, I implemented chunking inside the API logic, so large payloads are broken into smaller chunks and each chunk is sent to the PL/SQL procedure independently. This made the system stable and scalable for any payload size.

**Challenge 2 — Type Validation Without Frontend**
> Since this was a direct system-to-system integration, there was no frontend layer doing any validation. The client was sending raw JSON and any type mismatch was breaking the database inserts. I handled this entirely in the backend using TypeORM entities — defining strict type constraints so that only clean, correctly typed data would reach the SQL tables.

### 💡 Technical Benefits
- Chunking mechanism made the API handle any payload size without timeouts
- Staging table pattern gave a clear audit trail — you always know what came in vs what got rejected
- Smart reuse of existing PL/SQL procedures — no business logic rewritten, just cleanly plugged in
- TypeORM entity validation acted as a strong backend data contract

### 💼 Functional / Business Benefits
- Client could migrate and sync their master data without any manual intervention
- Rejected records returned with reasons — so the HUB team could fix and resend without guessing
- Entire integration was system-agnostic — any upstream system could call these APIs with the right JSON
- Reduced dependency on manual data entry — saving hours of effort daily

---

## 🗂️ Project 3 — AI Invoice Automation Pipeline

### What to Say
> My third and most recent project is an AI-powered invoice automation pipeline where we integrated Azure AI Agent with BlueYonder's ERP system.
>
> The business problem was simple — the client was receiving hundreds of invoices as PDFs from different vendors, and all of this was being processed manually. We automated the entire pipeline.
>
> The flow is — when an invoice PDF arrives, Azure Document Intelligence first parses it and extracts the raw text and structure. Then we invoke the Azure AI Agent — powered by GPT-4o — from our Node.js backend using the Azure SDK for JavaScript. The agent reads the extracted content and intelligently maps it to fields like Vendor Name, Product details, quantities, and PO references. This structured JSON is then pushed into BlueYonder's SDI staging tables. From there, business validation runs — if a vendor or product doesn't exist, the system auto-creates it. Once all validations pass, data moves to the final destination tables and triggers the relevant notifications or approvals.

### 🔧 Tech Stack
- **AI:** Azure AI Agent (GPT-4o), Azure Document Intelligence
- **Backend:** Node.js, Azure SDK for JavaScript, REST APIs
- **Database:** BlueYonder SDI Staging Tables, SQL, PL/SQL Procedures

### 🔥 Biggest Challenge — Different Invoice Formats
> The hardest real-world problem we faced was that every vendor has a completely different invoice format. One vendor puts the PO number on the top right, another embeds it in a footer, another calls it 'Reference Number' instead of 'PO Number'. Traditional rule-based parsing completely fails here.
>
> This is exactly where the AI Agent added real value — because GPT-4o understands context, not just position. It can read 'Ref No: 10045' and correctly map it to the PO Reference field based on surrounding context. We fine-tuned our agent prompt carefully to handle these variations, and we also built a fallback — if the agent confidence is low on a field, it flags that record for manual review instead of pushing wrong data downstream.

### 📊 Conversion Rate — How to Present It
> Currently the system is achieving around 70 to 80 percent straight-through accuracy — meaning that much data flows end-to-end without any human intervention. The remaining 20 to 30 percent gets flagged for review, mostly edge cases like scanned invoices or completely non-standard formats. We are actively improving this by enriching our prompt engineering and adding more format examples to the agent context. Even at current accuracy, it has drastically reduced manual effort compared to processing everything by hand.

### 💡 Technical Benefits
- Zero code change needed for new vendor invoice formats — the AI agent adapts
- Staged validation via SDI tables ensures no bad data reaches production tables
- Auto-creation of Vendor/Product reduces manual master data management
- Scalable pipeline — can process multiple invoices in parallel via async Node.js calls

### 💼 Functional / Business Benefits
- Eliminated hours of manual data entry per day for the client's AP team
- Faster PO matching — invoices that used to take days now process in minutes
- Audit trail maintained at every stage — staging → validation → destination
- Reduced human error in invoice-to-PO matching significantly
- Easily scalable to new vendors without any development effort

---

## 🎙️ Scenario Q1 — Unclear Requirements

**Question:** *"Tell me about a time when you were given unclear or incomplete requirements, and how did you handle it?"*

### Your Answer (STAR Format)

> **Situation:**
> There was a ticket assigned to me in Jira where the description had just a couple of lines and one screenshot — no clear steps to reproduce, no expected behavior mentioned, nothing about which module or flow was affected.

> **Task:**
> Rather than going back immediately and asking basic questions, I wanted to first do my own due diligence before raising it.

> **Action:**
> I did a thorough end-to-end analysis — I tested the entire PO creation and release flow myself to try and reproduce the issue. When I couldn't reproduce it, I documented my complete analysis — what I tested, what I found, and what information was still missing — and added it as a comment on the Jira ticket. In the next scrum call, I presented my findings to the manager and flagged that I needed more clarity before proceeding further.

> **Result:**
> My manager appreciated the analysis and said it was thorough. He then guided me on where to get the additional information needed. It also set the right expectation with the team — that the ticket needed more details before development could begin.

### 💡 Power Line
> *"I've learned that unclear requirements, if not handled early, can cost a lot of rework later. So I always try to front-load the clarity."*

---

## 🎙️ Scenario Q2 — Tough Situation / Deadline Pressure

**Question:** *"Tell me about a time you faced a tough technical problem or a situation where things were going wrong — maybe a deadline was at risk or something was breaking."*

### Your Answer (STAR Format)

> **Situation:**
> During our first client go-live, we hit a critical phase where multiple high-priority bugs were surfacing daily. The situation got serious enough that the go-live had to be postponed once — which was a significant business impact.

> **Task:**
> Before the next scheduled go-live date, the team needed to ensure that all critical and high business-impact issues were resolved — and this time there was no room for another postponement.

> **Action:**
> I along with my team did a thorough triage of all open Jira tickets — prioritizing them by business impact. We focused only on critical and high severity cases first. Each fix was done carefully — not rushed — because a wrong fix at go-live is worse than no fix. After every fix we did end-to-end functional testing ourselves to make sure nothing else broke. We stayed late nights as needed because the goal was clear — a successful go-live. The whole team was aligned on that one mission.

> **Result:**
> The go-live went extremely well. No critical issues post-launch. The team received a lot of appreciation from the client and management. It was one of the most satisfying moments professionally — because we had turned around a very stressful situation into a big success.

### 💡 Power Line
> *"That experience taught me that in high pressure situations, the most important thing is to stay structured, communicate clearly with the team, and never compromise on testing even when time is tight."*

---

## 🎙️ Scenario Q3 — Disagreement / Conflict

**Question:** *"Tell me about a time you had a disagreement with a teammate or senior, and how did you handle it?"*

### Your Answer (STAR Format)

> **Situation:**
> We were facing a performance issue on our product landing page — we had thousands of records loading, but our REST APIs were returning all columns from the table even though the UI only needed 4 or 5 fields. This was making the payload unnecessarily heavy and slowing down the page.

> **Task:**
> I wanted to propose a solution that could fix this not just for one screen but structurally — so that we don't keep facing this as the application grows.

> **Action:**
> I suggested introducing GraphQL for specific use cases — not replacing all REST APIs, but using GraphQL alongside REST where we need flexible, field-level querying. I backed my point by highlighting that many large companies in the industry successfully use GraphQL and REST together in the same application. One of my senior colleagues disagreed — his concern was around the complexity of introducing a new technology into an already large application, which is a completely valid point. We had a healthy technical discussion where both sides presented their reasoning. I made sure I listened to his concerns carefully and didn't dismiss them.

> **Result:**
> We didn't reach a final conclusion that day, and honestly that was okay — because this was a big architectural decision that needed more thought. But one of my leads appreciated that I had brought up a valid solution. For me personally, it was a great learning — I got more confident in presenting technical ideas, and I also learned that in a large application, the cost of change is a very real factor to weigh alongside the technical benefit.

### 💡 Power Line
> *"I believe disagreements like these are healthy — they push everyone to think deeper. As long as the discussion is about what's best for the product and not personal, it always leads to better outcomes."*

---

## 🎙️ Scenario Q4 — Career Goals

**Question:** *"Where do you see yourself in the next 2 to 3 years?"*

### Your Answer

> In the next 2 to 3 years, I have three clear goals for myself.
>
> First — I want to become technically very strong. Not just in my current stack, but in end-to-end architecture — where I can look at a system and make the right design decisions confidently. Projects like the AI pipeline and the data integration I've worked on are already pushing me in that direction.
>
> Second — I want to stay ahead of emerging technologies, especially AI. I've already started with the Azure AI Agent project and I've seen firsthand how much value it brings to a business. I want to be the kind of person who helps a company stay competitive by identifying and implementing the right technologies at the right time.
>
> Third — I want to grow into a senior or tech lead role where I can own an entire product end-to-end and drive a team of 5 to 10 people. Not just writing code, but mentoring, making architectural decisions, and taking full accountability for delivery.

### 💡 Power Line
> *"Optum feels like the right place for that journey — the scale of the problems here, the exposure to enterprise systems, and the opportunity to work with AI in healthcare is exactly the kind of environment where I can grow and contribute meaningfully."*

---

## ⚡ Quick Cheat Sheet

| Topic | Key Highlight |
|---|---|
| **Project 1** — HTS Integration | Full-stack, complex price calculation, 5 modules |
| **Project 2** — Data Integration | Sole architect, chunking fix, type validation |
| **Project 3** — AI Invoice Pipeline | GPT-4o, Azure SDK, 70-80% automation rate |
| **Unclear Requirements** | Self-analysis first, proactive communication |
| **Tough Situation** | Go-live rescue, prioritization, late nights |
| **Disagreement** | GraphQL vs REST, respectful, mature debate |
| **Career Goals** | Technical depth → AI adoption → Lead/Manager |

---

## 🏁 Last Minute Tips

1. **Speak slowly and confidently** — you know this material deeply
2. **Always give real examples** — never answer scenario questions abstractly
3. **End every answer positively** — what you learned, what the result was
4. **Listen carefully** before answering — it's okay to take 2-3 seconds to think
5. **Show enthusiasm** for the role — managers hire attitude as much as skill

---

> 💪 **You've got this! All the best for Monday!** 🚀
