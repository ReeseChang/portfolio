# NexusBrain — Project Summary

> A config-driven AI chatbot engine that qualifies leads and books appointments via SMS, 24/7.

---

## What It Is

NexusBrain is an AI-powered SMS chatbot engine that acts as a 24/7 appointment booking agent for local businesses. A realtor, dental clinic, or any service business signs up once — and from then on, every lead who texts them gets an intelligent, structured conversation that qualifies them and drops a confirmed booking on the calendar without a human ever touching it.

**One deployment serves unlimited clients.** Each client's config, credentials, and conversation history are fully isolated. It's multi-tenant by design.

---

## The Problem It Solves

| Problem | What NexusBrain Does |
|---|---|
| Leads going cold after hours | Replies instantly, 24/7, no delay |
| Staff spending hours qualifying & scheduling | Bot handles the full conversation, start to finish |
| Generic chatbots with random questions | Follows a structured, industry-specific qualification flow |
| AI hallucinating or going off-script | State machine gates what the LLM can do — programmatically |
| Double-bookings from duplicate webhooks | Deduplication by content + timestamp window |
| Credentials exposed in the database | Fernet-encrypted at rest, keys in Modal secrets |

---

## How a Conversation Works

```
Lead texts in → Greeting → Qualification → Confirmation → Booking → Confirmed
```

1. **Lead texts in** — GHL webhook fires, NexusBrain receives the message
2. **Greeting** — bot acknowledges the lead and identifies intent
3. **Qualification** — bot collects required fields (budget, timeline, lead type, etc.)
4. **Confirmation** — bot recaps collected data and asks the lead to verify
5. **Booking** — bot checks the GHL calendar and books an appointment slot
6. **Confirmed** — bot sends a confirmation with the appointment time

---

## The Core Innovation: The Anti-Hallucination Moat

The state machine is the most important part of the system.

**Tools are gated per phase.** The LLM physically cannot call `book_appointment` until all required qualification fields are collected and confirmed. This is a code-level guarantee enforced by `state_machine.py` — not a prompt instruction the model could ignore or misinterpret.

```
Phase: QUALIFICATION
  → Tools available: [report_extracted_data, knowledge_base]
  → book_appointment: NOT available

Phase: BOOKING
  → Tools available: [get_calendar_slots, book_appointment]
  → report_extracted_data: NOT available
```

Each phase defines its own allowed tools, required fields, transition conditions, and max turns. The LLM operates within those constraints — it cannot advance or book out of sequence.

---

## Industries Supported (Template-Driven)

Industry templates are JSON configs stored in Supabase. Adding a new industry means writing a new JSON template — no code changes required.

### Real Estate
**Phases:** Greeting → Buyer Qualification → Seller Qualification → Confirmation → Booking → Confirmed

**Qualification fields:** lead type (buyer/seller), budget, area, timeline, pre-approval status

### Dental
**Phases:** Greeting → Patient Qualification → Insurance → Confirmation → Booking → Confirmed

**Qualification fields:** reason for visit, insurance provider, new vs returning patient

---

## Tech Stack

### Infrastructure
- **Modal** — serverless Python deployment (auto-scales, zero DevOps)
- **Supabase** — Postgres database + pgvector for RAG (knowledge base embeddings)
- **FastAPI** — HTTP endpoint that receives GHL webhooks

### AI / LLM
- **Anthropic Claude** — primary LLM (conversation + extraction)
- **OpenAI** — fallback LLM + embeddings for knowledge base
- **Google Gemini** — second fallback LLM
- **LangChain** — LLM orchestration and tool binding
- **LangSmith** — optional prompt tracing and observability

### CRM Integration
- **GoHighLevel (GHL)** — CRM, SMS, calendar booking, contact management

### Security
- **Fernet (cryptography)** — symmetric encryption for client credentials at rest

---

## Database Schema (6 Tables)

| Table | Purpose |
|---|---|
| `client` | One row per business location. Stores encrypted GHL tokens, timezone, industry. |
| `contact` | One row per lead. Resolved by (location_id, source, source_id). |
| `process` | The bot's configuration — phase template, prompts, A/B variant. |
| `conversation` | One active session per contact. Owns phase cursor, extracted data, turn counters. |
| `message` | Full message log. Append-only. One row per message. |
| `knowledge_base` | RAG documents + pgvector embeddings. Populated via `populate_kb.py`. |
| `event` | Audit trail per conversation turn. |
| `prompt` | Full system prompt per turn for debugging. |

---

## Security & Hardening

### Prompt Injection Defence
Every system prompt is prepended with a `SECURITY_PREAMBLE` instructing the model to ignore role-change requests, refuse to reveal its instructions, and stay focused on appointment booking — regardless of what the user message says.

### Abuse Detection
Before any processing, `_check_abuse()` scans the incoming message for known injection patterns (`"ignore previous instructions"`, `"jailbreak"`, `"act as if"`, etc.) and enforces a rate limit of 10 messages per contact per 60 seconds. Returns HTTP 429 on abuse.

### Encrypted Credentials
Every client's GHL API token and security token is Fernet-encrypted before being stored in Supabase. The encryption key lives in Modal secrets — never in the database.

### Duplicate Message Guard
Webhooks from GHL can fire more than once for the same message. The bot detects duplicates by checking content + timestamp window and silently drops the repeat without double-replying or double-processing.

### LLM Cascade Resilience
If the primary LLM (Claude) fails or times out, the system automatically retries with OpenAI, then Gemini. The business never sees a dropped or unanswered message.

### Human Handover
When a GHL `nb:human-handover` tag is detected on a contact, the bot goes fully silent. A human agent can take over the conversation without the bot interrupting.

### Config Validation on Startup
All required client and process config fields are validated at boot. The system fast-fails with a clear error message rather than silently misbehaving at runtime.

---

## Key Protections for Businesses

| Risk | Protection |
|---|---|
| Lead replies after hours, no one responds | Bot responds instantly, always |
| Bot books before qualifying | State machine prevents this at code level |
| AI invents answers not in the knowledge base | RAG grounds answers in configured FAQs |
| Competitor or bad actor spams the webhook | Rate limiting + injection detection |
| Staff member leaves, credentials lost | All credentials encrypted + centrally stored |
| GHL webhook double-fires | Duplicate guard prevents double-booking |
| LLM provider has an outage | Three-provider fallback cascade |
| Human needs to take over | `nb:human-handover` tag silences bot immediately |

---

## Onboarding a New Client

The process takes approximately 20 minutes:

1. Collect client details (GHL credentials, timezone, industry, qualification fields)
2. Run `onboard_customer.py` — creates all database rows and loads the industry template
3. Run `populate_kb.py` — chunks the client's FAQ text and loads it into the knowledge base
4. Set the GHL webhook to the Modal endpoint URL
5. Done — the bot is live

---

## Roadmap

| Feature | Status |
|---|---|
| Voice integration (LiveKit + Deepgram) | Planned — post-launch |
| No-GHL widget (chatbot embed + email/Sheets delivery) | Planned |
| A/B testing (variant comparison + promotion scripts) | Built, triggers at 500+ leads/month |
| Multi-location franchise support | Planned |
| Contact memory with expiry (returning lead recognition) | Planned |
| Owner approval flow (pending bookings) | Planned |
| Smart Q&A (flag unanswered questions, email owner) | Planned |
| Self-serve onboarding UI | Planned (v2) |

---

## For Non-Technical Readers

Imagine hiring a staff member who:

- Reads every incoming text **instantly**, day or night
- Asks exactly the **right qualifying questions** in the right order, every time
- **Never books** before they've collected all the information you need
- Checks your **calendar** and confirms the appointment on the spot
- Handles **every lead simultaneously**, across every client location
- **Goes silent** the moment a human agent needs to take over
- **Never makes things up** — only answers from your approved FAQ content

That's NexusBrain. One setup. Then it just runs.

---

*Stack: Python · Modal · Supabase (Postgres + pgvector) · FastAPI · LangChain · Anthropic Claude · GoHighLevel CRM*
