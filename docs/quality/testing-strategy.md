# Quality Assurance & Testing Strategy

## 1. The Testing Pyramid
Given the financial nature of the system, we invert the typical pyramid slightly to favor **Property-Based Testing**.

| Level | Tool | Coverage Target | Focus |
| :--- | :--- | :--- | :--- |
| **Unit (Logic)** | JUnit 5 + MockK | 100% | Individual functions, Tier logic. |
| **Property (Fuzz)** | **jqwik** | High | Mathematical invariants (commutativity, associativity). |
| **Integration** | Testcontainers | 80% | Kafka <-> App <-> Postgres data flow. |
| **Mutation** | **Pitest** | >90% Score | Ensure tests fail if code changes. |

## 2. Property-Based Testing (jqwik)
We do not just test `1 + 1 = 2`. We test theorems.
* **Theorem 1:** Interest for a positive balance and positive rate is always $\ge 0$.
* **Theorem 2:** Splitting a balance into two accounts ($A = A1 + A2$) yields the same total interest (if linear tiers).

```kotlin
@Property
fun `interest is strictly monotonic with rate`(@ForAll balance: BigDecimal, @ForAll rate1: BigDecimal, @ForAll rate2: BigDecimal) {
    Assume.that(rate1 < rate2)
    val i1 = calculate(balance, rate1)
    val i2 = calculate(balance, rate2)
    Assertions.assertThat(i1).isLessThan(i2)
}
```

## 3. The "Torture Suite"
A specialized dataset containing:
* Leap years.
* Negative balances.
* Integer overflows (MAX_LONG + 1).
* 10-year backdated transactions.