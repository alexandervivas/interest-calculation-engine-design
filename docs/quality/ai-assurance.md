# AI Assurance & Safety Rails

## 1. The "Guardian" Architecture
We deploy AI agents as **Sidecars**, not as primary decision-makers in the critical path.

### 1.1 Component: The Config Auditor (Pre-Flight)
* **Trigger:** PR created for `product-config.json`.
* **Action:**
    1.  Parses the new rate structure.
    2.  Spins up a Sandbox CIE instance.
    3.  Replays yesterday's traffic (10k sampled accounts).
    4.  Compares Total Liability: `(New_Interest - Old_Interest)`.
* **Gate:** If Liability Shift > Threshold (e.g., +20%), Block PR and alert Human.

### 1.2 Component: Anomaly Detector (Post-Flight)
* **Trigger:** Stream of `InterestAccrued` events.
* **Model:** Isolation Forest (Unsupervised).
* **Signal:** "This specific accrual vector `[Bal=-500, Rate=0.05, Int=+20.0]` is highly improbable."
* **Action:** Route message to `DLQ-Audit` instead of `Ledger-Outbox`.

### 1.3 Component: The Code Guardian (PR Reviewer)
* **Trigger:** Every Pull Request (Code Change).
* **Agent:** CodiumAI powered by **Gemini 1.5 Pro** (See ADR-009).
* **Role:** Architectural Compliance & Static Analysis.
* **Mechanism:**
    * The agent ingests the Code Diff + System Documentation (ADRs/Architecture).
    * It checks for specific "Forbidden Patterns" defined in the configuration (e.g., Floating Point math, JSON serialization, Hardcoded Sharding).
* **Gate:** The agent posts comments on the PR. "Blocking" comments regarding architectural violations must be resolved before merging.

## 2. Explainability (RAG)
* **Goal:** Enable Tier-1 Support to explain math without code access.
* **Mechanism:**
    * **Context:** Daily Trace Logs (JSON) stored in Elasticsearch.
    * **LLM:** "Given this Trace JSON, explain in simple English why the interest was $0.05."
    * **Safety:** The LLM does **not** calculate. It only translates the structured log (which contains the math steps) into natural language.