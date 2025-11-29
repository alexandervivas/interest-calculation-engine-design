# ADR-009: AI-Driven Pull Request Reviews (The Gemini Guardian)

* **Status:** Accepted
* **Date:** 2024-XX-XX
* **Context:**
  * The system has strict architectural constraints (1024 partitions, BigDecimal precision, Avro serialization) that are easily missed during manual review or by standard linters.
  * Using the same AI model for coding and reviewing leads to "bias confirmation" (the reviewer agrees with the coder's hallucinations).
* **Decision:**
  * Implement an automated **AI PR Reviewer** using **CodiumAI (PR-Agent)**.
  * **Model Selection:** The underlying engine must be **Google Gemini 1.5 Pro**.
  * **Rationale:**
    1.  **Context Window:** Gemini's 1M+ token window allows injecting the *entire* `docs/` folder (ADRs, Architecture) into the review context, enabling it to spot violations of documented decisions.
    2.  **Separation of Concerns:** The coding agent (Junie) uses **Claude 3.5 Sonnet**. Using **Gemini** for reviews ensures a different reasoning path ("Adversarial AI"), reducing the chance of shared blind spots.
* **Consequences:**
  * **Positive:** Continuous auditing of code against ADRs. Reduced human fatigue on syntax/boilerplate reviews.
  * **Negative:** Requires managing Google Vertex AI credentials/API keys in CI/CD.