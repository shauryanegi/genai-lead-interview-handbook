# Agent Memory Systems: A Deep Dive

"Memory" is what separates a **Chatbot** from an **Agent**.

A chatbot answers your question and forgets.
An agent **learns**, **recalls**, and **evolves**.

---

## 1. The Memory Hierarchy

Just like a computer (RAM vs Disk) or a human (Working vs Long-term), agents need tiered memory.

| Tier | Type | Human Analogy | Technical Implementation |
|------|------|---------------|--------------------------|
| **L1** | **Short-Term (Working)** | "What did you just say?" | Context Window (Message List) |
| **L2** | **Episodic (Long-Term)** | "What happened last week?" | Vector Database (RAG) |
| **L3** | **Semantic (Knowledge)** | "What is the capital of France?" | Knowledge Graph / Learned Weights |
| **L4** | **Procedural (Muscle)** | "How do I ride a bike?" | Few-Shot Examples / Tool Definitions |

---

## 2. L1: Short-Term Memory Management

**Challenge**: Context windows are finite (even 1M tokens get slow/expensive).
**Solution**: We can't keep everything. We need **Eviction Policies**.

### 2.1 FIFO (Rolling Window)
*   **Mechanism**: Keep last $N$ turns. Drop the oldest.
*   **Pros**: Simple, predictable cost.
*   **Cons**: You forget the crucial instruction given at the start.

### 2.2 Conversation Summary Buffer
*   **Mechanism**: When the buffer fills, an LLM summarizes the oldest interactions into a "System Note" and appends it to the top.
*   **Prompt**: *"Summarize the conversation so far, retaining key preferences and facts."*
*   **Result**: Context remains small, but "gist" is preserved.

### 2.3 Token Selection (Key-Value)
*   **Mechanism**: Use an algorithm (like Attention sinks) to keep *only* high-importance tokens, dropping "stopwords" and filler.

---

## 3. L2: Episodic Memory (Vector Stores)

This is reliable "Recall".

### The "Time-Aware" Retrieval Problem
Standard RAG retrieves by *similarity*. But human memory retrieves by *similarity + recency + importance*.

**The "Memary" Architecture Strategy**:
Score = $(w_1 \times \text{Relevance}) + (w_2 \times \text{Recency}) + (w_3 \times \text{Importance})$

1.  **Relevance**: Cosine similarity (Standard RAG).
2.  **Recency**: Exponential decay function ($e^{-\lambda t}$). Recent events feature higher.
3.  **Importance**: An LLM "reflection" step that rates a memory's importance (1-10) at write time.

**Implementation**:
```python
def retrieve_memory(query, memories):
    for mem in memories:
        sim_score = cosine(query, mem.vector)
        recency_score = 0.99 ** (hours_since(mem.timestamp))
        final_score = (0.6 * sim_score) + (0.4 * recency_score)
    return top_k(final_score)
```

---

## 4. L3: Semantic Memory (Knowledge Graphs)

Vectors are fuzzy. Graphs are precise.

**Use Case**: User says *"I moved to London."*
*   **Vector**: Might vaguely associate user with UK cities.
*   **Graph**: Explicitly updates edge: `(User) --[LIVES_IN]--> (London)`.

**Architecture**:
1.  **Extraction Agent**: Runs after every turn.
2.  **Triples**: Extracts `(Subject, Predicate, Object)`.
3.  **Merge**: Upserts into a Graph DB (Neo4j).

> **Pro Tip**: Use this for **User Profiles**. Vectors for "vibes", Graphs for "facts".

---

## 5. L4: Procedural Memory (Tools & Reflexion)

Agents improve by remembering *how* to solve problems.

**Technique: Reflexion**
1.  Agent tries task -> Fails.
2.  Reflexion Step: "Why did I fail?" -> "I used the wrong date format."
3.  **Save this insight** to Procedural Memory.
4.  Next run: Retrieve relevant "past failures" and add to context: *"Tip: Don't use the wrong date format."*

---

## 6. Interview "Deep Dive" Questions

### Q: "How do you handle context overflow in long conversations?"
> **Answer**: "I use a **tiered strategy**.
> 1.  **Immediate Context**: Raw text for the last 10 turns.
> 2.  **Summary Buffer**: A running summary of the session for high-level context.
> 3.  **Vector Store**: For recalling specific details from 100 turns ago.
> This creates an 'infinite' memory feel without the latency of a 1M token window."

### Q: "Vector search often returns outdated info. How do you fix that?"
> **Answer**: "I implement a **Time-Decay Reranker**.
> I don't just sort by cosine similarity. I penalize older memories using an exponential decay function.
> If the user asks 'What is my address?', the address from 2024 should outweigh the one from 2020, even if the wording is identical."

---
