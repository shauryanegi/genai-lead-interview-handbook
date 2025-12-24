# Vector Indexing Implementation: Code Dictionary

A cheat sheet for implementing HNSW, IVF-PQ, and DiskANN using the most popular libraries.

---

## 1. Faiss (Facebook AI Similarity Search)

The standard for in-memory indexing.

### Scenario A: HNSW (Maximum Speed)

```python
import faiss

# D = Dimension (e.g., 768)
# M = Edges per node (32 is standard)
index = faiss.IndexHNSWFlat(d, 32)

# Tuning Build Time
index.hnsw.efConstruction = 64  # Higher = Better index, slower build

# Training (Not needed for HNSWFlat, but good practice)
index.add(vectors_np)

# Tuning Search Time
index.hnsw.efSearch = 64 # Trade-off: Latency vs Recall
D, I = index.search(query_np, k=5)
```

### Scenario B: IVF-PQ (Maximum Compression)

```python
import faiss

# Centroids (nlist): 4 * sqrt(N) is a good rule of thumb
nlist = 1000 
# M = Sub-vectors (e.g., 8 chunks of 96 dims each)
m = 8 
# Bits per chunk (8 bits = 256 centroids per chunk)
nbits = 8

quantizer = faiss.IndexFlatL2(d)
index = faiss.IndexIVFPQ(quantizer, d, nlist, m, nbits)

# Train the quantizer (Required!)
index.train(vectors_np)
index.add(vectors_np)

# Search
index.nprobe = 10 # Check 10 nearest centroids
D, I = index.search(query_np, k=5)
```

---

## 2. Qdrant (Production Vector DB)

A modern, Rust-based engine.

### Scenario: HNSW Configuration (Python Client)

```python
from qdrant_client.models import VectorParams, Distance, OptimizersConfigDiff

client.create_collection(
    collection_name="my_docs",
    vectors_config=VectorParams(size=768, distance=Distance.COSINE),
    
    # HNSW Tuning
    optimizers_config=OptimizersConfigDiff(
        memmap_threshold=20000, # Use mmap (disk) after 20k vectors
    ),
    hnsw_config={
        "m": 16,
        "ef_construct": 100,
        "full_scan_threshold": 10000 # Use brute force for small filters
    }
)
```

### Scenario: Scalar Quantization (INT8)

```python
from qdrant_client.models import ScalarQuantization, ScalarQuantizationConfig, ScalarType

client.create_collection(
    collection_name="quantized_docs",
    vectors_config=VectorParams(size=768, distance=Distance.COSINE),
    quantization_config=ScalarQuantization(
        scalar=ScalarQuantizationConfig(
            type=ScalarType.INT8,
            quantile=0.99, # Handle outliers
            always_ram=True # Keep quantized vectors in RAM
        )
    )
)
```

---

## 3. LanceDB (Embedded & Serverless)

Optimized for **Disk** storage (Lance format).

### Scenario: Creating an Index (IVF-PQ)

```python
import lancedb

db = lancedb.connect("./data")
tbl = db.create_table("docs", data=data)

# Create Index
tbl.create_index(
    metric="cosine",
    vector_column_name="vector",
    num_partitions=256, # Number of IVF partitions
    num_sub_vectors=96, # PQ chunks
    accelerator="cuda" # Use GPU for indexing
)
```

---

## 4. Interview "Whiteboard Coding" Question

**Q: "Write a naive KNN search function in Numpy."**
(Usually asked to test if you understand Dot Product).

```python
import numpy as np

def brute_force_knn(query, db_vectors, k=5):
    # query: (D,)
    # db_vectors: (N, D)
    
    # 1. Compute Cosine Similarity (Dot product if normalized)
    # scores shape: (N,)
    scores = np.dot(db_vectors, query)
    
    # 2. Get Top K indices
    # argsort is ascending, so take last k and reverse
    top_k_indices = np.argsort(scores)[-k:][::-1]
    
    return top_k_indices, scores[top_k_indices]
```

**Follow-up: "How do you make this faster?"**
1.  **Faiss**: Move to C++.
2.  **GPU**: `torch.matmul`.
3.  **Indexing**: This is $O(N)$. HNSW is $O(\log N)$.
