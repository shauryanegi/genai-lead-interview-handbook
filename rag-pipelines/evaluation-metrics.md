# Evaluation Techniques Deep Dive
## The Complete Handbook

This document covers the essential evaluation techniques for modern ML systems, ranging from LLM generation metrics to specialized RAG assessment frameworks.

1.  **LLM Generation Evaluation** (BLEU, ROUGE, BERTScore, Perplexity)
2.  **RAG Evaluation** (RAGAS Framework)
3.  **Classification/Extraction Evaluation** (Precision, Recall, F1)

---

## 1. LLM Generation Metrics

### 1.1 Perplexity (For Language Modeling)
**What it measures**: "How surprised is the model by the next token?"

**Formula**: `PPL = exp(Cross-Entropy Loss)`

**Interpretation**:
*   Lower is better.
*   PPL = 10 means the model is "as confused" as if it was randomly choosing from 10 equally likely options.

**🎯 When to use**: Comparing base models (e.g., "Is Llama-3 70B better than Mistral 7B?").
**Limitation**: Doesn't measure if output is *useful*, only if it's *likely*.

---

### 1.2 BLEU (Bilingual Evaluation Understudy)
**What it measures**: N-gram overlap between generated text and reference text.

**How it works**:
1.  Count matching unigrams, bigrams, trigrams, 4-grams.
2.  Apply a "brevity penalty" if generated text is shorter than reference.

**Formula** (Simplified):
```
BLEU = BP × exp(Σ w_n × log(precision_n))
```

**🎯 When to use**: Machine Translation.
**Limitation**: "The cat sat on the mat" vs "A feline rested upon the rug" → Low BLEU, but both are correct!

---

### 1.3 ROUGE (Recall-Oriented Understudy for Gisting Evaluation)
**What it measures**: Similar to BLEU, but focuses on **Recall** (how much of the reference is captured).

| Variant | What it measures |
|---------|------------------|
| **ROUGE-1** | Unigram overlap |
| **ROUGE-2** | Bigram overlap |
| **ROUGE-L** | Longest Common Subsequence |

**🎯 When to use**: Summarization.
**Limitation**: Same as BLEU—purely lexical, ignores semantics.

---

### 1.4 BERTScore
**What it measures**: Semantic similarity using BERT embeddings.

**How it works**:
1.  Get contextual embeddings for each token in generated and reference.
2.  Compute pairwise cosine similarity.
3.  Aggregate (greedy matching) for Precision, Recall, F1.

**🎯 When to use**: When you want to capture paraphrases.
**Example Scenario**:
> "BERTScore is ideal for evaluating the quality of generated Executive Summaries against human-written summaries. It handles paraphrasing significantly better than ROUGE, which can be overly sensitive to exact wording."

---

## 2. RAG Evaluation (The RAGAS Framework)
**This is critical for your interview.** CloudSEK will ask about RAG evaluation.

RAGAS (Retrieval Augmented Generation Assessment) provides 4 key metrics:

### 2.1 Faithfulness
**Question**: "Is the answer grounded in the retrieved context?"
**How to compute**: An LLM checks: "Can every claim in the Answer be attributed to the Context?"

**Score**: 0-1. Higher is better.

**Example Scenario**:
> "Tracking Faithfulness is critical for catching 'hallucinations'. If a model claims 'Revenue is $10M' but the retrieved source context only mentions '$8M', the Faithfulness score will flag the discrepancy."

---

### 2.2 Answer Relevancy
**Question**: "Is the answer relevant to the user's question?"
**How to compute**: Generate hypothetical questions from the answer, then compute semantic similarity to the original question.

**Score**: 0-1. Higher is better.

---

### 2.3 Context Precision
**Question**: "Are the retrieved chunks actually useful?"
**How to compute**: For each retrieved chunk, does it contribute to answering the question?

**Score**: 0-1. Higher means less noise in retrieval.

---

### 2.4 Context Recall
**Question**: "Did we retrieve all the relevant chunks?"
**How to compute**: Compare retrieved chunks to a ground-truth set of "ideal" chunks.

**Score**: 0-1. Higher means we didn't miss important info.

---

### RAGAS Summary Table

| Metric | Measures | Low Score Means |
|--------|----------|-----------------|
| Faithfulness | Hallucination Control | Model is making things up |
| Answer Relevancy | Response Quality | Answer is off-topic |
| Context Precision | Retrieval Quality | Too much irrelevant noise |
| Context Recall | Retrieval Coverage | Missed important docs |

**Example Answer**: "I implement a RAGAS pipeline for continuous monitoring. Every week, I sample queries and compute these 4 metrics. If **Faithfulness** drops below a defined threshold (e.g., 0.85), I trigger a manual review of the retrieved context and prompts."

---

## 3. Classification / Extraction Metrics

### 3.1 Precision, Recall, F1 (The Holy Trinity)

| Metric | Formula | Meaning |
|--------|---------|---------|
| **Precision** | TP / (TP + FP) | Of what I predicted, how much was correct? |
| **Recall** | TP / (TP + FN) | Of what was true, how much did I find? |
| **F1** | 2 * (P * R) / (P + R) | Harmonic mean (balanced) |

**General Principle**:
> "In high-stakes domains (like security or finance), **Recall is often more important**. A missed threat or risk (False Negative) is usually worse than a false alarm (False Positive). I focus on the **F2-Score** (which weights Recall 2x) as a primary metric while monitoring for 'Alert Fatigue'."

---

### 3.2 PR-AUC vs ROC-AUC

| Metric | Best For | Why |
|--------|----------|-----|
| **ROC-AUC** | Balanced datasets | Can be misleading if classes are imbalanced |
| **PR-AUC** | Imbalanced datasets (like Threat Detection) | Focuses on positive class performance |

**Example Scenario**:
> "When evaluating an Entity Extraction model for dense documents (e.g., extracting company names, dates, amounts), **PR-AUC** is preferred because the majority of tokens are 'Background'. ROC-AUC would be misleadingly high."

---

## 4. Human Evaluation (The Gold Standard)

**When metrics fail**: All automated metrics are proxies. Human judgment is the ultimate test.

**Framework**: The "Likert Scale" approach:
*   **Fluency**: 1-5 (Is the text grammatical and natural?)
*   **Factual Correctness**: 1-5 (Are the facts accurate?)
*   **Usefulness**: 1-5 (Does this answer help the user?)

**Example Scenario**:
> "For Human-in-the-Loop evaluation, focus on showing generated reports to domain experts. They should rate Factual Correctness on a scale (e.g., 1-5). Computing Inter-Rater Agreement (Cohen's Kappa) ensures consistency across reviewers."

---

## 5. Potential Interview Questions & Answers

### Q1: "How do you evaluate your RAG pipeline?"
> "I use the RAGAS framework with 4 metrics:
> 1. **Faithfulness**: Is the answer grounded in context? (Critical for avoiding hallucinations)
> 2. **Answer Relevancy**: Is the answer on-topic?
> 3. **Context Precision/Recall**: Is my retrieval component pulling the right chunks?
> I run weekly evaluations on a held-out test set of 50 queries."

### Q2: "Should you use BLEU or BERTScore for summarization?"
> "BERTScore. BLEU is purely lexical—it misses valid paraphrases. BERTScore uses contextual embeddings, so 'The company grew' and 'The firm expanded' would get high similarity."

### Q3: "Your model has 95% accuracy on threat detection. Is that good?"
> "Not necessarily. If threats are 0.1% of data, a model that always predicts 'No Threat' has 99.9% accuracy but is useless.
> I'd look at:
> 1. **Precision-Recall Curve**: Especially Recall at high precision thresholds.
> 2. **PR-AUC**: A single number that's robust to imbalance.
> 3. **F2-Score**: Because Recall matters more than Precision in security."

### Q4: "How do you handle evaluation when ground truth is expensive?"
> "Active Learning. I sample queries where the model is **least confident** (entropy sampling) and send only those to human annotators. This gives the most 'informative' labels for the least cost."

---
