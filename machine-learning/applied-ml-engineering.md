# Machine Learning: Applied Engineering

This cheatsheet covers practical challenges, debugging strategies, and metric selection for production ML systems.

---

## 1. Metric Selection (The "Business" View)

| Problem Type | Standard Metric | Lead-Level Metric | Why it matters |
|--------------|-----------------|--------------------|----------------|
| **Classification** | Accuracy | F1-Score | Handles imbalanced classes. |
| **Search/RAG** | Top-K Recall | **MRR** or **nDCG** | Considers the *order* of relevance. |
| **Financial/Medical** | F1-Score | **F-Beta (Beta=2)** | Weighs **Recall** higher (Missing a risk is worse than a false alarm). |
| **System Health** | P95 Latency | **Throughput (QPS)** | Measures cost-efficiency and capacity. |

---

## 2. Handling Data Imbalance (The "Security" Problem)
*In security/fraud, the "bad" events are rare (0.1%).*

1.  **SMOTE / Over-sampling**: Artificially creating more minority samples (Risk: Overfitting).
2.  **Loss Function weighting**: Tell the model that failing on a minority class is 10x more expensive.
3.  **Anomaly Detection**: Reframe as finding "the odd one out" rather than binary classification.

---

## 3. Model Debugging & "Hallucination" Controls

### Retrieval Hallucinations
- **The Symptom**: Answer sounds good but uses wrong numbers.
- **The Fix**: **Context Grounding**. Prompt the LLM: *"Use ONLY the provided context. If the answer is not there, say 'I do not know'."*

### Reasoning Hallucinations
- **The Symptom**: Model has the right facts but does the wrong math.
- **The Fix**: **Chain-of-Thought (CoT)**. Force the model to "show its work" step-by-step.
- **The Fix**: **Self-Consistency**. Run the query 3 times and pick the most common answer (Major voting).

---

## 4. Agentic Logic & Feedback Loops

### The ReAct Pattern
**Thought → Action → Observation**. This allows the model to:
1.  **Reflect** on the current status.
2.  **Tool-Call** to get missing info.
3.  **Update** its internal state.

### Improving Agent Reliability
- **Schema Engineering**: Use strictly typed tool definitions (Pydantic). Provide 2-3 examples of tool use in the docstrings.
- **Graceful Failure**: When a tool fails, return the error message to the LLM so it can try an alternative plan.
- **Human-in-the-Loop**: Route low-confidence agent steps to a human review queue before final execution.
