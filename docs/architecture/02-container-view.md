# C4 Container Diagram

## 1. Architectural Style
The system follows a **Stream-First (Kappa) Architecture** with local state optimization.
* **Communication:** Asynchronous via Kafka (for data) and REST (for config/management).
* **Concurrency:** Virtual Threads (Project Loom) for high-throughput I/O.

## 2. Container Diagram (Mermaid)

```mermaid
C4Container
    title Container Diagram - Core Interest Engine

    Container_Boundary(cie_boundary, "Interest Engine Cluster") {
        
        Container(ingest, "Ingestion Service", "Kotlin/Flink", "Consumes Ledger events, aggregates daily positions, updates State Store.")
        Container(calc, "Calculation Core", "Kotlin", "The brain. Fetches state, applies math rules, computes accruals & deltas.")
        Container(post, "Posting Service", "Kotlin", "Batches interest entries and reliably sends them to the Ledger.")
        Container(api, "Management API", "Spring Boot", "Exposes endpoints for Config, Audit, and Manual Triggers.")
        
        ContainerDb(state_store, "State Store", "Apache Pinot / RocksDB", "Read-optimized view of Daily Balances (Partitioned).")
        ContainerDb(config_db, "Config DB", "PostgreSQL", "Stores Products, Rates, and Rules.")
        ContainerDb(audit_log, "Audit Log", "PostgreSQL/Timescale", "Immutable history of all accrual records.")
    }

    System_Ext(kafka, "Kafka Cluster", "Event Backbone")

    Rel(kafka, ingest, "Stream: tx-events", "Consumer Group")
    Rel(ingest, state_store, "Upsert Balance", "KV/OLAP")
    
    Rel(api, config_db, "Read/Write Configs", "JDBC")
    
    Rel(calc, state_store, "Fetch History", "Query")
    Rel(calc, config_db, "Fetch Rules", "JDBC + Cache")
    Rel(calc, audit_log, "Append Accrual", "Batch Insert")
    Rel(calc, post, "Notify Ready", "Internal Event")
    
    Rel(post, kafka, "Publish: interest-commands", "Transactional Producer")
```

## 3. Component Responsibilities
* **Ingestion Service**: Responsible for "De-noising" the ledger stream. It converts thousands of raw transactions into a single "End-of-Day Balance" record per account per day.
* **Calculation Core**: Stateless logic. Input: `[History, Rules]`. Output: `[Accrual, Adjustment]`.
* **State Store**: A specialized store (Pinot or partitioned Postgres) designed to answer "What was the balance of Account X on Day Y?" in < 10ms.