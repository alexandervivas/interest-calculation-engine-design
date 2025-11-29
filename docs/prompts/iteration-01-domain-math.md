# Iteration 1: The Domain Core & Math

**Goal:** Implement the pure business logic (Money, Interest, Tiers) with zero external dependencies (no DB, no Kafka).
**Definition of Done:** The `InterestCalculator` is implemented, and Property-Based Tests (jqwik) pass 100%.

---

## 1. Domain Primitives (Agent: Junie)

**Context:** We need a safe way to handle money before doing any math. Floating point errors are forbidden (FR-6).
**Agreements:** Use `BigDecimal` with 9 decimal places of precision internally. Rounding mode is `HALF_EVEN`.

**Prompt:**
```text
You are a Senior Kotlin Engineer. We are building the Domain Layer.

1. **Create the `Money` Value Object:**
   - Location: `src/main/kotlin/com/bank/interest/domain/Money.kt`
   - Data: Holds a `private val amount: BigDecimal`.
   - Invariants: 
     - Always set scale to 9. 
     - Never allow `Double` in constructors (String or BigDecimal only).
   - Operations: `add`, `subtract`, `multiply(factor: BigDecimal)`.
   - Method `roundToCurrency()`: Returns a new Money scaled to 2 decimal places using `RoundingMode.HALF_EVEN`.
   - Override `equals`, `hashCode`, `toString`.

2. **Create the `AccountBalance` Value Object:**
   - Location: `src/main/kotlin/com/bank/interest/domain/AccountBalance.kt`
   - Fields: `accountId: String`, `amount: Money`, `valueDate: LocalDate`.

3. **Unit Tests:**
   - Write standard JUnit 5 tests ensuring math precision is maintained (e.g., adding 1/3 + 1/3 + 1/3 does not lose precision until rounding).

```

---

## 2. Interest Calculation Logic (Agent: Junie)

**Context:** We need the engine that accepts a Balance and a Rate Configuration and outputs the accrued interest.
**Agreements:** Day count convention is Actual/365.

**Prompt:**
```text
You are a Senior Kotlin Engineer. Implement the core Interest Engine logic.

1. **Define the Rate Configuration:**
   - Create data classes in `com.bank.interest.domain.policy`:
     - `Tier(minBalance: Money, maxBalance: Money?, apy: BigDecimal)`
     - `InterestRule(id: String, effectiveDate: LocalDate, tiers: List<Tier>)`
   - Note: `apy` is the Annual Percentage Yield (e.g., 0.05 for 5%).

2. **Implement the Calculator:**
   - Create `InterestCalculator` service.
   - Function: `calculate(balance: Money, rule: InterestRule): Money`
   - Logic:
     - Determine which Tier applies to the balance.
     - Formula: `Accrual = Balance * Rate / 365`.
     - Use `Money` operations created in step 1.
     - **Do not round** the result (keep 9 decimals).

3. **Handle Tiering Strategy:**
   - For now, assume "Entire Balance" tiering (if balance > 10k, whole amount gets the higher rate).
   - Raise an exception if no tier matches the balance.

```

---

## 3. Property-Based Testing (Agent: Junie)

**Context:** Standard unit tests are insufficient for financial guarantees. We need to fuzz-test the math properties using **jqwik**.
**Agreements:** `docs/quality/testing-strategy.md`.

**Prompt:**
```text
You are a QA Automation Expert. We need to harden the math logic.

1. **Add Dependencies:**
   - Update `build.gradle.kts` to include `net.jqwik:jqwik:1.8.0`.

2. **Create Property Tests:**
   - Location: `src/test/kotlin/com/bank/interest/domain/InterestMathProperties.kt`
   - Use `@Property` to verify these Theorems:
     - **Non-Negativity:** If Balance > 0 and Rate > 0, Interest must be >= 0.
     - **Monotonicity:** If Balance A > Balance B (and Rate is constant), Interest A > Interest B.
     - **Reversibility:** `(Balance + Interest) - Interest` should equal `Balance` (within precision limits).
   - Create `@Provide` methods to generate random Money amounts (positive, negative, zero, huge).

3. **Run the Suite:**
   - Ensure the tests run fast and cover edge cases (0 balance, negative balance).

```

