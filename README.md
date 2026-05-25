# AI Engineering & System Architecture Case Studies

Welcome! This repository serves as a technical portfolio showcasing the architecture, system design, and engineering methodologies of my core independent projects. 

Because these systems interact with paid production APIs (Anthropic, OpenAI) or proprietary business logic, the raw source code repositories are kept private. However, full architectural specifications, design rationales, and interface walkthroughs are documented transparently below.

---

## 🛠️ The Projects

### 1. Amazon Ads AI Optimization
**A two-layer keyword and listing analysis system that combines a deterministic rules engine with a multi-step AI pipeline to drive Amazon advertising decisions.**
* **Core Highlights:** Rules engine + LLM cascade separation, scenario-mode context injection, fan-out parallel listing analysis, ClickHouse historical analytics, and full per-call audit logging.
* **Tech Stack:** TypeScript, Next.js, Supabase, ClickHouse Cloud, Anthropic API.
* 👉 **[Read the Full System Breakdown](../amazon-ads-optimization/README.md)**

### 2. Atom Process Engine
**A domain-agnostic AI pipeline engine where markdown files define processing logic and a thin infrastructure layer executes them.**
* **Core Highlights:** Dependency-graph orchestration, runtime parallel execution, strict boundary schema validation, and real-time Supabase streaming.
* **Tech Stack:** TypeScript, Next.js, Supabase, Anthropic API, OpenRouter.
* 👉 **[Read the Deep-Dive Architecture & View Screenshots](./atom-process-engine/README.md)**

### 3. NexusBrain
**A multi-tenant, config-driven AI chatbot engine that qualifies leads and books appointments via SMS.**
* **Core Highlights:** Anti-hallucination code-level state machine gating, Fernet credential encryption at rest, 3-provider fallback cascade, and webhook duplicate guarding.
* **Tech Stack:** Python, Modal (Serverless), Supabase (pgvector), FastAPI, LangChain, Claude, GoHighLevel CRM.
* 👉 **[Read the Full System Breakdown](./nexusbrain/README.md)**

---

## 🔒 Code Review & Access
If you are a hiring manager or technical interviewer who wishes to inspect the raw source code or underlying database schemas for a hiring evaluation, I am happy to grant temporary GitHub read access or provide a repository walkthrough.

📧 **Contact:** via my application resume link.