# Infrastructure & Deployment

## 1. Kubernetes Topology
The system runs on **Kubernetes (EKS/GKE)**.

### 1.1 Node Pools
* **General Pool:** Runs API, Webhooks, Tooling.
* **Compute Pool (High CPU):** Runs `Calculation Core`. Optimized for math/logic.
* **Memory Pool (High RAM):** Runs `Ingestion Service` (Flink/Kafka Streams) for buffering state.

## 2. Autoscaling (KEDA)
We do not scale on CPU usage. We scale on **Kafka Consumer Lag**.

* **Trigger:** `keda.sh/v1alpha1:ScaledObject`
* **Metric:** `kafka_consumer_group_lag`
* **Logic:**
    * If Lag > 10,000 messages -> Scale Out.
    * If Lag < 1,000 messages -> Scale In.

## 3. Database Deployment
* **Engine:** PostgreSQL 16+.
* **High Availability:** Patroni + PGBouncer.
* **Optimization:**
    * `autovacuum` tuned for append-only workloads.
    * `fillfactor=100` for historical tables (they never change).
    * `fillfactor=70` for current balance tables (heavy updates).

## 4. Observability Stack
* **Metrics:** Prometheus. Critical Alert: `interest_calculation_lag_seconds`.
* **Tracing:** OpenTelemetry. All Kafka headers propagate `traceparent`.
* **Logs:** JSON Structured Logging (ELK/Loki).