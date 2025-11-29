# Iteration 3: Messaging & Event Streaming

**Goal:** Connect the engine to the outside world. Consume transactions from Kafka, and publish Interest events using Avro and CloudEvents.
**Definition of Done:** `make test` passes integration tests where a message sent to a Kafka container is consumed, processed, and a result message is published back.

---

## 1. Avro Schemas & Configuration (Agent: Aider)

**Context:** We are implementing ADR-005 (CloudEvents) and ADR-006 (Avro). We need the build toolchain to generate Java/Kotlin classes from Avro schemas.
**Agreements:** Use `com.github.davidmc24.gradle.plugin.avro` (or similar standard plugin). Schemas live in `src/main/resources/avro`.

**Prompt:**
```text
You are a DevOps/Backend Engineer. Set up the Avro Build Pipeline.

1. **Gradle Setup:**
   - Add the Avro Gradle Plugin to `build.gradle.kts` (`"com.github.davidmc24.gradle.plugin.avro" version "1.9.1"`).
   - Add dependencies:
     - `implementation("org.apache.avro:avro:1.11.3")`
     - `implementation("io.confluent:kafka-avro-serializer:7.5.0")`
     - `implementation("org.springframework.kafka:spring-kafka")`
   - Add the Confluent Maven Repository to `repositories {}` (`url = uri("https://packages.confluent.io/maven/")`).

2. **Schema Definition:**
   Create `src/main/resources/avro/TransactionSettled.avsc`:
   ```json
   {
     "namespace": "com.bank.ledger.events",
     "type": "record",
     "name": "TransactionSettled",
     "fields": [
       {"name": "eventId", "type": "string", "logicalType": "uuid"},
       {"name": "accountId", "type": "string"},
       {"name": "amountMicros", "type": "long"},
       {"name": "valueDate", "type": "int", "logicalType": "date"},
       {"name": "type", "type": {"type": "enum", "name": "TxType", "symbols": ["DEPOSIT", "WITHDRAWAL", "INTEREST"]}}
     ]
   }

   ```

   Create src/main/resources/avro/InterestPosted.avsc:
   ```json
   {
     "namespace": "com.bank.interest.events",
     "type": "record",
     "name": "InterestPosted",
     "fields": [
       {"name": "eventId", "type": "string", "logicalType": "uuid"},
       {"name": "accountId", "type": "string"},
       {"name": "amountMicros", "type": "long"},
       {"name": "date", "type": "int", "logicalType": "date"}
     ]
   }

   ```
3. **Build Task:**

   - Configure the `generateAvroJava` task to run before `compileKotlin`.
   - Run the build and ensure the classes `TransactionSettled` and `InterestPosted` are generated in `build/generated/...`.

---

## 2. Ingestion & Posting Logic (Agent: Junie)

**Context:** We need the Spring Kafka consumers and producers. They must handle the "CloudEvents over Avro" envelope. **Agreements:** * Consumer Group ID: `core-interest-engine-v1`.
   - Topic: `tx-events` (input), `interest-events` (output).
   - Use `IsolationLevel.READ_COMMITTED`.

**Prompt:**
```text
You are a Senior Backend Engineer. Implement the Messaging Layer.

1. **Configuration (`application.yml`):**
   - Configure Spring Kafka:
     - Bootstrap servers: `${KAFKA_BOOTSTRAP_SERVERS:localhost:9092}`
     - Consumer key-deserializer: StringDeserializer
     - Consumer value-deserializer: `io.confluent.kafka.serializers.KafkaAvroDeserializer`
     - Producer value-serializer: `io.confluent.kafka.serializers.KafkaAvroSerializer`
     - Schema Registry URL: `${SCHEMA_REGISTRY_URL:http://localhost:8081}`
     - Specific Avro Config: `specific.avro.reader: true`.

2. **Ingestion Service (`TransactionConsumer`):**
   - Create a `@KafkaListener` for topic `tx-events`.
   - Method signature: `fun consume(record: ConsumerRecord<String, TransactionSettled>)`.
   - Logic:
     - Convert Avro `TransactionSettled` to Domain `Money` and `AccountBalance`.
     - Call `PositionService.upsertBalance` (from Iteration 2).
     - Log the processing (MDC with traceId).

3. **Posting Service (`InterestPublisher`):**
   - Create a service wrapping `KafkaTemplate`.
   - Method: `publishInterest(interest: InterestPosted)`.
   - Logic: 
     - Wrap the payload in a CloudEvent (using the CloudEvents Java SDK or manually constructing headers if using Avro payload directly). 
     - *Simplification for now:* Just send the Avro record as the value. We will wrap in CloudEvents in Iteration 4 (Observability) to keep this step focused on connectivity.

```

---

## 3. Integration Testing (Agent: Junie)

**Context:** Verify the pipeline works with Testcontainers (Kafka + Schema Registry). This is complex because Schema Registry containers take time to start.

**Prompt:**
```text
You are a QA Engineer. Implement the Kafka Integration Test.

1. **Test Infrastructure:**
   - Update `AbstractIntegrationTest` to include:
     - `KafkaContainer` (Confluent image).
     - `GenericContainer` for Schema Registry (linked to Kafka).
   - Use `@DynamicPropertySource` to inject the random ports into the Spring Context.

2. **The Flow Test:**
   - Create `MessagingFlowTest`.
   - **Scenario:**
     1. Use a Test Producer to send a `TransactionSettled` event (Deposit $100) to `tx-events`.
     2. Wait (Awaitility) for the `DailyPositionRepository` to show a balance of $100.
     3. *Manual Trigger (Mock)*: Call the `InterestCalculator` logic manually in the test.
     4. Assert that an `InterestPosted` event appears on the `interest-events` topic.
    
```
