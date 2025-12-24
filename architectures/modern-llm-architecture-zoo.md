# Modern LLM Architecture Zoo

A technical comparison of the most prominent models in the GenAI ecosystem today.

---

## 1. Llama-3 (Meta)
*The Open Standard*

*   **Architecture**: Dense Transformer (not MoE).
*   **Attention**: **GQA** (Grouped Query Attention) with 8 KV heads.
*   **Tokenizer**: 128K Tiktoken-based (very efficient for code/non-English).
*   **RoPE**: Uses high base frequency ($500k$) for expanded context handling.
*   **Training**: Massive scale. 15 Trillion tokens for the 8B and 70B models.
*   **Key Insight**: Llama-3 proves that "Over-training" small dense models (8B) leads to performance that beats previously much larger models (Llama-2 70B).

---

## 2. DeepSeek-V3 / R1 (DeepSeek)
*The Efficiency Frontier*

*   **Architecture**: Multi-headed Latent Attention (**MLA**) + **DeepSeekMoE**.
*   **MLA**: Compresses KV cache into a latent space, allowing for massive context (128K) with minimal memory growth.
*   **MoE**: Fine-grained experts (64+ experts) with **Shared Experts** (always active for common knowledge).
*   **Training**: Uses Multi-Token Prediction (MTP) to speed up convergence.
*   **Key Insight**: DeepSeek-V3 is the most technically efficient model per-FLOP, achieving GPT-4o performance with significantly lower training and inference costs.

---

## 3. Claude 3.5 Sonnet (Anthropic)
*The "Reasoning" Specialist*

*   **Architecture**: Likely a large-scale MoE (details are proprietary but suspected).
*   **Key Feature**: Exceptional at **Long-Context Artifacts** and **Visual Reasoning**.
*   **Alignment**: Heavy focus on Constitutional AI and strictly formatted outputs.
*   **Key Insight**: Sonnet 3.5 is the current "State-of-the-Art" for coding and agentic workflows due to its precision and instruction-following.

---

## 4. GPT-4o (OpenAI)
*The Omnimodal Model*

*   **Architecture**: Native Multimodal.
*   **Innovation**: Instead of separate encoders for Audio/Vision/Text joined by a bridge, it is trained on all modalities in a **single transformer stream**.
*   **Interaction**: Very low latency (232ms for audio-to-audio) due to the unified architecture.
*   **Key Insight**: GPT-4o marks the transition from "Text-First" LLMs to "Native-Multimodal" LLMs.

---

## 5. Mistral / Mixtral (Mistral AI)
*The MoE Pioneers*

*   **Mixtral 8x7B**: The first highly successful open MoE. Uses 2 active experts per token.
*   **SWA**: Mistral models popularized **Sliding Window Attention** to handle 32k+ context on smaller GPUs.
*   **Key Insight**: Mistral focuses on "Density per Byte", making them the favorites for edge deployment.

---

## 💡 Summary Comparison Table

| Model | Type | Attention | Context | Best For |
|-------|------|-----------|---------|----------|
| **Llama-3** | Dense | GQA | 8k/128k | General Purpose / Local |
| **DeepSeek-V3** | MoE | MLA | 128k | Extreme Throughput / Value |
| **Claude 3.5** | Suspected MoE | Proprietary | 200k | Coding / Reasoning |
| **GPT-4o** | Omnimodal | Proprietary | 128k | Voice / Interactive |

---

## 7. Interview Question

### Q: "Why did DeepSeek choose MLA over GQA?"
> **Answer**: "DeepSeek's goal was maximizing **Inference Throughput**. 
> 
> GQA (Llama-3) reduces the KV cache size by a fixed factor (ratio of Q-heads to KV-heads). However, for very long contexts (128k+), even GQA's cache becomes a bottleneck for batch size. 
> 
> **MLA** (Multi-head Latent Attention) goes further. It uses low-rank latent compression. It compresses the Keys and Values into a shared latent vector ($d=512$) which is then up-projected at inference. This results in a KV cache that is **4-6x smaller than GQA**, allowing DeepSeek to run massive batches on the same hardware, which is why their API is so much cheaper than competitors."
