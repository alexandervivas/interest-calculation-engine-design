# Architecture: Future Extensibility (Loans & Assets)

## 1. Context
The system currently focuses on **Deposit** products (Savings/Checking). However, it is architected to eventually handle **Lending** products (Loans/Mortgages) which have distinct behaviors:
* **Schedules:** Loans have fixed payment dates.
* **Negative Balances:** A loan balance is inherently negative (Debit).
* **Amortization:** Interest calculation may depend on a specific schedule (French vs. German amortization).

## 2. The Extension Strategy
We utilize the **Strategy Pattern** to isolate calculation logic.

### 2.1 The `CalculationEngine` Interface
The Core Engine does not "know" about Savings or Loans. It asks a Factory:
> *"Give me the Strategy for Product Family 'LENDING'."*

### 2.2 Database Readiness
* **Balances:** The `daily_positions` table supports negative numbers (`DECIMAL` signed).
* **Rates:** The `product_config` table uses a JSONB column for rules, allowing us to store complex Loan parameters (e.g., "Variable Rate Index: SOFR + 2%") without altering the table schema.

### 2.3 What will change?
When we activate Loans, we will need to add:
1.  **New Strategy Class:** `AmortizedInterestStrategy.kt`.
2.  **New Event Types:** `LoanPaymentMissed` (triggers penalty interest).
3.  **Schedule Service:** A separate module/service to track "Next Payment Date" (since simple daily accrual isn't enough for loans).
