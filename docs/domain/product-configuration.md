# Domain Logic: Product & Rate Configuration

## 1. The Product Model
A **Product** is a named configuration template (e.g., "Gold Saver", "30-Year Fixed Mortgage").
An **Account** is an instance of a Product.

### 1.1 Product Family (Discriminator)
Every product belongs to a family that dictates the calculation engine's behavior.

| Family | Direction | Interest Logic | Future Scope? |
| :--- | :--- | :--- | :--- |
| **DEPOSIT** | Liability (Bank owes Customer) | `Accrual = Balance * Rate` (Expense) | **In Scope** |
| **LENDING** | Asset (Customer owes Bank) | `Accrual = Balance * Rate` (Revenue) | *Future* |

## 2. JSON Schema Configuration

We use a polymorphic configuration object. The `productFamily` field determines which extra fields are required.

```json
{
  "productId": "SAVINGS_GOLD_V1",
  "productFamily": "DEPOSIT",  // <--- The Critical Discriminator
  "currency": "USD",
  
  // Specific to DEPOSIT family
  "interestPostingFrequency": "MONTHLY",
  "compounding": "DAILY",
  
  // The Rate Matrix
  "rateConfig": {
    "type": "TIERED",
    "tiers": [
      { "min": 0, "apy": 0.05 }
    ]
  }
}
```

**Future Compatibility**: When we add Loans later, we will simply add a new schema for productFamily: "LENDING" which includes fields like amortizationMethod, penaltyRate, and gracePeriodDays.
