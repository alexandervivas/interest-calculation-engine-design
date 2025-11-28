# AI & Agentic Opportunities

| ID | Priority | Name | Description | Success Metric |
| :--- | :--- | :--- | :--- | :--- |
| **AI-1** | **Should** | **Pre-Flight Simulation Agent** | An agent that takes a proposed configuration change (e.g., rate hike) and runs a simulation against 10k sample accounts to forecast financial impact. | Forecast Variance < 5% vs Production. |
| **AI-2** | **Could** | **Anomaly Detection Sidecar** | An unsupervised model (Isolation Forest) reading the output stream to detect "Black Swan" math errors (e.g., massive negative interest) in real-time. | Detection Latency < 100ms. False Positive < 0.1%. |
| **AI-3** | **Could** | **Natural Language Auditor (RAG)** | A support tool allowing non-tech staff to query calculation traces using natural language (e.g., "Explain the adjustment on Jan 1"). | Correct explanation path returned for 90% of queries. |