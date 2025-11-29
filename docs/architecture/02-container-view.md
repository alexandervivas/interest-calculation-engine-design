# C4 Container Diagram

## 1. Architectural Style
The system follows a **Stream-First (Kappa) Architecture** with local state optimization.
* **Communication:** Asynchronous via Kafka (for data) and REST (for config/management).
* **Concurrency:** Virtual Threads (Project Loom) for high-throughput I/O.

## 2. Container Diagram (Mermaid)

```mermaid
C4Container
    title Container Diagram - Core Interest Engine (Refined)

    Container_Boundary(cie_boundary, "Interest Engine Cluster (K8s)") {
        
        Container(ingest, "Ingestion Service", "Kotlin/Spring", "Consumes transactions, updates 1,024 partitioned state stores locally.")
        Container(calc, "Calculation Core", "Kotlin/Virtual Threads", "Pure domain logic. Computes accruals and delta-adjustments.")
        Container(post, "Posting Service", "Kotlin/Spring", "Batches interest events (Avro) to the Ledger.")
        
        ContainerDb(state_store, "Partitioned State", "PostgreSQL/Pinot", "Sharded storage for Account Balances (Historical & Current).")
        ContainerDb(audit_log, "Audit Log", "TimescaleDB", "Immutable append-only log of every math step.")
        
        Container(ai_anomaly, "Anomaly Detector", "Python/Flink", "AI Sidecar: statistically scores outbound events for risk.")
    }

    System_Ext(ledger, "Thin Ledger", "Source of Truth")
    System_Ext(kafka, "Kafka Cluster", "Event Backbone (1024 Partitions)")
    System_Ext(registry, "Schema Registry", "Governs Avro Schemas")

    Rel(ledger, kafka, "Publishes Tx", "Avro")
    Rel(kafka, ingest, "Consumes Batch", "Group: cie-ingest")
    
    Rel(ingest, registry, "Validates Schema", "HTTPS")
    Rel(ingest, state_store, "Upsert Balance", "JDBC / Shard-Key")
    
    Rel(ingest, calc, "Invokes Logic", "In-Memory")
    Rel(calc, audit_log, "Writes Trace", "Async")
    
    Rel(calc, post, "Emits Result", "Internal Channel")
    Rel(post, ai_anomaly, "Stream Check", "Async Sidecar")
    Rel(post, kafka, "Publishes Interest", "Avro")
```
## 3. Interface Standards
* **Async API:** CloudEvents 1.0.2.
* **ID Format:** UUID v7 (Time-sortable) for all Event IDs.

## 4. Component Responsibilities
* **Ingestion Service**: Responsible for "De-noising" the ledger stream. It converts thousands of raw transactions into a single "End-of-Day Balance" record per account per day.
* **Calculation Core**: Stateless logic. Input: `[History, Rules]`. Output: `[Accrual, Adjustment]`.
* **State Store**: A specialized store (Pinot or partitioned Postgres) designed to answer "What was the balance of Account X on Day Y?" in < 10ms.