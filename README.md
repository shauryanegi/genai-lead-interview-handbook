# GenAI Lead Interview Handbook V2 (The "One-Stop Shop")

A definitive knowledge base for Machine Learning Lead and Generative AI Architect roles. This handbook covers the full stack: from **Training** (LoRA/DPO) to **Architecture** (Memory/Agents) to **Production Ops** (Security/Observability).

---

## 🏗️ 1. Advanced Architectures
Insights into the building blocks of modern GenAI systems.

*   [**Transformer Deep Dive**](architectures/transformer-deep-dive.md)  
    *The Core.* Self-attention, KV caching, RoPE, and MoE.
*   [**Agent Architecture Patterns**](architectures/agent-architecture-patterns.md)  
    *The Basics.* Scoping personas, tools, and orchestration validation.
*   [**Advanced Reasoning Patterns**](architectures/advanced-reasoning-patterns.md)  
    *The Brain.* ReAct vs Plan-and-Solve, Reflexion (Self-Correction), and Tree of Thoughts.
*   [**Agent Memory Systems**](architectures/agent-memory-systems.md)  
    *The Context.* Implementing L1 (Short-term), L2 (Vector/Episodic), and L3 (Knowledge Graph) memory.
*   [**Multi-Modal RAG**](architectures/multi-modal-rag.md)  
    *The Vision.* Beyond text: ColPali, Multi-Vector Retrieval, and Vision-Language Models (VLMs).

---

## 🧪 2. Model Training & Alignment
The "Scientist" side of the role: Adapting foundation models.

*   [**Fine-Tuning Strategies**](model-training/fine-tuning-strategies.md)  
    *The Adaptation.* Full FT vs PEFT. Deep dive into LoRA (Rank, Alpha, Modules) and QLoRA.
    *The Alignment.* RLHF (PPO) vs DPO vs ORPO.

---

## ⚡ 3. Inference Operations (LLMOps)
The "Engineer" side of the role: Serving models efficiently.

*   [**LLM Serving Optimization**](inference-ops/llm-serving-optimization.md)  
    *The Speed.* Quantization (AWQ/GPTQ), Continuous Batching, PagedAttention (vLLM), and Speculative Decoding.
*   [**GPU Architecture Guide**](inference-ops/gpu-architecture.md)  
    *The Hardware.* H100 vs A100. Memory Bandwidth vs Compute Bound. Multi-GPU strategies (Pipeline vs Tensor Parallelism).
*   [**Scaling Laws (Chinchilla)**](algorithms/scaling-laws.md)  
    *The Math.* Why Llama-3 8B is "over-trained" and how to calculate optimal model size.

---

## �️ 4. Production & Security Ops (New)
The "Lead" side of the role: Reliability and Safety.

*   [**Production Observability**](production-ops/observability-and-monitoring.md)  
    *The Watchtower.* Tracing (LangSmith), Metrics (TTFT, TPS), and Feedback Loops.
*   [**Generative AI Security**](safety-ops/generative-ai-security.md)  
    *The Shield.* Guardrails (NeMo, Llama Guard), Prompt Injection Defense, and PII Masking.

---

## �🔍 5. System Design & RAG
Blueprints for scaling AI applications.

*   [**System Design Interview Blueprint**](system-design/system-design-interview-blueprint.md)
*   [**Scalable RAG Systems**](system-design/scalable-rag-systems.md)
*   [**RAG Pipeline Deep Dive**](rag-pipelines/rag-pipeline-deep-dive.md)
*   [**Vector Indexing Algorithms**](rag-pipelines/vector-indexing-deep-dive.md)
    *   *The Engine.* HNSW (Speed) vs IVF (Memory) vs Quantization (PQ/SQ).
    *   *Decision Matrix.* When to use which for 10M vs 100M vectors.
*   [**Vector Indexing Code Examples**](rag-pipelines/vector-indexing-code-examples.md)
    *   *The Implementation.* Faiss (HNSW/IVF), Qdrant, and LanceDB snippets.
*   [**Tool Calling Optimization**](rag-pipelines/tool-calling-optimization.md)

---

## 🧠 6. Machine Learning Engineering
Applied patterns for production.

*   [**Applied ML Engineering**](machine-learning/applied-ml-engineering.md)
*   [**Human-In-The-Loop (HITL)**](machine-learning/human-in-the-loop-framework.md)
*   [**Coding for ML Engineers**](machine-learning/coding-for-ml-engineers.md)

---

## 💼 7. Interview Strategy
*   [**Resume Defense Strategies**](interview-prep/resume-defense-strategies.md)

---

## 🚀 Recommended Study Path (The "Lead" Track)

1.  **Level 1: The Basics** - Review `Transformer Deep Dive` and `RAG Pipeline Deep Dive`.
2.  **Level 2: The Agentic** - Master `Advanced Reasoning` and `Agent Memory`.
3.  **Level 3: The Optimization** - Understand `Serving Optimization` (vLLM) and `Fine-Tuning`.
4.  **Level 4: The Production Lead** - Master `Security`, `Observability`, and `GPU Architecture`.
