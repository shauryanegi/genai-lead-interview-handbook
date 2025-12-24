# Fine-Tuning Strategies: From PEFT to Full Training

As an ML Lead, you must choose the right adaptation strategy. "Just fine-tune it" is not a strategy.

---

## 1. The Decision Matrix: When to Fine-Tune?

| Scenario | Strategy | Why? |
|----------|----------|------|
| **New Knowledge** (e.g., "Company Policy 2024") | **RAG** | Knowledge changes faster than training cycles. |
| **New Format/Style** (e.g., "JSON Output", "SQL Queries") | **Fine-Tuning** | Models learn *style/syntax* efficiently via weights. |
| **New Domain Language** (e.g., "Medical specifics") | **Continued Pre-Training** | Model needs to learn new token correlations. |

---

## 2. PEFT (Parameter-Efficient Fine-Tuning)

Full fine-tuning updates 100% of weights. It's expensive and prone to **Catastrophic Forgetting**.
PEFT updates < 1% of weights.

### 2.1 LoRA (Low-Rank Adaptation)
**Concept**: Weight updates $\Delta W$ have a "low intrinsic rank." We don't need to update the huge matrix $W$, just two small matrices $A$ and $B$.

$$W_{new} = W_{frozen} + (A \times B)$$

**Key Hyperparameters (Interview Gold)**:
1.  **Rank ($r$)**: The dimension of the low-rank matrices.
    *   *Rule of Thumb*: 8-16 for general tasks. 64-128 for complex reasoning.
2.  **Alpha ($\alpha$)**: The scaling factor.
    *   *Rule of Thumb*: Set $\alpha = 2 \times r$. This helps stability.
3.  **Target Modules**: Which layers to adapt?
    *   *Basic*: `q_proj`, `v_proj` (Attention queries/values).
    *   *Advanced*: All linear layers (`gate_proj`, `up_proj`, `down_proj`). This fits more knowledge but uses more VRAM.

### 2.2 QLoRA (Quantized LoRA)
**The Game Changer**: Fine-tune a 70B model on a single GPU.
*   **Mechanism**:
    1.  Quantize base model to **4-bit** (NF4 format).
    2.  Keep LoRA adapters in **16-bit** (Brain Float).
    3.  Backpropagate through the frozen 4-bit weights into the 16-bit adapters.
*   **Trade-off**: Slightly slower training (de-quantization overhead) but massive memory savings.

---

## 3. Preference Alignment (RLHF vs DPO)

After Supervised Fine-Tuning (SFT), you align the model to human preferences.

### 3.1 RLHF (Reinforcement Learning from Human Feedback)
*   **The Old Standard** (used by GPT-4).
*   **Steps**: SFT -> Reward Model (RM) Training -> PPO (Proximal Policy Optimization).
*   **Pros**: Proven at massive scale.
*   **Cons**: Unstable, complex, requires training 4 models simultaneously (Policy, Reference, Reward, Value).

### 3.2 DPO (Direct Preference Optimization)
*   **The Efficient Standard** (used by Llama-3, Mistral).
*   **Concept**: We don't need a separate Reward Model. The Policy Model *is* the Reward Model.
*   **Loss Function**: Directly optimizes the likelihood of "Winning" response over "Losing" response.
*   **Pros**: Stable, requires only 2 models (Policy, Reference). No PPO complexity.

### 3.3 ORPO (Odds Ratio Preference Optimization)
*   **Latest Trend**: Combines SFT + DPO into a single stage.
*   **Why**: Doesn't require a separate SFT checkpoint. Faster convergence.

---

## 4. Coding a LoRA Configuration (Using `peft`)

```python
from peft import LoraConfig, TaskType

config = LoraConfig(
    r=16, # Rank
    lora_alpha=32, # Scaling
    target_modules=["q_proj", "v_proj", "k_proj", "o_proj"], # All attention blocks
    lora_dropout=0.05,
    bias="none",
    task_type="CAUSAL_LM"
)

# Application
model = get_peft_model(base_model, config)
print(f"Trainable params: {model.print_trainable_parameters()}")
# Output: "trainable params: 0.05% || all params: 7B"
```

---

## 5. Interview Questions & Answers

### Q: "Why choose DPO over RLHF?"
> **Answer**: "PPO (RLHF) is notoriously unstable and memory-intensive because you have to load four models. DPO is mathematically equivalent at the optimal solution but simply requires a classification loss on pairs of (chosen, rejected) samples. It consumes half the memory and converges faster for most open-source scale tasks."

### Q: "Does LoRA cause latency during inference?"
> **Answer**: "No. Since $W_{new} = W_{frozen} + AB$, we can **merge** the adapters back into the base model weights before serving.
> `model.merge_and_unload()`.
> The final model is identical in architecture to the base model—zero inference penalty."
