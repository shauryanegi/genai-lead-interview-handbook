# Transformer Architecture: Deep Dive

This guide covers the core architecture of Transformer models, recent performance optimizations, and common implementation patterns used in enterprise-scale AI systems.

---

## 1. Core Architecture

### 1.1 Self-Attention Mechanism
The intuition behind self-attention is that every token in a sequence should be able to "look at" and weigh its relationship with every other token to capture context.

**The Foundational Formula**:
```
Attention(Q, K, V) = softmax(Q × K^T / √d_k) × V
```

| Component | Role | Intuition |
|-----------|------|-----------|
| **Q** | Query | "What am I looking for?" |
| **K** | Key | "What information do I contain?" |
| **V** | Value | "The content to be weighted and sum/averaged." |
| **√d_k** | Scaling | Prevents the dot product from growing too large, which would stabilize gradients in the softmax layer. |

---

### 1.2 Multi-Head Attention (MHA)
Rather than computing a single attention score, we project Q, K, and V into multiple subspaces (heads) in parallel. This allows the model to simultaneously attend to information from different representation subspaces at different positions.

- **Example**: Head 1 might focus on syntax (which noun follows which adjective), while Head 2 focuses on coreference resolution ("he" refers to "the CEO").

---

### 1.3 Position Encodings
Since Transformers lack recurrence (unlike RNNs) or convolution (unlike CNNs), they have no inherent sense of the order of words. We add "positional encodings" to the input embeddings.

| Method | Description | Usage |
|--------|-------------|-------|
| **Sinusoidal** | Fixed sine/cosine waves of varying frequencies. | Original Transformer |
| **Learned Absolute** | Distinct embeddings for positions 1, 2, 3... | BERT, GPT-2 |
| **RoPE (Rotary)** | Rotates the Q and K vectors in a complex space. | Llama, Mistral |
| **ALiBi** | Adds a linear penalty to attention scores based on distance. | BLOOM |

---

## 2. Recent Performance Optimizations

### 2.1 Flash Attention
**The Problem**: Standard attention is $O(N^2)$ in memory, which makes long-context retrieval (e.g., 32k tokens) prohibitively expensive.
**The Solution**: Flash Attention computes the attention block by block (tiling) without materializing the massive attention matrix in memory.
**Impact**: 2-4x speedup and significant memory savings, enabling the long-context windows we see in modern LLMs.

---

### 2.2 Grouped Query Attention (GQA)
**Problem**: Storing the Key-Value (KV) cache for every head in Multi-Head Attention consumes massive GPU VRAM during inference.
**Solution**: Multiple "Query" heads share a single "Key" and "Value" head.
- **MHA**: 1 Q per 1 K/V
- **MQA (Multi-Query)**: All Qs share 1 K/V (Performance loss)
- **GQA**: A group of Qs share 1 K/V (The sweet spot used in Llama-3/Mistral)

---

### 2.3 Mixture of Experts (MoE)
Instead of a single large Feed-Forward Network, the model contains **N experts**. For every token, a **Router** selects only a small subset (e.g., 2 experts) to perform the computation.
- **Benefit**: You get the representational power of a 50B parameter model but the inference speed/cost of a 12B model (since only a fraction of parameters are active per token).

---

## 3. Serving Architectures: vLLM & PagedAttention

In production, the bottleneck is often **Memory Fragmentation** in the KV Cache.
- **Old Way**: Allocating a fixed, contiguous memory block for every request.
- **PagedAttention**: Divides the KV cache into non-contiguous "pages" (similar to virtual memory in OS).
- **Result**: Near-zero memory waste, allowing for significantly higher batch sizes (throughput) on the same hardware.

---

## 4. Whiteboard Diagram Template

```text
Input Tokens
     ↓
[Embedding + Position Encodings]
     ↓
┌─────────────────────────────────┐
│     Transformer Block (x N)     │
│  ┌───────────────────────────┐  │
│  │ Multi-Head Attention      │  │
│  │ (Scaled Dot-Product)      │  │
│  └───────────┬───────────────┘  │
│              │ Add & Norm       │
│              ↓                  │
│  ┌───────────────────────────┐  │
│  │ Feed-Forward Network      │  │
│  │ (Up-proj → non-lin → down)│  │
│  └───────────┬───────────────┘  │
│              │ Add & Norm       │
└──────────────┼──────────────────┘
               ↓
          Output Logits
```
