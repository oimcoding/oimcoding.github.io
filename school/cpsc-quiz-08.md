# Quiz 8: RAG and Vector Databases — Tutor Answers (Q1–Q25)

---

# Quiz 8, Question 1 — Fine-tuning vs RAG: The Verdict

> **According to the "Verdict" on memory types, what is the primary purpose of fine-tuning versus RAG?**
>
> **B. Fine-tuning is for form (behavior, tone); RAG is for facts.**

---

## Sub-questions

### What is fine-tuning?

$$\theta_\text{fine} = \theta_\text{pretrain} - \alpha \nabla \mathcal{L}_\text{task}$$

Updating model weights on a task-specific dataset. Changes the model's **internal parameters** — altering how it writes, what style it uses, what format it follows.

### What is RAG (Retrieval-Augmented Generation)?

$$\text{response} = \text{LLM}(\text{query} + \text{retrieved\_documents})$$

At inference time, retrieve relevant documents from an external database and inject them into the prompt. The model's weights are **unchanged** — it reads new facts from context.

### What is "form" vs "facts"?

- **Form**: behavior, tone, style, output structure, persona — how the model responds
- **Facts**: specific data, numbers, documents, up-to-date information — what the model knows

---

## Main answer

**B. Fine-tuning is for form (behavior, tone); RAG is for facts.**

```
Memory type mapping:

  ┌──────────────────────────────────────────────────┐
  │              What do you need?                   │
  │                                                  │
  │   FORM (behavior, tone, style)                   │
  │   ──────────────────────────                     │
  │   → Fine-tuning                                  │
  │   Changes weights → permanent behavior change    │
  │   Example: "Always respond in JSON format"       │
  │   Example: "Use formal medical terminology"      │
  │                                                  │
  │   FACTS (data, documents, knowledge)             │
  │   ────────────────────────────────               │
  │   → RAG                                          │
  │   Injects context → temporary knowledge          │
  │   Example: "What were Q3 earnings?"              │
  │   Example: "Summarize this legal contract"       │
  │                                                  │
  └──────────────────────────────────────────────────┘
```

| Aspect | Fine-tuning | RAG |
|---|---|---|
| Purpose | Form (behavior, tone) | Facts (knowledge, data) |
| Mechanism | Modify weights | Inject context at inference |
| Persistence | Permanent | Per-query |
| Update cycle | Retrain model | Update document database |
| Example | "Respond like a doctor" | "What does policy §4.2 say?" |

---
---

# Quiz 8, Question 2 — Why Fine-tuning Fails at Storing Facts

> **Why does fine-tuning fail at reliably storing facts?**
>
> **C. It stores statistical relationships between words, making it probabilistic and prone to hallucination.**

---

## Sub-questions

### How does fine-tuning "store" information?

$$\theta \leftarrow \theta - \alpha \nabla \mathcal{L}$$

Knowledge is encoded implicitly in the statistical correlations between weight values — not as discrete, addressable facts. The model learns that certain words tend to follow others in certain contexts.

### What is hallucination?

The model generates text that is fluent and confident but factually incorrect. Because knowledge is stored as probabilities rather than exact records, the model can "interpolate" between learned patterns and produce plausible-sounding but wrong facts.

### Why is probabilistic storage unreliable for facts?

A fact like "Company X revenue = $4.2B" is stored as weighted connections across millions of parameters. Small perturbations, conflicting training data, or similar-sounding facts can cause the model to output "$4.5B" or "$3.8B" — the model has no mechanism to verify or look up the exact value.

---

## Main answer

**C. It stores statistical relationships between words, making it probabilistic and prone to hallucination.**

```
How fine-tuning stores "facts":

  Fact: "Paris is the capital of France"

  ┌────────────────────────────────────────────┐
  │  Stored as:                                │
  │  P("capital" | "Paris", "France") = high   │
  │  P("Paris" | "capital of France") = high   │
  │                                            │
  │  NOT stored as:                            │
  │  database_row("France", capital="Paris")   │
  └────────────────────────────────────────────┘

  Problem: P("Lyon" | "capital of France") = small but nonzero
           → hallucination possible
```

```
Comparison of storage mechanisms:

  Fine-tuning (parametric memory):
    fact ──▶ distributed across millions of weights ──▶ probabilistic recall
                                                         ↓
                                                    can hallucinate

  RAG (non-parametric memory):
    fact ──▶ stored in document database ──▶ exact retrieval
                                              ↓
                                         faithful to source
```

| Property | Fine-tuning (Parametric) | RAG (Non-parametric) |
|---|---|---|
| Storage | Distributed in weights | Explicit in documents |
| Recall | Probabilistic | Exact lookup |
| Hallucination risk | High | Low (grounded in source) |
| Updatability | Retrain required | Add/remove documents |
| Best for | Behavior, style | Facts, data |

---
---

# Quiz 8, Question 3 — Cosine Similarity vs Euclidean Distance

> **In the context of embedding vectors, why is Cosine Similarity preferred over Euclidean (L2) distance for measuring semantic meaning?**
>
> **B. Cosine similarity ignores vector magnitude, evaluating directional intent regardless of document length.**

---

## Sub-questions

### What is Cosine Similarity?

$$\cos(\theta) = \frac{\mathbf{a} \cdot \mathbf{b}}{\|\mathbf{a}\| \cdot \|\mathbf{b}\|} = \frac{\sum_i a_i b_i}{\sqrt{\sum_i a_i^2} \cdot \sqrt{\sum_i b_i^2}}$$

Measures the angle between two vectors. Range: $[-1, 1]$. Value of 1 = same direction (semantically identical), 0 = orthogonal, -1 = opposite.

**Numeric example:** $\mathbf{a} = [1, 2], \mathbf{b} = [2, 4]$ (same direction, different magnitude):

$$\cos(\theta) = \frac{1 \cdot 2 + 2 \cdot 4}{\sqrt{5} \cdot \sqrt{20}} = \frac{10}{10} = 1.0$$

### What is Euclidean (L2) Distance?

$$d(\mathbf{a}, \mathbf{b}) = \|\mathbf{a} - \mathbf{b}\|_2 = \sqrt{\sum_i (a_i - b_i)^2}$$

Measures the straight-line distance between two points. Sensitive to both direction **and** magnitude.

**Numeric example:** Same vectors $\mathbf{a} = [1, 2], \mathbf{b} = [2, 4]$:

$$d = \sqrt{(1-2)^2 + (2-4)^2} = \sqrt{1 + 4} = \sqrt{5} \approx 2.24$$

Despite identical direction (same semantic meaning), Euclidean distance reports them as far apart.

### What is vector magnitude?

$$\|\mathbf{a}\| = \sqrt{\sum_i a_i^2}$$

The "length" of the vector. Longer documents tend to produce embeddings with larger magnitudes, but this doesn't mean they are more semantically relevant.

---

## Main answer

**B. Cosine similarity ignores vector magnitude, evaluating directional intent regardless of document length.**

```
Cosine vs Euclidean:

                 b = [2, 4]
                ╱
               ╱  ← same direction as a
              ╱      (cosine = 1.0)
             ╱
            ╱
  a = [1,2]●
            ╲
             ╲  Euclidean distance = √5 ≈ 2.24
              ╲  (says they are "far apart"!)

  Cosine sees:  SAME meaning (angle = 0°)
  Euclidean sees: DIFFERENT (distance = 2.24)
```

$$\text{Cosine: } \frac{\mathbf{a} \cdot \mathbf{b}}{\|\mathbf{a}\| \|\mathbf{b}\|} \quad \text{(normalizes out magnitude)}$$

$$\text{Euclidean: } \|\mathbf{a} - \mathbf{b}\| \quad \text{(magnitude affects result)}$$

| Property | Cosine Similarity | Euclidean Distance |
|---|---|---|
| Measures | Angle (direction) | Straight-line distance |
| Affected by magnitude | No | Yes |
| Short doc vs long doc (same topic) | High similarity | Large distance |
| Range | $[-1, 1]$ | $[0, \infty)$ |
| Preferred for semantics | **Yes** | No |

---
---

# Quiz 8, Question 4 — Embedding Long Documents into a Single Vector

> **What happens if you try to embed an entire 100-page PDF document into a single vector?**
>
> **B. The vector represents the "average" meaning, and specific signals get drowned out by noise.**

---

## Sub-questions

### What is an embedding?

$$\text{embed}: \text{text} \rightarrow \mathbb{R}^d$$

A function that maps text to a fixed-dimensional vector (e.g., $d = 768$ or $d = 1536$). The vector captures the semantic meaning of the input.

### What does "average meaning" mean?

An embedding model must compress all information into a fixed-size vector. For short texts, most information is preserved. For very long texts, the vector becomes a blend of all topics:

$$\mathbf{v}_{100\text{-page}} \approx \frac{1}{N}\sum_{i=1}^{N} \text{topic}_i \quad \text{(oversimplification, but captures the intuition)}$$

### Why do specific signals get "drowned out"?

A 100-page document covers many topics. The embedding vector is a fixed-size container — it cannot separately represent every detail. Dominant/repeated themes dominate the vector while rare but important details contribute negligibly.

---

## Main answer

**B. The vector represents the "average" meaning, and specific signals get drowned out by noise.**

```
Short text embedding (preserves meaning):

  "The cat sat on the mat"
       │
       ▼
  [0.12, -0.45, 0.78, ...]   ← clear, focused signal
  (768 dimensions)


Long document embedding (diluted):

  100-page PDF covering:
  - Chapter 1: Contracts
  - Chapter 2: Compliance
  - Chapter 3: HR Policies
  - Chapter 4: Safety
  - ...
       │
       ▼
  [0.01, -0.02, 0.03, ...]   ← average of everything
  (768 dimensions)              specific details lost
```

```
Signal dilution:

  Query: "What is the fire safety protocol?"

  ┌─────────────────┐     ┌─────────────────┐
  │ Chunk: "Fire     │     │ Whole PDF:       │
  │ safety protocol  │     │ 100 pages of     │
  │ requires..."     │     │ mixed content    │
  │                  │     │                  │
  │ cosine(q, chunk) │     │ cosine(q, pdf)   │
  │ = 0.92 ✓ match  │     │ = 0.31 ✗ weak   │
  └─────────────────┘     └─────────────────┘
```

This is why **chunking** is essential in RAG: split documents into smaller pieces so each embedding captures focused, retrievable meaning.

| Input Length | Embedding Quality | Specific Detail Retrieval |
|---|---|---|
| 1 sentence | Excellent (focused) | High |
| 1 paragraph | Good | Good |
| 1 page | Acceptable | Moderate |
| **100 pages** | **Poor (averaged)** | **Low (drowned in noise)** |

---
---

# Quiz 8, Question 5 — Semantic Chunking

> **How does "Semantic Chunking" determine where to slice a document?**
>
> **C. By dynamically measuring cosine similarity between sequential sentences and splitting when it drops drastically.**

---

## Sub-questions

### What is chunking?

Dividing a document into smaller pieces (chunks) for embedding and retrieval. Each chunk gets its own vector in the database.

### What is fixed-size chunking?

$$\text{chunk}_i = \text{text}[i \cdot \text{stride} : i \cdot \text{stride} + \text{chunk\_size}]$$

Split by token count (e.g., every 500 tokens with 50-token overlap). Simple but can cut mid-sentence or mid-paragraph, breaking semantic coherence.

### What is semantic chunking?

1. Embed each sentence individually: $\mathbf{v}_i = \text{embed}(\text{sentence}_i)$
2. Compute cosine similarity between consecutive sentences: $\text{sim}(i, i+1) = \cos(\mathbf{v}_i, \mathbf{v}_{i+1})$
3. When similarity drops below a threshold → insert a split point

The idea: a large drop in similarity means the topic changed.

---

## Main answer

**C. By dynamically measuring cosine similarity between sequential sentences and splitting when it drops drastically.**

```
Semantic chunking algorithm:

  Sentence:  S₁    S₂    S₃    S₄    S₅    S₆    S₇    S₈
  Topic:    [--- Topic A ---] [--- Topic B ---] [- Topic C -]

  Cosine similarity between consecutive sentences:
  sim:      0.89  0.91  0.32  0.87  0.85  0.28  0.90
                        ↑                  ↑
                     BIG DROP           BIG DROP
                     (split!)           (split!)

  Result:
  Chunk 1: [S₁, S₂, S₃]    ← Topic A
  Chunk 2: [S₄, S₅, S₆]    ← Topic B
  Chunk 3: [S₇, S₈]        ← Topic C
```

```
Pipeline:

  Document
     │
     ▼  Split into sentences
  [S₁, S₂, S₃, ..., Sₙ]
     │
     ▼  Embed each sentence
  [v₁, v₂, v₃, ..., vₙ]
     │
     ▼  Compute consecutive cosine similarities
  [sim(1,2), sim(2,3), ..., sim(n-1,n)]
     │
     ▼  Split where sim drops below threshold
  [Chunk₁, Chunk₂, ..., Chunkₖ]
```

| Chunking Method | Split Criterion | Respects Topic Boundaries | Complexity |
|---|---|---|---|
| Fixed-size | Token count | No | $O(1)$ |
| Paragraph-based | Newlines | Sometimes | $O(1)$ |
| **Semantic** | **Cosine similarity drop** | **Yes** | $O(n)$ embeddings |

---
---

# Quiz 8, Question 6 — Lost in the Middle

> **What is the "Lost in the Middle" phenomenon in Large Language Models?**
>
> **B. LLM attention perfectly recalls the first and last chunks but completely ignores chunks buried in the middle of a long prompt.**

---

## Sub-questions

### What causes this phenomenon?

The self-attention mechanism in Transformers exhibits a **positional bias**: tokens at the beginning and end of the context window receive disproportionately high attention weights, while tokens in the middle receive less.

### What is the practical impact for RAG?

If you retrieve 10 relevant documents and concatenate them into the prompt, the LLM may effectively ignore documents placed in positions 4–7, even if they contain the most relevant information.

### How is this measured?

Studies show that LLM accuracy on question-answering tasks follows a U-shaped curve when the position of the relevant information is varied:

$$\text{Accuracy}(i) \approx \begin{cases} \text{High} & i \text{ near start} \\ \text{Low} & i \text{ in middle} \\ \text{High} & i \text{ near end} \end{cases}$$

---

## Main answer

**B. LLM attention perfectly recalls the first and last chunks but completely ignores chunks buried in the middle of a long prompt.**

```
Accuracy vs position of relevant information:

  Accuracy
   ↑
  High│ ●                               ●
      │  ╲                             ╱
      │   ╲                           ╱
      │    ╲                         ╱
  Low │     ╲_______________________╱
      │           "Lost in the Middle"
      └──────────────────────────────────▶ Position
      Start                            End
      (1st chunk)                    (last chunk)
```

```
RAG prompt with 10 retrieved documents:

  [System prompt]
  [Doc 1]  ← ✓ high attention (primacy)
  [Doc 2]  ← ↓ declining
  [Doc 3]
  [Doc 4]  ← ✗ "lost" zone
  [Doc 5]  ← ✗ minimal attention
  [Doc 6]  ← ✗ "lost" zone
  [Doc 7]
  [Doc 8]  ← ↑ recovering
  [Doc 9]
  [Doc 10] ← ✓ high attention (recency)
  [Query]
```

**Mitigation strategy: Algorithmic Re-ordering** — place the most relevant chunks at the start and end of the context window, not in the middle.

| Position | Attention Level | Recall Quality |
|---|---|---|
| Start (primacy) | High | Strong |
| Middle | **Low** | **Weak ("lost")** |
| End (recency) | High | Strong |

---
---

# Quiz 8, Question 7 — ANN Search Time Complexity

> **What time complexity does Approximate Nearest Neighbor (ANN) search achieve, compared to the O(N) brute-force k-NN approach?**
>
> **C. O(log N)**

---

## Sub-questions

### What is brute-force k-NN?

$$\text{For each query, compute distance to all } N \text{ vectors} \implies O(N)$$

Compare the query against every single vector in the database. Guarantees exact results but scales linearly — impractical for billions of vectors.

### What is Approximate Nearest Neighbor (ANN)?

Trade a small amount of accuracy for dramatically faster search. Uses index structures (trees, graphs, hash tables) to skip most of the database and only examine a small subset of candidates.

### What does $O(\log N)$ mean?

$$\text{Doubling the database size adds only one extra step.}$$

**Numeric example:** $N = 10^6$ → $\sim 20$ steps. $N = 10^9$ → $\sim 30$ steps. Compared to brute-force: $10^9$ distance computations vs. $\sim 30$.

---

## Main answer

**C. O(log N)**

```
Search time scaling:

  Time
   ↑
   │             ╱  O(N) brute-force
   │           ╱
   │         ╱
   │       ╱
   │     ╱
   │   ╱───────────────────  O(log N) ANN
   │ ╱
   └──────────────────────────▶ N (database size)
   10³  10⁶       10⁹
```

$$\text{Brute-force: } O(N) = O(10^9) \text{ comparisons}$$
$$\text{ANN: } O(\log N) = O(\log 10^9) \approx O(30) \text{ comparisons}$$

| Method | Time Complexity | 1B vectors | Exact? |
|---|---|---|---|
| Brute-force k-NN | $O(N)$ | $10^9$ comparisons | Yes |
| **ANN (HNSW, IVF)** | **$O(\log N)$** | **~30 comparisons** | Approximate |
| Speedup | — | **~33 million×** | — |

---
---

# Quiz 8, Question 8 — Voronoi Cell

> **What is a "Voronoi Cell" in the context of a vector database?**
>
> **B. A mathematical proximity map where every point inside the boundary is closer to its specific "seed" than any other seed.**

---

## Sub-questions

### What is a Voronoi diagram?

Given a set of seed points (centroids) in space, a Voronoi diagram partitions the entire space into regions (cells). Each cell contains all points that are closer to its seed than to any other seed.

$$\text{Cell}_i = \{ \mathbf{x} \mid d(\mathbf{x}, \mathbf{c}_i) \leq d(\mathbf{x}, \mathbf{c}_j) \; \forall \; j \neq i \}$$

### How are Voronoi cells used in vector databases?

In **IVF (Inverted File Index)** search:

1. Cluster all vectors using K-Means → centroids become Voronoi seeds
2. Each vector is assigned to its nearest centroid's cell
3. At query time: find the nearest centroid(s) → only search vectors in those cells

This avoids scanning the entire database.

### What is a centroid?

$$\mathbf{c}_i = \frac{1}{|S_i|}\sum_{\mathbf{x} \in S_i} \mathbf{x}$$

The mean of all vectors assigned to cluster $i$. Acts as the "representative point" for its cell.

---

## Main answer

**B. A mathematical proximity map where every point inside the boundary is closer to its specific "seed" than any other seed.**

```
Voronoi diagram (2D example with 4 seeds):

  ┌─────────────────────────────────┐
  │         ╱    │                  │
  │   Cell A╱     │    Cell B       │
  │       ╱       │                 │
  │  ● A╱         │    ● B         │
  │    ╱           │                │
  │  ╱─────────────┼────────────    │
  │╱               │           ╲   │
  │    Cell C      │  Cell D    ╲  │
  │                │             ╲ │
  │   ● C          │   ● D        │
  │                │               │
  └─────────────────────────────────┘

  Every point in Cell A is closer to seed A
  than to seeds B, C, or D.
```

```
IVF search using Voronoi cells:

  Query q
     │
     ▼  Compare q to all centroids (few hundred)
  Nearest centroid = C₃
     │
     ▼  Search ONLY vectors in Cell₃ (not entire database)
  Top-k results from Cell₃
```

| Concept | Definition |
|---|---|
| Seed / Centroid | Representative point for a cluster |
| Voronoi Cell | Region of all points nearest to a given seed |
| Cell boundary | Where equidistant to two seeds |
| IVF application | Only search vectors in the query's nearest cell(s) |

---
---

# Quiz 8, Question 9 — nprobe Parameter in Voronoi Search

> **How does the "nprobe" parameter solve the border problem in Voronoi cell searches?**
>
> **B. It searches the closest N cells to the query vector, rather than just the single closest cell.**

---

## Sub-questions

### What is the border problem?

A query vector near the boundary between two Voronoi cells might have its true nearest neighbor in the **adjacent** cell, not its own cell. If we only search one cell, we miss that neighbor.

```
  ┌────────────────┬────────────────┐
  │   Cell A       │   Cell B       │
  │                │                │
  │        q ●───► │ ◄──● true NN  │
  │   (query is    │  (in Cell B!)  │
  │    in Cell A)  │                │
  └────────────────┴────────────────┘
```

### What does nprobe do?

$$\text{nprobe} = k \implies \text{search the } k \text{ nearest cells}$$

Instead of only searching the single closest cell, search the $k$ nearest cells. Higher nprobe → better recall, more computation.

**Numeric example:** nprobe = 1: search 1 cell. nprobe = 10: search 10 cells. If there are 1000 cells total, nprobe = 10 scans only 1% of the database.

---

## Main answer

**B. It searches the closest N cells to the query vector, rather than just the single closest cell.**

```
nprobe = 1 (single cell, border problem):

  ┌──────────┬──────────┬──────────┐
  │ Cell 1   │ Cell 2   │ Cell 3   │
  │          │          │          │
  │          │  q● ──── │──── ●nn  │
  │          │ searched │ NOT      │
  │          │          │ searched │
  └──────────┴──────────┴──────────┘
  Result: misses true nearest neighbor ✗


nprobe = 2 (two nearest cells):

  ┌──────────┬──────────┬──────────┐
  │ Cell 1   │ Cell 2   │ Cell 3   │
  │          │ ████████ │ ████████ │
  │          │ q● ──── │──── ●nn  │
  │          │ searched │ searched │
  │          │ ████████ │ ████████ │
  └──────────┴──────────┴──────────┘
  Result: finds true nearest neighbor ✓
```

$$\text{Recall} \uparrow \text{ as nprobe} \uparrow, \quad \text{Speed} \downarrow \text{ as nprobe} \uparrow$$

| nprobe | Cells Searched | Recall | Speed |
|---|---|---|---|
| 1 | 1 (single cell) | Low (border misses) | Fastest |
| 10 | 10 | High | Fast |
| 100 | 100 | Very high | Moderate |
| $N_\text{cells}$ | All | Perfect (brute-force) | Slowest |

---
---

# Quiz 8, Question 10 — Product Quantization: 256 Centroids

> **In Product Quantization (PQ), why is a high-dimensional vector divided into subvectors that are mapped to exactly 256 centroids?**
>
> **C. Because 256 unique values can be represented by precisely 1 byte of memory.**

---

## Sub-questions

### What is Product Quantization?

Split a high-dimensional vector into $M$ subvectors. For each subvector, learn a codebook of $K$ centroids via K-Means. Replace each subvector with its nearest centroid's **ID** (an integer).

$$\mathbf{v} = [\underbrace{v_1, \dots, v_{d/M}}_{\text{sub}_1}, \underbrace{v_{d/M+1}, \dots, v_{2d/M}}_{\text{sub}_2}, \dots, \underbrace{\dots, v_d}_{\text{sub}_M}]$$

### Why exactly 256 centroids?

$$2^8 = 256 \implies \text{centroid ID fits in exactly 1 byte (8 bits)}$$

This is the maximum number of unique values representable by a single byte — the most efficient encoding without wasting bits.

**Numeric example:** Centroid ID = 42 → stored as byte `0x2A` → exactly 8 bits. If we used 257 centroids, we'd need 9 bits → wastes 7 bits of a 2-byte representation.

### What is the memory savings?

Original subvector: $d/M$ floats × 4 bytes each. After PQ: 1 byte (centroid ID).

---

## Main answer

**C. Because 256 unique values can be represented by precisely 1 byte of memory.**

$$K = 256 = 2^8 \implies \lceil \log_2(256) \rceil = 8 \text{ bits} = 1 \text{ byte per subvector}$$

```
Product Quantization pipeline:

  Original vector (1536 dims, FP32):
  [0.12, -0.45, 0.78, 0.33, ..., 0.56]
  Size: 1536 × 4 bytes = 6,144 bytes

       │  Split into 96 subvectors of 16 dims each
       ▼

  Sub₁: [0.12, -0.45, ..., 0.21]  → nearest centroid → ID: 42   → 1 byte
  Sub₂: [0.78, 0.33, ..., -0.11]  → nearest centroid → ID: 183  → 1 byte
  ...
  Sub₉₆: [0.56, 0.09, ..., 0.34] → nearest centroid → ID: 7    → 1 byte

  Compressed: [42, 183, ..., 7]
  Size: 96 × 1 byte = 96 bytes
```

$$\text{Compression ratio} = \frac{6144}{96} = 64\times$$

| Aspect | Value | Why |
|---|---|---|
| Centroids per subspace | 256 | $2^8$ = 1 byte exactly |
| Bits per centroid ID | 8 | Fills a byte perfectly |
| If 257 centroids | Would need 9 bits → wasteful | Not byte-aligned |
| If 128 centroids | Only 7 bits → wastes 1 bit | Less representative |

---
---

# Quiz 8, Question 11 — HNSW Algorithm Components

> **What two concepts are combined to create the Hierarchical Navigable Small World (HNSW) algorithm?**
>
> **B. Navigable Small World (NSW) Graphs and Skip Lists**

---

## Sub-questions

### What is a Navigable Small World (NSW) graph?

A graph where each node is a vector and edges connect it to its approximate nearest neighbors. Search works by greedy routing: starting from any node, follow edges to neighbors that are progressively closer to the query.

The "small world" property: any two nodes are reachable in a small number of hops, even in a massive graph.

### What is a Skip List?

A multi-layered linked list data structure. The bottom layer contains all elements. Each higher layer is a "fast lane" containing a subset of elements, enabling $O(\log N)$ search by skipping over large sections.

```
Layer 3:  1 ─────────────────────── 9
Layer 2:  1 ────── 4 ────────────── 9
Layer 1:  1 ── 3 ── 4 ── 6 ── 7 ── 9
Layer 0:  1  2  3  4  5  6  7  8  9
```

### How does HNSW combine them?

HNSW builds an NSW graph at multiple layers (like a skip list). Upper layers have fewer nodes with long-range connections (for fast coarse navigation). Lower layers have more nodes with short-range connections (for precise local search).

---

## Main answer

**B. Navigable Small World (NSW) Graphs and Skip Lists**

```
HNSW structure:

  Layer 2 (sparse):    ● ────────────────── ●
                       (few nodes, long-range links)
                       (fast, coarse navigation)

  Layer 1 (medium):    ● ──── ● ──── ● ──── ● ──── ●
                       (more nodes, medium links)

  Layer 0 (dense):     ● ─ ● ─ ● ─ ● ─ ● ─ ● ─ ● ─ ● ─ ●
                       (all nodes, short-range links)
                       (precise, local search)
```

```
Search flow:

  Query q
     │
     ▼  Enter at top layer (Layer 2)
  Greedy walk on sparse graph → find approximate region
     │
     ▼  Descend to Layer 1
  Greedy walk on medium graph → refine region
     │
     ▼  Descend to Layer 0
  Greedy walk on dense graph → find exact nearest neighbors
     │
     ▼
  Result: top-k nearest neighbors
```

| Component | Contribution to HNSW |
|---|---|
| NSW Graph | Greedy routing on neighbor edges → navigable search |
| Skip List | Hierarchical layers → $O(\log N)$ complexity |
| **Combined** | **Multi-layer navigable graph → fast, accurate ANN** |

---
---

# Quiz 8, Question 12 — HNSW Search Progression

> **In the HNSW architecture, how does the search physically progress?**
>
> **B. It starts at a random entry point in the uppermost sparse layer and descends to denser layers.**

---

## Sub-questions

### Why start at the top (sparse) layer?

The top layer has very few nodes with long-range connections. This allows the search to quickly jump to the approximate region of the vector space where the query belongs — like taking a highway before using local roads.

### What happens during descent?

At each layer, the algorithm performs a greedy walk: move to the neighbor closest to the query. When no closer neighbor exists at the current layer, descend to the next layer (which has more nodes and shorter edges) and continue.

### What happens at the bottom layer?

Layer 0 contains **all** vectors. The greedy walk here performs fine-grained local search among the actual nearest neighbors, producing the final result.

---

## Main answer

**B. It starts at a random entry point in the uppermost sparse layer and descends to denser layers.**

```
HNSW search progression:

  Layer 2:  ● ─ ─ ─ ─ ─ ● ─ ─ ─ ─ ─ ●    ← ENTER here
            (3 nodes)    ↓ greedy walk      (jump to approximate region)
                         ↓ descend

  Layer 1:  ● ── ● ── ● ── ● ── ● ── ●    ← refine
            (6 nodes)       ↓ greedy walk
                            ↓ descend

  Layer 0:  ●─●─●─●─●─●─●─●─●─●─●─●─●     ← precise local search
            (all N nodes)   ↓
                            ↓
                    TOP-K RESULTS
```

```
Analogy:

  Layer 2:  "Which country?"     ← coarse (fast, long jumps)
  Layer 1:  "Which city?"        ← medium
  Layer 0:  "Which street?"      ← fine (slow, short steps)
```

| Layer | Nodes | Edge Length | Purpose |
|---|---|---|---|
| Top (sparse) | Few | Long-range | Coarse navigation |
| Middle | Medium | Medium | Refine region |
| Bottom (dense) | All $N$ | Short-range | Precise local search |

---
---

# Quiz 8, Question 13 — L2 Normalization for Vector Databases

> **How do vector databases solve the paradox of needing to calculate semantic angles (Cosine) but relying on physical distance (Euclidean) for speed?**
>
> **B. They apply L2 Normalization to force every vector to have a length of exactly 1.**

---

## Sub-questions

### What is L2 Normalization?

$$\hat{\mathbf{v}} = \frac{\mathbf{v}}{\|\mathbf{v}\|_2} \implies \|\hat{\mathbf{v}}\|_2 = 1$$

Divide each vector by its magnitude, projecting it onto the unit hypersphere.

### Why does this solve the paradox?

For unit vectors ($\|\mathbf{a}\| = \|\mathbf{b}\| = 1$):

$$\|\mathbf{a} - \mathbf{b}\|_2^2 = \|\mathbf{a}\|^2 + \|\mathbf{b}\|^2 - 2\mathbf{a} \cdot \mathbf{b} = 1 + 1 - 2\cos(\theta) = 2(1 - \cos(\theta))$$

$$\text{Euclidean}^2 = 2(1 - \text{Cosine Similarity})$$

Euclidean distance becomes a **monotonic function** of cosine similarity. Minimizing Euclidean distance is equivalent to maximizing cosine similarity.

### Why use Euclidean for speed?

Euclidean distance is computationally simpler — it can leverage SIMD instructions, GPU parallelism, and optimized index structures (HNSW, IVF) that are designed for $L_2$ distance.

---

## Main answer

**B. They apply L2 Normalization to force every vector to have a length of exactly 1.**

```
The equivalence trick:

  Step 1: L2 Normalize all vectors
  ┌─────────────────────┐
  │ v̂ = v / ‖v‖        │
  │ ‖v̂‖ = 1 for all v  │
  └─────────────────────┘

  Step 2: Use fast Euclidean distance on normalized vectors
  ┌─────────────────────────────────────────────┐
  │ ‖â - b̂‖² = 2(1 - cos(θ))                   │
  │                                             │
  │ min Euclidean ⟺ max Cosine Similarity      │
  └─────────────────────────────────────────────┘
```

```
On the unit circle (2D illustration):

         â●
        ╱  ╲  ‖â - b̂‖ = Euclidean distance
       ╱ θ  ╲
      ╱      ╲
  ───●────────●b̂───
     origin
     ‖â‖ = ‖b̂‖ = 1

  Small θ → small Euclidean → high cosine similarity
  Large θ → large Euclidean → low cosine similarity
```

$$\|\hat{\mathbf{a}} - \hat{\mathbf{b}}\|^2 = 2 - 2\cos(\theta) = 2(1 - \cos(\theta))$$

| Step | What | Why |
|---|---|---|
| L2 Normalize | $\hat{\mathbf{v}} = \mathbf{v}/\|\mathbf{v}\|$ | Project onto unit sphere |
| Use Euclidean | $\|\hat{\mathbf{a}} - \hat{\mathbf{b}}\|$ | Fast hardware, index-friendly |
| **Result** | **Euclidean ranking = Cosine ranking** | **Best of both worlds** |

---
---

# Quiz 8, Question 14 — Bi-Encoder Limitations

> **Why does a standard Bi-Encoder sometimes struggle with linguistic nuance?**
>
> **A. It embeds documents and queries completely independently without them ever "seeing" each other.**

---

## Sub-questions

### What is a Bi-Encoder?

$$\mathbf{v}_q = \text{Encoder}(\text{query}), \quad \mathbf{v}_d = \text{Encoder}(\text{document})$$

$$\text{score}(q, d) = \cos(\mathbf{v}_q, \mathbf{v}_d)$$

Two separate forward passes — one for the query, one for the document. They produce independent vectors that are compared via cosine similarity.

### Why is independent embedding a problem?

Without cross-attention between query and document, the encoder cannot capture interactions like:

- Negation: "What is NOT a cause of..." vs "What is a cause of..."
- Paraphrase: query uses different words than the document
- Context-dependent meaning: same word means different things depending on surrounding text

### What is the alternative (Cross-Encoder)?

$$\text{score}(q, d) = \text{Encoder}([q \; \text{[SEP]} \; d]) \rightarrow \text{scalar}$$

Concatenate query and document as a single input. Full cross-attention between all tokens — the model can see both simultaneously and reason about their relationship.

---

## Main answer

**A. It embeds documents and queries completely independently without them ever "seeing" each other.**

```
Bi-Encoder (independent):

  Query: "non-toxic paint"     Document: "paint that is safe"
       │                              │
       ▼                              ▼
  Encoder(query)                 Encoder(doc)
       │                              │
       ▼                              ▼
  v_q = [0.3, 0.8, ...]         v_d = [0.4, 0.7, ...]
       │                              │
       └──────── cosine(v_q, v_d) ─────┘
                 = 0.72

  ✗ No cross-attention between query and document
  ✗ "non-toxic" ≠ "safe" at word level (needs reasoning)


Cross-Encoder (joint):

  Input: ["non-toxic paint" [SEP] "paint that is safe"]
       │
       ▼
  Encoder sees BOTH together
  Full cross-attention between all tokens
       │
       ▼
  Score = 0.94  ← understands "non-toxic" ≈ "safe"
```

| Property | Bi-Encoder | Cross-Encoder |
|---|---|---|
| Encoding | Independent | Joint |
| Cross-attention | None | Full |
| Linguistic nuance | **Weak** | Strong |
| Speed | Fast (pre-compute doc embeddings) | Slow (must run per query-doc pair) |
| Scalability | $O(1)$ per query (pre-computed) | $O(N)$ per query |
| Use in RAG | Stage 1: fast retrieval | Stage 2: re-ranking |

---
---

# Quiz 8, Question 15 — Cross-Encoder in Two-Stage RAG

> **In a Two-Stage RAG Architecture, what is the role of the Cross-Encoder?**
>
> **C. It takes a smaller subset of retrieved documents and performs a deep analysis to output highly accurate relevance scores.**

---

## Sub-questions

### What is the Two-Stage RAG Architecture?

1. **Stage 1 (Retrieval):** Bi-Encoder quickly retrieves top-$k$ candidates from the full database
2. **Stage 2 (Re-ranking):** Cross-Encoder deeply scores each candidate and re-orders them by relevance

### Why not use Cross-Encoder for everything?

Cross-Encoder runs a full forward pass for each (query, document) pair. For $N = 10^6$ documents:

$$\text{Cross-Encoder on all: } 10^6 \text{ forward passes → hours per query}$$

By using Bi-Encoder first to narrow to 20–50 candidates, the Cross-Encoder only needs 20–50 forward passes → milliseconds.

---

## Main answer

**C. It takes a smaller subset of retrieved documents and performs a deep analysis to output highly accurate relevance scores.**

```
Two-Stage RAG pipeline:

  Query
    │
    ▼  Stage 1: Bi-Encoder (fast, approximate)
  ┌─────────────────────────────────────┐
  │ Search 1,000,000 documents          │
  │ via cosine similarity               │
  │ Return top-50 candidates            │
  │ Time: ~10ms                         │
  └─────────────────────────────────────┘
    │
    ▼  50 candidates
    │
    ▼  Stage 2: Cross-Encoder (slow, precise)
  ┌─────────────────────────────────────┐
  │ Deep cross-attention analysis       │
  │ on each (query, doc) pair           │
  │ Re-rank 50 candidates               │
  │ Time: 50 × 25ms = 1,250ms          │
  └─────────────────────────────────────┘
    │
    ▼  Re-ranked top-k results → LLM
```

| Stage | Model | Input | Speed | Accuracy |
|---|---|---|---|---|
| 1 (Retrieve) | Bi-Encoder | All $N$ docs | Fast (~10ms) | Good |
| **2 (Re-rank)** | **Cross-Encoder** | **Top-$k$ only** | **Moderate** | **High** |

---
---

# Quiz 8, Question 16 — How Frozen LLMs "Learn" from Context

> **How can an LLM mathematically "learn" from injected context despite having frozen weights?**
>
> **A. Through the Self-Attention mechanism calculating the relevance of every token dynamically ($Q \cdot K^T \cdot V$).**

---

## Sub-questions

### What does "frozen weights" mean?

$$\theta_\text{inference} = \theta_\text{pretrained} \quad \text{(no gradient updates)}$$

The model's parameters do not change. No fine-tuning, no learning in the traditional sense.

### How does self-attention enable dynamic "learning"?

$$\text{Attention}(Q, K, V) = \text{Softmax}\!\left(\frac{QK^T}{\sqrt{d_k}}\right) V$$

The attention weights are computed **at inference time** based on the actual input tokens. When you inject retrieved documents into the prompt, every query token can attend to every document token. The model dynamically computes which parts of the context are relevant to each output token.

### What changes when you inject context?

The **input tokens** change → $Q$, $K$, $V$ matrices change → attention weights change → output changes. The model "learns" the injected facts through the attention mechanism, not through weight updates.

---

## Main answer

**A. Through the Self-Attention mechanism calculating the relevance of every token dynamically ($Q \cdot K^T \cdot V$).**

```
How frozen LLMs "learn" from context:

  Without RAG context:
  ┌─────────────────────────────────┐
  │ Input: "What is X?"             │
  │ Attention over: query tokens    │
  │ Answer: from parametric memory  │
  │         (may hallucinate)       │
  └─────────────────────────────────┘

  With RAG context:
  ┌─────────────────────────────────┐
  │ Input: [retrieved docs] +       │
  │        "What is X?"             │
  │ Attention over: docs + query    │
  │ Answer: attends to relevant     │
  │         document tokens         │
  │         (grounded in facts)     │
  └─────────────────────────────────┘
```

$$\text{Same weights } \theta, \text{ different input} \implies \text{different } Q, K, V \implies \text{different output}$$

```
Attention mechanism as "soft lookup":

  Query token: "revenue"
       │
       ▼  Q·Kᵀ computes relevance to every context token
  ┌─────────────────────────────────────────────┐
  │ "The"=0.01  "company"=0.05  "earned"=0.12  │
  │ "$4.2B"=0.82  "in"=0.01  "Q3"=0.15  ...   │
  └─────────────────────────────────────────────┘
       │
       ▼  Softmax → attention weights → weighted sum of V
  Output: heavily influenced by "$4.2B" token
```

| Mechanism | What Changes | What Stays Fixed |
|---|---|---|
| Weight update (fine-tuning) | $\theta$ (parameters) | — |
| **Self-Attention (RAG)** | **$Q, K, V$ (from input)** | **$\theta$ (frozen)** |
| RAG "learning" is | Dynamic, per-query | Not persistent |

---
---

# Quiz 8, Question 17 — Algorithmic Re-ordering

> **What is "Algorithmic Re-ordering" in the context of prompt construction?**
>
> **B. Placing the most highly relevant retrieved chunks at the very edges (start and end) of the context window to maximize attention.**

---

## Sub-questions

### Why is placement important?

Due to the "Lost in the Middle" phenomenon, LLMs attend most strongly to tokens at the **beginning** and **end** of the context window. Chunks placed in the middle receive less attention.

### What is Algorithmic Re-ordering?

After retrieving and ranking chunks by relevance, rearrange them in the prompt so the most important chunks occupy the high-attention positions:

1. Most relevant → **start** of context
2. Least relevant → middle of context
3. Second-most relevant → **end** of context (just before the query)

---

## Main answer

**B. Placing the most highly relevant retrieved chunks at the very edges (start and end) of the context window to maximize attention.**

```
Standard ordering (by relevance score):

  [Doc1: 0.95] [Doc2: 0.88] [Doc3: 0.82] [Doc4: 0.75] [Doc5: 0.70]
   high attn    ↓ declining                              high attn
                    ↓
            Doc3, Doc4 get LOST in the middle


Re-ordered (edges strategy):

  [Doc1: 0.95] [Doc4: 0.75] [Doc5: 0.70] [Doc3: 0.82] [Doc2: 0.88]
   high attn    ← least important in middle →           high attn
   (best doc)    (expendable)                          (2nd best doc)
```

```
Attention distribution with re-ordering:

  Attention
   ↑
  High│ ●Doc1                               ●Doc2
      │  ╲                                 ╱
      │   ╲                               ╱
  Low │    ╲___Doc4____Doc5____Doc3______╱
      │           (least important here)
      └──────────────────────────────────────▶ Position
      Start                                End
```

| Position | Attention Level | Assigned Chunk |
|---|---|---|
| Start | High (primacy) | Most relevant |
| Middle | Low ("lost") | Least relevant |
| End | High (recency) | Second-most relevant |

---
---

# Quiz 8, Question 18 — Enforcing Grounding via Citations

> **Which of the following is a strict method for enforcing grounding and mitigating hallucinations?**
>
> **B. Prompting the model to append a citation (Document ID) to every sentence, and verifying it via Regex.**

---

## Sub-questions

### What is grounding?

Ensuring every claim in the model's output is traceable to a specific source document. The opposite of hallucination.

### What is a citation-based approach?

Instruct the model: "After every factual sentence, include a citation like [Doc-3]." Then programmatically verify that every sentence contains a valid document ID using regex.

$$\text{Regex: } \texttt{\\[Doc-\\d+\\]} \quad \text{matches } \texttt{[Doc-1]}, \texttt{[Doc-42]}, \text{etc.}$$

### Why is regex verification "strict"?

It's **programmatic and deterministic** — not relying on the model's self-assessment. Any sentence without a valid citation is flagged automatically. The system can reject or re-prompt for uncited claims.

---

## Main answer

**B. Prompting the model to append a citation (Document ID) to every sentence, and verifying it via Regex.**

```
Grounding pipeline:

  Retrieved docs: [Doc-1: "Revenue was $4.2B", Doc-2: "Growth was 12%"]
       │
       ▼  LLM generates response with citations
  "The company earned $4.2B in revenue [Doc-1].
   Year-over-year growth reached 12% [Doc-2]."
       │
       ▼  Regex verification
  Sentence 1: "earned $4.2B..." → [Doc-1] ✓ found
  Sentence 2: "growth reached 12%..." → [Doc-2] ✓ found
       │
       ▼  All sentences cited → PASS


  Hallucination detection:
  "The company plans to expand to Asia."
       │
       ▼  Regex verification
  No [Doc-X] found → ✗ FLAGGED (potential hallucination)
```

| Method | Verification | Strictness | Scalable |
|---|---|---|---|
| Manual review | Human reads output | Low (subjective) | No |
| Model self-check | Ask LLM "is this true?" | Medium (unreliable) | Yes |
| **Citation + Regex** | **Programmatic pattern match** | **High (deterministic)** | **Yes** |

---
---

# Quiz 8, Question 19 — RAGAS Evaluation Metrics

> **What are the three core evaluation metrics used in the RAGAS framework?**
>
> **C. Context Precision, Faithfulness, Answer Relevance**

---

## Sub-questions

### What is RAGAS?

**R**etrieval **A**ugmented **G**eneration **A**ssessment — a framework for evaluating RAG pipelines. Uses LLMs to automatically score the quality of retrieval and generation.

### What is Context Precision?

$$\text{Context Precision} = \frac{\text{relevant chunks retrieved}}{\text{total chunks retrieved}}$$

Did the retriever return the **right** documents? High precision = few irrelevant results.

### What is Faithfulness?

$$\text{Faithfulness} = \frac{\text{claims supported by context}}{\text{total claims in answer}}$$

Is the answer **grounded** in the retrieved documents? High faithfulness = no hallucinated claims.

### What is Answer Relevance?

$$\text{Answer Relevance} = \cos(\text{embed}(\text{answer}), \text{embed}(\text{question}))$$

Does the answer actually **address** the question? An answer can be faithful to context but not answer the user's actual question.

---

## Main answer

**C. Context Precision, Faithfulness, Answer Relevance**

```
RAGAS evaluates the full RAG pipeline:

  Query ──▶ Retriever ──▶ Context ──▶ Generator ──▶ Answer
              │                         │              │
              ▼                         ▼              ▼
         Context Precision        Faithfulness    Answer Relevance
         "Did we retrieve         "Is the answer  "Does the answer
          the right docs?"         grounded?"      address the query?"
```

```
Example scoring:

  Query: "What was Q3 revenue?"
  Retrieved: [Doc about Q3 earnings, Doc about Q2 earnings, Doc about HR]
  Answer: "Q3 revenue was $4.2B based on the earnings report."

  Context Precision: 1/3 relevant = 0.33 (retrieved irrelevant docs)
  Faithfulness: 1/1 claims supported = 1.0 (fully grounded)
  Answer Relevance: high cosine similarity = 0.95 (answers the question)
```

| Metric | What it Measures | Target Component | Ideal |
|---|---|---|---|
| Context Precision | Retrieval quality | Retriever | 1.0 |
| Faithfulness | Groundedness (no hallucination) | Generator | 1.0 |
| Answer Relevance | Query alignment | Generator | 1.0 |

---
---

# Quiz 8, Question 20 — Contrastive Training for Q&A Embeddings

> **Why is contrastive training required for specialized Q&A embedding models?**
>
> **A. Because questions and answers have different structural and grammatical syntax, yet must map to the exact same neighborhood in latent space.**

---

## Sub-questions

### What is the problem with standard embedding models?

A standard embedding model embeds text based on surface-level similarity. Questions and answers have very different structures:

- Question: "What is the capital of France?"
- Answer: "Paris is the capital of France."

These share some words but have different syntax. A naive model might place them far apart.

### What is contrastive training?

$$\mathcal{L} = -\log \frac{e^{\cos(\mathbf{v}_q, \mathbf{v}_a^+)/\tau}}{\sum_{j} e^{\cos(\mathbf{v}_q, \mathbf{v}_j)/\tau}}$$

Train the encoder to pull matching (query, answer) pairs **together** and push non-matching pairs **apart** in embedding space.

- Positive pairs: (question, correct answer) → high similarity
- Negative pairs: (question, wrong answer) → low similarity

### What is latent space?

The embedding vector space $\mathbb{R}^d$. "Same neighborhood" means the embeddings of a question and its answer have high cosine similarity.

---

## Main answer

**A. Because questions and answers have different structural and grammatical syntax, yet must map to the exact same neighborhood in latent space.**

```
Before contrastive training:

  Embedding space:
       Q: "What is the capital of France?"
       ●  ← question cluster (interrogative syntax)

                    (large gap)

       ● ← A: "Paris is the capital of France."
            answer cluster (declarative syntax)

  cosine(Q, A) = LOW (different syntax → different embeddings)


After contrastive training:

  Embedding space:
       Q: "What is the capital of France?"  ●●  A: "Paris is the capital..."
                                        (same neighborhood)

  cosine(Q, A) = HIGH (learned to match despite syntax difference)
```

```
Contrastive loss pushes and pulls:

  ●Q ─── PULL ───▶ ●A⁺ (correct answer)     → closer
  ●Q ─── PUSH ───▶ ●A⁻ (wrong answer)       → farther
```

$$\text{Goal: } \cos(\mathbf{v}_q, \mathbf{v}_{a^+}) \gg \cos(\mathbf{v}_q, \mathbf{v}_{a^-})$$

| Property | Standard Embedding | Contrastive-trained Q&A |
|---|---|---|
| Syntax sensitivity | High (Q ≠ A in style) | Low (Q ≈ A semantically) |
| Q-A proximity | Low (different structure) | **High (same neighborhood)** |
| Training signal | Generic text similarity | (question, answer) pairs |
| Use case | Document similarity | **RAG retrieval** |

---
---

# Quiz 8, Question 21 — Cross-Encoder Re-Ranking Code (Essay)

> **Coding: Cross-Encoder Re-Ranking.** You are writing the Python script for the re-ranking stage of a Two-Stage RAG pipeline. You have a `user_query` (string) and a list of 20 retrieved document objects called `initial_20_docs` (the text is accessed via `doc.page_content`). Assuming numpy is imported as np and the `cross_encoder` model is already loaded, fill in the blanks to successfully re-rank these documents.
>
> ```
> # 1. Format inputs as pairs of [Query, Document Text] for the model
> cross_encoder_inputs = __code1__
>
> # 2. Run the forward pass to get the relevance logits
> logits = __code2__
>
> # 3. Get the sorted indices in descending order (highest score first)
> sorted_indices = __code3__
> ```

---

## Sub-questions

### What format does a Cross-Encoder expect?

A Cross-Encoder takes pairs of texts as input:

$$\text{input} = [[\text{query}, \text{doc}_1], [\text{query}, \text{doc}_2], \dots, [\text{query}, \text{doc}_n]]$$

Each pair is concatenated internally as `[CLS] query [SEP] document [SEP]` and processed jointly.

### What does `cross_encoder.predict()` return?

A 1D array of relevance scores (logits), one per input pair. Higher score = more relevant.

### What does `np.argsort()` do?

$$\texttt{np.argsort(x)} \rightarrow \text{indices that would sort } x \text{ in ascending order}$$

To get **descending** order (highest first): `np.argsort(x)[::-1]`.

---

## Main answer

### code1: Format input pairs

```python
cross_encoder_inputs = [[user_query, doc.page_content] for doc in initial_20_docs]
```

Creates a list of 20 `[query, document_text]` pairs.

### code2: Get relevance logits

```python
logits = cross_encoder.predict(cross_encoder_inputs)
```

Runs all 20 pairs through the cross-encoder in a single batch. Returns an array of 20 scores.

### code3: Sort by descending score

```python
sorted_indices = np.argsort(logits)[::-1]
```

`argsort` returns indices for ascending order. `[::-1]` reverses to descending (highest relevance first).

```
Full pipeline:

  # 1. Build pairs
  cross_encoder_inputs = [[user_query, doc.page_content]
                           for doc in initial_20_docs]
  → [["What is X?", "Doc1 text..."],
     ["What is X?", "Doc2 text..."],
     ...,
     ["What is X?", "Doc20 text..."]]

  # 2. Score all pairs
  logits = cross_encoder.predict(cross_encoder_inputs)
  → [0.82, 0.15, 0.94, ..., 0.67]   (20 scores)

  # 3. Sort indices by descending score
  sorted_indices = np.argsort(logits)[::-1]
  → [2, 0, 19, ..., 1]               (index 2 had highest score)

  # Usage: re-ranked documents
  re_ranked_docs = [initial_20_docs[i] for i in sorted_indices]
```

| Step | Code | Purpose |
|---|---|---|
| 1 | `[[user_query, doc.page_content] for doc in initial_20_docs]` | Format query-doc pairs |
| 2 | `cross_encoder.predict(cross_encoder_inputs)` | Get relevance scores |
| 3 | `np.argsort(logits)[::-1]` | Descending sort indices |

---
---

# Quiz 8, Question 22 — Brute-Force k-NN Scaling (Essay)

> **Brute Force Operations:** A brute-force k-NN search is computationally expensive. If a database of 1 billion documents embedded in 1536 dimensions requires approximately 1.5 trillion floating-point operations per user query, exactly how many floating-point operations would be required for a single query if the database scaled up to 2 billion documents embedded in the exact same 1536 dimensions?

---

## Sub-questions

### What is the brute-force k-NN cost formula?

$$\text{FLOPS} = N \times d \times 2$$

For each of $N$ vectors, compute the distance in $d$ dimensions. Each dimension requires a subtraction + multiplication (or multiply + add), giving $\sim 2d$ operations per vector.

### How does cost scale with $N$?

$$\text{FLOPS} \propto N$$

Linear in the number of database vectors. Doubling $N$ doubles the computation.

---

## Main answer

$$\text{2 billion} = 2 \times \text{1 billion} \implies \text{FLOPS} = 2 \times 1.5 \text{ trillion} = 3.0 \text{ trillion}$$

$$\boxed{3{,}000{,}000{,}000{,}000 = 3 \times 10^{12} \text{ floating-point operations}}$$

```
Linear scaling:

  FLOPS
   ↑
  3.0T│               ●  (2B docs)
      │             ╱
  1.5T│      ●    ╱    (1B docs)
      │    ╱    ╱
      │  ╱    ╱
      │╱    ╱
      └──────────────▶ N (documents)
      0    1B       2B
```

| Database Size | Dimensions | FLOPS per Query |
|---|---|---|
| 1 billion | 1536 | 1.5 trillion ($1.5 \times 10^{12}$) |
| **2 billion** | **1536** | **3.0 trillion ($3.0 \times 10^{12}$)** |
| Scaling factor | $2\times$ | $2\times$ (linear) |

---
---

# Quiz 8, Question 23 — Two-Stage Retrieval Latency (Essay)

> **Two-Stage Retrieval Latency:** Assume your Bi-Encoder can search a database of 1,000,000 documents and retrieve the Top-50 candidates in exactly 10ms. Your Cross-Encoder takes exactly 25ms to perform deep cross-attention on a single query-document pair.
>
> A: Calculate the total time (in milliseconds) to run a single query through the entire Two-Stage pipeline (Bi-Encoder retrieval + Cross-Encoder re-ranking of the candidates).
>
> B: If you skipped the Bi-Encoder entirely and naively ran the Cross-Encoder on all 1,000,000 documents, how long would the query take in hours?

---

## Sub-questions

### What is Stage 1 cost?

$$t_\text{Bi-Encoder} = 10 \;\text{ms} \quad \text{(retrieve top-50 from 1M docs)}$$

Bi-Encoder uses pre-computed embeddings and ANN search. Very fast.

### What is Stage 2 cost?

$$t_\text{Cross-Encoder} = 50 \times 25 \;\text{ms} = 1{,}250 \;\text{ms}$$

Cross-Encoder processes each of the 50 candidates individually: 50 forward passes × 25ms each.

### What would Cross-Encoder on all docs cost?

$$t_\text{naive} = 1{,}000{,}000 \times 25 \;\text{ms}$$

Full forward pass for every document in the database.

---

## Main answer

### Part A: Two-Stage Pipeline

$$t_\text{total} = t_\text{Bi-Encoder} + t_\text{Cross-Encoder}$$

$$t_\text{total} = 10 \;\text{ms} + (50 \times 25 \;\text{ms}) = 10 + 1{,}250 = 1{,}260 \;\text{ms}$$

$$\boxed{1{,}260 \;\text{ms} \;(\approx 1.26 \;\text{seconds})}$$

### Part B: Cross-Encoder on All 1M Documents

$$t = 1{,}000{,}000 \times 25 \;\text{ms} = 25{,}000{,}000 \;\text{ms}$$

$$= 25{,}000 \;\text{s} = 416.67 \;\text{min} \approx 6.94 \;\text{hours}$$

$$\boxed{\approx 6.94 \;\text{hours}}$$

```
Latency comparison:

  Two-Stage:
  ┌──────────────────┐    ┌─────────────────────────┐
  │ Bi-Encoder       │───▶│ Cross-Encoder (50 docs) │───▶ Result
  │ 10ms             │    │ 50 × 25ms = 1,250ms     │
  └──────────────────┘    └─────────────────────────┘
  Total: 1,260ms (1.26 seconds)


  Naive Cross-Encoder only:
  ┌──────────────────────────────────────────────────┐
  │ Cross-Encoder (1,000,000 docs)                   │───▶ Result
  │ 1,000,000 × 25ms = 25,000,000ms                 │
  └──────────────────────────────────────────────────┘
  Total: 25,000,000ms (6.94 hours)
```

$$\text{Speedup} = \frac{25{,}000{,}000}{1{,}260} \approx 19{,}841\times$$

| Approach | Computation | Time | Practical? |
|---|---|---|---|
| **Two-Stage** | 10ms + 50×25ms | **1.26 seconds** | **Yes** |
| Naive Cross-Encoder | 1M × 25ms | **6.94 hours** | No |
| Speedup | — | **~19,841×** | — |

---
---

# Quiz 8, Question 24 — Applied Product Quantization (Essay)

> **Applied Product Quantization (PQ):** Your company uses massive 1536-dimensional vectors. A single uncompressed vector using float32 numbers takes up exactly 6,144 bytes of memory. You apply Product Quantization: split each 1536-dimensional vector into exactly 96 equal subvectors. For each subvector slot, you use K-Means to build a Codebook containing exactly 256 centroids.
>
> A: Calculate the final compressed memory size (in bytes) of this single vector.
> B: Calculate the compression ratio (Original Size / Compressed Size).

---

## Sub-questions

### What is the original vector size?

$$1536 \text{ dims} \times 4 \text{ bytes/float32} = 6{,}144 \text{ bytes}$$

### What is the subvector size?

$$\frac{1536}{96} = 16 \text{ dimensions per subvector}$$

### How is each subvector compressed?

Each subvector is replaced by its nearest centroid's ID. With 256 centroids:

$$\lceil \log_2(256) \rceil = 8 \text{ bits} = 1 \text{ byte per subvector}$$

---

## Main answer

### Part A: Compressed Memory Size

$$96 \text{ subvectors} \times 1 \text{ byte each} = 96 \text{ bytes}$$

### Part B: Compression Ratio

$$\text{Ratio} = \frac{6{,}144}{96} = 64\times \text{ compression}$$

```
Product Quantization breakdown:

  Original vector (1536 dims, float32):
  ┌────────────────────────────────────────────────────────┐
  │ [0.12, -0.45, 0.78, ..., 0.56]                        │
  │ 1536 × 4 bytes = 6,144 bytes                          │
  └────────────────────────────────────────────────────────┘

       │  Split into 96 subvectors of 16 dims
       ▼

  ┌────┐ ┌────┐ ┌────┐         ┌────┐
  │sub₁│ │sub₂│ │sub₃│  ...    │sub₉₆│  (96 subvectors)
  └──┬─┘ └──┬─┘ └──┬─┘         └──┬──┘
     │      │      │              │
     ▼      ▼      ▼              ▼    Quantize each to centroid ID
  [42]   [183]   [7]    ...    [201]   (1 byte each, 256 centroids)

  Compressed: 96 bytes
```

| Metric | Value |
|---|---|
| Original size | 6,144 bytes |
| Subvectors | 96 (16 dims each) |
| Centroids per codebook | 256 ($= 2^8$ → 1 byte per ID) |
| **Compressed size** | **96 bytes** |
| **Compression ratio** | **64×** |

---
---

# Quiz 8, Question 25 — Chunking Overlap Mathematics (Essay)

> **Chunking Overlap Mathematics:** You are writing a Python script to parse a corporate manual that contains exactly 3,200 tokens. To prevent context loss, you use a fixed-size chunking strategy with a `chunk_size` of 500 tokens and a `chunk_overlap` of 50 tokens. Calculate the total number of chunks this document will be divided into. Show your mathematical logic.

---

## Sub-questions

### What is the stride (step size)?

$$\text{stride} = \text{chunk\_size} - \text{chunk\_overlap} = 500 - 50 = 450 \text{ tokens}$$

Each new chunk starts 450 tokens after the previous chunk started. The 50-token overlap ensures no information is lost at boundaries.

### How does the first chunk work?

$$\text{Chunk 1: tokens } [0, 500)$$

Covers the first 500 tokens. Remaining tokens after chunk 1: $3200 - 500 = 2700$.

### How many additional chunks are needed?

$$\text{Additional chunks} = \left\lceil \frac{\text{remaining}}{\text{stride}} \right\rceil = \left\lceil \frac{2700}{450} \right\rceil = \left\lceil 6.0 \right\rceil = 6$$

---

## Main answer

### Step 1: Calculate stride

$$\text{stride} = 500 - 50 = 450$$

### Step 2: Count chunks

$$\text{Chunk 1: tokens } [0, 500)$$

$$\text{Remaining after chunk 1: } 3200 - 500 = 2700$$

$$\text{Additional chunks: } \frac{2700}{450} = 6.0 \text{ (exact)}$$

$$\text{Total chunks} = 1 + 6 = 7$$

```
Chunk layout (3200 tokens):

  Chunk 1: [  0 ... 499 ]                    (500 tokens)
  Chunk 2: [450 ... 949 ]   ← 50 overlap     (500 tokens)
  Chunk 3: [900 ... 1399]   ← 50 overlap     (500 tokens)
  Chunk 4: [1350 ... 1849]  ← 50 overlap     (500 tokens)
  Chunk 5: [1800 ... 2299]  ← 50 overlap     (500 tokens)
  Chunk 6: [2250 ... 2749]  ← 50 overlap     (500 tokens)
  Chunk 7: [2700 ... 3199]  ← 50 overlap     (500 tokens)

  Token: 0         500       1000      1500      2000      2500      3000  3200
         |──────────|──────────|──────────|──────────|──────────|──────────|──|
  Stride:     450       450       450       450       450       450
```

### Verification

$$\text{Last chunk starts at: } 0 + 6 \times 450 = 2700$$
$$\text{Last chunk ends at: } 2700 + 500 = 3200 \; \checkmark \; \text{(exactly covers the document)}$$

| Parameter | Value |
|---|---|
| Document length | 3,200 tokens |
| Chunk size | 500 tokens |
| Overlap | 50 tokens |
| Stride | $500 - 50 = 450$ tokens |
| First chunk | tokens $[0, 500)$ |
| Remaining | $3200 - 500 = 2700$ |
| Additional chunks | $2700 / 450 = 6$ |
| **Total chunks** | **$1 + 6 = 7$** |
