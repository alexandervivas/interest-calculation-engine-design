# ADR-002: Shadow Ledger (Materialized View) Pattern

* **Status:** Accepted
* **Context:** The upstream Transaction Ledger cannot support 100M read queries per day for interest calculation. We need local access to balances.
* **Decision:**
    * The CIE will maintain its own local "Shadow Ledger" (Materialized View) of daily EOD balances.
    * This state is built by consuming the transaction event stream.
* **Consequences:**
    * **Positive:** Decouples Interest Engine from Ledger uptime/performance. Allows optimized indexing for time-series queries.
    * **Negative:** Complexity of ensuring the Shadow Ledger is consistent with the Real Ledger (requires careful event handling and reconciliation).