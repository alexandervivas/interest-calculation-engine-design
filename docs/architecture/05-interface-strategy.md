# Interface Strategy: The "Hard Shell" Protocol

## 1. Overview
The Core Interest Engine (CIE) rejects the "REST for everything" default. We use protocols fit for purpose:
* **Humans/Tools** need flexibility -> **GraphQL**.
* **Machines/Services** need strictness & speed -> **gRPC**.

## 2. The GraphQL Gateway (Northbound)
* **Scope:** Public-facing API (Admins, Product Managers, Dashboards).
* **Location:** `mod-api-gateway` module.
* **Philosophy:** **Schema-First**. We define `.graphqls` files before writing code.
* **Capabilities:**
  * Querying complex nested structures (Account -> History -> Trace).
  * Mutating configurations (Product Rates).

## 3. The gRPC Mesh (Southbound & Internal)
* **Scope:** High-volume service-to-service communication.
* **Location:** `src/main/proto`.
* **Standard:**
  * **Transport:** Netty / HTTP/2.
  * **Serialization:** Protocol Buffers (v3).
  * **Error Handling:** Standard gRPC Status Codes mapped to Domain Errors.

### 3.1 Known gRPC Connections
| Source Module | Target Service | Purpose | Type |
| :--- | :--- | :--- | :--- |
| `mod-posting` | **Ledger System** | Posting Interest Transactions | Unary & Streaming |
| `mod-calculation` | **Audit Lake** | Streaming Trace Logs | Client-Side Streaming |
| `mod-config` | **Config Service** | Fetching Global Bank Rates | Unary (Cached) |
| `mod-ingestion` | **Feature Store** | Fetching ML Features | Unary |

## 4. Forbidden Patterns
* ❌ Exposing a REST Controller for domain logic.
* ❌ Using JSON over HTTP to call the Ledger.
* ❌ Sharing JPA Entities directly in the GraphQL Schema.
