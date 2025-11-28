# C4 Context Diagram

## 1. System Overview
The **Core Interest Engine (CIE)** is a critical subsystem within the Banking Platform. It acts as a sidecar to the main Transaction Ledger, responsible specifically for the complex domain of interest accrual, capitalization, and tax withholding calculation.

## 2. Context Diagram (Mermaid)

```mermaid
C4Context
    title System Context Diagram for Core Interest Engine

    Person(pm, "Product Manager", "Configures interest rates and tiering rules")
    Person(ops, "Bank Ops / Auditor", "Reviews adjustments and audit trails")

    System(cie, "Core Interest Engine", "Calculates daily interest, manages accruals, handles backdated corrections")

    System_Ext(ledger, "Thin Ledger (Core)", "Source of Truth for transactions. Receives interest posting commands.")
    System_Ext(analytics, "Data Lake / BI", "Ingests historical accrual data for reporting.")
    System_Ext(notify, "Notification System", "Alerts customers of interest payouts.")

    Rel(pm, cie, "Configures Products", "HTTPS/REST")
    Rel(ops, cie, "Audits Traces", "HTTPS/UI")
    
    Rel(ledger, cie, "Publishes Transaction Events", "Kafka/CloudEvents")
    Rel(cie, ledger, "Posts Interest Entries", "Kafka/Command")
    
    Rel(cie, analytics, "Exports Daily Accruals", "Parquet/S3")
    Rel(cie, notify, "Triggers Payout Alerts", "Async Event")
```
## 3. Key Interactions
* **Inbound**: The CIE consumes a high-volume stream of TransactionSettled events from the Ledger.
* **Outbound**: The CIE emits PostTransactionCommand messages back to the Ledger to book interest.
* **Configuration**: Product Managers define rates (e.g., "Gold Savings = 5.0%") via the Configuration API.