# LLM Inference Optimization: The Engineering of Speed

In production, **Latency is Revenue**. A Lead Engineer must know how to squeeze every millisecond out of the GPU.

---

## 1. The Bottleneck: Memory Bandwidth vs. Compute

LLM inference is usually **Memory Bound**, not Compute Bound.
*   **The Problem**: Moving weights from VRAM to Compute Units takes longer than doing the math.
*   **The Goal**: Move less data (Quantization) or move it smarter (Attention).

---

## 2. Quantization Techniques

Reducing precision from FP16 (16-bit) to INT8 or INT4.

### 2.1 Post-Training Quantization (PTQ)
Compress the weights *without* re-training.
*   **GPTQ**: Takes calibration data and minimizes the output error layer-by-layer. High accuracy 4-bit.
*   **AWQ (Activation-aware Weight Quantization)**: 
    *   *Insight*: Not all weights are equal. Weights that multiply with *large activations* are critical.
    *   *Method*: Identify salient weights (1%) and keep them precise. Quantize the rest.
    *   *Result*: Better than GPTQ for low-bit (3-bit/4-bit) inference.

### 2.2 SmoothQuant (INT8 for Activations)
*   **Concept**: Weights are easy to quantize. Activations (outliers) are hard.
*   **Trick**: Mathematically "smooth" the activation outliers by migrating the scale factor to the weights.

---

## 3. KV Cache & PagedAttention (vLLM)

### The KV Cache Problem
In autoregressive generation, we re-calculate Attention for previous tokens every step.
*   **Solution**: Cache the Key (K) and Value (V) matrices.
*   **New Problem**: The Cache grows. We must pre-allocate contiguous RAM. This leads to **Fragmentation** (wasted memory).

### The PagedAttention Fix (vLLM)
*   **Insight**: Inspired by OS Virtual Memory paging. In OS, paging prevents external fragmentation.
*   **Mechanism**: 
    1.  Divide the KV Cache into physical **blocks**.
    2.  Each block contains the Key and Value vectors for a fixed number of tokens (e.g., 16).
    3.  A **Block Table** maps logical token indices to physical block addresses.
    4.  As tokens generate, we only fetch the necessary blocks.
*   **Result**: 
    *   **Zero External Fragmentation**: We don't need contiguous memory.
    *   **Memory Sharing**: For "Copy-on-write" or parallel sampling, multiple sequences can share the same physical blocks for the shared prompt (massive memory saving for Multi-agent apps).
*   **Impact**: Increases throughput by 2-4x over HuggingFace `generate()`.

---

## 4. Speculative Decoding

**Concept**: Big models are smart but slow. Small models are dumb but fast.

**Workflow**:
1.  **Draft**: A tiny "Draft Model" (e.g., Llama-68M) quickly generates 5 tokens.
2.  **Verify**: The big "Target Model" (Llama-70B) runs *one* forward pass to check all 5 tokens in parallel.
3.  **Accept/Reject**: If Draft tokens match Target probabilities, keep them. If not, discard and correct.

**Gain**: If the draft is good, you get 5 tokens for the cost of 1 forward pass. typically **2x latency reduction**.

---

## 5. Continuous Batching (Orca)

**Traditional Batching**: Wait for all requests to finish. Short queries get blocked by long ones.
**Continuous Batching**: It's iteration-level scheduling.
*   When Query A finishes, immediately evict it and slot in Query C, while Query B is still generating.
*   **Result**: GPUs never sit idle. 20x throughput improvement.

---

## 6. Interview "Deep Dive" Questions

### Q: "How would you optimize latency for a chatbot vs. throughput for batch processing?"
> **Answer**:
> *   **Chatbot (Latency Sensitive)**:
>     *   Use **Quantization** (AWQ 4-bit) to fit weights in faster memory tiers.
>     *   Use **Speculative Decoding** to generate tokens faster.
>     *   Batch size = 1 (or small).
> *   **Batch (Throughput Sensitive)**:
>     *   Use **vLLM** with **Continuous Batching**.
>     *   Maximize Batch Size until VRAM is full.
>     *   Latency per user increases, but Total Tokens/Sec skyrockets.

### Q: "Explain the difference between GPTQ and AWQ."
> **Answer**: "Both are PTQ methods.
> *   **GPTQ** optimizes weights to minimize reconstruction error based on the Hessian of the loss.
> *   **AWQ** focuses on activations. It assumes that weights corresponding to large activation magnitudes are more important. It creates a scaling factor to protect those salient weights before quantization.
> *   Practically: AWQ provides better perplexity at very low bitrates (4-bit) and has faster inference kernels in some frameworks."
