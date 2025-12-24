# Vector Indexing Algorithms: The Expert's Guide

A deep dive into the engine room of RAG. "Which index?" is a million-dollar infrastructure decision.

---

## 1. The Landscape of Indexing

| Index Type | Complexity | Recall | RAM Usage | Best For |
|------------|------------|--------|-----------|----------|
| **Flat (Brute Force)** | $O(N)$ | 100% | High (1x) | < 100k vectors, or absolute precision reqs. |
| **IVF (Inverted File)** | $O(N/K)$ | 90-95% | Medium | Mid-scale (1M-50M), RAM constrained. |
| **HNSW (Graph)** | $O(\log N)$ | 98%+ | Huge (1.5x) | Low latency (<50ms), High RAM budget. |
| **DiskANN / Vamana** | $O(\log N)$ | 95%+ | Low (0.1x) | Massive scale (100M+), SSD-based. |

---

## 2. HNSW (Hierarchical Navigable Small World)

The industry standard for low-latency, in-memory search.

### Allocating Memory (The "1.5x" Rule)
HNSW constructs a multi-layer graph. It doesn't just store vectors; it stores **Edges**.
*   **Formula**: $\text{RAM} \approx \text{Raw Vector Size} + \text{Graph Overhead}$
*   **Graph Overhead**: $2 \times M \times N \times 4$ bytes (roughly).
    *   Where $M$ = max links per node (e.g., 16 or 32).
    *   Each link is an integer ID (4 bytes).
*   **Implication**: For 100M vectors (768-dim), raw size is 300GB. HNSW overhead adds ~50-100GB.

### Tuning Knobs (Iterview Gold)
1.  **`M` (Neighbors)**:
    *   Higher $M$ (e.g., 64) = Better Recall, Robust to Deletions.
    *   Cost: Slower insertion, more RAM.
2.  **`ef_construction`**: Deep search during build time.
    *   Set high (e.g., 200) for better graph quality. No effect on query speed.
3.  **`ef_search`**: Deep search during runtime.
    *   The trade-off knob. `ef=100` -> High Recall, Slow. `ef=10` -> Fast, Low Recall.

---

## 3. Dissecting IVF (Inverted File Index)

Best for when HNSW is too expensive.

### The Algorithm
1.  **Train**: Use K-means to find $C$ centroids (e.g., 4096).
2.  **Assign**: Vectors fall into their nearest "Voronoi Cell".
3.  **Search**: Query matches Centroid A. Simply scan all vectors in Cell A.

### The "Cardinality Explosion" Problem (Filtering)
*   **Scenario**: `Select * from docs where user_id = '123'` (Matches 5 docs).
*   **Post-filtering**: Retrieve 100 vectors via ANN -> Filter by user_id -> Result: 0 docs (because the top 100 didn't belong to user 123).
*   **Pre-filtering**: Filter by user_id -> Run KNN on the 5 remaining docs.
*   **Solution**: **Acapulco** or **Block-Bloom** schemes. Modern DBs (Qdrant, Milvus) handle this with optimized HNSW traversal (skipping deleted nodes).

---

## 4. DiskANN (The "Vamana" Graph)

**The Problem**: RAM is expensive ($5/GB). SSD is cheap ($0.1/GB).
**The Solution**: Keep the graph structure on Disk.

### How Vamana Works
*   Unlike HNSW (Layered), Vamana is a flat, random graph optimized for **long edges**.
*   It ensures you can hop from any node to any node in very few SSD reads.
*   **Beam Search**: Instead of greedy search, it keeps a "beam" of promising candidates, fetching their neighbors from SSD in parallel (Async IO).
*   **The Gain**: Run 1 Billion vectors on a single 64GB RAM machine (storing mostly on 2TB NVMe).

---

## 5. Compression & Quantization

Scaling to Billions requires shrinking the vector.

### 5.1 Product Quantization (PQ)
*   **Technique**: Slice the vector into 8 sub-vectors (chunks). Cluster each chunk into 256 centroids (1 byte).
*   **Compression Ratio**: 768 dims (3KB) $\rightarrow$ 8 bytes. **~384x compression**.
*   **Cost**: Recall drops drastically. Requires "Re-ranking" with full resolution to fix.

### 5.2 Scalar Quantization (SQ / INT8)
*   **Technique**: Map min/max range of floats to -127..128.
*   **Compression**: 4x (FP32 -> INT8).
*   **Quality**: Very high preservation (Recall > 99%). Recommended default.

### 5.3 Binary Quantization (BQ)
*   **Technique**: `x > 0 ? 1 : 0`. The vector becomes a bit-array.
*   **Distance**: Hamming Distance (XOR). Extremely fast hardware instruction (`_mm_popcnt`).
*   **Use Case**: OpenAI `text-embedding-3`, Cohere v3. High dim (1536+) vectors specifically trained for this.
*   **Speed**: 30x faster than FP32 search.

### 5.4 Matryoshka Representation Learning (MRL)
*   **Insight**: Why send 1536 dims if 64 dims works?
*   **Training**: Train the model so "coarse" information is in the first $N$ dimensions, and "fine" detail is in the later ones.
*   **Adaptive Retrieval**:
    1.  Search using only first 64 dims (Ultra fast).
    2.  Rescore Top-N using full 1536 dims.
*   **Supported Models**: `text-embedding-3`, Nomic v1.5.

---

## 6. Sizing Guide: "How much RAM do I need?"

**Scenario**: 10 Million Documents, 768-dim vectors.

*   **Raw Vectors**: $10M \times 768 \times 4B = 30 \text{ GB}$.
*   **Option A: HNSW (FP32)**
    *   RAM: $30 \text{ GB} + 15 \text{ GB (Graph)} \approx 45 \text{ GB}$.
    *   Latency: < 5ms.
    *   Cost: $\$ \$ \$$.
*   **Option B: Hybrid (HNSW + PQ)**
    *   RAM: Graph (15 GB) + Compressed Vectors ($10M \times 16B = 160MB$) $\approx 16 \text{ GB}$.
    *   Latency: < 10ms.
    *   Cost: $\$ \$$.
*   **Option C: DiskANN**
    *   RAM: ~2 GB (Cache). SSD: 50 GB.
    *   Latency: 20-50ms (SSD I/O bound).
    *   Cost: $\$$.

---

## 7. Interview "Gotchas"

### Q: "Does adding more dimensions always help?"
> **Answer**: "No. It exacerbates the 'Curse of Dimensionality'. In very high dimensions, all points become roughly equidistant. Plus, computational cost scales linearly. MRL (Matryoshka) is solving this by allowing flexible truncated dimensions."

### Q: "How do you handle Metadata Filtering with ANN?"
> **Answer**: "Naive post-filtering kills recall (you find 10 neighbors, filter them all out, result 0).
> The correct approach is **Filtered-HNSW**: The traversal algorithm checks the filter condition *during* the graph walk, skipping 'blocked' nodes and exploring only valid neighbors."
