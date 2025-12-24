# Production Observability & Monitoring

"It worked on my laptop" is not enough. You need to know why it failed at 3:00 AM.

---

## 1. The Three Pillars of LLM Ops

| Pillar | Question | Tools |
|--------|----------|-------|
| **Tracing** | "What happened inside the chain?" | LangSmith, Arize Phoenix, Honeycomb |
| **Metrics** | "How fast/expensive is it?" | Prometheus, Grafana, Datadog |
| **Evaluation** | "Is the answer actually good?" | Ragas (Online), User Feedback |

---

## 2. Tracing (The "X-Ray")

Standard logging (`logger.info`) fails for Agentic workflows because they are non-deterministic and recursive.

**What to Trace:**
1.  **Inputs/Outputs**: Diff checking.
2.  **Tool Calls**: Did it call `search` with the right arguments?
3.  **Latency Spans**: Which step was slow? (e.g., Vector DB took 2ms, but LLM took 4s).
4.  **Token Usage**: Cost attribution per step.

**Example Trace Structure (OpenTelemetry)**:
```json
{
  "trace_id": "txn_123",
  "spans": [
    {"name": "Orchestrator", "duration": 4500, "children": [
        {"name": "Retrieve_Docs", "duration": 200},
        {"name": "LLM_Reasoning", "duration": 4100, "tokens": 1024}
    ]}
  ]
}
```

---

## 3. Key Metrics to Watch (The "Dashboard")

### 3.1 Operational Metrics (Red/Green)
*   **TTFT (Time To First Token)**: Critical for user perception. Target < 200ms.
*   **TPS (Tokens Per Second)**: Reading speed. Target > 50 tokens/s.
*   **Error Rate**: 4xx/5xx errors, but also JSON parsing failures.

### 3.2 Quality Metrics (RAG)
*   **Empty Retrieval Rate**: % of queries where Vector DB returned nothing useful.
*   **Re-ranking Disagreement**: How often does Re-ranker significantly shuffle the vector results? (High shuffle = Vector DB is weak).

---

## 4. Collecting "Gold" (Feedback Loops)

Data from production is worth 10x more than synthetic data.

**Explicit Feedback**:
*   Thumbs Up/Down buttons.
*   "Regenerate" clicks (Strong negative signal).

**Implicit Feedback**:
*   **Copy/Paste Rate**: If user copies code, it was likely good.
*   **Conversation Length**: Short = Good answer? Or User gave up? (Context dependent).

**Data Flywheel Strategy**:
1.  Log all `Thumbs Down` interactions.
2.  Weekly: Review this dataset.
3.  Curate into a "Hard Negatives" dataset.
4.  Align via DPO.

---

## 5. Interview Questions

### Q: "How do you debug 'It feels slower today'?"
> **Answer**: "I look at my **Latency Decomposition** dashboard.
> 1.  Did **TTFT** spike? -> Probably Queueing at the LLM Gateway (Concurrency limit).
> 2.  Did **TPOT** (Time Per Output Token) drop? -> GPU saturation.
> 3.  Did **Retrieval Latency** spike? -> Vector DB index fragmentation or network issues."

### Q: "How do you monitor for hallucinations in production?"
> **Answer**: "It's hard to do in real-time without adding latency. I use **Sampling**.
> 1.  **Online**: Basic assertion checks (e.g., 'Does the answer contain the phone number retrieved?').
> 2.  **Offline**: Async evaluation queue. Sample 1% of logs and run them through a 'Judge Model' (GPT-4) to score Faithfulness. Alert if the daily average drops."
