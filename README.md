# Sally — Production Customer Support AI Agent (n8n + MCP)

A 24/7 production customer support AI agent built in **n8n** for **Modupeola Ventures**, a wholesale and retail beverage distributor in Ilorin, Kwara State, Nigeria. Built as Week 1 of the **Stackreel AI Automation Internship** (Status: Complete).

Sally handles after-hours product inquiries, live stock verification, accurate multi-item price quotes, store policy lookups via **Model Context Protocol (MCP)**, and instant **Gmail human handoff**—locked down with strict business and anti-jailbreak guardrails.

---

## Demo & Architecture


> **2-Minute Walkthrough Video:** [Watch Demo](https://drive.google.com/file/d/1bF_mmQCfVKD0GLI8M3QswOgTOFjyS8_W/view?usp=drivesdk)

### System Architecture
- **Orchestration:** n8n AI Agent (`Max Iterations: 5`) with `Simple Memory` keyed to each customer's `sessionId`.
- **LLM Strategy:** Groq primary model for sub-4s latency, paired with `Retry on Fail` and Google Gemini fallback support to handle free-tier rate limits cleanly.
- **Tools (4 Total):**
  1. **Google Sheets (Live Inventory):** Reads the store's stock sheet, separating `Wholesale stock status` (cases/packs) from `Retail stock status` (single units), packaging types (Plastic PET vs. Bottle), and case sizes.
  2. **Calculator:** Enforced for every multi-item total, mixed-case calculation, and discount deduction to eliminate LLM arithmetic errors.
  3. **MCP Client → Dedicated n8n MCP Server:** Connects to a separate published MCP Server workflow serving verified store policies (opening hours, Sango Ilorin address, pickup-only terms, payment methods, and the 70% refundable empty-crate deposit).
  4. **Gmail (Human Handoff):** Sends a structured `[ESCALATION]` email to the store manager with customer name, request summary, and escalation reason in under 1 second.

---

## Business & Security Guardrails

- **No Credit Policy:** Strictly enforces full upfront payment; rejects installments or pay-later requests.
- **Discount Caps:** Zero discounts on retail or wholesale plastic items; hard **₦100 max total discount** on wholesale bottled orders regardless of case quantity.
- **Wholesale & Mix-and-Match Rules:** 1-full-case wholesale minimum; flavor mixing permitted only when items share identical wholesale price, packaging type, and case size.
- **Zero Hallucination & Fuzzy Typo Matching:** Never invents unstocked items; scans live inventory to catch common brand typos (e.g., `"Fiju Chocolate"` → *"Did you mean Viju Chocolate?"*).
- **Anti-Jailbreak & Off-Topic Protection:** Resists prompt-injection attempts (`"ignore your rules"`, fake owner claims), refuses off-topic tasks, never leaks system instructions, and escalates policy pushback, complaints, or manager requests via Gmail.

---

## Testing & Evaluation (15-Question Suite)

Tested across a structured **5 Normal / 5 Tricky / 5 Attacks** rubric covering stock lookups, ambiguous packaging, typo matching, out-of-stock handling, policy pushback, prompt injection, and live Gmail escalation.

- **Initial Groq-only run:** 13/15 Passed (2 multi-step Calculator calls hit Groq's 8,000 TPM free-tier ceiling).
- **Post-fix:** Resolved via ~55% system prompt compression, tightened memory windowing, and Gemini fallback routing.
- Full test log: [`docs/15-question-test-suite.md`](https://docs.google.com/document/d/12YEcKf54nfeaTjj2W4Ba7GfEiDGqmavhv4ALmbQ05cs/edit?usp=drivesdk))

---

## Engineering Challenges Solved

 **Groq 8,000 TPM Rate-Limit Optimization:** Loading a 39-row stock sheet plus chat history spiked requests to ~15,900 tokens. Compressed the system prompt by ~55% without losing guardrails, capped `Simple Memory` context length, and added Retry/Fallback handling to bring typical tool runs down to ~3.9s and ~4,800 tokens.

---

## Quick Setup

1. Import `workflows/mcp-store-policies-server.json` into n8n, connect your Google Sheets credential, and **Publish** the workflow.
2. Copy the MCP Server's **Production URL** (`/mcp/...`).
3. Import `workflows/sally-support-agent.json`, paste the MCP URL into the **MCP Client Tool**, and attach your Groq/Gemini, Google Sheets, and Gmail credentials.
4. Open the n8n Chat panel and test.
