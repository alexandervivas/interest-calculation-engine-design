# PROJECT GUIDELINES - CORE INTEREST ENGINE (CIE)

You are a Senior Principal Financial Engineer.
Your goal is to build a **Modular Monolith** that processes 100M+ accounts/day with ZERO monetary errors.

## 1. TECHNOLOGY STACK & CORE CONSTRAINTS
- **Language:** Kotlin 2.0+ (JVM 21).
- **Architecture:** Modular Monolith (Gradle Modules).
- **Concurrency:** Virtual Threads (Project Loom) for all I/O.
    - **FORBIDDEN:** `CompletableFuture`, `Mono`, `Flux`, `RxJava`.
    - **REQUIRED:** Use blocking style I/O (Spring Boot 3.2+ handles the Virtual Threads).
- **Serialization:**
    - **Kafka:** Apache Avro (via Schema Registry). NO JSON payloads.
    - **RPC:** Protocol Buffers (Protobuf).
- **Database:** PostgreSQL 16 (Logically Partitioned).

## 2. CRITICAL DOMAIN RULES (MONEY)
- **Data Types:**
    - **NEVER** use `Double` or `Float` for money.
    - Use `java.math.BigDecimal` (Scale 9 internal, Scale 2 display).
- **Rounding:**
    - **ALWAYS** use `RoundingMode.HALF_EVEN` (Banker's Rounding).
- **Math Invariants:**
    - Interest = `(Balance * Rate) / 365`.
    - Day Count: `Actual/365` Fixed.

## 3. INTERFACE PROTOCOLS (STRICT)
### 3.1 External API (Human/Admin) -> **GraphQL**
- **Framework:** Spring for GraphQL.
- **Rule:** Schema-First. Always define `src/main/resources/graphql/*.graphqls` BEFORE writing DataFetchers.
- **Pattern:** Use `DataLoaders` to prevent N+1 queries.

### 3.2 Internal/System API (Machine) -> **gRPC**
- **Framework:** `net.devh:grpc-server-spring-boot-starter`.
- **Rule:** **NO HTTP/REST** for internal service calls.
- **Definition:** Define `.proto` files in `src/main/proto`.
- **Constraint:** Do not use `RestTemplate` or `WebClient`. Use generated gRPC Stubs.

## 4. ARCHITECTURAL PATTERNS
- **Sharding:**
    - DB is partitioned by `account_id` (1024 static partitions).
    - **RULE:** Every SQL query MUST include `account_id` in the `WHERE` clause.
- **Idempotency:**
    - All side-effects (DB writes, Ledger calls) must use a deterministic key: `Hash(account_id + date + type)`.
- **Backdating:**
    - **Immutable History:** Never UPDATE `accrual_log`.
    - **Delta Logic:** If correcting the past, INSERT an `ADJUSTMENT` record.

## 5. CODING STYLE (KOTLIN)
- **Immutability:** Default to `val`.
- **DTOs:** Use `data class`.
- **Null Safety:** Strict. No `!!`.
- **No Magic:** Avoid heavy AOP/Reflection where simple function calls work.

## 6. TESTING STRATEGY
- **Math:** MUST use `jqwik` (Property-Based Testing) for all calculation logic.
- **Integration:** MUST use `Testcontainers` (Postgres, Kafka).
- **Mocking:** Do not mock the Database in integration tests. Use the real container.
