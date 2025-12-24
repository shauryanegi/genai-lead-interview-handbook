# Vector Indexing Algorithms: HNSW vs. IVF+PQ

At scale (10M+ vectors), you cannot use brute-force cosine similarity (KNN). You need **ANN** (Approximate Nearest Neighbor).

The choice of index determines your **Latency**, **Recall** (Accuracy), and **Memory Cost**.

---

## 1. The Trade-Off Triangle

You can only pick two:
1.  **Speed** (Low Latency)
2.  **Accuracy** (High Recall)
3.  **Efficiency** (Low RAM)

---

## 2. HNSW (Hierarchical Navigable Small World)
*The "Speed King" (Graph-Based)*

**How it works**:
*   Think of it like a **Skip List** for graphs.
*   **Layer 0 (Bottom)**: Contains all data points. Highly connected.
*   **Layer N (Top)**: Contains very few points. Acts as an "Express Highway".
*   **Search**: You start at the top layer, take big jumps to get close to the target, then drill down to lower layers for fine-grained steps.

| Feature | Details |
|---------|---------|
| **Pros** | Extremely fast query speeds. High Recall (98%+). |
| **Cons** | **Memory Hungry**. Stores millions of edges/pointers. Construction is slow. |
| **Use Case** | Real-time RAG (< 50ms requirement) where RAM is plentiful. |

---

## 3. IVF (Inverted File Index)
*The "Memory Saver" (Cluster-Based)*

**How it works**:
*   **Train**: Cluster the vector space into $K$ centroids (Voronoi cells).
*   **Index**: Assign every vector to its nearest centroid.
*   **Search**:
    1.  Find the nearest centroid to your query.
    2.  Search *only* the vectors inside that centroid (and maybe its neighbors).
*   **nprobe**: A hyperparameter (e.g., 10). "Check the top 10 closest centroids." Higher = Better Recall, Slower.

| Feature | Details |
|---------|---------|
| **Pros** | Lower memory usage than HNSW. Fast training. |
| **Cons** | Slower query speed than HNSW. Recall drops if `nprobe` is too low. |
| **Use Case** | Mid-size datasets (1M - 10M) where RAM is constrained. |

---

## 4. Quantization (Compression)

Typically used *with* IVF to compress the vectors themselves.

### 4.1 PQ (Product Quantization)
**Concept**: Break the 768-dim vector into 8 sub-vectors (96 dims each).
*   Run K-Means on each sub-section to find 256 "codebooks".
*   Replace the float values with the *ID* of the codebook.
*   **Compression**: Reduces a 768-float vector (3KB) to just 8 bytes. **300x compression**.
*   **Trade-off**: Lossy. Distance calculations are approximations.

### 4.2 SQ (Scalar Quantization)
**Concept**: Convert FP32 (4 bytes) to INT8 (1 byte).
*   **Compression**: 4x reduction.
*   **Trade-off**: Less accuracy loss than PQ, but less compression.

---

## 5. The "God Mode": HNSW + Quantization (e.g., in Qdrant/Milvus)

Modern vector DBs allow hybrid approaches.
*   **HNSW on disk**: Keep the graph structure on SSD.
*   **Compressed Vectors in RAM**: Keep PQ/INT8 vectors in memory for distance calc.
*   **Rescoring**:
    1.  Retrieve top 100 using Compressed vectors (Fast).
    2.  Fetch full FP32 vectors for those 100 from disk.
    3.  Re-calculate exact distance.

---

## 6. Interview Questions

### Q: "How do you choose between HNSW and IVF-PQ for 100M vectors?"
> **Answer**: "It depends on the Memory Budget vs. Latency SLA.
> 1.  **HNSW**: If I have massive RAM (or budget for it), HNSW is strictly better for low-latency queries.
> 2.  **IVF-PQ**: If I need to fit 100M vectors on a single node, I must use quantization (PQ). HNSW would require terabytes of RAM for the graph edges.
>
> *Rule of Thumb*: <1M vectors = HNSW. >100M vectors = IVF-PQ (or DiskANN)."

### Q: "What is the 'nprobe' parameter in IVF?"
> **Answer**: "`nprobe` controls how many clusters (Voronoi cells) we inspect.
> *   `nprobe=1`: fast but might miss the answer (low recall).
> *   `nprobe=100`: slow but accurate (high recall).
> It's a runtime knob to trade off latency for accuracy without re-indexing."
