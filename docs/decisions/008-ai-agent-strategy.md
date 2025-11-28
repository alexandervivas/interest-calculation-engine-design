# ADR-008: AI-Augmented Development Strategy

* **Status:** Accepted
* **Date:** 2024-XX-XX
* **Context:**
  * The Core Interest Engine (CIE) requires strict adherence to Modular Monolith boundaries, gRPC protocols, and specific "Money Math" rules (BigDecimal/Half-Even).
  * Standard "autocomplete" AI tools often suggest generic, unsafe code (e.g., using `Double` or `REST`) that violates these constraints.
  * We need an AI strategy that can reason across multiple files, strictly adhere to architectural rules, and operate safely within a Kotlin codebase.

## 1. Market Analysis & Evaluation

We evaluated three primary AI agent solutions against our specific requirements: **Kotlin proficiency**, **Architectural Rule Adherence**, and **Multi-file Reasoning**.

| Agent | Type | Key Capabilities | Strengths for CIE | Weaknesses |
| :--- | :--- | :--- | :--- | :--- |
| **JetBrains Junie** | **Native Agent** | **PSI-Native:** Uses the IDE's Abstract Syntax Tree (AST) for precise code understanding.<br>**Test Loop:** Can run tests, detect failures, and self-correct code.<br>**Guidelines:** Supports `.junie/guidelines.md` for project rules. | **Safety:** Highest reliability for refactoring (rename/move) without breaking builds.<br>**Verification:** Can automatically run `jqwik` property tests to verify math logic. | **Cost:** Requires AI Ultimate/Enterprise credits.<br>**Availability:** Newer product, evolving feature set. |
| **Cursor (Composer)** | **IDE Fork** | **Context Pinning:** Strict enforcement via `.cursorrules`.<br>**Speed:** Extremely fast multi-file editing (Composer mode).<br>**Model:** Claude 3.5 Sonnet optimized. | **Strictness:** Excellent at enforcing negative constraints (e.g., "NO JSON").<br>**Velocity:** Best for massive initial scaffolding. | **Fragility:** Text-based editing can sometimes break imports or Gradle configurations that a native IDE would catch. |
| **Aider** | **CLI Tool** | **Git-Ops:** Auto-commits changes.<br>**Self-Healing:** Reads compiler errors from the terminal and fixes code. | **Ops/Infra:** Best for setting up Docker, K8s manifests, and CI pipelines. | **UX:** Context switching between Terminal and IDE.<br>**Visibility:** Harder to review complex logic changes in real-time. |

## 2. Decision: JetBrains Junie

We have selected **JetBrains Junie** as the primary development agent for the Core Interest Engine.

### Rationale
1.  **The "Test-Loop" Capability:** Junie is unique in its ability to **run tests autonomously**. For a financial system, this is critical. We can ask Junie to "Implement the tiered interest logic to pass the `InterestMathTest` suite," and it will iterate until the test bar is green.
2.  **Context Pinning:** The discovery of `.junie/guidelines.md` allows us to enforce our strict architectural constraints (Modular Monolith, gRPC-only) just as effectively as Cursor, but natively within IntelliJ.
3.  **Compiler Awareness:** Unlike text-based agents, Junie understands Kotlin structure. It is significantly less likely to hallucinate non-existent methods or break strict typing rules.

## 3. Implementation Strategy

### 3.1 Configuration
* **Rules File:** We will maintain a strict system prompt in `.junie/guidelines.md` at the project root. This file will contain the "Zero Tolerance" rules for Money types, Protocol buffers, and Threading models.
* **Exclusions:** We will use `.aiignore` to prevent the agent from consuming build artifacts, logs, or sensitive configuration files, keeping the context window focused on Domain Logic.

### 3.2 Workflow
* **Domain Logic:** Developers will use Junie to implement core business rules, relying on its ability to run Property-Based Tests (`jqwik`) for verification.
* **Refactoring:** Large-scale structural changes (e.g., moving a package across modules) will be delegated to Junie due to its PSI safety.
* **Infrastructure:** (Optional) Aider may be used as a secondary tool for pure DevOps tasks (Docker/Kubernetes) if Junie struggles with non-Kotlin files.

## 4. Consequences

* **Positive:**
    * **Higher Correctness:** Automated test execution by the agent reduces the likelihood of subtle math bugs entering the codebase.
    * **Architectural Drift Prevention:** The Guidelines file ensures that even "lazy" prompts result in compliant code (e.g., gRPC instead of REST).
    * **Developer Experience:** No need to switch IDEs; leverages the existing IntelliJ ecosystem.
* **Negative:**
    * **Licensing Cost:** Usage requires specific JetBrains AI subscriptions.
    * **Latency:** Deep reasoning and test execution loops can take longer than simple text-streaming auto-complete.
