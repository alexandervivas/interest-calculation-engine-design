# ADR-001: Technology Stack Selection

* **Status:** Accepted
* **Date:** 2024-XX-XX
* **Context:** We need a language and runtime capable of high-throughput math, complex domain modeling, and concurrent I/O (DB/Kafka).
* **Decision:**
    * **Language:** Kotlin.
    * **Runtime:** JVM 21+ (utilizing Virtual Threads / Project Loom).
    * **Framework:** Spring Boot 3.2+ (or Micronaut) with minimal overhead.
* **Consequences:**
    * **Positive:** Kotlin provides type safety and null safety for domain logic. Virtual Threads allow writing imperative, readable code that performs like non-blocking Reactive code.
    * **Negative:** JVM startup time (mitigated by long-running batch processes).