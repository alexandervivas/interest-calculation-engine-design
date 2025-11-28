# Core Interest Engine (CIE) - Project Home

## 1. Context & Scope
The **Core Interest Engine (CIE)** is a specialized high-throughput financial system designed to calculate, accrue, and post interest for a Tier-1 scale banking ledger. It operates as a "Thin Ledger" sidecar, maintaining its own read-optimized state to avoid overwhelming the transactional core.

* **Primary Objective:** Process daily interest accruals for **100M+ accounts** within a 4-hour batch window.
* **Critical Constraint:** Zero monetary error tolerance. 100% auditability.
* **Key Challenge:** Handling retroactive (backdated) changes to balances without rewriting history, using a Delta-Adjustment strategy.

## 2. Technical Snapshot
* **Language:** Kotlin (JVM 21+) using Virtual Threads (Project Loom).
* **Architecture:** Event-Driven Kappa Architecture (Stream-first) with local state snapshots.
* **Infrastructure:** Kubernetes (K8s), KEDA, Kafka (Events), PostgreSQL (Config), Apache Pinot/RocksDB (State).
* **Sharding:** 1,024 static partitions aligned by `hash(account_id)`.

## 3. Navigation
* [Requirements](./requirements/) - Functional, Non-Functional, and AI goals.
* [Architecture](./architecture/) - C4 Diagrams, topology, and infrastructure.
* [Domain Logic](./domain/) - Math formulas, backdating rules, and product configuration.
* [Decisions (ADRs)](./decisions/) - Log of architectural decisions and trade-offs.
* [Quality & Safety](./quality/) - Testing strategies and AI Assurance layers.

> **Status:** Active Development (Iteration 0)
> **Maintainers:** Core Ledger Team