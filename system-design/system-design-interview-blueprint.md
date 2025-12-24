# System Design Interview: Complete Answer Blueprint

This is a word-for-word blueprint for answering a system design question for a large-scale document retrieval and analysis system.

---

## The Scenario
**"Design a system that ingests 1-2 million documents and lets users query them with a latency of under 2 seconds."**

---

## Step 1: Clarifying Questions (Ask These First!)

Before diving in, ask clarifying questions to show you think like a Lead:

1.  **Data**: "What format are the documents? PDFs, JSON, or raw text?"
2.  **Scale**: "Is the 1M number the total corpus, or daily ingestion volume?"
3.  **Latency**: "Is the 2-second SLA for P50 or P99?"
4.  **Freshness**: "How 'real-time' does the index need to be? Minutes or hours after ingestion?"

---

## Step 2: High-Level Architecture (Draw This)

```
┌─────────────────────────────────────────────────────────────────────┐
│                       INGESTION PIPELINE (Batch)                     │
│  ┌─────────┐      ┌─────────────┐      ┌─────────────────────────┐  │
│  │ Sources │─────▶│ Kafka Queue │─────▶│ Spark Processing Cluster│  │
│  │ (PDFs)  │      │ (Buffering) │      │ (Parse + Chunk + Embed) │  │
│  └─────────┘      └─────────────┘      └───────────┬─────────────┘  │
│                                                    ↓                │
│                                          ┌─────────────────┐        │
│                                          │   Vector DB     │        │
│                                          │ (Milvus/Qdrant) │        │
│                                          └─────────────────┘        │
└─────────────────────────────────────────────────────────────────────┘

┌─────────────────────────────────────────────────────────────────────┐
│                       RETRIEVAL PIPELINE (Real-Time)                │
│  ┌───────────┐    ┌───────────────┐    ┌─────────────────────────┐  │
│  │ User Query│───▶│ Hybrid Search │───▶│  Cross-Encoder Re-rank  │  │
│  └───────────┘    │(Dense + BM25) │    │    (Top 100 → Top 5)    │  │
│                   └───────────────┘    └───────────┬─────────────┘  │
│                                                    ↓                │
│                                          ┌─────────────────┐        │
│                                          │ LLM Generation  │        │
│                                          │ (vLLM / Triton) │        │
│                                          └─────────────────┘        │
└─────────────────────────────────────────────────────────────────────┘
```

---

## Step 3: Component Deep-Dives

### 3.1 Why Kafka?
> "Kafka acts as a **buffer** between data sources and processing. If our Spark cluster slows down or restarts, Kafka retains the messages. It also lets us **replay** data if we need to re-index after a schema change."

### 3.2 Why Spark?
> "Spark provides **distributed processing**. When we have 1 million documents, we can spin up 100 executors to process 10,000 documents each in parallel. It handles failures gracefully—if one executor dies, Spark reassigns its work."

### 3.3 Why Milvus over Chroma?
> "Chroma is excellent for prototyping. But Milvus is built for **billion-scale vectors**. It supports **sharding** across nodes, **replicas** for high availability, and **IVF_PQ** indexes for faster approximate search. For 1-2M documents, I'd start with a single Milvus node but design for horizontal scaling."

### 3.4 Why Cross-Encoder Re-ranking?
> "Vector search is fast but 'fuzzy'. A Cross-Encoder is more expensive but sees the Query and Document **together**, enabling fine-grained matching. I use it to prune 100 candidates down to the 5 most relevant before hitting the LLM. This reduces hallucination and improves answer quality."

---

## Step 4: Latency Optimization (The "Lead" Answer)

To hit 2-second P99:

| Component | Optimization | Impact |
|-----------|--------------|--------|
| **Embedding** | Pre-compute & cache for common queries (Redis) | -200ms |
| **Vector Search** | Use IVF_PQ index, `nprobe=32` | -100ms |
| **Re-ranking** | Limit to Top-100 candidates | -300ms |
| **LLM** | vLLM with PagedAttention, Streaming | -500ms |

**Total Saved: ~1100ms**. This brings a naive 3s pipeline under 2s.

---

## Step 5: Handling Follow-Up Questions

### "What if the LLM is the bottleneck?"
> "I'd implement **Semantic Caching**. If a user asks a question semantically similar to a previous one (cosine similarity > 0.95), I return the cached answer. For novel queries, I'd add more GPU replicas behind a load balancer."

### "How do you handle document updates?"
> "Each document has a unique ID. On update, we delete the old vectors from Milvus (by ID filter) and re-ingest. For very frequent updates, I'd consider a 'soft delete' pattern and periodic compaction."

### "What about security/access control?"
> "I would add a `user_id` or `tenant_id` metadata field to every chunk. At query time, the vector search filter includes `tenant_id == current_user.tenant`. This ensures data isolation in a multi-tenant system."

---

## Conclusion: The Closing Statement

> "In summary, my design separates **Ingestion** (Kafka → Spark → Milvus) from **Retrieval** (Hybrid Search → Re-rank → LLM). This decoupling allows each layer to scale independently. The key to hitting 2-second latency is Semantic Caching and a well-tuned Re-ranking stage. I'd start with this architecture and iterate based on real-world P99 metrics."
