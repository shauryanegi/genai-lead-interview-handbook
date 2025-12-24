# Attention Mechanisms: MHA, MQA, GQA, MLA, and SWA

The evolution of the "Attention" layer is driven by one goal: **Inference Efficiency**.

---

## 1. MHA (Multi-Head Attention)
*The Original (Attention Is All You Need)*

*   **Mechanism**: Every Query head ($Q$) has its own Key ($K$) and Value ($V$) head.
*   **Problem**: **KV Cache Bloat**. For long sequences, the $K$ and $V$ matrices consume massive memory (Memory Bandwidth Bound).
*   **Formula**: $\text{KV Cache Size} \propto 2 \times \text{layers} \times \text{heads} \times \text{dim} \times \text{seq\_len}$

---

## 2. MQA (Multi-Query Attention)
*The Extreme Optimizer (PaLM)*

*   **Mechanism**: Multiple $Q$ heads share a **single** $K$ and $V$ head.
*   **Pro**: Drastically reduces KV cache size (by $1/\text{heads}$). Massive inference speedup.
*   **Con**: **Quality Drop**. The model loses the ability to distinguish different semantic relationships across different head subspaces.

---

## 3. GQA (Grouped-Query Attention)
*The Sweet Spot (Llama-2, Llama-3)*

*   **Mechanism**: $Q$ heads are grouped, and each group shares one $K$ and $V$ head.
*   **Llama-3 example**: 32 $Q$ heads, 8 $KV$ groups (4 $Q$ heads per $KV$).
*   **Impact**: Near-MHA quality with near-MQA speed. The industry standard for production serving.

| Mechanism | Ratio (Q:K:V) | Memory Usage | Quality |
|-----------|---------------|--------------|---------|
| **MHA** | $H : H : H$ | 100% | Ultra High |
| **MQA** | $H : 1 : 1$ | ~5% | Medium |
| **GQA** | $H : G : G$ | ~25% | Very High |

---

## 4. MLA (Multi-Head Latent Attention)
*The DeepSeek-V3 Masterpiece*

*   **Mechanism**: Low-rank compression of the KV Cache. Instead of storing full $K$ and $V$, it stores a **compressed latent vector**.
*   **Up-projection**: During inference, it uses a small matrix to "up-project" the latent vector back to the original $K, V$ space on the fly.
*   **The Result**: DeepSeek-V3 achieves **higher accuracy than GQA** while having a **smaller KV cache footprint than MQA**. It is the current state-of-the-art for throughput.

---

## 5. SWA (Sliding Window Attention)
*Handling Infinite Context (Mistral-7B)*

*   **Mechanism**: A token only attends to the last $W$ tokens (the "Window"), rather than the whole context.
*   **Memory Gain**: Constant KV cache size regardless of sequence length.
*   **The Trick (Dilated/Global tokens)**: By stacking SWA layers, higher layers effectively have a larger receptive field ("Information jumps"), allowing for long-range dependency without $O(N^2)$ cost.

---

## 6. Interview Deep Dive Question

### Q: "Explain the transition from MHA to GQA in production LLMs."
> **Answer**: "The main bottleneck in LLM inference is **Memory Bandwidth**, specifically the loading of the **KV Cache** from VRAM to the compute units. 
> 
> In **MHA**, the KV Cache scales linearly with the number of heads, which limits our batch size (throughput). **MQA** solved this by sharing one KV pair for all Q heads, but it hurt model reasoning because the head subspaces were too constrained.
> 
> **GQA** (used in Llama-3) is the compromise: we group heads (e.g., 4 Q heads per KV head). This reduces memory traffic by 8x compared to MHA while maintaining almost identical performance benchmarks. It allows for significantly larger batch sizes and higher throughput in frameworks like **vLLM**."
