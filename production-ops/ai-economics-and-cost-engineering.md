# AI Economics & Cost Engineering

A Lead AI Engineer isn't just a scientist; they are a **P&L Manager**. Every token has a price.

---

## 1. The Unit Economics of AI

### 1.1 The Token Tax
In production, your primary costs are:
1.  **Input Tokens** (The Context)
2.  **Output Tokens** (The Generation)
*   **Optimization**: RAG context (Input) is often 10x larger than the answer (Output). This makes **Prompt Compression** and **Context Caching** high-priority engineering tasks.

---

## 2. Cost-Reduction Strategies

### 2.1 Model Cascading (The Router Pattern)
Don't use GPT-4 for everything.
*   **Step 1**: Use a cheap model (Llama-3 8B / GPT-4o-mini) to classify the query difficulty.
*   **Step 2**: If "Easy" $\rightarrow$ Answer with cheap model.
*   **Step 3**: If "Hard" $\rightarrow$ Route to expensive model (Claude 3.5 / GPT-4o).
*   **ROI**: Can reduce bills by **60-80%** with < 2% drop in accuracy.

### 2.2 Speculative Decoding
Using a small model (the "Drafter") to guess the next 5 tokens, and a big model (the "Verifier") to check them in parallel.
*   **Impact**: Up to **2x speedup** on inference without changing the output quality.

---

## 3. RAG vs. Fine-Tuning: The ROI Test

| Factor | RAG | Fine-Tuning |
|--------|-----|-------------|
| **Cost** | High (Per request context) | High (Upfront training) |
| **Updates** | Real-time | Static (Until re-train) |
| **Precision** | High (Source citation) | Medium (Hallucinations) |
| **Recommendation** | Use RAG for **Knowledge**. | Use Fine-Tuning for **Format/Style/Behavior**. |

---

## 4. The "GPU Math" for Leads
*   **Compute Bound**: When you have a massive prompt and small output. (Load the model once, do lots of math).
*   **Memory Bandwidth Bound**: When you are generating long text. (The KV Cache must be pulled from VRAM for every single token).
*   **Strategy**: To save money on long generations, move to **Quantized models (GGUF/EXE)** or **GQA-based models**.

---

## 5. Interview Questions

### Q: "How do you justify the cost of an LLM project to a CFO?"
> **Answer**: "I focus on **Total Cost of Ownership (TCO)** and **Productivity Gain**. 
> 1.  **Baseline**: How much does it cost humans to do this today? (e.g., $50/hour).
> 2.  **LLM Cost**: ($0.50/request). 
> 3.  **Efficiency**: Even with a Human-in-the-Loop review, if the LLM speeds up the process by 10x, the project pays for itself in 3 months. I also show the roadmap for **Model Cascading** to drop costs further as we scale."

### Q: "What is Context Caching and how does it save money?"
> **Answer**: "In many apps, users ask questions about the same 100-page PDF. Without caching, the LLM 're-reads' the PDF for every question (expensive). **Context Caching** (available in Gemini/DeepSeek/Anthropic) allows us to store the pre-computed KV Cache of that PDF on the server. Subsequent requests only pay a small 'storage fee' and zero 'input fee' for the cached part, often reducing costs by 90% for high-context apps."
