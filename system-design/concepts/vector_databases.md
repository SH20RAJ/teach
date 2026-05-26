# System Design: Vector Databases (Semantic Search & RAG)
> **Format:** Concept-First Whiteboard Explanation
> **Topic:** Storing and Querying High-Dimensional Vectors for AI Systems

---

## 1. First Understand the Problem

Imagine you run an online bookstore.
A user types in the search bar:
`"Looking for stories about small magical creatures that live in the woods and build houses in trees."`

### Method 1: Keyword-Based Search (SQL `LIKE` or Elasticsearch BM25)
Your database tries to match exact words: `magical`, `creatures`, `woods`, `trees`.
- If a book description says: *"A fantasy tale of tiny elves residing in oak branches,"* the keyword search **fails** to find it!
- Why? Because none of the exact words (`woods`, `trees`, `creatures`) match the synonyms (`branches`, `residing`, `elves`).

Keyword search only understands character strings. It does **not** understand the **meaning (semantic context)** of the words.

### The AI Solution
We use a Machine Learning model (Embedding Model) to convert text into an array of numbers called an **Embedding Vector**.
For example:
- `Embedding("magical creature") = [0.12, -0.85, 0.43, ... 512 dimensions]`
- `Embedding("elf residing in branches") = [0.11, -0.84, 0.45, ... 512 dimensions]`

These numbers represent coordinates in a massive **high-dimensional space**.
Because the concepts are similar, their coordinates are very close to each other in this space!

**The Crisis:**
If you have **10 Million book descriptions**, you have 10 Million vectors of 1,536 dimensions.
Standard database indexes (like B-Trees) can only index 1-dimensional values (numbers, strings). They cannot index coordinates in a 1,536-dimensional space!

If a user searches, you cannot run a loop to calculate the distance between the search vector and all 10 Million vectors. It would take seconds, causing your search to lag.

We need a database optimized to store vectors and find the "nearest neighbors" in microseconds.
That database is a **Vector Database** (like Pinecone, Milvus, Chroma).

---

## 2. Imagine the Map Analogy

Imagine a massive sheet of paper (2D Space).
You write words on the sheet based on their meaning:
- **King** is written at coordinate `(10, 10)`.
- **Queen** is written at coordinate `(10, 9)`.
- **Apple** is written at coordinate `(1, 1)`.
- **Orange** is written at coordinate `(1, 2)`.

```text
  Y-Axis
   ▲
10 │    [Queen]   [King]
   │
   │
 2 │    [Orange]
 1 │    [Apple]
 0 └────────────────────────► X-Axis
        1         10
```

### The Search:
If you ask: *"Show me things similar to 'Pear'."*
1. You calculate Pear's coordinate: `(1.2, 1.2)`.
2. You look at the paper. What is closest to `(1.2, 1.2)`?
3. It is **Apple** and **Orange**!
4. You return those results. You never look at King and Queen.

In a Vector Database, we do the exact same thing, but instead of a 2D sheet of paper, our map has **1,536 dimensions**!

---

## 3. How Vector Indexing Works: Step-by-Step

How do we query 10 Million vectors in under 10ms? We use **Approximate Nearest Neighbor (ANN)** indexing algorithms.

Let's study the primary algorithm: **HNSW (Hierarchical Navigable Small World)** on the whiteboard.

HNSW builds a multi-layer graph, similar to the Skip List data structure.

```text
Layer 2 (Express Nodes)     [Node A] ────────────────────────► [Node G]
                                │                                 │
Layer 1 (Local Highway)     [Node A] ─────► [Node D] ────────► [Node G]
                                │               │                 │
Layer 0 (All Vectors)       [Node A] ─► [B] ─► [C] ─► [D] ─► [E] ─► [F] ─► [G]
```

### The Search Process:
1. **Start at top layer (Layer 2):** You enter the graph. The nodes are sparse (express highways). You jump from Node A to Node G because Node G is closest to your query vector.
2. **Drop down to Layer 1:** Around Node G, you drop down a layer to see local neighbors. You find Node F.
3. **Drop down to Layer 0:** You search the dense network of local connections around Node F to find the absolute closest vectors (e.g. Node E and F).
This skip-graph search reduces the search time from linear $O(N)$ scanning to logarithmic $O(\log N)$ navigation!

---

## 4. Vector Distance Metrics

How do we mathematically define if two coordinates are "close"?

1. **Cosine Similarity:** Measures the angle between two vectors. It ignores the length of the vectors and only checks if they point in the same direction. (Best for text search).
2. **Euclidean Distance (L2):** Measures the straight-line distance between two points. (Best for physical measurements).
3. **Dot Product:** Multiplies the coordinates. (Fastest, if vectors are normalized to length 1).

---

## 5. Common Confusion

### Confusion: Can you store raw user data inside a vector index?
*   **Correction:** A vector index itself only holds the raw floats (e.g. `[0.1, -0.4, ...]`) and a record ID. It does not hold the raw paragraph text, book title, or user details. To return actual search results, the vector database must map the matched record ID back to a traditional document store (metadata storage) to fetch the actual text string to display to the user.

---

## 6. Interview-Safe Definitions

1. **Vector Embedding:** A representation of real-world data (text, images, audio) as high-dimensional coordinates that capture semantic meaning.
2. **Semantic Search:** A search technique that understands the intent and contextual meaning of a query, rather than relying on literal keyword matching.
3. **HNSW (Hierarchical Navigable Small World):** A graph-based index structure that enables fast approximate nearest neighbor searches in high-dimensional vector spaces.

---

## 7. Active Recall Questions

1. **Why do traditional indexing structures like B-Trees fail when applied to high-dimensional vector searches?**
2. **Explain the difference between Cosine Similarity and Euclidean Distance. In what context is vector length important?**
3. **What is Retrieval-Augmented Generation (RAG) and how does a vector database integrate with a Large Language Model (LLM) in this pattern?**
