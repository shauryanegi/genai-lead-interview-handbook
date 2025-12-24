# Mixture of Experts (MoE): Sparse Scaling

MoE is the dominant architecture for ultra-large models (GPT-4, Mixtral, DeepSeek). It allows models to have massive parameter counts while keeping the "Cost-per-Token" low.

---

## 1. Dense vs. Sparse Models

*   **Dense Model**: Every token activates every parameter in the model (e.g., Llama-3 70B uses 70B params per token).
*   **Sparse MoE**: A token only activates a small subset of the total parameters (e.g., Mixtral 8x7B has 47B total params, but only 13B are active per token).

**The Benefit**: You get the "Knowledge Capacity" of a massive model with the "Inference Speed" of a much smaller one.

---

## 2. The Core Components

### 2.1 The Router (Gating Network)
*   **Mechanism**: For each token, the Router (a small linear layer) predicts which $K$ experts are best suited to process it.
*   **Top-K Routing**: Typically $k=2$. A token is sent to the top 2 experts, and their outputs are weighted-summed.

### 2.2 Expert Layers (FFNs)
*   Instead of one giant Feed-Forward Network (FFN), you have $N$ independent FFN "Experts".
*   Attention layers are usually shared across all experts (Dense Attention).

---

## 3. High-Level Challenges (Interview Depth)

### 3.1 Expert Specialization
*   **Myth**: One expert learns "French" and another learns "Physics".
*   **Reality**: Specialization is more syntactic/token-level than conceptual. One expert might handle "Prepositional phrases" while another handles "Numerical reasoning".

### 3.2 Routing Collapse & Load Balancing
*   **Problem**: The Router is lazy. It might find 2 "Good" experts and send *every* token to them. The other 6 experts never learn anything and die (Gradient starvation).
*   **Solution**: **Auxiliary Loss**. We add a penalty to the loss function if the token distribution across experts is unbalanced.

### 3.3 Expert Capacity & Dropping
*   In distributed training, we set a "Capacity Factor". If one expert is overloaded (too many tokens assigned), extra tokens are **dropped** (not processed by that layer) to prevent hardware idle time.

---

## 4. DeepSeekMoE: The Multi-Head Latent Expert
*DeepSeek-V3 Innovation*

DeepSeek improved MoE in two massive ways:
1.  **Fine-Grained Experts**: Instead of 8 large experts, use 64+ smaller ones. This allows for much more precise routing.
2.  **Shared Experts**: some experts are **always active** for every token. These capture "Common Knowledge", while the routed experts capture "Specialized Knowledge". This fixes many stability issues in standard MoE.

---

## 5. MoE Training vs. Inference

| Stage | Challenge | Solution |
|-------|-----------|----------|
| **Training** | VRAM overhead. | **Expert Parallelism**: Different GPUs store different experts. |
| **Inference** | High VRAM requirement (Model doesn't fit). | **Quantization** or **Offloading**. Even if only 13B params are active, all 47B must be in memory. |

---

## 6. Interview Question

### Q: "If a MoE model only uses 10B parameters per token, why can't I run it on a 16GB GPU if the total params are 50B?"
> **Answer**: "Because MoE is **Computationally Sparse** but **Memory Dense**. 
> 
> Even though a token only activates 10B parameters for math, all 50B parameters must reside in VRAM (or be swapped very fast). The router might send the next token in the sequence to *any* of the 50B parameters. Unless you have enough VRAM to hold the entire expert 'library', you will hit a massive 'IO bottleneck' trying to swap experts from Disk/CPU to GPU on every token generation. This is why MoE models are great for throughput on large clusters but hard to run on consumer hardware."
