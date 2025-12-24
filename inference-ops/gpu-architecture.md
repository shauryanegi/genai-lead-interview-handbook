# GPU Architecture for GenAI Leads

You don't need to be a CUDA kernel engineer, but you must know why your H100s are worth the money (or not).

---

## 1. The Compute Hierarchy

| Component | Function | Analogy |
|-----------|----------|---------|
| **VRAM (HBM)** | Stores Weights & KV Cache. | The Library Bookshelf. |
| **SRAM (L1/L2)** | Ultra-fast scratchpad for Tensor Cores. | The Desk. |
| **Tensor Cores** | Matrix Multiplication Units (matmul). | The Calculator. |
| **Memory Bandwidth** | Speed of moving data VRAM <-> Compute. | The Librarian's running speed. |

---

## 2. The Bottleneck: Memory Wall

**Crucial Concept**: LLM Inference is **Memory Bandwidth Bound**, not Compute Bound.
*   **Math**: An A100 has 312 TFLOPS (Compute) but only 2 TB/s (Bandwidth).
*   **Result**: The GPU spends most of its time *waiting* for data to arrive from VRAM.

**Arithmetic Intensity**:
$$ \text{Intensity} = \frac{\text{FLOPS}}{\text{Bytes Access}} $$
*   **Prompt Phase**: High Intensity (Compute Bound). We process 1000s of tokens at once matrix-style.
*   **Generation Phase**: Low Intensity (Memory Bound). We generate 1 token, read ALL weights, generate next token, read ALL weights again.

**Implication**: Buying an H100 vs A100 for *generation* is often about the **3TB/s vs 2TB/s** bandwidth upgrade, not just the FLOPs.

---

## 3. GPU Comparison Cheat Sheet

| GPU | VRAM | Bandwidth | Best For |
|-----|------|-----------|----------|
| **H100** | 80GB | 3.35 TB/s | Training, High-Throughput Serving (vLLM) |
| **A100 (80GB)** | 80GB | 2.0 TB/s | Standard Enterprise Serving |
| **A100 (40GB)** | 40GB | 1.6 TB/s | Smaller Models (7B-13B) |
| **L40S** | 48GB | 0.86 TB/s | Fine-Tuning (LoRA), Graphics |
| **A10G** | 24GB | 0.6 TB/s | Budget Inference, Dev Environments |

---

## 4. Multi-GPU Strategies

When the model doesn't fit on one card.

### 4.1 Pipeline Parallelism (PP)
*   **How**: Layers 1-10 on GPU0, Layers 11-20 on GPU1.
*   **Pro**: Simple.
*   **Con**: **Bubble**. GPU1 sits idle while GPU0 works (and vice versa).

### 4.2 Tensor Parallelism (TP)
*   **How**: Split the *matrices* themselves. $A \times B$ becomes $A_1 \times B$ on GPU0 and $A_2 \times B$ on GPU1.
*   **Pro**: Extremely fast. No bubbles.
*   **Con**: Requires massive interconnect speed (NVLink). Cannot run effectively over Ethernet/InfiniBand.

### 4.3 Data Parallelism (DP) - Training Only
*   **How**: Copy full model to GPU0 and GPU1. Give Batch A to GPU0, Batch B to GPU1. Average gradients.
*   **Pro**: Linear scaling.

---

## 5. Interview Questions

### Q: "Why is the H100 so much faster than the A100?"
> **Answer**: "It's not just the FP8 Tensor Cores (which are 6x faster). The real killer feature for inference is the **Transformer Engine** and the **3.35 TB/s Bandwidth**. It allows us to move weights faster, which is the bottleneck for single-user latency."

### Q: "How do you size a GPU for a 70B model?"
> **Answer**:
> 1.  **Weights**: 70B parameter $\times$ 2 bytes (FP16) = 140GB VRAM.
> 2.  **KV Cache**: Needs ~20-30GB for long contexts.
> 3.  **Total**: ~170GB.
> 4.  **Hardware**: Requires **2x A100 (80GB)** or **4x A6000 (48GB)** using **Tensor Parallelism**.
