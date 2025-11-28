# Non-Functional Requirements (NFR)

## 1. Performance & Scalability
* **NFR-1 (Throughput):** System must sustain ~7,000 account calculations per second to meet the 4-hour window for 100M accounts.
* **NFR-2 (Latency):** Interactive queries (e.g., "Preview Interest") must respond in < 100ms (P95).
* **NFR-3 (Horizontal Scaling):** System must support adding consumer nodes without downtime (leveraging the 1,024 partition strategy).

## 2. Reliability & Correctness
* **NFR-4 (Precision):** Internal calculations use Micros (Long) or BigDecimal with 9 decimal places. Rounding to 2 decimal places occurs **only** at the final Ledger Posting step.
* **NFR-5 (Isolation):** A failure in one partition (e.g., "Poison Pill" message) must not halt the entire system. Only the affected partition should stall/DLQ.

## 3. Observability
* **NFR-6 (Tracing):** 100% of calculations must emit OpenTelemetry spans.
* **NFR-7 (Alerting):** Variance in total daily interest accrual > 5% vs previous day triggers critical PagerDuty alert.

## 4. Security
* **NFR-8 (Immutable Logs):** Interest Accrual Logs are append-only. No updates/deletes allowed on historical records.