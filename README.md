# GenAI Lead Interview Handbook V2 (The "One-Stop Shop")

A definitive knowledge base for Machine Learning Lead and Generative AI Architect roles. This handbook covers the full stack: from **Training** (LoRA/DPO) to **Architecture** (Memory/Agents) to **Inference Engineering** (vLLM/Quantization).

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

---

## 🧪 2. Model Training & Alignment
The "Scientist" side of the role: Adapting foundation models.

*   [**Fine-Tuning Strategies**](model-training/fine-tuning-strategies.md)  
    *The Adaptation.* Full FT vs PEFT. Deep dive into LoRA (Rank, Alpha, Modules) and QLoRA.
*   [**Preference Alignment**](model-training/preference-alignment.md) *(Coming Soon)*  
    *The Safety.* RLHF (PPO) vs DPO vs ORPO.

---

## ⚡ 3. Inference Operations (LLMOps)
The "Engineer" side of the role: Serving models efficiently.

*   [**LLM Serving Optimization**](inference-ops/llm-serving-optimization.md)  
    *The Speed.* Quantization (AWQ/GPTQ), Continuous Batching, PagedAttention (vLLM), and Speculative Decoding.
*   [**GPU Architecture Guide**](inference-ops/gpu-architecture.md) *(Coming Soon)*  
    *The Hardware.* Memory Bandwidth vs Compute Bound.

---

## 🔍 4. System Design & RAG
Blueprints for scaling AI applications.

*   [**System Design Interview Blueprint**](system-design/system-design-interview-blueprint.md)
*   [**Scalable RAG Systems**](system-design/scalable-rag-systems.md)
*   [**RAG Pipeline Deep Dive**](rag-pipelines/rag-pipeline-deep-dive.md)
*   [**Tool Calling Optimization**](rag-pipelines/tool-calling-optimization.md)

---

## 🧠 5. Machine Learning Engineering
Applied patterns for production.

*   [**Applied ML Engineering**](machine-learning/applied-ml-engineering.md)
*   [**Human-In-The-Loop (HITL)**](machine-learning/human-in-the-loop-framework.md)
*   [**Coding for ML Engineers**](machine-learning/coding-for-ml-engineers.md)

---

## 💼 6. Interview Strategy
*   [**Resume Defense Strategies**](interview-prep/resume-defense-strategies.md)

---

## 🚀 Recommended Study Path (The "Lead" Track)

1.  **Level 1: The Basics** - Review `Transformer Deep Dive` and `RAG Pipeline Deep Dive`.
2.  **Level 2: The Agentic** - Master `Advanced Reasoning` and `Agent Memory`.
3.  **Level 3: The Optimization** - Understand `Serving Optimization` (vLLM) and `Fine-Tuning` (LoRA).
4.  **Level 4: The System** - Practice `System Design` using the Blueprint.
