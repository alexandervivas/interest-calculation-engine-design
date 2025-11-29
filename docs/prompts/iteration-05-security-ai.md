# Iteration 5: Security Hardening & AI "Guardian" Layer

**Goal:** Secure the application secrets, enforce dependency safety, and implement the "AI Assurance" components (Simulation Sandbox and Anomaly Detection).
**Definition of Done:** 1. OWASP Dependency Check runs during build.
2. A `SimulationService` exists that can dry-run config changes without affecting the real ledger.
3. An `AnomalyDetector` flags suspicious interest calculations to a dead-letter queue.

---

## 1. Security & Dependency Scanning (Agent: Aider)

**Context:** We need to ensure we aren't shipping vulnerabilities and that our secrets (DB passwords) aren't hardcoded.
**Agreements:** Use OWASP Dependency Check. Use Spring Security for Actuator endpoints.

**Prompt:**
```text
You are a Security Engineer. Harden the application.

1. **Supply Chain Security:**
   - Add the OWASP Dependency Check plugin to `build.gradle.kts` (`org.owasp.dependencycheck`).
   - Configure it to fail the build if a High Severity (CVSS > 7.0) vulnerability is found.

2. **API Security:**
   - Add `implementation("org.springframework.boot:spring-boot-starter-security")`.
   - Configure `SecurityFilterChain` in a new `SecurityConfig.kt`:
     - Require **Basic Auth** for all `/actuator/**` endpoints.
     - Allow anonymous access to `/health` (for K8s probes).
     - Set default user/pass in `application.yml` via environment variables (`${ADMIN_USER}`, `${ADMIN_PASS}`).

3. **Secrets Management:**
   - Update `docker-compose.yml` to use a `.env` file for Postgres and Kafka passwords.
   - Ensure no hardcoded credentials remain in `application.yml`.

```

---

## 2. AI Use Case 1: The Simulation Engine (Agent: Junie)

**Context:** Implementing **FR-AI-1**. Before an AI Agent (or human) applies a new Interest Rate Config, we must simulate it against a subset of users to prevent financial disasters. We need the "Sandbox" logic.

**Prompt:**
```text
You are a Senior Backend Engineer. Implement the Pre-Flight Simulation Engine.

1. **Create `SimulationService`:**
   - Method: `runSimulation(proposedConfig: InterestRule, sampleSize: Int): SimulationResult`.
   - Logic:
     1. Fetch `sampleSize` random accounts from `DailyPositionRepository`.
     2. For each account:
        - Calculate interest using the *Current* Active Config (Baseline).
        - Calculate interest using the *Proposed* Config (Scenario).
     3. Aggregate the results.

2. **Define `SimulationResult`:**
   - Data class containing:
     - `totalCurrentLiability: BigDecimal`
     - `totalProposedLiability: BigDecimal`
     - `variancePercentage: Double`
     - `impactedAccountsCount: Int` (How many accounts saw a change).

3. **Safety:**
   - Ensure this service is **Read-Only**. It must NOT save any `InterestAccrued` records to the database. It is a dry-run tool.

```

---

## 3. AI Use Case 2: Anomaly Detection Sidecar (Agent: Junie)

**Context:** Implementing **FR-AI-2**. We need a statistical guardrail that watches the output stream. If the engine calculates a massive outlier (e.g., $1M interest on a $100 balance), we must flag it.

**Prompt:**
```text
You are a Data Engineer. Implement the Anomaly Detector.

1. **The Detector Logic:**
   - Create `AnomalyDetector` component.
   - Implement a simple heuristic rule (Placeholder for future ML model):
     - **Rule A:** Interest Amount > 5% of Principal Balance (Unlikely for daily accrual).
     - **Rule B:** Interest Amount is Negative AND Balance is Positive (Logic error).
     - **Rule C:** Interest Amount > $10,000 (Hard cap for manual review).

2. **Stream Integration:**
   - Update the `InterestPublisher` (from Iteration 3).
   - Before publishing to `interest-events`:
     - Pass the result through `AnomalyDetector.check(event)`.
     - **If Anomalous:** Publish to a new topic `interest-anomalies` (or DLQ) and increment a metric `interest.anomaly.detected`.
     - **If Safe:** Publish to `interest-events` as normal.

3. **Test:**
   - Write a Unit Test `AnomalyDetectorTest` that feeds in a "Poison Logic" event (e.g., 50% interest) and asserts it returns `IS_ANOMALY`.