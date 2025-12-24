# Advanced Reasoning Patterns

Moving beyond "Prompt Engineering" to "Flow Engineering".

## 1. The Reasoning Hierarchy

| Pattern | Logic | Best For |
|---------|-------|----------|
| **Zero-Shot** | Just answer. | Simple Q&A |
| **CoT** (Chain of Thought) | "Let's think step by step." | Math, Logic |
| **ReAct** (Reason+Act) | Loop: Thought -> Action -> Obs. | Tool use |
| **Plan-and-Solve** | Plan first, then execute. | Complex Constraints |
| **Reflexion** | Try -> Fail -> Analyze -> Retry. | Coding, Agents |
| **ToT** (Tree of Thoughts) | Explore multiple paths, backtrack. | Creative Writing, Strategy |

---

## 2. ReAct vs. Plan-and-Solve

**ReAct (The Standard):**
*   **Pros**: Dynamic. Can change course if a tool reveals new info.
*   **Cons**: Myopic. Can get stuck in loops ("Search google", "Search google again").

**Plan-and-Solve (The Strategist):**
*   **Prompt**: "First, create a step-by-step plan. Then execute each step."
*   **Mechanism**:
    1.  Generate Plan: `[1. Search X, 2. Filter Y, 3. Calculate Z]`
    2.  Execute Step 1.
    3.  Execute Step 2...
*   **Pros**: Better global coherence.
*   **Cons**: Rigid. If Step 1 fails, the plan might fail.

**Hybrid Approach (Best of Both)**: 
*   "Replanning Agents". Create a plan, execute one step, then *re-evaluate* the plan.

---

## 3. Reflexion (Self-Correction)

Agents that don't learn from errors are useless.

**Workflow**:
1.  **Actor**: Generates code/text.
2.  **Evaluator**: Checks output (Unit Tests or LLM Review).
3.  **Self-Reflection**: "Why did I fail? I missed the edge case." -> Adds to Short-Term Memory.
4.  **Retry**: Actor tries again with the Reflection in context.

**Code Pattern**:
```python
while attempts < max_retries:
    result = agent.act(task, context)
    if evaluation.pass(result):
        return result
    
    critique = reflector.reflect(task, result, evaluation.errors)
    context += f"Previous Attempt Failed: {critique}"
```

---

## 4. Tree of Thoughts (ToT)

Classic Search algorithms (BFS/DFS) applied to LLM reasoning.

**Mechanism**:
1.  **Node**: A state of text.
2.  **Expand**: Generate 3 possible next steps.
3.  **Evaluate**: Score each step (Values).
4.  **Select**: Pick the best implementation (or backtrack).

**Use Case**: Writing a novel chapter (Branching storylines) or rigorous logical proofs.

---

## 5. Interview Questions

### Q: "When does Chain-of-Thought FAIL?"
> **Answer**: "CoT fails when the logic requires **external knowledge** not present in the weights (hallucination risk) or when the reasoning chain is **too long** and errors propagate (error cascading). In those cases, we need RAG (for knowledge) or decompose-and-verify (for long chains)."

### Q: "How do you prevent Agents from getting stuck in loops?"
> **Answer**:
> 1.  **Hard Limit**: Max 10 steps.
> 2.  **Loop Detection**: Check if the last 3 tool calls are identical.
> 3.  **Temperature Kick**: If stuck, increase temperature to 0.7 for one turn to force diversity.
> 4.  **Supervisor LLM**: A separate lightweight model monitoring the trace. If it detects a loop, it injects a 'Stop' or 'Hint' interruption."
