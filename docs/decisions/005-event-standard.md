# ADR-005: Adoption of CloudEvents 1.0.2

* **Status:** Accepted
* **Date:** 2024-XX-XX
* **Context:** The system is event-driven. We need a standard envelope.
* **Decision:**
  * All asynchronous messages must adhere to the **CloudEvents 1.0.2** specification.
  * **Format:** The implementation will use the **Avro Format** (as defined in ADR-006).
  * The `type` attribute must follow the pattern: `com.bank.ledger.transaction.posted.v1`.
* **Consequences:**
  * **Positive:** Interoperability + Efficiency.
  * **Negative:** Slightly higher complexity in setting up the SerDe (Serializer/Deserializer) configuration.