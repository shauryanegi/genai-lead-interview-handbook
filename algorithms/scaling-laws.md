# Scaling Laws: The Physics of LLMs

Why do we train 7B, 13B, and 70B models? It's not random. It's **Chinchilla Optimal**.

---

## 1. The Kaplan Scaling Laws (2020) vs. Chinchilla (2022)

### Kaplan (OpenAI)
*   **Hypothesis**: Model Size ($N$) matters most.
*   **Recommendation**: Big models, fewer tokens.
*   **Result**: GPT-3 (175B params) was trained on only 300B tokens. It was **undertrained**.

### Chinchilla (DeepMind)
*   **Hypothesis**: Compute ($C$) is the constraint. For a fixed compute budget, you need to balance $N$ and Data ($D$).
*   **The Golden Ratio**: **20 Tokens per Parameter**.
    *   $D \approx 20N$
*   **Implication**:
    *   For a 10B model, you need 200B tokens.
    *   For a 70B model, you need 1.4T tokens (Llama-2 used 2T, Llama-3 used 15T).

---

## 2. "Llama-3" and Over-Training

Models like **Llama-3 8B** defy Chinchilla.
*   **Chinchilla Optimal**: 8B params $\times$ 20 = 160B tokens.
*   **Llama-3 Training**: 15 Trillion tokens (approx 100x optimal).

**Why? Inference Economics.**
*   **Chinchilla** optimizes *Training Cost*.
*   **Production** cares about *Inference Cost*.
*   A small model (8B) trained on massive data (15T) is "Smarter" than a large model (70B) trained on little data, but is **10x cheaper to serve**.
*   **Conclusion**: For open weights, we **massively over-train** small models to make them punch above their weight class.

---

## 3. The Power Laws

Performance ($L$) Scales as a power law with compute ($C$):

$$ L(C) \propto C^{-\alpha} $$

*   **Meaning**: Diminishing returns. To decrease error linearly, you must increase compute exponentially.
*   **Unlock**: This allows us to **Predict** the performance of GPT-4 by training a small GPT-2 scale model. We extrapolate the curve.

---

## 4. Interview Questions

### Q: "I have 1 Billion Tokens of proprietary data. How big should my model be?"
> **Answer**: "Using the Chinchilla ratio ($20:1$):
> $1,000,000,000 / 20 = 50,000,000$ params.
> A **50M parameter model** is optimal.
> Training a 7B model on 1B tokens is a waste of parameters; it will overfit instantly. You fit a Nano-model, or you find more data."

### Q: "Why is Llama-3 8B better than Llama-2 13B?"
> **Answer**: "Because Llama-3 was trained on 15T tokens vs Llama-2's 2T. Even though it has fewer parameters, the **Data Quality and Volume** compensated. It sat in the 'optimization oven' much longer, compressing more knowledge into fewer weights."
