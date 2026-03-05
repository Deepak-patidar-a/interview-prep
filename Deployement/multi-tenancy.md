# Multi-Tenant Architecture

## ❓ Interview Question
> "You mentioned your system uses multi-tenant deployment. Can you explain what multi-tenancy means in your project — and how does your architecture handle tenant isolation? Is data separated at the database level, application level, or both?"

---

## 🙋 My Answer (What I Said)
- 3 customers, each with 2-3 environments
- Each environment has separate database and separate server
- Same base code for all customers
- Customer-specific enhancements on top of base code
- Separate Git branches per customer (cust1_main, cust2_main etc.)

---

## ✅ Model Answer (What I Should Say)

"We use the Silo Model of multi-tenancy — the strongest form of tenant isolation. Each customer has completely separate servers and separate databases per environment. No customer's data can ever leak to another customer.

All customers share the same base application code, but each has their own Git branch for customer-specific enhancements layered on top. Deployment is handled per customer per environment, giving each customer a fully isolated stack.

The tradeoff is that this approach doesn't scale well beyond a handful of customers. Going from 3 to 30 customers would require 30 separate deployment pipelines and branches. The long-term solution for scale would be feature flags — where all customers run the same codebase and customer-specific behavior is controlled through database configuration rather than separate code branches."

---

## 🔑 Key Concepts To Remember

### Three Models of Multi-Tenancy
| Model | Isolation | Scale | Cost |
|-------|-----------|-------|------|
| **Silo** (what we use) | Separate DB + server per tenant | Low | High |
| **Bridge** | Separate DB, shared servers | Medium | Medium |
| **Pool** | Shared DB with tenant_id column | High | Low |

### Our Model — Silo
```
Customer 1:  [Server A] → [Database A]
Customer 2:  [Server B] → [Database B]
Customer 3:  [Server C] → [Database C]

All running same base code + customer-specific enhancements
```

### Git Branch Strategy
```
develop (base code — shared)
    ↓ merge
cust1_release  ←  merge  ← cust1_main (customer enhancements)
cust2_release  ←  merge  ← cust2_main
cust3_release  ←  merge  ← cust3_main
```

**Critical merge order:** develop first, then customer branch on top — never reverse this.

### Scaling Problem — The Honest Answer
| Customers | Branch-per-customer model |
|-----------|--------------------------|
| 3 | Manageable |
| 10 | Getting painful |
| 30 | Nightmare — avoid |

**Solution at scale: Feature Flags**
- All customers run identical codebase
- Customer behavior controlled via database config
- One branch, one deployment pipeline
- Customer A sees Feature X, Customer B does not — controlled by config not code

### Merge Conflict Prevention Tips
- Customer-specific code should never touch same files as base code
- Keep customizations in separate config/override files
- Sync develop → customer branches frequently (small merges beat big merges)
- Assign one developer per customer branch merge responsibility

---

## 💡 Interview Tip
Always proactively mention the scaling limitation — saying "this works for 3 customers but would need feature flags at 30" shows architectural maturity and forward thinking.
