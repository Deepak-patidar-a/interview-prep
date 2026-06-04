# 🧾 Invoice Automation — Interview Prep Guide

> Architecture: Azure Blob Storage → PDF Picker Tool → Azure AI Agent → Node/REST API → MerchOps Platform

---

## 🗺️ Architecture Overview

```
Azure Blob Storage
      │
      │  (Event trigger on upload)
      ▼
PDF Picker Tool          ← Decides which PDF to send to the agent
      │
      ▼
Azure AI Agent           ← Reads PDF, extracts structured data
      │
      │  (Returns JSON)
      ▼
Custom Node/REST API     ← Validates, transforms, calls MerchOps
      │
      ▼
MerchOps Platform        ← Final destination for invoice data
```

---

## ❓ Interview Questions & Answers

---

### 1. Where do you store the PDF and for how long?

**Answer:**
> We store PDFs in Azure Blob Storage in a dedicated container scoped per client. We apply a **lifecycle management policy** that automatically deletes blobs after **7 days** — long enough for reprocessing or audit, but short enough for cost control and data minimization compliance.
>
> Each blob is tagged with metadata like upload timestamp and client ID for traceability.

**Key points to mention:**
- Azure Blob lifecycle policy (auto-delete after 7 days)
- Per-client scoped container
- Metadata tagging for audit trail
- Data minimization principle (GDPR-friendly)

---

### 2. How do you decide which PDF the agent should read?

**Answer:**
> It's **event-driven**. When a PDF is uploaded to Blob Storage, it triggers an **Azure Event Grid event** (or an Azure Function Blob trigger). The PDF picker tool receives that event, extracts the blob URL and metadata, and passes it directly to the agent.
>
> There's no manual selection — each upload maps to one agent invocation. The trigger itself is the decision.

**Key points to mention:**
- Azure Event Grid / Blob trigger on upload
- Event carries the blob URL + metadata
- One upload = one agent invocation
- Fully automated, no human selection needed

---

### 3. Why use Azure AI Agent instead of building your own?

**Answer:**
> Classic **build vs. buy** decision. Three reasons we chose Azure AI Agent:
>
> 1. **Speed** — Building a reliable PDF extraction pipeline with retries, tool use, and structured output parsing from scratch would take weeks. Azure AI Agent gives us that out of the box.
> 2. **PDF complexity** — Invoices come in many layouts: tables, multi-column formats, varied templates. The agent handles these semantically, far better than regex or rule-based parsers.
> 3. **Azure ecosystem fit** — We're already on Azure, so it integrates naturally with our IAM, networking (VNet), and compliance setup.
>
> The velocity gain at this stage outweighs the control tradeoff.

**Key points to mention:**
- Build vs. buy trade-off reasoning
- Handles varied invoice layouts (tables, multi-column)
- Already on Azure — ecosystem consistency
- Can always migrate later if needed

---

### 4. What kind of prompt do you give the Azure AI Agent?

**Answer:**
> We use a **structured extraction prompt**. Something like:
>
> *"You are an invoice data extraction assistant. From the attached PDF, extract the following fields as a JSON object: product number, product name, quantity, unit price, tax rate, total amount, vendor name, and invoice date. If a field is not found, return null for that key. Do not infer or guess values."*
>
> The key instruction is **return null rather than guess** — downstream validation then catches missing fields cleanly instead of silently accepting wrong data.

**Key points to mention:**
- Structured JSON output format
- Explicit field list: product no., name, qty, price, tax, vendor, date
- `null` for missing fields — never guess/infer
- Keeps extraction deterministic and auditable

**Bonus — prompt best practices:**
- Use system prompt to set role ("You are an invoice extraction assistant")
- Use few-shot examples for complex formats
- Instruct agent to flag ambiguous values instead of assuming

---

### 5. Does the agent store extracted data somewhere, or call the API directly?

**Answer:**
> The agent itself is **stateless** — it doesn't persist anything. It returns structured JSON in the response. Our **Node.js orchestration layer** receives that JSON and immediately calls the internal REST API with the extracted payload.
>
> Think of the agent as a pure function: input PDF → output JSON. The Node layer owns all state and downstream calls.

**Key points to mention:**
- Agent = stateless, pure extraction function
- Node.js orchestration layer receives JSON response
- Node layer owns state management and API calls
- Clean separation of concerns

---

### 6. What security measures exist between the agent and your APIs?

**Answer:**
> Two layers:
>
> 1. **Network isolation** — The Node API is inside an Azure **Virtual Network (VNet)**. It's not publicly exposed. The orchestration service lives in the same VNet, so calls never go over the public internet.
>
> 2. **Authentication** — We use **Azure Managed Identity**. The orchestration service has an assigned identity and calls the API with a bearer token from Azure AD. No hardcoded secrets, no API keys in code.
>
> Additionally, the API side applies **rate limiting** and **input sanitization** to protect against malformed agent output.

**Key points to mention:**
- Azure VNet — no public exposure
- Managed Identity — no hardcoded secrets
- Azure AD bearer token authentication
- Rate limiting + input sanitization on API side

---

### 7. Can the agent call your API directly, or does it need a Node layer?

**Answer:**
> Technically yes — Azure AI Agent supports **function calling** and **OpenAPI plugins** that could call the API directly. But we chose to keep the Node orchestration layer for three reasons:
>
> 1. **Transform layer** — We validate and shape the agent's JSON before sending it to the API.
> 2. **Error handling** — Retries, logging, and failure notifications live in one place.
> 3. **Decoupling** — If the internal API contract changes, we update the Node layer only. The agent setup stays untouched.

**Key points to mention:**
- Agent *can* call APIs via function calling / OpenAPI plugin
- Chose Node layer for: transformation, error handling, decoupling
- Single responsibility — agent extracts, Node orchestrates
- Easier to maintain and evolve independently

---

### 8. What if the agent can't find the required data in the PDF?

**Answer:**
> Handled at two levels:
>
> 1. **Prompt level** — Agent is instructed to return `null` for missing fields, never guess.
> 2. **Node layer pre-validation** — Before calling the API, we check for null critical fields (product number, total amount). If any are missing, we skip the API call and send the user a notification: *"Invoice could not be fully processed — missing fields: [list]."*
>
> The partial extraction result and blob path are logged so the user can review the PDF manually and resubmit.

**Key points to mention:**
- Agent returns `null` for missing fields
- Node layer validates before calling API
- User gets a clear error message with specific missing fields
- Blob path + partial data logged for manual review

---

### 9. What if the agent finds data but the API validation fails?

**Answer:**
> We **don't automatically re-invoke the agent** — that could create infinite retry loops on bad data.
>
> Instead:
> - The API returns a structured error response with the specific failing field and reason
> - We log it with the original extracted payload and a correlation ID
> - The UI shows the user a clear error: *"Field X failed validation — expected format YYYY-MM-DD, received 15/03/25"*
> - The user can correct and resubmit
>
> For financial data specifically, we prefer **human-in-the-loop** on failures rather than automated re-extraction — accuracy matters more than speed here.

**Key points to mention:**
- No auto-retry loop on validation failure
- Structured error from API → shown on UI
- Correlation ID for full traceability
- Human-in-the-loop for financial accuracy
- Future option: targeted re-extraction for specific field types

---

### 10. How is the process invoked and how is it stopped?

**Answer:**
> **Invocation:** Event-driven. An Azure Blob trigger or Event Grid subscription fires on new PDF upload → triggers an Azure Function → starts the orchestration.
>
> **Stopping:** Each invocation is stateless and short-lived — there's no long-running process to kill. To pause all processing, we can:
> - Disable the Blob trigger at the Function App level
> - Add a feature flag check at the start of the function
>
> Every invocation has a **correlation ID** for end-to-end tracing in Azure Monitor / Application Insights.

**Key points to mention:**
- Event-driven via Azure Event Grid / Blob trigger
- Stateless, short-lived invocations — easy to stop
- Disable trigger or use feature flag to pause
- Correlation ID for full observability

---

### 11. What are the business benefits?

**Answer:**
> Before this, the team manually keyed invoice data — slow, error-prone, and not scalable.
>
> This system delivers:
> - **Speed** — Processing time drops from hours to seconds per invoice
> - **Accuracy** — Eliminates manual keying errors on financial data (tax, qty, price)
> - **Scalability** — Handle 10x invoice volume without adding headcount
> - **Faster payment cycles** — Quicker invoice ingestion = faster PO matching and vendor payments
> - **Cost efficiency** — Azure AI Agent costs well under a cent per invoice at typical token usage

**Key points to mention:**
- Manual → automated (hours → seconds)
- Reduces human error on financial fields
- Scales with volume, not headcount
- Faster vendor payment cycles
- Measurable ROI per invoice

---

## 🎯 Bonus — Extra Questions an Interviewer May Ask

---

### 12. How do you handle different invoice templates from different vendors?

> The agent handles template variability semantically, not positionally — so it doesn't break when a vendor moves a column. For vendors with consistently problematic formats, we maintain **vendor-specific prompt hints** in our config. Long-term, few-shot examples per vendor would be the next step.

---

### 13. What's your cost model — is this expensive at scale?

> Azure AI Agent is priced per token. A typical invoice extraction is a few thousand tokens — well under a cent per invoice. We also cache the agent configuration (not re-created per call) and could batch low-priority invoices during off-peak hours to reduce cost. At high volume, we'd evaluate batching strategies.

---

### 14. How do you handle multi-page PDFs or PDFs with multiple invoices?

> Currently we assume **one invoice per PDF** — that's our v1 constraint. For multi-invoice PDFs, we'd need a pre-processing step to split by invoice boundary, either heuristically (detecting header/footer patterns) or with a classification call. Known limitation, planned for v2.

---

### 15. How do you monitor and alert on failures?

> Every invocation writes structured logs to **Azure Monitor / Application Insights**:
> - Correlation ID
> - Blob path
> - Extraction result status
> - API response code
> - Processing duration
>
> Alerts are set for failure rates above a threshold. Dashboard shows daily volume, success rate, and average extraction time.

---

### 16. Is this GDPR / data compliance friendly?

> Yes:
> - **7-day blob retention** — data doesn't linger
> - Extracted data written only to internal systems, never logged in plain text
> - Blob container access scoped by **RBAC** to the service principal only
> - PII in invoices (vendor name, addresses) handled under internal data classification policy

---

### 17. What happens if Azure AI Agent is down or returns garbage output?

> We have a fallback strategy:
> - **Retry with exponential backoff** (3 attempts) on transient failures
> - If all retries fail, the invoice is marked as `pending_manual_review`
> - User is notified and can resubmit when the issue is resolved
> - We use Azure's health endpoint to detect service degradation early

---

### 18. How would you extend this for real-time processing vs batch?

> Currently it's near-real-time (event-driven per upload). For batch scenarios — e.g., processing 500 invoices at month-end — we'd use an **Azure Durable Functions fan-out pattern**: one orchestrator spawning parallel activity functions per PDF, with a final aggregation step. Cost-optimised and parallelised.

---

## 🔑 Key Terms to Know

| Term | What it means in this context |
|---|---|
| Azure Blob Storage | File storage for uploaded PDFs |
| Azure Event Grid | Event bus that triggers processing on blob upload |
| Azure Function | Serverless function that starts the orchestration |
| Azure AI Agent | LLM-based agent that reads and extracts invoice data |
| Managed Identity | Azure AD identity for service-to-service auth (no secrets) |
| VNet | Private Azure network — keeps API off public internet |
| Application Insights | Monitoring + logging for all invocations |
| Correlation ID | Unique ID per invocation for end-to-end tracing |
| Feature flag | Toggle to pause/enable processing without a deployment |

---

## ✅ Quick Revision Checklist

- [ ] Can explain the full flow end-to-end in 60 seconds
- [ ] Know why Event Grid is used (event-driven, not polling)
- [ ] Can explain build vs. buy decision for Azure AI Agent
- [ ] Know the prompt strategy (null for missing, no guessing)
- [ ] Can explain Managed Identity vs API key auth
- [ ] Know the two failure scenarios (missing data vs. validation failure)
- [ ] Can give 3 business benefits confidently
- [ ] Know the GDPR / compliance angle
- [ ] Know the monitoring story (correlation ID, Application Insights)

---

*Good luck with the interview! 🚀*
