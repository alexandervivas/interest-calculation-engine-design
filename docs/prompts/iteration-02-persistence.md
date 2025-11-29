# Iteration 2: Persistence & Migrations

**Goal:** Establish the Database layer, create the schema for configurations and balances, and verify read/write operations using Testcontainers.
**Definition of Done:** `make test` passes integration tests that spin up a real Postgres Docker container, run migrations, and persist the Domain objects.

---

## 1. Database Dependencies & Migrations (Agent: Aider)

**Context:** We need to connect our Kotlin application to PostgreSQL. We will use **Flyway** for schema versioning to ensure reproducible environments.
**Agreements:** * Use `HikariCP` for connection pooling.
* Use `Flyway` for migrations.
* Schema must support the High-Precision Money types (DECIMAL 19,9).

**Prompt:**
```text
You are a DevOps/Backend Engineer. We are adding the Persistence Layer.

1. **Gradle Dependencies:**
   - Update `build.gradle.kts` to add:
     - `implementation("org.postgresql:postgresql:42.7.1")`
     - `implementation("com.zaxxer:HikariCP:5.1.0")`
     - `implementation("org.flywaydb:flyway-core:10.0.0")`
     - `testImplementation("org.testcontainers:postgresql:1.19.3")`
     - `testImplementation("org.testcontainers:junit-jupiter:1.19.3")`

2. **Flyway Setup:**
   - Create the directory structure: `src/main/resources/db/migration`.
   - Configure a `DataSourceConfig` class (or standard Spring Boot application.yml if we are using Spring Boot - assume manual config if pure Kotlin, or Spring Boot if we are using it. *Assumption: We are using Spring Boot as per Iteration 0.*)
   - Ensure application.properties/yml has the datasource url, user, password placeholders.

3. **Schema Definition (V1):**
   Create `src/main/resources/db/migration/V1__initial_schema.sql`.
   
   **Table 1: `product_config`**
   - `product_id` (VARCHAR, PK)
   - `effective_date` (DATE, PK) - (Composite PK to allow versioning)
   - `version` (INT)
   - `config_json` (JSONB) - Stores the Tiering rules
   - `created_at` (TIMESTAMP)

   **Table 2: `daily_positions` (The Core Ledger State)**
   - `account_id` (VARCHAR NOT NULL)
   - `value_date` (DATE NOT NULL)
   - `balance_amount` (DECIMAL(19, 9) NOT NULL)
   - `accrued_interest` (DECIMAL(19, 9) NOT NULL)
   - `partition_id` (INT GENERATED ALWAYS AS (hashtext(account_id) % 1024) STORED)
   - Primary Key: `(partition_id, account_id, value_date)`
   - Note: Include the `PARTITION BY LIST (partition_id)` syntax but just create a `default` partition for now to keep it runnable locally without creating 1024 tables yet.
   - `CREATE TABLE daily_positions_default PARTITION OF daily_positions DEFAULT;`

4. **Verification:**
   - Provide a command to run `./gradlew flywayMigrate` (if plugin is added) or ensuring the app boot triggers it.

```

---

## 2. Repository Implementation (Agent: Junie)

**Context:** We need Kotlin code to read/write the Domain Objects (`Money`, `AccountBalance`) to the Database.
**Agreements:** Use **Spring Data JDBC** or **Exposed** (Let's stick to **Spring Data JDBC** for simplicity/speed in this iteration, or plain **JdbcTemplate** if complex mapping is needed. Let's ask for **Spring Data JDBC** repositories).

**Prompt:**
```text
You are a Senior Backend Engineer. Implement the Repository Layer.

1. **Dependencies:**
   - Ensure `org.springframework.boot:spring-boot-starter-data-jdbc` is in build.gradle.kts.

2. **Domain Mapping:**
   - Map the `product_config` table to a `ProductConfigEntity` data class.
     - Note: You will need to map the `JSONB` column. Use a custom Converter or simple String mapping for now.
   - Map the `daily_positions` table to a `DailyPositionEntity` data class.
     - Ensure `balance_amount` and `accrued_interest` map to `BigDecimal`.

3. **Repositories:**
   - Create `ProductConfigRepository` (CrudRepository).
   - Create `DailyPositionRepository` (CrudRepository).
   - Add a custom query method: `findByAccountIdAndValueDate(accountId: String, date: LocalDate)`.

4. **Service Integration:**
   - Create a `PositionService`.
   - Method: `upsertBalance(accountId: String, date: LocalDate, balance: Money)`.
   - Logic: Convert Domain Object -> Entity -> Save.

```

---

## 3. Integration Testing (Agent: Junie)

**Context:** We must verify that the database interactions work with real Postgres, especially the Precision (9 decimals) and the JSONB handling.

**Prompt:**
```text
You are a QA Engineer. We need an Integration Test using Testcontainers.

1. **Create Base Test Class:**
   - `AbstractIntegrationTest`: Uses `@Container` to spin up a generic Postgres 16 container.
   - Sets the `spring.datasource.url` dynamically.

2. **Repository Test:**
   - Create `DailyPositionRepositoryTest` extending the base class.
   - **Test Case 1: Precision Preservation**
     - Save a balance of `100.123456789`.
     - Read it back.
     - Assert it equals exactly `100.123456789` (no rounding to 2 decimals).
   - **Test Case 2: Upsert Semantics**
     - Save a record for Day 1.
     - Update the record for Day 1.
     - Assert count is 1, and value is updated.

3. **Execution:**
   - Run the test. Ensure it passes.

```