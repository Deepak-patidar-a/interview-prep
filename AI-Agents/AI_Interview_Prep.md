# 🤖 AI & GenAI Interview Preparation Guide

> **Purpose:** Revise your AI/GenAI resume points, understand them deeply, and prepare for probable interview questions with confident answers.

---

## 📋 Table of Contents

1. [Your Resume Skills Overview](#1-your-resume-skills-overview)
2. [Project 1 — Invoice Automation Pipeline (BlueYonder + Azure AI)](#2-project-1--invoice-automation-pipeline)
3. [Project 2 — AI-Powered Plant Diagnosis](#3-project-2--ai-powered-plant-diagnosis)
4. [Project 3 — LLM-Powered Content Discovery (GPT-4o-mini)](#4-project-3--llm-powered-content-discovery)
5. [Core Concepts You Must Know](#5-core-concepts-you-must-know)
6. [General AI/GenAI Interview Questions](#6-general-aigenai-interview-questions)

---

## 1. Your Resume Skills Overview

```
AI & GenAI: Azure AI Agent, OpenAI API (GPT-4o-mini), LLM integration,
Prompt Engineering, Chain-of-Thought (CoT) reasoning,
Model Fine-tuning, Hugging Face LLM Inference API
```

### What Each Term Means (In Simple Words)

| Skill | What It Means | How to Say It in Interview |
|---|---|---|
| **Azure AI Agent** | A service by Microsoft that lets an AI autonomously use tools, call APIs, and make decisions | "I used Azure AI Agent to orchestrate the invoice automation workflow" |
| **OpenAI API (GPT-4o-mini)** | API to call OpenAI's GPT model programmatically from your code | "I integrated GPT-4o-mini via REST API to enable natural language understanding" |
| **LLM Integration** | Connecting a Large Language Model into your application/system | "I embedded LLM capabilities into existing backend services" |
| **Prompt Engineering** | Crafting the right instructions/questions to get accurate output from an AI model | "I designed prompts to extract structured invoice fields from unstructured PDF text" |
| **Chain-of-Thought (CoT)** | A technique where you ask AI to think step-by-step before giving the final answer | "I used CoT prompting to improve accuracy in multi-step invoice validation logic" |
| **Model Fine-tuning** | Training a pre-trained AI model further on your own data to improve its performance on specific tasks | "I explored fine-tuning to improve domain-specific invoice field extraction" |
| **Hugging Face Inference API** | A platform that hosts open-source AI models you can call via API without hosting them yourself | "I used Hugging Face's hosted inference API to run a plant disease detection model" |

---

## 2. Project 1 — Invoice Automation Pipeline

### 📝 Resume Line
> *"Developing an AI-powered invoice automation pipeline by integrating Azure AI Agent with BlueYonder — enabling intelligent PDF extraction, staged business validation, and automated PO/Vendor/Product creation at scale."*

---

### 🔍 What This Project Actually Does (Simple Explanation)

Think of it like this:

```
Invoice PDF (from Supplier)
        ↓
Azure AI Agent reads the PDF
        ↓
Extracts: Vendor Name, Items, Quantity, Price, Tax ID, PO Number
        ↓
Sends data → BlueYonder SDI / Staging Tables
        ↓
Business Validations run:
  - Does this Vendor exist?  → No → Auto-create Vendor
  - Does this Product exist? → No → Auto-create Product
  - Does a PO exist?         → No → Auto-create PO
        ↓
Data moves to Main Destination Table
        ↓
Notification sent to the system / approver
```

**Why it matters:** Without AI, someone had to manually read each invoice, type the data, and create POs. This automates the entire flow.

---

### 🧱 Tech Stack Breakdown

| Component | Technology | Role |
|---|---|---|
| AI Orchestration | Azure AI Agent | Manages the workflow, calls tools |
| Language Model | GPT-4o (via Azure OpenAI) | Understands and extracts invoice data |
| PDF Parsing | Azure Document Intelligence | Reads PDF structure and text |
| ERP System | BlueYonder | Staging tables, business logic, PO/Vendor/Product creation |
| Data Flow | SDI / Staging Tables | Intermediate layer for validation before main tables |

---

### ❓ Probable Interview Questions & How to Answer

---

**Q1. Can you walk me through the invoice automation pipeline you built?**

> ✅ *Answer:*
> "Sure! We integrated Azure AI Agent with BlueYonder's ERP system. When an invoice PDF comes in, Azure Document Intelligence first parses the PDF to extract raw text and structure. Then the Azure AI Agent — powered by GPT-4o — reads this extracted data and maps it to fields like Vendor Name, Product details, quantities, and PO references. This structured data is pushed into BlueYonder's SDI staging tables. From there, business validation rules run — for example, checking if the vendor or product already exists. If not, the system auto-creates them. Once validated, the data moves to the main destination tables and triggers the relevant notifications or approvals."

---

**Q2. What is an AI Agent? How is it different from just calling GPT-4o directly?**

> ✅ *Answer:*
> "A regular GPT-4o API call is a one-shot interaction — you send a prompt, you get a response. An AI Agent is more autonomous. It can decide which tools to call, in what order, and loop through steps until the task is complete. For example, in our pipeline, the agent doesn't just extract text — it decides to call the PDF parser first, then validate the output, then call the BlueYonder API. Azure AI Agent provides this orchestration layer on top of the LLM."

---

**Q3. What is the role of SDI/Staging tables? Why not directly insert into the main table?**

> ✅ *Answer:*
> "Staging tables act as a buffer or sandbox. The AI might extract data that looks correct but could violate business rules — like a duplicate vendor or an invalid product code. By putting data in staging first, we can run all validations before touching the main production tables. It also makes error handling easier — if something fails, we can fix it in staging without corrupting real data."

---

**Q4. What is Prompt Engineering and how did you use it here?**

> ✅ *Answer:*
> "Prompt Engineering is about designing the instruction you give to the LLM so it returns accurate, structured output. In our invoice pipeline, instead of just saying 'read this invoice,' we crafted prompts like: 'Extract the following fields in JSON format: vendor_name, invoice_number, line_items (array of product, quantity, unit_price), total_amount, tax_id.' This structured prompt ensures the model gives consistent, parseable output rather than a free-form paragraph."

---

**Q5. What is Chain-of-Thought reasoning? Did you use it here?**

> ✅ *Answer:*
> "Chain-of-Thought is a prompting technique where you instruct the model to reason step-by-step before giving the final answer. For example, instead of asking 'Is this a valid invoice?', you ask the model to first identify the vendor, then check if all required fields are present, then verify the math — and then conclude. This reduces errors, especially for complex multi-field validation. In our pipeline, we explored CoT for handling ambiguous invoices where fields were missing or formatted inconsistently."

---

**Q6. What challenges did you face in this project?**

> ✅ *Answer:*
> "A few key challenges: First, PDFs are inconsistent — different vendors format their invoices differently, so extraction wasn't always clean. We handled this by combining Azure Document Intelligence's layout model with GPT-4o's contextual understanding. Second, staging validation rules were complex — we had to make sure the AI output conformed to BlueYonder's data schema. Third, since this is still in development, we're iterating on edge cases like multi-currency invoices and line items with vague descriptions."

---

**Q7. The project is still in progress — what stage is it at?**

> ✅ *Answer:*
> "We've completed the core integration — the PDF extraction, AI field mapping, and staging table flow are working. We're currently in the business validation and testing phase, refining the rules for PO/Vendor/Product auto-creation and handling edge cases. The projected scale is 10K+ invoices/month with around 50% reduction in manual processing effort."

---

**Q8. Why auto-create a PO from an invoice? Isn't a PO created before an invoice?**

> ✅ *Answer:*
> "Great question — in an ideal procurement cycle, yes, the PO comes first. But in reality, especially with smaller vendors or emergency purchases, invoices sometimes arrive without a prior PO. In those cases, the system creates a PO retroactively to match the invoice — this is called a 'post-facto PO' or 'invoice-backed PO.' It ensures the purchase is still recorded, traceable, and goes through the approval workflow."

---

## 3. Project 2 — AI-Powered Plant Diagnosis

### 📝 Resume Line
> *"AI-powered plant diagnosis using Hugging Face LLM inference API."*

---

### 🔍 What This Project Does (Simple Explanation)

```
User uploads image of a plant (or describes symptoms)
        ↓
Hugging Face Inference API called
  (Model: image classification or vision-language model)
        ↓
Model identifies: disease name, confidence score
        ↓
Returns diagnosis + suggested treatment to user
```

---

### ❓ Probable Interview Questions & How to Answer

---

**Q1. What is Hugging Face and why did you use it instead of OpenAI?**

> ✅ *Answer:*
> "Hugging Face is a platform that hosts thousands of open-source AI models — for text, images, audio, and more. I used their Inference API which lets you call these models via a simple HTTP request without needing to host or manage the model yourself. For plant diagnosis, there are specialized open-source models trained on plant disease datasets — using Hugging Face made more sense than OpenAI since these domain-specific models perform better for this use case and are also more cost-effective."

---

**Q2. What kind of model did you use for plant diagnosis?**

> ✅ *Answer:*
> "I used an image classification model — specifically one fine-tuned on plant disease datasets like PlantVillage. The model takes a plant image as input and classifies it into disease categories like 'Tomato Late Blight' or 'Healthy.' Hugging Face hosts several such models under the image-classification pipeline."

---

**Q3. What is the difference between an LLM and an image classification model?**

> ✅ *Answer:*

| Feature | LLM (e.g. GPT-4o) | Image Classification Model |
|---|---|---|
| Input | Text | Image |
| Output | Text (generated) | Label/Category |
| Use Case | Q&A, summarization, extraction | Disease detection, object recognition |
| Example | GPT-4o, Claude | ResNet, ViT, EfficientNet |

> "For plant diagnosis, I used a vision model for disease detection. You can also use vision-language models like LLaVA for a more conversational experience."

---

## 4. Project 3 — LLM-Powered Content Discovery

### 📝 Resume Line
> *"Integrated large language model (LLM) capabilities using the OpenAI GPT-4o-mini chat API to enable natural-language-based content discovery driven by user intent, preferences, genre, and mood."*

---

### 🔍 What This Project Does (Simple Explanation)

```
User says: "Show me something feel-good, like a romantic comedy but not too cheesy"
        ↓
GPT-4o-mini understands: mood=feel-good, genre=romance/comedy, tone=light
        ↓
Maps to filters / search parameters
        ↓
Returns relevant content recommendations
```

Instead of dropdowns and filters, the user just *talks* to the system naturally.

---

### ❓ Probable Interview Questions & How to Answer

---

**Q1. Why GPT-4o-mini and not GPT-4o?**

> ✅ *Answer:*
> "GPT-4o-mini is a lighter, faster, and cheaper version of GPT-4o. For content discovery, the task doesn't require heavy reasoning — it's mainly understanding user intent and mapping it to parameters. GPT-4o-mini handles this well while being more cost-efficient for high-frequency API calls. If the task required complex reasoning or document analysis, I'd upgrade to GPT-4o."

---

**Q2. How did you handle user intent extraction?**

> ✅ *Answer:*
> "I designed a structured prompt that asked GPT-4o-mini to extract specific attributes from the user's natural language input — like genre, mood, tone, era, and preferences — and return them as a JSON object. This JSON was then used to query the content database. Prompt Engineering was key here to ensure consistent, structured output regardless of how the user phrased their request."

---

**Q3. What is the difference between GPT-4o and GPT-4o-mini?**

| Feature | GPT-4o | GPT-4o-mini |
|---|---|---|
| Performance | Higher accuracy, better reasoning | Slightly lower but very capable |
| Speed | Moderate | Faster |
| Cost | Higher | ~10x cheaper |
| Best For | Complex tasks, document analysis | Simple NLP, classification, intent parsing |

---

## 5. Core Concepts You Must Know

### What is an LLM?
A **Large Language Model** is an AI model trained on massive amounts of text data. It can understand and generate human-like text. Examples: GPT-4o, Claude, Gemini, LLaMA.

### What is RAG? (Retrieval-Augmented Generation)
Combining a search/retrieval system with an LLM. Instead of relying only on the model's training data, you first fetch relevant documents, then pass them to the LLM as context. Useful when you need up-to-date or domain-specific answers.

### What is Fine-tuning?
Taking a pre-trained model and training it further on your own dataset so it performs better on your specific task. Example: fine-tuning GPT on invoice data so it extracts fields more accurately.

### What is Prompt Engineering?
The practice of designing effective instructions/prompts for an LLM to get desired, accurate, and structured outputs.

### What is Chain-of-Thought (CoT)?
Prompting technique: "Think step by step before answering." Improves accuracy on complex, multi-step tasks.

### What is an AI Agent?
An autonomous AI system that can plan, decide which tools to use, call APIs, and complete multi-step tasks without human intervention at each step.

### What are Tokens?
Tokens are chunks of text the model processes — roughly 1 token ≈ 0.75 words. Models have a token limit per request (context window). GPT-4o has a 128K token context window.

### What is a Context Window?
The maximum amount of text (in tokens) an LLM can "see" and consider at one time. If your input exceeds this, older content gets cut off.

---

## 6. General AI/GenAI Interview Questions

**Q: What is the difference between AI, ML, and GenAI?**
> AI = broad field of making machines smart. ML = subset where machines learn from data. GenAI = subset of ML where models *generate* new content (text, images, code).

**Q: How do you handle hallucinations in LLMs?**
> Use structured prompts, validate outputs, use RAG to ground responses in real data, and add post-processing validation layers (like your staging table approach).

**Q: What is the difference between zero-shot, one-shot, and few-shot prompting?**
> Zero-shot = no examples given. One-shot = one example. Few-shot = multiple examples. More examples generally = better accuracy for specific formats.

**Q: What is the difference between a model and an API?**
> The model is the actual AI brain (weights, parameters). The API is the interface that lets you talk to it over the internet without hosting it yourself.

**Q: What is Azure OpenAI vs OpenAI directly?**
> Both give access to GPT-4o. Azure OpenAI is hosted on Microsoft's infrastructure — preferred for enterprise because of compliance, data privacy, SLAs, and integration with other Azure services like Document Intelligence and AI Agent.

---

## 🎯 Quick Revision Cheat Sheet

```
Invoice Pipeline:    Azure AI Agent → PDF Extraction → Staging Tables → Validation → PO/Vendor/Product
Plant Diagnosis:     User Image → Hugging Face Inference API → Vision Model → Disease + Treatment
Content Discovery:   User Text → GPT-4o-mini → Intent Extraction → Content Recommendations

Key Terms:
  LLM         = Large Language Model (GPT, Claude, LLaMA)
  Agent       = Autonomous AI that uses tools and makes decisions
  RAG         = Retrieval + LLM for grounded answers
  Fine-tuning = Training model further on your own data
  CoT         = Step-by-step reasoning prompt technique
  Tokens      = Units of text LLMs process (~0.75 words each)
  Staging     = Intermediate data layer for validation before production
```

---

*Good luck with your interviews! You've built real, production-relevant AI projects — own it confidently.* 🚀
