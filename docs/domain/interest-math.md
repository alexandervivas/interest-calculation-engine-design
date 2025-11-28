# Domain Logic: Interest Mathematics

## 1. The Standard Formula
For a given day $d$, the accrued interest $I$ is calculated as:

$$I = B_{eod} \times \frac{r}{D_{year}}$$

Where:
* $B_{eod}$: End-of-Day Balance for day $d$.
* $r$: Annual Interest Rate (as a decimal, e.g., 5% = 0.05).
* $D_{year}$: Days in the year (Day Count Convention).

## 2. Conventions & Rules

### 2.1 Day Count Convention
We use **Actual/365** Fixed.
* Denominator is always 365, even in leap years.
* *rationale:* Simplifies consistency across years; standard in consumer banking.

### 2.2 Precision & Data Types
* **Internal Calculation:** `BigDecimal` with **9 decimal places**.
* **Storage (DB):** `DECIMAL(19, 9)`.
* **Transport (Kafka):** `String` (to avoid float corruption) or Scaled Long (Micros).

### 2.3 Rounding Strategy
We apply **Banker's Rounding (HALF_EVEN)**.
* Rounding occurs **only** at the moment of **Posting** (creating a transaction).
* Daily accruals remain unrounded (9 decimals) in the accumulation table to prevent "penny drift" over the month.

**Example:**
* Day 1 Accrual: `0.333333333`
* Day 2 Accrual: `0.333333333`
* Day 3 Accrual: `0.333333333`
* **Total:** `0.999999999` -> Rounds to `$1.00`
* *Bad approach (Daily Rounding):* $0.33 + 0.33 + 0.33 = 0.99$ (Loss of 1 cent).