# ADR-005: Adoption of CloudEvents 1.0.2

* **Status:** Accepted
* **Date:** 2024-XX-XX
* **Context:** The system is event-driven. Without a strict envelope standard, services will invent proprietary header formats for metadata (trace IDs, event types, timestamps, sources), leading to parsing inconsistencies and tighter coupling between producers and consumers.
* **Decision:**
  * All asynchronous messages (Kafka, RabbitMQ, etc.) **must** adhere to the **CloudEvents 1.0.2** specification using the **Structured Content Mode** (JSON Format).
  * The `data` attribute will contain the domain payload.
  * The `type` attribute must follow the reverse-DNS pattern: `com.bank.ledger.transaction.posted.v1`.
* **Consequences:**
  * **Positive:** Uniformity in event routing, tracing, and deserialization. Middleware (like KEDA or Event Mesh) can inspect metadata without parsing the domain payload.
  * **Negative:** Slight payload size overhead due to verbose JSON headers.
  * **Compliance:** The Ingestion Service will reject any message that does not pass the CloudEvents JSON Schema validation.