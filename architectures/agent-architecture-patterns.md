# How to Engineer Specialized Agents
## "The Recipe" for Multi-Agent Systems

Designing specialized agents (like a **Financial Analyst** vs. **Risk Analyst**) involves strict scoping of **Context**, **Instructions**, and **Tools**.

Here is the blueprint for engineering these agents in multi-agent workflows.

---

## 🏗️ The 3 Ingredients

To make an agent feel "real" and distinct, you need to configure three layers:

### 1. The Persona (System Prompt) 🧠
This is the most obvious part. You give the LLM a specific role and set of constraints.

> **Example Implementation**:
> ```python
> AgentRole.RISK_ANALYST: """You are an expert Risk Analyst.
> Identify: financial, operational, market risks. Provide SWOT analysis.
> Be balanced - highlight both risks AND mitigants."""
> ```

**Why it works**: It forces the model to ignore irrelevant details (like "marketing slogan") and focus on "risk factors".

### 2. The Tools (Capabilities) 🛠️
**This is the differentiator.** A "Risk Analyst" becomes truly specialized when it has *different tools* than the others.

*   **Financial Analyst**: Gets access to `calculate_ratio` and `retrieve_financial_tables`.
*   **Risk Analyst**: Gets access to `web_search` (to find news about lawsuits/scandals) but *not* the raw implementation details.
*   **Compliance Analyst**: Gets access to `retrieve_regulatory_docs`.

**How to implement (Advanced)**:
Instead of passing *all* tools to every agent, pass a subset:
```python
risk_agent = ReActAgent(
    llm=llm,
    tools={"web_search": web_search, "retrieve": retrieve_news} # Only news tools
)

finance_agent = ReActAgent(
    llm=llm,
    tools={"calculate": calculator, "retrieve": retrieve_10k} # Only math tools
)
```

### 3. The Orchestration (The Workflow) 🎼
How do they talk?
*   **Parallel (Map-Reduce)**: Everyone works at once, then the boss summarizes. (This is what your `MultiAgentOrchestrator` does).
*   **Sequential (Chain)**: Finance output -> Risk Input -> Compliance Input.

---

## 🔍 Deep Dive: Implementation Patterns

Common multi-agent systems use the **Parallel Specialist** pattern to manage complexity.

### How it works today:
1.  **Orchestrator** defines tasks:
    *   To Finance Agent: "Analyze financial statements"
    *   To Risk Agent: "Identify risks"
2.  **Parallel Execution**: It uses `asyncio.gather` to run all 3 prompts simultaneously.
3.  **Synthesis**: The Orchestrator takes the 3 text blobs and writes the final Executive Summary.

### 🚀 How to Articulate the Value
Specialization is a powerful pattern to highlight.

**Lead Perspective:**
> "By building a **specialized agent swarm**, we mimic an expert committee.
> 1.  A **Financial Agent** focuses strictly on quantitative accuracy and data extraction.
> 2.  A **Risk Agent** focuses on qualitative red flags and SWOT analysis.
> 3.  A **Lead Agent** synthesizes these views into a final recommendation.
> This avoid the 'generalist' problem where a single prompt tries to handle too many conflicting goals, resulting in higher quality and more focused outputs."

---

## 💻 Code Snippet: Improving Specialization

To make them even better, you can inject **different context** into each agent.

```python
class SpecializedAgent:
    def __init__(self, role, llm, tools):
        self.role = role
        self.tools = tools # <--- Distinct tools per agent!
    
    async def process(self, document_chunk):
        # The Risk Analyst mostly cares about the "Legal Proceedings" section
        if self.role == AgentRole.RISK_ANALYST:
             relevant_section = extract_section(document_chunk, "Risk Factors")
             return self.llm.analyze(relevant_section)
             
        # The Financial Analyst cares about the "Tables"
        elif self.role == AgentRole.FINANCIAL_ANALYST:
             tables = extract_tables(document_chunk)
             return self.llm.analyze(tables)
```

## 🎯 Summary Checklist for "Making Agents"

1.  [x] **Define Enum**: `AgentRole.RISK`, `AgentRole.FINANCE`
2.  [x] **System Prompt**: Write a specific "You are X, your goal is Y" prompt for each.
3.  [ ] **Tool Scoping**: (Next Level) Give the Risk agent web search, give Finance agent calculator.
4.  [x] **Orchestrator**: Write the code that combines their outputs (`" ".join(outputs)`).

**Final Tip**: "Specialization" is really just **Prompt Engineering + Tool Restriction**. You are 'making' an agent by narrowing its focus so it performs better on a smaller task.
