# Functional Requirements (FR)

| ID | Priority | Feature | Description | Acceptance Criteria |
| :--- | :--- | :--- | :--- | :--- |
| **FR-1** | **Must** | **High-Scale Processing** | Process interest accruals for 100M+ accounts daily. | Benchmark: 100M accounts processed in < 4 hours (~7k/sec). |
| **FR-2** | **Must** | **Idempotent Posting** | Interest posting to the ledger must be idempotent. | Re-running a batch results in exactly 0 duplicate ledger entries. |
| **FR-3** | **Must** | **Exactly-Once Processing** | No events lost, no events processed twice effectively. | Kafka `isolation.level=read_committed`, transactional producers enabled. |
| **FR-4** | **Must** | **State Recovery** | System must recover state after crash/restart. | RTO < 5 minutes. Resume from last committed Kafka offset. |
| **FR-5** | **Must** | **Audit Traceability** | Every accrual must be traceable to a balance and rate. | API exists to retrieve calculation inputs for any historical day. |
| **FR-6** | **Must** | **Zero Money Error** | Strict decimal precision handling. | Use `BigDecimal` (Java). Rounding Mode: `HALF_EVEN`. Precision: 9 decimals internal. |
| **FR-7** | **Medium** | **Schema Governance** | Events must use Avro with Schema Registry validation. | 100% of produced events pass Schema Registry compatibility checks. |
| **FR-8** | **Must** | **Backdated Handling** | Automatically handle transactions effective in the past. | Detect $t_{val} < t_{current}$, trigger Delta-Adjustment logic. |
| **FR-8.1** | **Must** | **Delta Calculation** | Compute difference between *Ideal* and *Actual* interest. | Logic: $\Delta = I_{new} - I_{paid}$. Post adjustment entry. |
| **FR-8.2** | **Must** | **Tier Crossing** | Re-evaluate interest tiers during backdated replays. | If backdate changes avg balance, apply correct tier rate to history. |