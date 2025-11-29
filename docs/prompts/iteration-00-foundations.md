# Iteration 0: Foundations & Skeleton

**Goal:** Establish the infrastructure "Walking Skeleton" and the empty codebase structure.
**Definition of Done:** `make infra-up` starts all containers, and `./gradlew build` passes with clean code checks.

---

## 1. Infrastructure Bootstrapping (Agent: Aider)

**Context:** We are initializing the repository. We need the container runtime and automation scripts before we write application code.

**Prompt:**
```text
You are an expert DevOps engineer. We are building a high-throughput financial system called "Core Interest Engine".

Please scaffold the infrastructure for Iteration 0.

1. **Docker Compose:**
   Create a `docker-compose.yml` at the root with the following services:
   - `postgres`: Use image `postgres:16-alpine`. Expose port 5432. Set environment `POSTGRES_USER=user`, `POSTGRES_PASSWORD=password`, `POSTGRES_DB=interest_db`. Add a healthcheck.
   - `zookeeper`: Use `confluentinc/cp-zookeeper:7.5.0`. Environment: `ZOOKEEPER_CLIENT_PORT=2181`.
   - `kafka`: Use `confluentinc/cp-kafka:7.5.0`. Depends on zookeeper. Expose port 9092. Environment: `KAFKA_ZOOKEEPER_CONNECT=zookeeper:2181`, `KAFKA_ADVERTISED_LISTENERS=PLAINTEXT://localhost:9092`.
   - `schema-registry`: Use `confluentinc/cp-schema-registry:7.5.0`. Depends on kafka. Expose port 8081. Environment: `SCHEMA_REGISTRY_KAFKASTORE_BOOTSTRAP_SERVERS=kafka:9092`.

2. **Automation:**
   Create a `Makefile` at the root with these phony targets:
   - `infra-up`: Runs `docker compose up -d`
   - `infra-down`: Runs `docker compose down`
   - `infra-logs`: Runs `docker compose logs -f`
   - `clean`: Removes build artifacts (gradle clean).

3. **Git Configuration:**
   Create a `.gitignore` suitable for a Kotlin/IntelliJ/Gradle/Docker project. Ensure `.idea`, `build/`, and `.gradle` are ignored.

4. **Validation:**
   Run `docker compose config` to verify the syntax is valid.

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
