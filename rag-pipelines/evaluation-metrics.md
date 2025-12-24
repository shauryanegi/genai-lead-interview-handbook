# Evaluation Techniques Deep Dive
## The Complete Guide (Mayank's Focus Area #1b)

Mayank said: *"Evaluation techniques around various use-cases."*

This document covers:
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
**Your Credit Memo Link**:
> "I use BERTScore to evaluate the quality of my generated Executive Summary against human-written summaries. It handles paraphrasing better than ROUGE."

---

## 2. RAG Evaluation (The RAGAS Framework)
**This is critical for your interview.** CloudSEK will ask about RAG evaluation.

RAGAS (Retrieval Augmented Generation Assessment) provides 4 key metrics:

### 2.1 Faithfulness
**Question**: "Is the answer grounded in the retrieved context?"
**How to compute**: An LLM checks: "Can every claim in the Answer be attributed to the Context?"

**Score**: 0-1. Higher is better.

**🎯 Your Credit Memo Link**:
> "I track Faithfulness to catch 'hallucinations'. If my model says 'Revenue is $10M' but the retrieved chunks only mention '$8M', Faithfulness flags it."

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

**🎯 Interview Answer**: "I implemented a RAGAS pipeline in my Credit Memo project. Every week, I sample 50 queries and compute these 4 metrics. If **Faithfulness** drops below 0.85, I trigger a prompt tuning review."

---

## 3. Classification / Extraction Metrics

### 3.1 Precision, Recall, F1 (The Holy Trinity)

| Metric | Formula | Meaning |
|--------|---------|---------|
| **Precision** | TP / (TP + FP) | Of what I predicted, how much was correct? |
| **Recall** | TP / (TP + FN) | Of what was true, how much did I find? |
| **F1** | 2 * (P * R) / (P + R) | Harmonic mean (balanced) |

**🎯 Security Domain (CloudSEK)**:
> "For Threat Detection, **Recall is more important**. A missed threat (FN) is worse than a false alarm (FP). But too many FPs cause 'Alert Fatigue'. I use **F2-Score** (weights Recall 2x) as my primary metric."

---

### 3.2 PR-AUC vs ROC-AUC

| Metric | Best For | Why |
|--------|----------|-----|
| **ROC-AUC** | Balanced datasets | Can be misleading if classes are imbalanced |
| **PR-AUC** | Imbalanced datasets (like Threat Detection) | Focuses on positive class performance |

**🎯 Your Credit Memo Link**:
> "When I evaluate my Entity Extraction model (finding company names, dates, amounts), I use **PR-AUC** because 90% of tokens are 'O' (no entity). ROC-AUC would be misleadingly high."

---

## 4. Human Evaluation (The Gold Standard)

**When metrics fail**: All automated metrics are proxies. Human judgment is the ultimate test.

**Framework**: The "Likert Scale" approach:
*   **Fluency**: 1-5 (Is the text grammatical and natural?)
*   **Factual Correctness**: 1-5 (Are the facts accurate?)
*   **Usefulness**: 1-5 (Does this answer help the user?)

**🎯 Your Credit Memo Link**:
> "For my Human-in-the-Loop evaluation, I showed generated memos to 3 credit analysts. They rated Factual Correctness using a 5-point scale. We compute Inter-Rater Agreement (Cohen's Kappa) to ensure consistency."

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
