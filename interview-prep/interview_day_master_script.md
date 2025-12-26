# 🚀 Tech Lead Interview - "Flight Deck" Script
**Use this document during the interview. Scroll to the relevant section.**

---

## 🟢 PART 1: The Intro & Resume Defense
*For the Hiring Manager & First Round*

### 🗣️ "Tell me about yourself." (30s)
> "I'm an ML engineer with **6+ years of experience**, specializing in **production LLM systems**.
>
> I have a Master's from **Georgia Tech** (Deep Learning/NLP specialization).
>
> Most recently at Arya.ai, I built an **Agentic RAG platform** for credit analysis that reduced the reporting cycle from **3 weeks to 3 days** (85% reduction).
>
> I'm excited about CloudSEK because I want to apply my expertise in **Retrieval & Agents** to **cybersecurity at scale**—catching threats before they cause damage."

### 🗣️ "Tell me about your RAG Project." (The Star)
> "I built an **end-to-end Agentic RAG pipeline** to automate Credit Memo generation.
>
> **The Problem**: Analysts spent weeks digging through PDFs for numbers.
>
> **The Solution**:
> 1. **Hybrid Retrieval**: Combined Vector Search (ChromaDB) with Keyword Search to catch specific financial terms.
> 2. **Cross-Encoder Re-ranking**: Improved precision by re-scoring top candidates, boosting retrieval accuracy from 70% to 91%.
> 3. **MCP Agents**: Built specialized agents (Risk, Financial) that coordinated via **Model Context Protocol** to generate the final report."
>
> **The Impact**: 85% time reduction."

### ⚡ Technical Deep Dives (Be Ready)
*   **Why Cross-Encoders?**: "Bi-Encoders satisfy speed (latency), Cross-Encoders satisfy precision. I use Cross-Encoders only on the Top 20 results to get the best of both worlds."
*   **Why vLLM?**: "To solve memory fragmentation. **PagedAttention** allowed me to increase batch size from 4 to 12, reducing P99 latency by 22%."
*   **Why MCP?**: "Standardization. It decoupled my agents from the tools. When backend changed a tool, I didn't have to rewrite the agent prompt."

---

## 🟡 PART 2: System Design (The Whiteboard)
*For Puneet & Technical Panel*

### ❓ Prompt: "Design a Scalable RAG System" (2 mins)

**1. Clarify**: "Scale = 2M docs? Latency < 2s? Freshness = Minutes?"

**2. The Ingestion Pipeline**:
> "I'd use **Kafka** for backpressure handling of incoming documents.
> **Spark Streaming** consumers pull from Kafka to chunk and embed documents in parallel.
> **Milvus** (Distributed) stores the vectors with an **IVF_PQ** index for 16x memory compression."

**3. The Retrieval Pipeline**:
> "Hybrid Search (Dense + Sparse).
> **Reciprocal Rank Fusion (RRF)** combines the scores.
> **Cross-Encoder** re-ranks top 50.
> **vLLM** serves the Llama-3 model for final answer generation."

**4. The "CloudSEK" Twist (Threat Intel)**:
> "For Threat Intel specifically, I would add **Entity Extraction (NER)** before embedding to pull out IP addresses and CVEs. I'd also consider a **Graph Database** to link 'Actor A' to 'Malware B', which simple vector search might miss."

---

## 🔵 PART 3: Evaluation & Robustness
*For Sravanthi & Research Panel*

### 🗣️ "How do you evaluate RAG?"
> "I generally use the **RAGAS Framework**.
> 1. **Faithfulness**: Is the answer grounded in the context?
> 2. **Context Precision**: Did we retrieve the right chunks?
>
> In production, I also track **Human Correction Rate**—using analyst edits as a feedback loop to find 'Hard Negatives' for my retriever."

### 🗣️ "How do you handle Hallucinations?"
> "Defensively.
> 1. **Strict System Prompt**: 'If not in context, say I don't know.'
> 2. **Citation Requirement**: Model must output `[Doc ID, Page #]` for every claim.
> 3. **Validation**: A post-processor Regex checks if the cited text actually exists in the retrieved chunk."

---

## 🟣 PART 4: Behavioral & Closing
*For Leadership*

### 🗣️ "Why CloudSEK?"
> "Two reasons: **Scale** and **Mission**.
> You deal with internet-scale threat data—that's an engineering challenge I want.
> And stopping cyberattacks is impactful work. I want my code to protect people, not just optimize clicks."

### 🗣️ Questions to Ask Them
1.  **For Puneet**: "How do you currently balance Recall vs. Alert Fatigue? Do you prefer missing a threat or spamming the analyst?"
2.  **For Mayank**: "What is the biggest technical bottleneck currently preventing the team from scaling GenAI?"
3.  **For Team**: "What does a 'successful first 6 months' look like for this role?"

---

## 📝 Cheatsheet Numbers
*   **Latency**: Reduced by **22%** (vLLM).
*   **Hallucinations**: Reduced from **23%** to **4%** (Strict Prompting + Eval).
*   **Impact**: **3 weeks** -> **3 days** (85% faster).
*   **Boilerplate Code**: Reduced by **60%** (MCP).
