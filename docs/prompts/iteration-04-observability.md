# Iteration 4: Observability & Performance

**Goal:** Turn on the lights. Implement distributed tracing, business metrics, and performance optimizations (Batching & Caching) to meet the 100M/day throughput target.
**Definition of Done:** 1. `docker compose up` includes Grafana/Prometheus/Jaeger.
2. A Load Test (k6) runs against the local stack, processing 10k events, and traces appear in Jaeger.

---

## 1. Observability Stack & Tracing (Agent: Aider)

**Context:** We need to see what's happening inside the black box. We will use OpenTelemetry for traces and Micrometer/Prometheus for metrics.
**Agreements:** * Export traces to Jaeger (OTLP).
* Export metrics to Prometheus.
* Propagate trace context over Kafka headers.

**Prompt:**
```text
You are a DevOps Engineer. Setup the Observability Infrastructure.

1. **Docker Compose:**
   - Add `jaeger` (all-in-one image) exposing ports 16686 (UI) and 4317 (OTLP gRPC).
   - Add `prometheus` configured to scrape the application on port 8080 (`/actuator/prometheus`).
   - Add `grafana` depends on prometheus.

2. **Gradle Dependencies:**
   - Add `implementation("org.springframework.boot:spring-boot-starter-actuator")`
   - Add `implementation("io.micrometer:micrometer-registry-prometheus")`
   - Add `implementation("io.opentelemetry.javaagent:opentelemetry-javaagent:1.32.0")` (or the Spring Boot Starter for OTel if preferred).
   - Add `implementation("com.github.ben-manes.caffeine:caffeine")` (for caching later).

3. **Configuration:**
   - `application.yml`: 
     - Enable actuator endpoints (`health`, `info`, `prometheus`, `metrics`).
     - Configure OTel exporter endpoint to `http://jaeger:4317`.
     - Set `management.tracing.sampling.probability: 1.0` (for dev).

```

---

## 2. Metrics & Caching Logic (Agent: Junie)

**Context:** We need specific business metrics (not just CPU usage) and caching to reduce DB load.
**Agreements:** * Cache `ProductConfig` (Read-mostly).
* Metric: `interest.calculation.executed` (Counter).
* Metric: `interest.batch.size` (DistributionSummary).

**Prompt:**
```text
You are a Senior Backend Engineer. Instrument the code.

1. **Caching Strategy:**
   - Enable Caching in Spring Boot.
   - Decorate `ProductConfigRepository.findActiveConfig(...)` with `@Cacheable("product-configs")`.
   - Configure Caffeine to expire entries after 10 minutes.

2. **Custom Metrics:**
   - Inject `MeterRegistry` into `InterestCalculator`.
   - Increment a counter `interest_calculation_total` (tags: `status=success|failure`) every time a calculation runs.
   - Record a timer `interest_calculation_latency` around the math logic.

3. **Trace Propagation:**
   - Verify (or configure) that the Kafka Consumer reads the `traceparent` header to continue the trace started by the Producer. (Spring Boot 3 + OTel usually does this automatically, but verify via config).

```

---

## 3. High-Performance Batch Consumer (Agent: Junie)

**Context:** Processing messages one-by-one is too slow for 100M accounts. We need to implement the "Smart Batch" logic from the Architecture.
**Agreements:** Use Spring Kafka Batch Listener.

**Prompt:**
```text
You are a Performance Engineer. Refactor the Consumer for Batch Processing.

1. **Batch Configuration:**
   - Update `application.yml` for Kafka Consumer:
     - `max-poll-records: 5000`
     - `fetch-min-size: 1MB`
     - `wait-max-ms: 500`
     - `listener.type: batch`

2. **Refactor Ingestion Service:**
   - Change `consume(record)` to `consumeBatch(records: List<ConsumerRecord>)`.
   - **Optimization Logic:**
     - `val distinctAccounts = records.map { it.accountId }.distinct()`
     - `val currentBalances = repository.findAllById(distinctAccounts)` (Bulk Read).
     - Iterate through records in memory, updating the Map of balances.
     - `repository.saveAll(updatedBalances)` (Bulk Write).
   - *Goal:* Turn 5000 DB calls into 2 DB calls (Read + Write).

3. **Error Handling:**
   - If a batch fails, fall back to "Retry with smaller batch" or "Process one-by-one" (BatchErrorHandler).

```

---

## 4. Load Testing (Agent: Aider)

**Context:** We need to prove the system handles load.
**Tool:** k6 (JavaScript-based load testing tool).

**Prompt:**
```text
You are a QA Automation Engineer. Create a Load Test Script.

1. **Script Generation:**
   - Create `tests/load/k6-script.js`.
   - The script should:
     - Connect to the Kafka Broker (using a k6-kafka extension or just hitting a Test Controller endpoint if we exposed one. *Decision: Create a simple HTTP Controller `POST /debug/seed` that accepts count=10000 and publishes that many random events to Kafka internally*).
   - Trigger the generation of 10,000 transaction events.

2. **Execution:**
   - Provide the command to run the test and observe Grafana panels.

```
