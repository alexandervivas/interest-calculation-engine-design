# ADR-006: Avro Serialization & Schema Registry

* **Status:** Accepted
* **Date:** 2024-XX-XX
* **Context:** The system expects billions of events. JSON is verbose (field names repeated in every message) and type-unsafe (no guarantee `amount` is a number). Breaking schema changes can crash consumers.
* **Decision:**
  * Use **Apache Avro** as the binary serialization format for all Kafka payloads.
  * Use a centralized **Schema Registry** (e.g., Confluent/Apicurio) to manage versioning.
  * Enforce **Forward Compatibility** policies (old consumers must be able to read new events).
  * CloudEvents will be serialized using the **Avro Format Binding**.
* **Consequences:**
  * **Positive:** Payload size reduced by ~40-60%. Faster serialization/deserialization. Strict contract enforcement prevents "poison pills" (invalid data) from entering the topic.
  * **Negative:** Requires running a Schema Registry. Events are not human-readable without tooling (cannot just use `kafkacat` or `console-consumer` without Avro flags).