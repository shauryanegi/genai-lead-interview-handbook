# 🧠 Technical Deep Dive Script (The "10/10" Answers)
**Use this for the specific technical drill-down questions.**

---

## ⚡ SECTION 1: Transformers & Attention Mechanisms

### Q: "Explain the evolution of Attention from MHA to GQA."
> "It's all about the **KV Cache bottleneck**.
>
> 1.  **MHA (Multi-Head Attention)**: The original. Every Query head has its own Key/Value head. Great quality, but the KV Cache grows huge (Memory Bound).
> 2.  **MQA (Multi-Query Attention)**: The extreme fix. All Query heads share **one** KV head. Fast, but quality drops because the model loses capacity to track different semantic subspaces.
> 3.  **GQA (Grouped-Query Attention)**: The Goldilocks zone (Llama-3). We group Query heads (e.g., 4 Queries share 1 KV). It cuts memory usage by 8x compared to MHA while keeping near-perfect accuracy. It's the standard for 70B+ models."

### Q: "What is the innovation in DeepSeek's MLA (Multi-Head Latent Attention)?"
> "MLA attacks the KV Cache size using **Low-Rank Compression**.
>
> Instead of storing the full Key and Value matrices in VRAM, it projects them down into a much smaller 'Latent Vector' (compressed form). During inference, it uses a 'RoPE-aware' up-projection to recover the attention scores.
>
> **The Impact**: It allows DeepSeek to serve massive context windows with significantly less VRAM than even GQA models."

### Q: "How does Sliding Window Attention (SWA) work?"
> "Used in Mistral. Instead of attending to *all* previous tokens (Quadratic cost), each token only attends to a fixed window (e.g., last 4096 tokens).
>
> **The Trick**: Because layers stack, the 'Receptive Field' still grows effectively infinite at higher layers, but the compute cost remains linear."

---

## 📂 SECTION 2: Advanced RAG Architectures

### Q: "Bi-Encoder vs. Cross-Encoder vs. ColBERT?"
> "This is the trade-off between **Speed** and **Precision**.
>
> 1.  **Bi-Encoder (Dense Retrieval)**: Encodes Query and Doc independently. fast (dot product) but misses nuance. Use this for **Recall** (finding top 100).
> 2.  **Cross-Encoder**: Encodes (Query + Doc) together. Very precise but slow (full attention). Use this for **Re-ranking** (sorting top 10).
> 3.  **ColBERT (Late Interaction)**: The middle ground. It keeps token-level embeddings and computes MaxSim. It's 10x faster than Cross-Encoder with similar precision, but requires 10x more storage for the index."

### Q: "How do you solve the 'Lost in the Middle' phenomenon?"
> "LLMs tend to ignore information in the middle of a long context.
>
> 1.  **Re-ranking Strategy**: I simply re-order the retrieved chunks so the **most relevant** chunks are at the **start** and **end** of the prompt, leaving the less relevant ones in the middle.
> 2.  **Context Windowing**: Or I break the context into smaller overlapping windows dealing with specific sub-questions."

### Q: "GraphRAG vs. Vector RAG?"
> "Vector RAG finds **similar types** of things (Semantic similarity).
> GraphRAG finds **connected** things (Structural relationships).
>
> **Example**: If I ask 'What connects Actor A and Malware B?', Vector search might fail if their text descriptions are different. GraphRAG traverses the edge `Actor A --wrote--> Malware B` to give the exact answer."

---

## 🤖 SECTION 3: Agentic Systems (The "Brain")

### Q: "Explain ReAct architecture."
> "**ReAct** stands for **Re**asoning + **Act**ing.
>
> It's a loop where the LLM generates a **Thought** ('I need to check the balance'), then an **Action** (Tool Call: `get_balance()`), waits for the **Observation** (API Result: `$100`), and then repeats.
>
> It's powerful for ambiguous tasks but can get stuck in loops. I prefer **Plan-and-Solve** for production, where the plan is generated upfront."

### Q: "What is the Model Context Protocol (MCP)?"
> "It's the 'USB-C' for Agents.
>
> Instead of hardcoding tool definitions (Zapier, Google Drive) into the Agent's prompt, MCP defines a standard client-host handshake. The Agent connects to an MCP Server, which declares 'Here are the tools I have'.
>
> This decouples the **Model** (Claude/GPT) from the **Context** (Data/Tools), making the system modular and secure."

### Q: "How do you handle Multi-Agent Orchestration?"
> "I use a **Hierarchical (Supervisor)** pattern for complex reports like the Credit Memo.
>
> 1.  **Supervisor Agent**: Breaks the user request into sub-tasks (Risk, Financial, Legal).
> 2.  **Worker Agents**: Execute specific RAG tasks in parallel.
> 3.  **Supervisor**: Synthesizes the outputs.
>
> This reduces latency (parallelism) and hallucinations (specialized prompts for each worker)."

---

## 📊 SECTION 4: Training & Evaluation

### Q: "RLHF vs. DPO?"
> "They both align models to human preference, but **DPO is mathematically simpler**.
>
> *   **RLHF (PPO)**: Requires training a separate **Reward Model** to score outputs, then using PPO to update the Policy Model. It's unstable and RAM-heavy.
> *   **DPO (Direct Preference Optimization)**: Removes the Reward Model. It uses the binary preference data (Winner vs. Loser) directly in the loss function to effectively 're-weight' the model. It's stable and runs on standard SFT hardware."

### Q: "How do you evaluate a RAG system?"
> "I use the **RAGAS** framework to measure the independent failure points:
>
> 1.  **Faithfulness**: (Generator Check) Did the LLM make things up not in the context?
> 2.  **Context Recall**: (Retriever Check) Did we fail to retrieve the ground truth?
>
> Monitoring these separately tells me if I need to fix my **Prompt** or my **Vector DB**."
