# ADR-003: Delta-Adjustment for Retroactive Changes

* **Status:** Accepted
* **Context:** Transactions effectively occur in the past (Backdated). We cannot rewrite historical Ledger entries (immutable).
* **Decision:**
    * Implement a "Replay and Adjust" mechanism.
    * When $t_{val} < t_{now}$, re-calculate interest for the window $[t_{val}, t_{now}]$.
    * Compute $\Delta = I_{calculated\_new} - I_{already\_accrued}$.
    * Post the difference $\Delta$ as a new adjustment record.
* **Consequences:**
    * **Positive:** Preserves immutability. Provides clear audit trail of corrections.
    * **Negative:** High computational cost for deep backdates. Requires access to historical rate configurations and balances.