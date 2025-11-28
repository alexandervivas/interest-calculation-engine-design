# ADR-007: Modular Monolith & Interface Protocols

* **Status:** Accepted
* **Date:** 2024-XX-XX
* **Context:**
  * The system requires high performance (low latency) and strict contracts.
  * We want the simplicity of a Monolith (single deployment unit) but the discipline of Microservices.
  * REST/HTTP is too loose and verbose for our internal high-throughput requirements.
* **Decision:**
  * **Architecture:** **Modular Monolith**. Code is organized into strict modules (`api`, `calculation`, `ingestion`, `posting`).
  * **External API (Northbound):** **GraphQL** (Spring for GraphQL). This serves UI, Admins, and Auditors.
  * **Internal Integration (Southbound/East-West):** **gRPC** (Protobuf). This serves connections to the Ledger, Audit Lake, and other internal banking services.
  * **Constraint:** HTTP/REST is **forbidden** for machine-to-machine communication. No `RestTemplate` or `WebClient` allowed for internal traffic.
* **Consequences:**
  * **Positive:** "Hard Shell, Fast Core." GraphQL allows flexible data querying without over-fetching. gRPC ensures type-safe, high-performance binary communication with upstream/downstream systems.
  * **Negative:** Higher learning curve (Protobuf, Schema-first GraphQL). Requires build tooling to generate Java sources from `.proto` and `.graphqls` files.
