# Domain Logic: Product & Rate Configuration

## 1. Configuration Model
Interest rates are not static scalars. They are **Tiered Structures** versioned by time.

### 1.1 JSON Schema Example
```json
{
  "productId": "SAVINGS_GOLD_V1",
  "effectiveDate": "2024-01-01T00:00:00Z",
  "currency": "USD",
  "tiers": [
    {
      "minBalance": 0,
      "maxBalance": 10000,
      "apy": 0.0100  // 1.0%
    },
    {
      "minBalance": 10000.01,
      "maxBalance": null, // Infinity
      "apy": 0.0450  // 4.5%
    }
  ],
  "calculationMethod": "SPLIT_TIER" 
  // SPLIT_TIER = First 10k gets 1%, rest gets 4.5%
  // ENTIRE_BALANCE = If > 10k, whole amount gets 4.5%
}
```

## 2. Versioning Rule
* Configs are **Append-Only**.
* To change a rate, you insert a new config row with `effectiveDate = FutureDate`.
* The Calculation Engine selects the config where `config.effectiveDate <= calculationDate` (Limit 1 Descending).