# ADR-009: AI-Driven Pull Request Reviews (The Gemini Guardian)

* **Status:** Accepted
* **Date:** 2024-XX-XX
* **Context:**
  * The system has strict architectural constraints (1024 partitions, BigDecimal precision, Avro serialization).
  * We need an automated auditor to enforce these rules on every PR.
* **Decision:**
  * Implement an automated **AI PR Reviewer** using **Qodo (formerly CodiumAI)** technology.
  * **Tool:** We will use the open-source **`pr-agent`** (maintained by Qodo) running via GitHub Actions.
  * **Model Selection:** We will configure the agent to use **Google Gemini 1.5 Pro**.
  * **Rationale:**
    1.  **Context:** Gemini's 1M+ token window allows ingesting the entire `docs/` folder.
    2.  **Tooling:** Qodo's `pr-agent` provides the best "Chat with Code" UX on GitHub.
* **Consequences:**
  * **Positive:** Enterprise-grade review workflow with custom architectural rule injection.
  * **Negative:** Requires managing Google Vertex AI credentials.
