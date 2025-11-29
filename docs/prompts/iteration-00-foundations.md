# Iteration 0: Foundations & Skeleton

**Goal:** Establish the infrastructure "Walking Skeleton" and the empty codebase structure.
**Definition of Done:** `make infra-up` starts all containers, and `./gradlew build` passes with clean code checks.

---

## 1. Infrastructure Bootstrapping (Agent: Aider)

**Context:** We are initializing the repository. We need the container runtime and automation scripts.

**Prompt:**
```text
You are an expert DevOps engineer. We are building a high-throughput financial system.

Please scaffold the infrastructure for Iteration 0 with a focus on "Local Development" efficiency.

1. **Docker Compose:**
   Create a `docker-compose.yml` at the root:
   - `postgres`: Image `postgres:16-alpine`. Port 5432. Env: `POSTGRES_USER=user`, `POSTGRES_PASSWORD=password`, `POSTGRES_DB=interest_db`.
   - `zookeeper`: Image `confluentinc/cp-zookeeper:7.5.0`.
   - `kafka`: Image `confluentinc/cp-kafka:7.5.0`.
     - Depends on `zookeeper`.
     - Env: `KAFKA_AUTO_CREATE_TOPICS_ENABLE: "false"`. (We will create them explicitly).
     - Env: `KAFKA_OFFSETS_TOPIC_REPLICATION_FACTOR: 1`.
   - `schema-registry`: Use `confluentinc/cp-schema-registry:7.5.0`. Depends on kafka. Expose port 8081.
   - `init-kafka`: Image `confluentinc/cp-kafka:7.5.0`.
     - Depends on `kafka`.
     - Entrypoint: A shell script that runs:
       ```bash
       # Default to 16 partitions for local dev (Production will be 1024)
       PARTITIONS=${KAFKA_PARTITION_COUNT:-16}
       echo "Creating topics with $PARTITIONS partitions..."
       cub kafka-ready -b kafka:9092 1 20
       kafka-topics --create --if-not-exists --bootstrap-server kafka:9092 --partitions $PARTITIONS --replication-factor 1 --topic tx-events
       kafka-topics --create --if-not-exists --bootstrap-server kafka:9092 --partitions $PARTITIONS --replication-factor 1 --topic interest-events
       ```

2. **Environment Config:**
   - Create a `.env` file at root with `KAFKA_PARTITION_COUNT=16`.

3. **Automation:**
   Create a `Makefile` at the root with these phony targets:
   - `infra-up`: Runs `docker compose up -d`
   - `infra-down`: Runs `docker compose down`
   - `infra-logs`: Runs `docker compose logs -f`
   - `clean`: Removes build artifacts (gradle clean).

4. **Git Configuration:**
   Create a `.gitignore` suitable for a Kotlin/IntelliJ/Gradle/Docker project. Ensure `.idea`, `build/`, and `.gradle` are ignored.

5. **Validation:**
   Run `docker compose config` to verify the syntax is valid.

6. **AI Reviewer Setup (The Gemini Guardian):**
   - Create a `.pr_agent.toml` file at the root to configure CodiumAI.
   - **Model Configuration:**
     - Set `pr_reviewer.model = "vertex_ai/gemini-1.5-pro"`. (Or `google/gemini-1.5-pro` depending on provider config).
     - Set `pr_reviewer.require_score_review = true`.
     - Set `pr_reviewer.enable_auto_approval = false`.
   
   - **Custom Instructions (`extra_instructions`):**
     Add this specific block to force architectural compliance:
     """
     You are the Architectural Auditor of the Core Interest Engine.
     
     **STRICT ENFORCEMENT PROTOCOL:**
     
     1. **ADR-004 (Sharding):** Reject hardcoded partition counts. Code must use `ShardingConfig` (defaults: 16 local / 1024 prod).
     2. **FR-6 (Precision):** Reject `Double` or `Float` for money. Require `BigDecimal` or `Money` class.
     3. **ADR-006 (Avro):** Reject JSON serialization. Ensure Avro schemas (`.avsc`) are used.
     4. **ADR-003 (Backdating):** If logic touches transaction dates, ensure it handles `valueDate < now`.
     
     If a PR violates these, comment with "BLOCKER: Violates [Doc Name]".
     """

   - **GitHub Workflow:**
     - Create `.github/workflows/pr_agent.yml` that triggers on `pull_request` types `[opened, reopened, synchronize]`.
     - Ensure it has access to the `GEMINI_API_KEY` or Vertex AI credentials.

```

---

## 2. Codebase Initialization (Agent: Junie)

**Context:** The infrastructure is ready. Now we need the JVM project structure. Agreements: Kotlin 1.9+, JVM 21, Gradle Kotlin DSL, Ktlint, Detekt.

**Prompt:**
```text
You are a Senior Kotlin Engineer. We are initializing the "Core Interest Engine" repository.

Please perform the following tasks to set up the Application Skeleton:

1. **Gradle Setup:**
   - Initialize a Gradle project using Kotlin DSL.
   - `settings.gradle.kts`: Project name = `core-interest-engine`.
   - `build.gradle.kts`:
     - Plugins: `kotlin("jvm") version "1.9.22"`, `application`, `io.gitlab.arturbosch.detekt:1.23.0`, `org.jlleitschuh.gradle.ktlint:12.0.0`.
     - Java Toolchain: Set to languageVersion 21.
     - Dependencies: 
       - `implementation("org.jetbrains.kotlinx:kotlinx-coroutines-core:1.8.0")`
       - `implementation("io.github.microutils:kotlin-logging:3.0.5")`
       - `implementation("org.slf4j:slf4j-simple:2.0.9")` (for logging output).

2. **Directory Structure:**
   - Create the standard layout: `src/main/kotlin/com/bank/interest` and `src/test/kotlin/com/bank/interest`.
   - Create a main entry point: `src/main/kotlin/com/bank/interest/Application.kt`.
   - Content of `Application.kt`: A simple `main` function that prints "Core Interest Engine: Starting..." and keeps the process alive (e.g., `Thread.sleep` or a simple input read) so Docker doesn't exit immediately when we eventually dockerize the app.

3. **Quality Gates:**
   - Configure `detekt` to fail on build if code smells are found.
   - Configure `ktlint` to enforce standard formatting.

4. **Verification:**
   - Run the build wrapper. Ensure the project compiles and linting is active.

```
