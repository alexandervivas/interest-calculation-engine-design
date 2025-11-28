# ADR-004: Static Partitioning Strategy

* **Status:** Accepted
* **Context:** Processing 100M accounts requires horizontal scaling. Dynamic sharding is complex.
* **Decision:**
    * Use **1,024 Static Logical Partitions**.
    * Routing: `PartitionID = Hash(AccountID) % 1024`.
    * Kafka Topics and Database Tables will align with this partitioning scheme.
    * "Shared Nothing" architecture: A worker node owns a subset of partitions exclusively.
* **Consequences:**
    * **Positive:** Guarantees strict ordering per account. Maximizes cache locality. Simplifies database writes (bulk inserts per partition).
    * **Negative:** Resharding (e.g., moving to 2048 partitions) is a "Big Bang" migration event. (Mitigated by choosing a high initial number: 1024).