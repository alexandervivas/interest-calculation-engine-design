# Domain Logic: Backdated Transaction Handling

## 1. The Principle
**"History is Immutable, Correction is Additive."**
We never update an existing `interest_accrued` record. We append a specific `adjustment` record.

## 2. The Algorithm: Delta Replay
When a transaction arrives with `value_date < current_date`:

1.  **Identify Window:** Determine affected range $[t_{start}, t_{end}]$.
2.  **Fetch Snapshots:** Retrieve $B_{daily}$ (Balance) and $R_{daily}$ (Rate) for each day in the window.
3.  **Simulate:** Calculate $I_{ideal}$ using $(B_{daily} + Tx_{amount})$.
4.  **Compare:** Fetch sum of existing accruals $I_{actual}$ for that day.
5.  **Delta:** $\Delta = I_{ideal} - I_{actual}$.
6.  **Book:** Insert `AccrualRecord` with type `ADJUSTMENT`.


```mermaid
sequenceDiagram
    autonumber
    participant Tx as Incoming Tx (T-3)
    participant Engine
    participant History as History Store
    participant Ledger

    Note over Tx, Ledger: Today is Jan 4th. Tx is effective Jan 1st.

    Tx->>Engine: Deposit $10k (ValueDate: Jan 1)
    
    Engine->>History: Fetch Daily Balances (Jan 1, Jan 2, Jan 3)
    History-->>Engine: [1k, 1k, 1k]
    
    loop Replay Jan 1 to Jan 3
        Engine->>Engine: New Balance = Old + 10k
        Engine->>Engine: Calc New Interest (e.g. $5.00)
        Engine->>History: Fetch Already Paid (e.g. $0.50)
        Engine->>Engine: Delta = $4.50
    end
    
    Engine->>History: Update Snapshots (+10k to all days)
    Engine->>Ledger: Post "Adjustment Int" (Sum of Deltas)
    Note right of Ledger: Booking Date = Jan 4th<br/>(Immutable History Preserved)

```

## 3. Scenario: The "Overdraft Flip"
**Situation:**
* Day 1 Balance: `$100` (Credit). Rate: `+2%`. Accrued: `+$0.005`.
* Backdated Tx: `-$200` effective Day 1.
* Revised Balance: `-$100` (Debit). Rate: `-10%` (Overdraft).

**Execution:**
1.  **Reversal:** We must claw back the paid interest. Delta 1 = `-$0.005`.
2.  **Charge:** We must charge the overdraft. Ideal Interest = `-$0.027`.
3.  **Net Adjustment:** `-$0.032`.

> **Note:** The system must support negative accruals (charges) even on savings products.