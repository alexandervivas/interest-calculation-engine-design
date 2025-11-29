# Iteration 6: Operations, Deployment & Explainability

**Goal:** Prepare for Production. Dockerize the application securely, generate Kubernetes manifests, and implement the "Explainability" data assembler (for AI-3).
**Definition of Done:** A production-ready `Dockerfile` exists, Kubernetes manifests are valid, and the system can export a "Calculation Trace" for human/AI review.

---

## 1. Production Dockerization (Agent: Aider)

**Context:** We need a secure, minimal container image. The "Fat Jar" from Gradle is good, but we want a Multi-Stage build to keep the image small and secure (Distroless).
**Agreements:** Use `gcr.io/distroless/java21-debian12`.

**Prompt:**
```text
You are a DevOps Engineer. Create the Production Dockerfile.

1. **Multi-Stage Build:**
   - Create `Dockerfile` at the root.
   - **Stage 1 (Build):** Use `gradle:8.5-jdk21` image. Copy src, build.gradle.kts, settings.gradle.kts. Run `gradle clean build -x test --no-daemon`.
   - **Stage 2 (Runtime):** Use `gcr.io/distroless/java21-debian12` (or `eclipse-temurin:21-jre-alpine` if distroless is too restrictive for debugging, but prefer distroless for security).
   - Copy the JAR from Stage 1 to `/app/app.jar`.

2. **Entrypoint:**
   - Command: `java -jar /app/app.jar`.
   - Ensure environment variables (DB_URL, KAFKA_BOOTSTRAP) can be passed in.

3. **Optimization:**
   - Add a `.dockerignore` to exclude `.git`, `build`, `.gradle`, `test` results.

```

---

## 2. Kubernetes Manifests (Agent: Aider)

**Context:** We need to deploy this to K8s. We need the basic manifests for the Application and the ConfigMaps.
**Agreements:** Standard K8s YAMLs.

**Prompt:**
```text
You are a Cloud Engineer. Generate Kubernetes Manifests.

1. **Directory:**
   - Create `k8s/base/`.

2. **Deployment (`deployment.yaml`):**
   - Replicas: 2.
   - Image: `core-interest-engine:latest`.
   - Resources: Requests (CPU: 500m, RAM: 1Gi), Limits (CPU: 2000m, RAM: 2Gi).
   - **Environment Variables:**
     - `SHARD_COUNT`: Value **"1024"** (Critical: Overrides the local default of 16).
     - `KAFKA_PARTITION_COUNT`: Value **"1024"**.
   - Probes:
     - Liveness: HTTP GET `/actuator/health/liveness` on port 8080.
     - Readiness: HTTP GET `/actuator/health/readiness` on port 8080.
   - EnvFrom: ConfigMapRef `cie-config`, SecretRef `cie-secrets`.

3. **Service (`service.yaml`):**
   - Type: ClusterIP.
   - Port 80 (Target 8080).

4. **KEDA Autoscaling (`scaledobject.yaml`):**
   - Reference the `deployment`.
   - Trigger: `kafka`.
   - Metadata: `bootstrapServers`, `consumerGroup`, `topic`, `lagThreshold: "10000"`.

```

---

## 3. AI Use Case 3: Trace Data Assembler (Agent: Junie)

**Context:** Implementing **AI-3** (Natural Language Auditor). Before an LLM can explain "Why interest was $0.05", we need code to fetch all the raw ingredients (Daily Balance, Rate Config, Math Steps) and bundle them into a context object.

**Prompt:**
```text
You are a Senior Backend Engineer. Implement the Trace Assembler.

1. **Create `TraceService`:**
   - Method: `assembleContext(accountId: String, date: LocalDate): CalculationContext`.

2. **Implementation Logic:**
   - Fetch `DailyPosition` for the specific date.
   - Fetch the `InterestRule` that was active on that date.
   - Fetch the specific `InterestAccrued` record (if it exists).
   - *Key Step:* Re-run the calculation logic in "Debug Mode" (if supported) or manually construct a `CalculationStep` list:
     - "Balance = 100.00"
     - "Tier Applied = Gold (> 50.00)"
     - "Rate = 0.05 / 365"
     - "Raw Result = 0.01369..."

3. **Output Format:**
   - The `CalculationContext` class should override `toString()` to produce a clean, Markdown-formatted block.
   - *Why:* This string will be the "System Prompt" fed to an external LLM agent in the future.
   - Example Output:
     ```markdown
     # Context for Account 123 on 2024-01-01
     * **EOD Balance:** 100.00 USD
     * **Applied Rate:** 5.0% (Tier: Gold)
     * **Math:** 100 * 0.05 / 365
     ```

```

---

## 4. Final Smoke Test (Agent: Junie)

**Context:** The system is done. We need a script to verify the whole loop.

**Prompt:**
```text
You are a QA Engineer. Create the End-to-End Smoke Script.

1. **Script (`scripts/smoke_test.sh`):**
   - Requires `curl`, `jq`, and access to the Kafka container.
   - **Steps:**
     1. Wait for `http://localhost:8080/actuator/health` to be UP.
     2. Post a dummy transaction via a helper endpoint (or produce to Kafka).
     3. Sleep 5 seconds.
     4. Query the `TraceService` (expose a temporary GET endpoint `/api/debug/trace/{id}/{date}`) to see if the calculation happened.
     5. Check the `prometheus` endpoint to ensure `interest_calculation_total` > 0.

2. **Execution:**
   - Make the script executable.

```
