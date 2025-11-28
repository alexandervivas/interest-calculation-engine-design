# ADR-008: AI-Augmented Development Strategy

* **Status:** Accepted
* **Date:** 2024-XX-XX
* **Context:**
  * The Core Interest Engine (CIE) has strict architectural constraints: Modular Monolith, gRPC, Virtual Threads, and zero-tolerance Monetary Math rules.
  * Standard "autocomplete" AI tools often suggest generic, unsafe code (e.g., using `Double` or `REST`) that violates these constraints.
  * We need an AI strategy that can reason across multiple files and strictly adhere to our `.cursorrules` context.

## 1. Market Analysis & Evaluation

We evaluated four primary AI agent solutions against our specific requirements (Kotlin proficiency, rule adherence, and multi-file reasoning).

| Agent | Primary Model | Strengths | Weaknesses | Best For |
| :--- | :--- | :--- | :--- | :--- |
| **Cursor (Composer)** | Claude 3.5 Sonnet | **Rule Adherence**: Strictly follows `.cursorrules`.<br>**Multi-file**: Can refactor entire modules via "Composer".<br>**Speed**: Fast streaming (~2-5s). | Requires switching IDE from IntelliJ. | **Primary Development**. Writing domain logic and enforcing architecture. |
| **JetBrains AI** | GPT-4o / Gemini | **PSI Awareness**: Understands Kotlin semantics/syntax tree deeply.<br>**Integrated**: Native to IntelliJ. | Smaller context window.<br>Harder to enforce broad architectural rules via prompt. | **Safety & Polish**. Refactoring safely and writing KDocs. |
| **Aider (CLI)** | Claude 3.5 / GPT-4o | **Self-Healing**: Runs builds/tests and fixes its own errors.<br>**Git-Aware**: Auto-commits changes. | Slower (iterative loops).<br>CLI-based UX. | **Scaffolding & Ops**. Setting up Gradle, Docker, and writing boilerplate tests. |
| **GitHub Workspace** | GPT-4o | **Planning**: Great at turning Issues into Plans. | Execution often misses strict custom constraints (like "No JSON"). | **Planning**. High-level feature roadmap. |

## 2. Decision: The Hybrid Stack

We will adopt a **Hybrid AI Approach** to maximize velocity and correctness.

### 2.1 Primary Driver: Cursor (with Claude 3.5 Sonnet)
* **Role:** The "Senior Pair Programmer".
* **Usage:** All core domain logic, module creation, and architectural refactoring.
* **Configuration:** Must always run with the project's root `.cursorrules` active to enforce Money/gRPC constraints.
* **Feature:** Developers should use **Composer Mode (Cmd+I)** for multi-file feature implementation.

### 2.2 Secondary Support: Aider
* **Role:** The "Ops & Test Engineer".
* **Usage:**
    * fixing build scripts (`build.gradle.kts`).
    * writing Property-Based Tests (`jqwik`) where "brute force" generation is needed.
    * fixing Docker Compose networking issues (Self-healing loop).

### 2.3 Quality Gate: IntelliJ IDEA
* **Role:** The "Linter & compiler".
* **Usage:** Periodically opening the project in IntelliJ to run deep static analysis and resolve specific JVM dependencies that text-based AIs miss.

## 3. Operational Guidelines

1.  **Context Pinning:** The `.cursorrules` file is the source of truth for the AI. If the AI hallucinates (e.g., uses `Double`), **do not** just correct the code. Update `.cursorrules` to explicitly forbid that pattern.
2.  **Prompt Strategy:** Prompts should reference the documentation.
    * *Bad:* "Write the interest calculation."
    * *Good:* "Implement the interest calculation following the formula in `docs/domain/interest-math.md` and the type rules in `.cursorrules`."
3.  **Code Review:** AI-generated code is treated as "Junior Engineer" code. It requires human review, specifically focusing on:
    * Math correctness.
    * Partition pruning in SQL.
    * Proper gRPC error mapping.

## 4. Consequences

* **Positive:**
    * Drastic reduction in boilerplate code generation.
    * Automated enforcement of "boring" rules (DTOs, Proto files).
    * "One-shot" refactoring of cross-module dependencies.
* **Negative:**
    * Tooling Cost (Cursor subscription).
    * Risk of "Review Fatigue" where developers blindly accept AI code. (Mitigated by strict Property-Based Testing requirements).
