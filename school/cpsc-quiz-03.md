# Quiz 3: Embeddings and Positional Encoding — Tutor Answers (Q1–Q25)

---

# Quiz 3, Question 1 — Sliding Window Stride

> **In the context of a sliding window DataLoader, what is the primary consequence of setting the stride equal to the context_length (non-overlapping windows) compared to a stride of 1?**
>
> **C. It minimizes data redundancy but requires a larger dataset to achieve the same number of training updates.**

---

## Sub-questions

### What is a sliding window DataLoader?

A method that generates training samples from a token stream by sliding a fixed-size window across it.

$$\text{sample}_i = \text{tokens}[i \cdot S \;:\; i \cdot S + L]$$

$L$ = context length (window size), $S$ = stride (step size between consecutive windows).

### What is stride?

$$S = \text{number of tokens the window advances between samples}$$

$S = 1$: maximum overlap — each sample shares $L - 1$ tokens with the previous.
$S = L$: zero overlap — no token appears in two samples.

### How does stride affect dataset size?

$$N_\text{samples} = \left\lfloor \frac{T - L}{S} \right\rfloor + 1$$

$T$ = total tokens in corpus.

**Numeric example:** $T = 1000, L = 512$:
- $S = 1$: $N = 489$ samples
- $S = 512$: $N = 1$ sample (only 1 full window fits)

---

## Main answer

**C. It minimizes data redundancy but requires a larger dataset to achieve the same number of training updates.**

```
Stride = 1 (maximum overlap):

  tokens:  [A B C D E F G H I J ...]
  sample1: [A B C D E]
  sample2: [B C D E F]   ← 4 tokens shared
  sample3: [C D E F G]   ← highly redundant

Stride = L (no overlap):

  tokens:  [A B C D E F G H I J ...]
  sample1: [A B C D E]
  sample2: [F G H I J]   ← zero tokens shared
                          ← zero redundancy
```

$$\frac{N_{S=1}}{N_{S=L}} = \frac{T - L + 1}{\lfloor(T - L)/L\rfloor + 1} \approx L$$

Stride $= L$ produces roughly $L\times$ fewer samples. Each sample is unique (no redundancy), but you need a proportionally larger corpus to generate the same number of training updates.

| Property | Stride = 1 | Stride = L |
|---|---|---|
| Overlap | $L - 1$ tokens | 0 tokens |
| Redundancy | High | None |
| Samples from $T$ tokens | $\approx T$ | $\approx T / L$ |
| Needs larger dataset? | No | Yes |

---
---

# Quiz 3, Question 2 — Shifted Target Computational Complexity

> **The "Shifted Target" strategy allows a Transformer to calculate the loss for N tokens in a single forward pass. Without this parallelization (i.e., using a naive sequential approach), what would be the computational complexity for training a sequence of length N?**
>
> **C. O(N^2) (Quadratic)**

---

## Sub-questions

### What is the Shifted Target strategy?

One forward pass processes all $N$ tokens simultaneously. The model predicts token $t+1$ at each position $t$ in parallel. Loss is computed for all $N-1$ predictions at once.

$$\text{Cost}_\text{shifted} = O(N) \quad \text{(one forward pass)}$$

### What is the naive sequential approach?

Without parallelization, predicting each token requires a separate forward pass over all preceding tokens:

- Predict token 2: forward pass on 1 token
- Predict token 3: forward pass on 2 tokens
- ...
- Predict token $N$: forward pass on $N-1$ tokens

### What is the total cost?

$$\text{Total} = \sum_{t=1}^{N-1} t = \frac{N(N-1)}{2} = O(N^2)$$

**Numeric example:** $N = 1024$ → $\frac{1024 \times 1023}{2} = 523{,}776$ forward-pass token operations vs. $1024$ with the shifted strategy.

---

## Main answer

**C. O(N^2) (Quadratic)**

```
Naive sequential approach:

  Step 1:  Process [tok₁]             → predict tok₂       (1 token)
  Step 2:  Process [tok₁, tok₂]       → predict tok₃       (2 tokens)
  Step 3:  Process [tok₁, tok₂, tok₃]  → predict tok₄       (3 tokens)
  ...
  Step N-1: Process [tok₁ ... tok_{N-1}] → predict tok_N     (N-1 tokens)

  Total = 1 + 2 + 3 + ... + (N-1) = N(N-1)/2 = O(N²)
```

```
Comparison:

  Shifted Target:  1 forward pass → N predictions → O(N)
  Naive Sequential: N forward passes → N predictions → O(N²)
```

$$\frac{O(N^2)}{O(N)} = O(N) \quad \text{speedup from parallelization}$$

| Approach | Forward Passes | Total Tokens Processed | Complexity |
|---|---|---|---|
| Shifted Target | 1 | $N$ | $O(N)$ |
| Naive Sequential | $N - 1$ | $\frac{N(N-1)}{2}$ | $O(N^2)$ |

---
---

# Quiz 3, Question 3 — Polysemous Words in Static Embeddings

> **In a static embedding space, how is a polysemous word like "Bank" (river vs. finance) represented geometrically?**
>
> **B. It is represented as the weighted average of its meanings, often placing the vector in a semantic "no-man's land" with shorter magnitude.**

---

## Sub-questions

### What is a static embedding?

$$\text{Embed}: w \rightarrow \mathbf{v}_w \in \mathbb{R}^d \quad \text{(one fixed vector per word)}$$

Every occurrence of a word maps to the **same** vector regardless of context. Word2Vec, GloVe, and FastText produce static embeddings.

### What is polysemy?

A single word form with multiple distinct meanings:
- "Bank" → financial institution **or** river bank
- "Bat" → flying mammal **or** baseball bat

### What happens when training averages multiple contexts?

The training objective (e.g., predicting surrounding words) sees "bank" in both financial and river contexts. The gradient pulls the vector toward **both** meaning clusters simultaneously.

$$\mathbf{v}_\text{bank} \approx \alpha \cdot \mathbf{v}_\text{finance} + (1-\alpha) \cdot \mathbf{v}_\text{river}$$

$\alpha$ = relative frequency of each meaning in the training corpus.

---

## Main answer

**B. The vector sits at the weighted average of its meanings — a semantic "no-man's land" with shorter magnitude.**

```
2D projection of embedding space:

            finance cluster
               ●  ●
              ●  ●  ●
                    ○ ← "bank" (averaged)
                  ↗   ↖
        weighted     weighted
        pull         pull

              ●  ●
             ●  ●  ●
            river cluster
```

$$\|\mathbf{v}_\text{bank}\| < \|\mathbf{v}_\text{finance}\| \quad \text{and} \quad \|\mathbf{v}_\text{bank}\| < \|\mathbf{v}_\text{river}\|$$

The averaged vector has shorter magnitude because the two meaning directions partially cancel. It sits between the clusters — close to neither meaning, accurate for neither context.

```
Why shorter magnitude:

  v_finance = [0.8, 0.2]     (points toward finance cluster)
  v_river   = [-0.6, 0.7]   (points toward river cluster)
  v_bank    = [0.1, 0.45]   (average — short, between both)

  ‖v_bank‖ = 0.46  <  ‖v_finance‖ = 0.82  <  ‖v_river‖ = 0.92
```

| Property | Static Embedding | Contextual Embedding (BERT/GPT) |
|---|---|---|
| Vectors per word | 1 (fixed) | Different per context |
| Polysemy handling | Averaged — "no-man's land" | Distinct vectors per meaning |
| "Bank" (finance) | Same vector | Unique vector near finance cluster |
| "Bank" (river) | Same vector | Unique vector near river cluster |

---
---

# Quiz 3, Question 4 — Anisotropy (Cone Effect)

> **High-dimensional embedding spaces often suffer from "Anisotropy" (the Cone Effect). What is the primary cause of this phenomenon?**
>
> **C. Frequency domination, where common words (e.g., "the", "is") pull all vectors toward a common mean direction.**

---

## Sub-questions

### What is anisotropy in embedding spaces?

$$\cos(\mathbf{v}_i, \mathbf{v}_j) \approx \text{high positive value} \quad \forall \; i, j$$

Vectors cluster into a narrow cone instead of spreading uniformly across all directions. Most pairs of vectors have high cosine similarity — even unrelated words.

### What is the Cone Effect?

All word vectors point in roughly the same direction, forming a cone. The angular spread is much smaller than expected in a high-dimensional space.

### Why do frequent words cause this?

High-frequency words ("the", "is", "a") appear in virtually every context. Their gradient updates dominate training, pulling all vectors toward a shared mean direction:

$$\hat{\mu} = \frac{1}{|V|}\sum_{w \in V} \mathbf{v}_w \neq \mathbf{0}$$

The mean $\hat{\mu}$ is far from the origin. Every vector has a large component along $\hat{\mu}$, reducing effective angular diversity.

---

## Main answer

**C. Frequency domination — common words pull all vectors toward a common mean direction.**

```
Isotropic (ideal):                    Anisotropic (Cone Effect):

         ↗  ↑  ↖                              ↗ ↗ ↑
        ←       →                             ↗ ↗ ↑
         ↙  ↓  ↘                             ↗ ↗ ↗

  Vectors spread                      All vectors point
  uniformly in all                    in similar direction
  directions                          (narrow cone)
```

$$\text{Expected } \cos(\mathbf{v}_i, \mathbf{v}_j) \approx 0 \quad \text{(isotropic, high-d)}$$
$$\text{Observed } \cos(\mathbf{v}_i, \mathbf{v}_j) \approx 0.5\text{–}0.9 \quad \text{(anisotropic)}$$

```
Cause chain:

  High-freq words ──▶ Dominate gradients ──▶ Shared mean direction μ ≠ 0
       │                                           │
       ▼                                           ▼
  All vectors acquire ──▶ Narrow cone ──▶ Cosine similarity inflated
  large component along μ                  (even for unrelated words)
```

| Property | Isotropic | Anisotropic (Cone) |
|---|---|---|
| Mean vector $\hat{\mu}$ | $\approx \mathbf{0}$ | Far from $\mathbf{0}$ |
| Cosine similarity (random pair) | $\approx 0$ | $\gg 0$ |
| Angular spread | Full sphere | Narrow cone |
| Cause | — | High-frequency word domination |
| Fix | — | Mean-centering, Layer Normalization |

---
---

# Quiz 3, Question 5 — Layer Normalization Geometric Effect

> **What is the geometric effect of applying Layer Normalization to a set of embedding vectors?**
>
> **A. It projects all vectors onto the surface of a hypersphere (unit length), forcing the model to rely on direction rather than magnitude.**

---

## Sub-questions

### What is Layer Normalization?

$$\text{LayerNorm}(\mathbf{x}) = \gamma \cdot \frac{\mathbf{x} - \mu}{\sigma + \epsilon} + \beta$$

$$\mu = \frac{1}{d}\sum_{i=1}^{d} x_i, \qquad \sigma = \sqrt{\frac{1}{d}\sum_{i=1}^{d}(x_i - \mu)^2}$$

Centers the vector (subtract mean), scales to unit variance (divide by standard deviation). $\gamma, \beta$ are learnable scale/shift parameters.

### What is a hypersphere?

$$S^{d-1} = \{\mathbf{x} \in \mathbb{R}^d : \|\mathbf{x}\| = r\}$$

The set of all points at distance $r$ from the origin in $d$-dimensional space. When $r = 1$, this is the unit hypersphere.

### What does "direction over magnitude" mean?

After normalization, all vectors have roughly the same norm. The only distinguishing feature is their **direction** (angle), not their length.

$$\|\text{LayerNorm}(\mathbf{x})\| \approx \|\gamma\| \quad \forall \; \mathbf{x}$$

---

## Main answer

**A. Layer Normalization projects vectors onto a hypersphere, forcing reliance on direction.**

```
Before LayerNorm:                After LayerNorm:

  Vectors have                    All vectors have
  varying lengths:                equal length (on hypersphere):

        ↗ (long)                       ↗ (unit)
    →   (medium)                   →   (unit)
      ↘ (short)                      ↘ (unit)

  Information in both             Information only
  direction AND magnitude         in direction (angle)
```

$$\text{Before: } \|\mathbf{v}_1\| = 3.2, \; \|\mathbf{v}_2\| = 0.8, \; \|\mathbf{v}_3\| = 5.1$$
$$\text{After: } \|\mathbf{v}_1'\| \approx \|\mathbf{v}_2'\| \approx \|\mathbf{v}_3'\| \approx \|\gamma\|$$

This also helps combat anisotropy — mean-centering pulls vectors away from the shared cone direction, and variance normalization ensures uniform spread.

| Property | Before LayerNorm | After LayerNorm |
|---|---|---|
| Magnitude | Varies widely | Approximately equal |
| Geometry | Scattered in volume | Projected onto hypersphere surface |
| Semantic info | Direction + magnitude | Direction only |
| Anisotropy | Present (cone) | Reduced (mean-centered) |

---
---

# Quiz 3, Question 6 — Blessing of Dimensionality

> **According to the "Blessing of Dimensionality," what is the expected angle between two randomly initialized vectors in a 768-dimensional space?**
>
> **C. Approximately 90 degrees (Orthogonal)**

---

## Sub-questions

### What is the dot product and cosine similarity?

$$\cos\theta = \frac{\mathbf{a} \cdot \mathbf{b}}{\|\mathbf{a}\| \|\mathbf{b}\|}$$

Two random vectors are orthogonal when $\cos\theta = 0$, i.e., $\theta = 90°$.

### Why does high dimensionality push toward orthogonality?

For random vectors $\mathbf{a}, \mathbf{b} \in \mathbb{R}^d$ with i.i.d. components:

$$\mathbb{E}[\cos\theta] = 0$$

$$\text{Var}[\cos\theta] \approx \frac{1}{d}$$

$$\text{Std}[\cos\theta] \approx \frac{1}{\sqrt{d}}$$

**Numeric example:** $d = 768$ → $\text{Std} \approx \frac{1}{\sqrt{768}} \approx 0.036$. The cosine similarity concentrates tightly around 0.

---

## Main answer

**C. Approximately 90 degrees (Orthogonal)**

$$\theta \approx 90° \pm \frac{1}{\sqrt{d}} \text{ radians} \approx 90° \pm 2.1°$$

```
Concentration of angles as d increases:

  d = 2:     Any angle 0°–180° equally likely
  d = 10:    Angles spread around 90° (wide)
  d = 100:   Angles cluster near 90° (±6°)
  d = 768:   Angles tightly concentrated at 90° (±2°)
  d = ∞:     All angles exactly 90°
```

This is the "Blessing of Dimensionality" for embeddings: in $d = 768$, random vectors are nearly orthogonal. This means a 768-dimensional space can accommodate a vast number of approximately orthogonal directions — each word gets its own "slot" without interfering with others.

$$\text{Max near-orthogonal vectors} \gg d \quad \text{(exponential in } d \text{)}$$

| Dimension $d$ | Std of $\cos\theta$ | Angular spread around 90° | Near-orthogonal capacity |
|---|---|---|---|
| 2 | 0.707 | $\pm 45°$ | Very few |
| 100 | 0.100 | $\pm 5.7°$ | Moderate |
| 768 | 0.036 | $\pm 2.1°$ | Very high |
| 4096 | 0.016 | $\pm 0.9°$ | Enormous |

---
---

# Quiz 3, Question 7 — Self-Attention Permutation Invariance

> **Why is the Self-Attention mechanism mathematically defined as "Permutation Invariant"?**
>
> **B. Because the output for a given token depends only on the set of input values, not their order in the sequence.**

---

## Sub-questions

### What is permutation invariance?

$$f(\pi(\mathbf{X})) = \pi(f(\mathbf{X}))$$

Reordering the input rows produces the same reordering of the output rows. The function treats the inputs as a **set**, not a sequence.

### How does self-attention compute outputs?

$$\text{Attention}(Q, K, V) = \text{Softmax}\!\left(\frac{QK^T}{\sqrt{d_k}}\right) V$$

For token $i$: $\text{output}_i = \sum_j \alpha_{ij} \mathbf{v}_j$, where $\alpha_{ij} = \text{Softmax}_j\!\left(\frac{\mathbf{q}_i \cdot \mathbf{k}_j}{\sqrt{d_k}}\right)$.

The attention weight $\alpha_{ij}$ depends only on the **content** of tokens $i$ and $j$ (their Q, K values) — not on their position indices.

### Why is this a problem for language?

"The cat sat on the mat" and "mat the on sat cat the" would produce **identical** attention patterns and outputs. Word order carries meaning in language, but self-attention ignores it.

---

## Main answer

**B. The output for a given token depends only on the set of input values, not their order.**

```
Permutation invariance demonstrated:

  Input A: [The, cat, sat]  →  Attention → output_A
  Input B: [sat, The, cat]  →  Attention → output_B

  output_B = permuted(output_A)
  (same values, just reordered to match the new input order)

  Self-Attention cannot distinguish A from B!
```

$$\alpha_{ij} = \frac{e^{\mathbf{q}_i \cdot \mathbf{k}_j / \sqrt{d_k}}}{\sum_m e^{\mathbf{q}_i \cdot \mathbf{k}_m / \sqrt{d_k}}}$$

No position index appears anywhere in this formula. The score depends purely on $\mathbf{q}_i$ and $\mathbf{k}_j$ content.

```
Why Positional Encodings are needed:

  Self-Attention alone ──▶ Permutation Invariant ──▶ No word order
                                                        │
  + Positional Encoding ──▶ Breaks invariance    ──▶ Order-aware
```

| Property | Self-Attention (alone) | Self-Attention + Pos. Encoding |
|---|---|---|
| Order-aware | No | Yes |
| Permutation invariant | Yes | No (broken by PE) |
| "cat sat" vs "sat cat" | Same output | Different output |

---
---

# Quiz 3, Question 8 — Learned vs. Sinusoidal Positional Embeddings

> **Which of the following is a specific disadvantage of Learned Positional Embeddings compared to Sinusoidal ones?**
>
> **B. They cannot extrapolate to sequence lengths longer than what was seen during training (e.g., the "1024 limit").**

---

## Sub-questions

### What are Learned Positional Embeddings?

$$\mathbf{P} \in \mathbb{R}^{L_{\max} \times d}$$

A trainable parameter matrix where each row is a learned vector for a specific position. Row $i$ = embedding for position $i$.

### What are Sinusoidal Positional Encodings?

$$PE_{(pos, 2i)} = \sin\!\left(\frac{pos}{10000^{2i/d}}\right), \quad PE_{(pos, 2i+1)} = \cos\!\left(\frac{pos}{10000^{2i/d}}\right)$$

Fixed, deterministic formula. Can generate an encoding for **any** position — no upper limit.

### What is extrapolation?

$$\text{Extrapolation: using the model at position } pos > L_{\max}$$

If trained with $L_{\max} = 1024$ and inference requires position 2000, the model must represent position 2000.

---

## Main answer

**B. Learned embeddings cannot extrapolate beyond the maximum training sequence length.**

```
Learned PE at inference:

  Position:  0    1    2   ...  1023  1024  1025
  Embedding: v₀   v₁   v₂  ... v₁₀₂₃  ???   ???
                                        ↑
                                   No learned vector exists!
                                   (IndexError or garbage)

Sinusoidal PE at inference:

  Position:  0    1    2   ...  1023  1024  1025  ...  ∞
  Encoding:  sin/cos formula produces valid vector for ANY position ✓
```

Learned embeddings are stored as a lookup table of size $L_{\max} \times d$. Positions beyond $L_{\max}$ have no entry — the model has never seen them and cannot generalize.

| Property | Learned PE | Sinusoidal PE |
|---|---|---|
| Representation | Trainable matrix $\mathbb{R}^{L_{\max} \times d}$ | Fixed formula |
| Max position | $L_{\max}$ (hard limit) | Unlimited |
| Extrapolation | Cannot | Can |
| Trainable parameters | $L_{\max} \times d$ | 0 |
| Performance at trained lengths | Slightly better (adapted to data) | Slightly worse |

---
---

# Quiz 3, Question 9 — Linear Property of Sinusoidal Encodings

> **The "Linear Property" of Sinusoidal Encodings allows the model to shift attention by a relative offset k. Mathematically, this is achieved because the encoding for pos+k can be derived from pos via:**
>
> **B. A fixed rotation matrix that depends only on k.**

---

## Sub-questions

### What is the Linear Property?

$$PE_{pos+k} = M_k \cdot PE_{pos}$$

The encoding for position $pos + k$ can be obtained by multiplying the encoding for position $pos$ by a **fixed matrix** $M_k$ that depends only on the offset $k$, not on $pos$.

### What is a rotation matrix?

$$M_\theta = \begin{pmatrix} \cos\theta & -\sin\theta \\ \sin\theta & \cos\theta \end{pmatrix}$$

Rotates a 2D vector by angle $\theta$. Preserves the vector's length. In sinusoidal PE, each pair of dimensions $(2i, 2i+1)$ has its own rotation frequency.

### How does the rotation work for sinusoidal PE?

For dimension pair $(2i, 2i+1)$ with frequency $\omega_i = \frac{1}{10000^{2i/d}}$:

$$\begin{pmatrix} \sin(\omega_i(pos+k)) \\ \cos(\omega_i(pos+k)) \end{pmatrix} = \begin{pmatrix} \cos(\omega_i k) & \sin(\omega_i k) \\ -\sin(\omega_i k) & \cos(\omega_i k) \end{pmatrix} \begin{pmatrix} \sin(\omega_i \cdot pos) \\ \cos(\omega_i \cdot pos) \end{pmatrix}$$

This follows directly from the angle addition identities: $\sin(A+B) = \sin A \cos B + \cos A \sin B$.

---

## Main answer

**B. A fixed rotation matrix that depends only on k.**

```
Linear Property:

  PE(pos) ──▶ × M_k ──▶ PE(pos + k)

  M_k is fixed for a given offset k
  M_k does NOT depend on pos
  The model can learn M_k to attend "k positions ahead/behind"
```

$$M_k = \text{block-diag}\!\left(R(\omega_1 k), \; R(\omega_2 k), \; \dots, \; R(\omega_{d/2} k)\right)$$

Each 2×2 block is a rotation by angle $\omega_i k$:

$$R(\omega_i k) = \begin{pmatrix} \cos(\omega_i k) & \sin(\omega_i k) \\ -\sin(\omega_i k) & \cos(\omega_i k) \end{pmatrix}$$

**Numeric example:** To shift by $k = 3$ positions, the same matrix $M_3$ works regardless of whether $pos = 0, 50,$ or $999$.

| Property | Value |
|---|---|
| Transformation type | Rotation (per dimension pair) |
| Depends on $k$ | Yes |
| Depends on $pos$ | No |
| Preserves vector norm | Yes (rotation preserves length) |
| Enables relative attention | Yes (model learns $M_k$) |

---
---

# Quiz 3, Question 10 — Geometric Progression of Wavelengths

> **Why does the Sinusoidal formula use a geometric progression of wavelengths (from $2\pi$ to $10000 \cdot 2\pi$) across dimensions?**
>
> **A. To ensure that low dimensions capture fine-grained positional changes while high dimensions capture global position, preventing collisions.**

---

## Sub-questions

### What is the wavelength for each dimension?

$$\lambda_i = 2\pi \cdot 10000^{2i/d}$$

Dimension $i = 0$: $\lambda_0 = 2\pi$ (shortest wavelength, fastest oscillation).
Dimension $i = d/2 - 1$: $\lambda = 2\pi \cdot 10000$ (longest wavelength, slowest oscillation).

### What is a geometric progression?

$$\lambda_0, \; \lambda_0 \cdot r, \; \lambda_0 \cdot r^2, \; \dots$$

Each wavelength is a constant multiple of the previous. The ratio $r = 10000^{2/d}$.

**Numeric example ($d = 512$):**

$$r = 10000^{2/512} = 10000^{0.0039} \approx 1.018$$

### What is a "collision"?

$$PE_{pos_a} = PE_{pos_b} \quad \text{where } pos_a \neq pos_b$$

Two different positions produce identical encoding vectors. If all dimensions had the same wavelength, the encoding would repeat every $\lambda$ positions.

---

## Main answer

**A. Low dimensions = fine-grained; high dimensions = global. This multi-scale encoding prevents collisions.**

```
Wavelength progression across dimensions:

  Dim 0,1:     λ = 2π ≈ 6.28          (repeats every ~6 positions)
  Dim 2,3:     λ = 2π · 10000^(2/d)   (slightly slower)
  ...
  Dim d-2,d-1: λ = 2π · 10000         (repeats every ~62,832 positions)

  Low dims:   ∿∿∿∿∿∿∿∿  fast oscillation → detects nearby positions
  High dims:  ∿         slow oscillation → detects far-apart positions
```

This is analogous to a **clock**: the second hand (low dims) detects fine differences, the hour hand (high dims) detects coarse differences. Together, they uniquely identify any time.

```
Analogy — binary odometer:

  Bit 0 (fast):  0 1 0 1 0 1 0 1   ← changes every position
  Bit 1:         0 0 1 1 0 0 1 1   ← changes every 2 positions
  Bit 2:         0 0 0 0 1 1 1 1   ← changes every 4 positions
  Bit 3 (slow):  0 0 0 0 0 0 0 0   ← changes every 8 positions

  Sinusoidal PE works the same way but with smooth sin/cos waves
```

| Dimension Range | Wavelength | Captures | Analogy |
|---|---|---|---|
| Low (0, 1, ...) | Short ($\approx 2\pi$) | Fine-grained (adjacent positions) | Second hand |
| Mid | Medium | Medium-range patterns | Minute hand |
| High (..., $d-1$) | Long ($\approx 2\pi \cdot 10000$) | Global position | Hour hand |
| Combined | Multi-scale | Unique encoding per position | Full clock |

---
---

# Quiz 3, Question 11 — Subword Tokenization Benefit

> **Modern LLMs use subword tokenization (like BPE) rather than whole-word vocabularies. What is the primary architectural benefit of this?**
>
> **B. It allows the model to construct the meaning of rare or unknown words from common fragments (compositionality).**

---

## Sub-questions

### What is whole-word tokenization?

$$\text{vocab} = \{\text{cat}, \text{dog}, \text{running}, \text{unforgettable}, \dots\}$$

Each unique word gets its own token. Unknown words → `<UNK>` token (total information loss).

### What is subword tokenization (BPE)?

$$\text{"unforgettable"} \rightarrow [\text{"un"}, \text{"forget"}, \text{"table"}]$$

Words are split into frequent subword units. Rare words are composed from common pieces. The model never sees `<UNK>`.

### What is compositionality?

$$\text{meaning}(\text{"un"} + \text{"forget"} + \text{"table"}) \approx \text{meaning}(\text{"unforgettable"})$$

The model learns that "un-" = negation, "-able" = capability. It can generalize to novel combinations like "unbreakable" even if that exact word was rare in training.

---

## Main answer

**B. Subword tokenization enables compositionality — constructing meaning of rare/unknown words from common fragments.**

```
Whole-word vs. Subword:

  Whole-word:  "unforgettable" → [unforgettable]  (must be in vocab)
               "ungooglable"   → [<UNK>]          (not in vocab — lost!)

  Subword:     "unforgettable" → ["un", "forget", "table"]
               "ungooglable"   → ["un", "goo", "gl", "able"]  ✓
```

$$V_\text{whole-word} \sim 100{,}000\text{+} \quad \text{vs.} \quad V_\text{subword} \sim 32{,}000$$

Smaller vocabulary → smaller embedding matrix $W_E \in \mathbb{R}^{V \times d}$ → fewer parameters.

| Property | Whole-word | Subword (BPE) |
|---|---|---|
| Vocabulary size | Very large ($>100$K) | Moderate ($\sim 32$K) |
| Unknown words | `<UNK>` (lost) | Composed from fragments |
| Compositionality | No | Yes ("un" + "break" + "able") |
| Embedding matrix size | $V_\text{large} \times d$ | $V_\text{small} \times d$ |

---
---

# Quiz 3, Question 12 — DataLoader num_workers

> **In the PyTorch DataLoader, what is the function of the num_workers parameter?**
>
> **B. It controls the number of CPU subprocesses used to pre-load and tokenize data in parallel to prevent GPU starvation.**

---

## Sub-questions

### What is the DataLoader?

PyTorch utility that feeds batches of data to the model. Handles batching, shuffling, and parallelized data loading.

### What is GPU starvation?

The GPU sits idle waiting for the next batch because CPU-side data loading (reading files, tokenizing, preprocessing) is too slow. The GPU's compute capacity is wasted.

### What does num_workers do?

$$\texttt{num\_workers} = n \implies n \text{ CPU subprocesses load data in parallel}$$

$n = 0$: data loading runs in the main process (serial). $n > 0$: $n$ worker processes prefetch upcoming batches while the GPU trains on the current batch.

---

## Main answer

**B. num_workers controls CPU subprocesses for parallel data prefetching to prevent GPU starvation.**

```
num_workers = 0 (serial):

  CPU: [load batch 1] ──▶ [load batch 2] ──▶ [load batch 3]
  GPU:                [train 1]          [train 2]          [train 3]
                              ↑ idle           ↑ idle

num_workers = 4 (parallel):

  Worker 1: [load batch 2] [load batch 6] ...
  Worker 2: [load batch 3] [load batch 7] ...
  Worker 3: [load batch 4] [load batch 8] ...
  Worker 4: [load batch 5] [load batch 9] ...
  GPU:      [train 1][train 2][train 3][train 4] ...  (no idle time)
```

| num_workers | Behavior | GPU utilization |
|---|---|---|
| 0 | Serial loading in main process | Low (frequent stalls) |
| 1–4 | Moderate parallelism | Good |
| 4–8+ | Full pipeline parallelism | High (GPU never starved) |

---
---

# Quiz 3, Question 13 — Manifold Hypothesis

> **The "Manifold Hypothesis" states that valid, meaningful language occupies what region of the high-dimensional embedding space?**
>
> **B. A tiny, low-dimensional structure twisting through the mostly empty high-dimensional void.**

---

## Sub-questions

### What is a manifold?

A smooth, low-dimensional surface embedded in a higher-dimensional space. A sheet of paper (2D manifold) curving through 3D space.

$$\dim(\text{manifold}) \ll \dim(\text{ambient space})$$

### What does "mostly empty" mean?

In $\mathbb{R}^{768}$, the overwhelming majority of possible vectors correspond to meaningless gibberish. Valid language embeddings occupy a vanishingly small fraction of the total volume.

$$\frac{\text{Volume}(\text{language manifold})}{\text{Volume}(\mathbb{R}^{768})} \approx 0$$

---

## Main answer

**B. A tiny, low-dimensional structure (manifold) twisting through the mostly empty high-dimensional void.**

```
Visualization (3D ambient space, 1D manifold):

              · · · · · · · · · · · ·    ← empty space (meaningless vectors)
           · · · ● ─ ● ─ ● · · · ·
          · · ·/· · · · · ·\· · · ·     ← curve = language manifold
         · · ● · · · · · · · ● · · ·
          · · \· · · · · · ·/ · · ·
           · · · ● ─ ● ─ ● · · · ·
              · · · · · · · · · · · ·

  Most of the space is void.
  Valid language lives on the thin curve.
```

The model learns to map all meaningful text onto this manifold. Points on the manifold = valid language. Points off the manifold = nonsense.

| Concept | Description |
|---|---|
| Ambient dimension | 768 (or $d_\text{model}$) |
| Manifold dimension | Much smaller (unknown, but $\ll 768$) |
| On-manifold points | Valid, meaningful language |
| Off-manifold points | Meaningless / incoherent |

---
---

# Quiz 3, Question 14 — High-Dimensional Volume Distribution

> **In high-dimensional geometry (e.g., a hypercube), where is the vast majority of the volume located relative to the center?**
>
> **C. Pushed into the "spiky corners" (spines) far away from the center.**

---

## Sub-questions

### What is a hypercube?

A $d$-dimensional cube. In $\mathbb{R}^d$, the unit hypercube spans $[-0.5, 0.5]^d$.

### What is the distance from center to face vs. corner?

$$d_\text{face} = 0.5 \quad \text{(always, regardless of dimension)}$$

$$d_\text{corner} = \sqrt{d \cdot 0.5^2} = \frac{\sqrt{d}}{2}$$

**Numeric example ($d = 100$):**
- Distance to face: $0.5$
- Distance to corner: $\frac{\sqrt{100}}{2} = 5.0$

The corner is $10\times$ farther than the nearest face.

### Why does volume concentrate near corners?

$$\frac{V_\text{inscribed sphere}}{V_\text{hypercube}} \rightarrow 0 \quad \text{as } d \rightarrow \infty$$

The inscribed sphere (all points within $0.5$ of center) occupies a vanishing fraction of the cube. Almost all volume is in the regions near corners, far from center.

---

## Main answer

**C. Volume concentrates in the spiky corners, far from the center.**

$$d_\text{corner} = \frac{\sqrt{d}}{2} \quad \text{vs.} \quad d_\text{face} = 0.5$$

$$\text{Ratio} = \frac{d_\text{corner}}{d_\text{face}} = \sqrt{d}$$

```
Volume distribution shift with dimension:

  d = 2:    Most volume near center (familiar circle-in-square)
  d = 10:   Volume migrating outward
  d = 100:  Virtually all volume in corners
  d = 768:  Center is essentially empty
```

**Numeric example ($d = 100$):**

$$d_\text{corner} = \frac{\sqrt{100}}{2} = 5.0, \quad d_\text{face} = 0.5$$

$$\text{Corner is } \frac{5.0}{0.5} = 10\times \text{ farther than the face}$$

| Dimension $d$ | $d_\text{face}$ | $d_\text{corner}$ | Ratio | Volume near center |
|---|---|---|---|---|
| 2 | 0.5 | 0.71 | 1.4 | Most |
| 10 | 0.5 | 1.58 | 3.2 | Some |
| 100 | 0.5 | 5.0 | 10 | Almost none |
| 768 | 0.5 | 13.9 | 27.7 | Essentially zero |

---
---

# Quiz 3, Question 15 — Combining Positional Encodings with Token Embeddings

> **How are Positional Encodings (P) physically combined with Token Embeddings (E) in the standard Transformer architecture?**
>
> **C. They are summed element-wise (E + P).**

---

## Sub-questions

### What is a Token Embedding?

$$\mathbf{e}_w = W_E[w] \in \mathbb{R}^d$$

Lookup in the embedding matrix $W_E \in \mathbb{R}^{V \times d}$. Encodes the **identity** (meaning) of the token.

### What is a Positional Encoding?

$$\mathbf{p}_{pos} \in \mathbb{R}^d$$

A vector encoding the **position** of the token in the sequence. Same dimensionality $d$ as the token embedding.

### Why element-wise addition (not concatenation)?

Addition preserves the dimensionality at $d$. Concatenation would double it to $2d$, increasing all downstream computation costs.

$$\text{Addition: } \mathbb{R}^d + \mathbb{R}^d = \mathbb{R}^d$$
$$\text{Concatenation: } [\mathbb{R}^d; \mathbb{R}^d] = \mathbb{R}^{2d}$$

---

## Main answer

**C. Element-wise addition: $\mathbf{x} = \mathbf{e}_w + \mathbf{p}_{pos}$**

```
Combining token and positional information:

  Token Embedding:     [0.2, -0.5,  0.8, ..., 0.1]    (what)
                        +      +     +          +
  Positional Encoding: [0.0,  0.3, -0.1, ..., 0.4]    (where)
                        =      =     =          =
  Input to Transformer:[0.2, -0.2,  0.7, ..., 0.5]    (what + where)
```

$$\mathbf{x}_{pos,w} = \underbrace{W_E[w]}_{\text{identity}} + \underbrace{PE(pos)}_{\text{position}}$$

| Combination Method | Output Dim | Compute Cost | Used in Practice |
|---|---|---|---|
| Addition ($E + P$) | $d$ | No extra cost | Yes (standard) |
| Concatenation ($[E; P]$) | $2d$ | Doubles downstream cost | No |

---
---

# Quiz 3, Question 16 — Sinusoidal Encoding Repetition Misconception

> **A common misconception is that Sinusoidal Encodings repeat after position 10,000 because of the denominator term. Why is this false?**
>
> **B. Because the wavelength formula includes $2\pi$, meaning the cycle actually repeats at approx position 62,832.**

---

## Sub-questions

### What is the sinusoidal encoding formula?

$$PE_{(pos, 2i)} = \sin\!\left(\frac{pos}{10000^{2i/d}}\right)$$

The argument of the sine function is $\frac{pos}{10000^{2i/d}}$.

### When does a sine wave repeat?

$$\sin(\theta) \text{ repeats when } \theta \text{ increases by } 2\pi$$

### What is the wavelength of the slowest dimension?

The slowest dimension ($i = d/2 - 1$) has denominator $\approx 10000$:

$$\sin\!\left(\frac{pos}{10000}\right) \text{ repeats when } \frac{pos}{10000} = 2\pi$$

$$pos = 10000 \times 2\pi \approx 62{,}832$$

---

## Main answer

**B. The full cycle includes the $2\pi$ factor, so repetition occurs at $\approx 62{,}832$, not 10,000.**

$$\lambda_\text{max} = 2\pi \times 10000 \approx 62{,}832 \text{ positions}$$

```
Common misconception vs. reality:

  Misconception:  sin(pos/10000) repeats at pos = 10,000  ✗
                  (confuses denominator with wavelength)

  Reality:        sin(pos/10000) repeats when pos/10000 = 2π
                  pos = 10000 × 2π ≈ 62,832               ✓
```

Even this is only for the **slowest single dimension**. The full encoding vector combines $d/2$ different frequencies. Their combined repetition period (LCM) is effectively infinite (see Q17 — Odometer Effect).

| Claim | Position | Correct? |
|---|---|---|
| Repeats at 10,000 | $10000^{2i/d}$ = denominator | No (missing $2\pi$) |
| Repeats at $\approx 62{,}832$ | $2\pi \times 10000$ | Yes (slowest dim only) |
| Full vector repeats | LCM of all wavelengths | Effectively never |

---
---

# Quiz 3, Question 17 — Odometer Effect and Uniqueness

> **How does the "Odometer Effect" contribute to the uniqueness of Sinusoidal Positional Encodings?**
>
> **B. It creates an effectively infinite Least Common Multiple (LCM) by combining multiple diverse frequencies, preventing vector repetition.**

---

## Sub-questions

### What is the Odometer Effect?

Like a car odometer, each "digit" (dimension pair) cycles at a different rate. The combined reading is unique until **all** digits simultaneously return to their starting values.

### What is the Least Common Multiple (LCM)?

$$\text{LCM}(\lambda_1, \lambda_2, \dots, \lambda_{d/2}) = \text{smallest period where ALL waves align}$$

For the full vector to repeat, every dimension pair must simultaneously complete full cycles.

### Why is the LCM effectively infinite?

The wavelengths $\lambda_i = 2\pi \cdot 10000^{2i/d}$ are **irrational** multiples of each other (because $10000^{2i/d}$ is irrational for most $i$). The LCM of incommensurable numbers is undefined (infinite).

---

## Main answer

**B. Multiple diverse frequencies create an effectively infinite LCM — the full vector never repeats.**

```
Odometer analogy (base-10):

  Digit 0 (ones):      cycles every 1 step    (fastest)
  Digit 1 (tens):      cycles every 10 steps
  Digit 2 (hundreds):  cycles every 100 steps
  Digit 3 (thousands): cycles every 1000 steps (slowest)

  Full odometer repeats after LCM(1,10,100,1000) = 1000 steps

Sinusoidal PE:

  Dim pair 0:     λ₀ = 2π                     (fastest)
  Dim pair 1:     λ₁ = 2π · 10000^(2/d)
  ...
  Dim pair d/2-1: λ = 2π · 10000              (slowest)

  Full vector repeats after LCM(λ₀, λ₁, ...) ≈ ∞
  (wavelengths are incommensurable)
```

$$\text{LCM}(2\pi, \; 2\pi \cdot 10000^{2/d}, \; 2\pi \cdot 10000^{4/d}, \; \dots) = \infty$$

| Dimension Pair | Wavelength | Cycles | Role |
|---|---|---|---|
| 0 | $2\pi$ | Very fast | Fine position |
| Middle | Intermediate | Medium | Medium scale |
| $d/2 - 1$ | $2\pi \cdot 10000$ | Very slow | Coarse position |
| **Combined** | **LCM $\approx \infty$** | — | **Unique encoding per position** |

---
---

# Quiz 3, Question 18 — Vector Databases for RAG

> **What is the primary application of Vector Databases (like Pinecone or Milvus) in the context of LLMs?**
>
> **B. To perform high-speed semantic similarity search for Retrieval-Augmented Generation (RAG).**

---

## Sub-questions

### What is a Vector Database?

A specialized database optimized for storing and querying high-dimensional vectors. Uses approximate nearest neighbor (ANN) algorithms for fast similarity search.

### What is semantic similarity search?

$$\text{top-}k = \underset{\mathbf{v} \in \text{DB}}{\operatorname{argmax}} \; \cos(\mathbf{q}, \mathbf{v})$$

Given a query vector $\mathbf{q}$ (the user's question, embedded), find the $k$ most similar vectors in the database. "Similar" = close in embedding space = semantically related.

### What is RAG (Retrieval-Augmented Generation)?

$$\text{response} = \text{LLM}(\text{query} + \text{retrieved context})$$

Instead of relying solely on the LLM's parametric memory, RAG retrieves relevant documents from an external database and injects them into the prompt. This grounds the response in factual, up-to-date information.

---

## Main answer

**B. Vector databases enable high-speed semantic similarity search for RAG.**

```
RAG pipeline:

  User Query: "What is our Q4 revenue?"
       │
       ▼  Embed query
  q = Embed("What is our Q4 revenue?")    (vector)
       │
       ▼  Similarity search in Vector DB
  Top-k documents = VectorDB.search(q, k=5)
       │
       ▼  Augment prompt
  Prompt = query + top-k documents
       │
       ▼  Generate
  LLM(prompt) → "Q4 revenue was $12.3M based on..."
```

| Component | Role |
|---|---|
| Embedding model | Converts text → vectors |
| Vector Database | Stores vectors, finds nearest neighbors fast |
| ANN algorithm (HNSW, IVF) | Approximate search in milliseconds |
| LLM | Generates answer using retrieved context |
| RAG = Retrieval + Generation | Grounded, factual responses |

---
---

# Quiz 3, Question 19 — Zero-Shot Transfer via Multilingual Embeddings

> **How do multilingual embedding spaces enable "Zero-Shot Transfer" (e.g., training on English but testing on Spanish)?**
>
> **C. By aligning concepts geometrically, such that the vector for "Cat" (English) and "Gato" (Spanish) point in the same direction.**

---

## Sub-questions

### What is a multilingual embedding space?

$$\text{Embed}(\text{"cat"}) \approx \text{Embed}(\text{"gato"}) \approx \text{Embed}(\text{"猫"})$$

A single embedding model trained on multiple languages, where translation-equivalent words map to nearby vectors.

### What is Zero-Shot Transfer?

$$\text{Train on language A} \rightarrow \text{Test on language B}$$

The model never sees labeled examples in language B, yet performs well because the shared embedding space transfers knowledge across languages.

### How does geometric alignment enable this?

If "cat" and "gato" have the same vector, then a classifier trained to recognize the concept "cat" in English automatically recognizes "gato" in Spanish — no Spanish training data needed.

---

## Main answer

**C. Translation equivalents point in the same direction, enabling zero-shot transfer.**

```
Aligned multilingual embedding space:

  English "cat"  ────▶ ●  ← same region
  Spanish "gato" ────▶ ●
  Chinese "猫"   ────▶ ●

  English "dog"  ────▶ ○  ← different region
  Spanish "perro"────▶ ○
  Chinese "狗"   ────▶ ○

  A classifier trained on English embeddings
  works on Spanish/Chinese for free (zero-shot)
```

$$\cos(\text{Embed}(\text{"cat"}), \; \text{Embed}(\text{"gato"})) \approx 1.0$$

| Property | Monolingual | Multilingual (aligned) |
|---|---|---|
| Languages per model | 1 | Many |
| "cat" ↔ "gato" | Different spaces | Same vector |
| Zero-shot transfer | Impossible | Yes |
| Training data needed per language | Full labeled set | English only (transfers) |

---
---

# Quiz 3, Question 20 — Embedding Initialization with Small Variance

> **Why is it standard practice to initialize embedding vectors with a normal distribution of small variance (e.g., $\mathcal{N}(0, 0.02)$)?**
>
> **C. To preserve orthogonality and keep the model in a linear regime, preventing exploding gradients at the start of training.**

---

## Sub-questions

### What is $\mathcal{N}(0, 0.02)$?

$$x \sim \mathcal{N}(\mu=0, \;\sigma=0.02)$$

Normal distribution centered at zero with standard deviation $0.02$. Values are tiny: $\sim 95\%$ fall in $[-0.04, 0.04]$.

### Why near-zero initialization?

Small values keep activations in the **linear regime** of activation functions (like GELU, SiLU), where gradients are well-behaved:

$$f(x) \approx x \quad \text{for small } x \quad \text{(linear approximation)}$$

Large initial values push activations into saturation regions where gradients vanish or explode.

### Why preserve orthogonality?

$$\mathbb{E}[\mathbf{v}_i \cdot \mathbf{v}_j] = 0 \quad \text{for random } \mathcal{N}(0, \sigma) \text{ vectors}$$

Small random initialization keeps vectors approximately orthogonal (see Q6). This means each token starts with maximum information capacity — no two tokens are confused at initialization.

---

## Main answer

**C. Small-variance initialization preserves orthogonality and prevents exploding gradients.**

$$\sigma = 0.02 \implies \|\mathbf{v}\| \approx \sigma\sqrt{d} = 0.02\sqrt{768} \approx 0.55$$

```
Initialization effects:

  σ too large (e.g., 1.0):
  ──▶ Large activations ──▶ Saturation ──▶ Exploding/vanishing gradients
  ──▶ Loss of orthogonality ──▶ Token confusion at start

  σ too small (e.g., 1e-6):
  ──▶ All vectors ≈ 0 ──▶ All tokens identical ──▶ Symmetry problem
  ──▶ No learning signal

  σ just right (0.02):
  ──▶ Small activations ──▶ Linear regime ──▶ Stable gradients
  ──▶ Near-orthogonal ──▶ Each token starts distinct
```

| $\sigma$ | Norm $\approx \sigma\sqrt{d}$ | Gradients | Orthogonality | Training |
|---|---|---|---|---|
| $\gg 1$ | Very large | Explode | Lost | Unstable |
| $0.02$ | $\sim 0.55$ | Stable | Preserved | Smooth |
| $\ll 0.001$ | $\approx 0$ | Vanish | Trivial (all $\approx \mathbf{0}$) | Stalled |

---
---

# Quiz 3, Question 21 — Parameter Efficiency Calculation (Essay)

> **Parameter Efficiency Calculation:** Consider a Transformer model with vocabulary size $V = 50{,}000$ and embedding dimension $d_\text{model} = 768$.
> a) Calculate the total number of parameters in the Input Embedding Matrix ($W_E$).
> b) If the model uses Weight Tying, how many parameters are saved by not creating a separate Output Unembedding Matrix? Express your answer in millions of parameters.

---

## Sub-questions

### What is the Input Embedding Matrix?

$$W_E \in \mathbb{R}^{V \times d_\text{model}}$$

Maps each token ID to a $d$-dimensional vector. One row per vocabulary entry.

### What is the Output Unembedding Matrix?

$$W_U \in \mathbb{R}^{d_\text{model} \times V}$$

Projects the final hidden state back to vocabulary-sized logits. Same shape as $W_E^T$.

### What is Weight Tying?

$$W_U = W_E^T$$

Reuse the embedding matrix (transposed) as the output projection. Saves one full copy of the matrix.

---

## Main answer

### Part (a): Input Embedding Matrix parameters

$$|W_E| = V \times d_\text{model} = 50{,}000 \times 768 = 38{,}400{,}000$$

$$= 38.4 \text{ million parameters}$$

### Part (b): Parameters saved by Weight Tying

Without weight tying, a separate $W_U \in \mathbb{R}^{d_\text{model} \times V}$ is needed:

$$|W_U| = d_\text{model} \times V = 768 \times 50{,}000 = 38{,}400{,}000$$

$$\text{Parameters saved} = 38.4 \text{ million}$$

```
Without Weight Tying:

  W_E  (50000 × 768) = 38.4M params   ← input embedding
  W_U  (768 × 50000) = 38.4M params   ← output unembedding
  Total embedding params = 76.8M

With Weight Tying:

  W_E  (50000 × 768) = 38.4M params   ← shared for both
  W_U  = W_E^T                         ← no extra params
  Total embedding params = 38.4M       ← 50% reduction
```

| Component | Without Tying | With Tying | Saved |
|---|---|---|---|
| $W_E$ (Input) | 38.4M | 38.4M | — |
| $W_U$ (Output) | 38.4M | 0 (reuse $W_E^T$) | **38.4M** |
| **Total** | **76.8M** | **38.4M** | **38.4M (50%)** |

---
---

# Quiz 3, Question 22 — High-Dimensional Hypercube Distance (Essay)

> **High-Dimensional Geometry:** In a high-dimensional unit hypercube (where each dimension extends from -0.5 to 0.5), the distance from the center to the nearest face is always 0.5. Calculate the distance from the center to a corner for a dimension of $d = 100$. How many times farther is the corner than the face?

---

## Sub-questions

### What is the center of the hypercube?

$$\text{center} = (0, 0, 0, \dots, 0) \in \mathbb{R}^d$$

### What is a corner of the hypercube?

$$\text{corner} = (\pm 0.5, \pm 0.5, \pm 0.5, \dots, \pm 0.5) \in \mathbb{R}^d$$

Each of the $d$ coordinates is either $+0.5$ or $-0.5$. There are $2^d$ corners.

### What is the Euclidean distance formula?

$$d_\text{corner} = \sqrt{\sum_{i=1}^{d} (0.5)^2} = \sqrt{d \cdot 0.25} = \frac{\sqrt{d}}{2}$$

---

## Main answer

### Distance from center to corner ($d = 100$)

$$d_\text{corner} = \frac{\sqrt{100}}{2} = \frac{10}{2} = 5.0$$

### How many times farther than the face?

$$d_\text{face} = 0.5$$

$$\text{Ratio} = \frac{d_\text{corner}}{d_\text{face}} = \frac{5.0}{0.5} = 10$$

**The corner is 10 times farther from the center than the nearest face.**

```
Distance comparison (d = 100):

  Center ──── 0.5 ────▶ Face      (always 0.5)
  Center ──── 5.0 ────▶ Corner    (√100 / 2 = 5.0)

  Ratio = 5.0 / 0.5 = 10×

General formula:
  Ratio = √d / (2 × 0.5) = √d
```

$$\text{Ratio} = \frac{\sqrt{d}/2}{0.5} = \sqrt{d}$$

| Dimension $d$ | $d_\text{face}$ | $d_\text{corner}$ | Ratio ($\sqrt{d}$) |
|---|---|---|---|
| 2 | 0.5 | 0.71 | 1.41 |
| 10 | 0.5 | 1.58 | 3.16 |
| **100** | **0.5** | **5.0** | **10** |
| 768 | 0.5 | 13.86 | 27.7 |

---
---

# Quiz 3, Question 23 — Dataset Expansion & Stride Analysis (Essay)

> **Dataset Expansion & Stride Analysis:** You have a raw training corpus containing $T = 50{,}000$ tokens. Your model uses a context window of $L = 512$ tokens. We can generate training samples using a sliding window with configurable stride $S$. Assume the last incomplete training sample will be dropped.
>
> (a) **Case A (No Overlap):** Calculate the total number of training samples if $S = 512$.
> (b) **Case B (Max Overlap):** Calculate the total number of training samples if $S = 1$.
> (c) **The Efficiency Ratio:** How many times larger is the dataset in Case B compared to Case A? Explain why training on Case B might not yield proportionally better results despite the massive increase in data.

---

## Sub-questions

### How many samples does a sliding window produce?

$$N = \left\lfloor \frac{T - L}{S} \right\rfloor + 1$$

$T$ = total tokens, $L$ = window length, $S$ = stride.

### What does "drop the last incomplete sample" mean?

If the window extends beyond the corpus, that sample is discarded. The floor function $\lfloor \cdot \rfloor$ handles this.

---

## Main answer

### Part (a): Case A — $S = 512$ (No Overlap)

$$N_A = \left\lfloor \frac{50{,}000 - 512}{512} \right\rfloor + 1 = \left\lfloor \frac{49{,}488}{512} \right\rfloor + 1 = \lfloor 96.656 \rfloor + 1 = 96 + 1 = 97$$

### Part (b): Case B — $S = 1$ (Max Overlap)

$$N_B = \left\lfloor \frac{50{,}000 - 512}{1} \right\rfloor + 1 = 49{,}488 + 1 = 49{,}489$$

### Part (c): Efficiency Ratio

$$\text{Ratio} = \frac{N_B}{N_A} = \frac{49{,}489}{97} \approx 510\times$$

Case B produces $\sim 510\times$ more samples, but training on it will **not** yield $510\times$ better results because:

**Massive data redundancy.** Adjacent samples ($S = 1$) share 511 out of 512 tokens. Each sample provides only 1 new token of information. The model sees the same token contexts repeatedly, so the effective diversity of training signal is much lower than the sample count suggests.

$$\text{Unique information per sample} = S = 1 \text{ token}$$
$$\text{Total unique tokens} = T = 50{,}000 \text{ (same in both cases)}$$

```
Overlap visualization (S = 1):

  Sample 1: [tok₁  tok₂  tok₃  ... tok₅₁₂]
  Sample 2: [tok₂  tok₃  tok₄  ... tok₅₁₃]   ← 511/512 overlap
  Sample 3: [tok₃  tok₄  tok₅  ... tok₅₁₄]   ← 511/512 overlap

  Enormous sample count, but nearly identical samples
  Gradients from adjacent samples are highly correlated
```

| Metric | Case A ($S = 512$) | Case B ($S = 1$) |
|---|---|---|
| Samples | 97 | 49,489 |
| Ratio | 1× | ~510× |
| Overlap | 0 tokens | 511 tokens per pair |
| Unique info per sample | 512 tokens | 1 token |
| Total unique tokens | 50,000 | 50,000 (same!) |
| Gradient correlation | Low | Very high |

---
---

# Quiz 3, Question 24 — Positional Wavelength Calculation (Essay)

> **Positional Wavelength Calculation:** The wavelength of the sinusoidal encoding for a given dimension index $i$ is determined by the denominator term: $10000^{2i/d_\text{model}}$. For a model with $d_\text{model} = 512$:
>
> (a) Calculate the denominator value for dimension index $i = 256$ (the midpoint of the embedding vector).
> (b) Briefly explain what this value implies about the "speed" (frequency) of the sine wave at this dimension compared to index $i = 0$.

---

## Sub-questions

### What is the denominator formula?

$$\text{denom}(i) = 10000^{2i/d_\text{model}}$$

This is the scaling factor that determines the frequency of the sinusoidal encoding at dimension $i$.

### What is the relationship between denominator and frequency?

$$f_i = \frac{1}{\text{denom}(i)} = \frac{1}{10000^{2i/d}}$$

Larger denominator → lower frequency → slower oscillation → longer wavelength.

---

## Main answer

### Part (a): Denominator at $i = 256$, $d_\text{model} = 512$

$$\text{denom}(256) = 10000^{2 \times 256 / 512} = 10000^{512/512} = 10000^1 = 10{,}000$$

### Part (b): Speed comparison to $i = 0$

At $i = 0$:

$$\text{denom}(0) = 10000^{0} = 1$$

$$f_0 = \frac{1}{1} = 1 \quad \text{(fastest oscillation)}$$

At $i = 256$:

$$f_{256} = \frac{1}{10000} = 0.0001$$

$$\frac{f_{256}}{f_0} = \frac{0.0001}{1} = 10^{-4}$$

The sine wave at dimension $i = 256$ oscillates **10,000 times slower** than at $i = 0$. It completes one full cycle every $2\pi \times 10{,}000 \approx 62{,}832$ positions, versus every $2\pi \approx 6.28$ positions for $i = 0$.

```
Speed comparison:

  i = 0:    denom = 1      → wavelength = 2π ≈ 6.28 positions (fastest)
  i = 256:  denom = 10000  → wavelength = 2π × 10000 ≈ 62,832 positions (slowest)

  Ratio: 10,000× slower
```

| Property | $i = 0$ | $i = 256$ ($d/2$) |
|---|---|---|
| Denominator | 1 | 10,000 |
| Frequency | 1 | $10^{-4}$ |
| Wavelength | $2\pi \approx 6.3$ | $2\pi \times 10^4 \approx 62{,}832$ |
| Captures | Fine-grained position | Global position |
| Speed ratio | 1× | $10{,}000\times$ slower |

---
---

# Quiz 3, Question 25 — Composition of Shifts / Rotation Matrix Proof (Essay)

> **Composition of Shifts (Matrix Proof):** A key property of Sinusoidal Encodings is that applying the rotation matrix twice is equivalent to shifting the attention by double the distance.
>
> Let $M_\theta = \begin{pmatrix} \cos\theta & \sin\theta \\ -\sin\theta & \cos\theta \end{pmatrix}$ be the rotation matrix for a 1-step shift (angle $\theta$).
>
> **Task:** Perform the matrix multiplication $M_\theta \times M_\theta$. Show your steps and prove that the result equals the rotation matrix for a 2-step shift ($M_{2\theta}$).
>
> *Hint: Use the double-angle identities: $\cos(2\theta) = \cos^2\theta - \sin^2\theta$ and $\sin(2\theta) = 2\sin\theta\cos\theta$.*

---

## Sub-questions

### What is matrix multiplication for 2×2 matrices?

$$\begin{pmatrix} a & b \\ c & d \end{pmatrix} \begin{pmatrix} e & f \\ g & h \end{pmatrix} = \begin{pmatrix} ae+bg & af+bh \\ ce+dg & cf+dh \end{pmatrix}$$

### What are the double-angle identities?

$$\cos(2\theta) = \cos^2\theta - \sin^2\theta$$
$$\sin(2\theta) = 2\sin\theta\cos\theta$$

---

## Main answer

### Step 1: Set up the multiplication

$$M_\theta \times M_\theta = \begin{pmatrix} \cos\theta & \sin\theta \\ -\sin\theta & \cos\theta \end{pmatrix} \begin{pmatrix} \cos\theta & \sin\theta \\ -\sin\theta & \cos\theta \end{pmatrix}$$

### Step 2: Compute each entry

**Top-left:**
$$(\cos\theta)(\cos\theta) + (\sin\theta)(-\sin\theta) = \cos^2\theta - \sin^2\theta$$

**Top-right:**
$$(\cos\theta)(\sin\theta) + (\sin\theta)(\cos\theta) = 2\sin\theta\cos\theta$$

**Bottom-left:**
$$(-\sin\theta)(\cos\theta) + (\cos\theta)(-\sin\theta) = -2\sin\theta\cos\theta$$

**Bottom-right:**
$$(-\sin\theta)(\sin\theta) + (\cos\theta)(\cos\theta) = \cos^2\theta - \sin^2\theta$$

### Step 3: Assemble the result

$$M_\theta^2 = \begin{pmatrix} \cos^2\theta - \sin^2\theta & 2\sin\theta\cos\theta \\ -2\sin\theta\cos\theta & \cos^2\theta - \sin^2\theta \end{pmatrix}$$

### Step 4: Apply double-angle identities

$$\cos^2\theta - \sin^2\theta = \cos(2\theta)$$
$$2\sin\theta\cos\theta = \sin(2\theta)$$

$$M_\theta^2 = \begin{pmatrix} \cos(2\theta) & \sin(2\theta) \\ -\sin(2\theta) & \cos(2\theta) \end{pmatrix} = M_{2\theta} \quad \blacksquare$$

```
Summary:

  M_θ × M_θ = M_{2θ}

  One rotation by θ, applied twice = one rotation by 2θ

  Generalization: M_θ^n = M_{nθ}  (rotation by n steps)
```

This proves that sinusoidal encodings support **linear composition of shifts**: shifting by $k$ positions is a single matrix multiplication, and composing two shifts is equivalent to adding their offsets.

| Operation | Matrix | Equivalent |
|---|---|---|
| 1-step shift | $M_\theta$ | Rotation by $\theta$ |
| 2-step shift | $M_\theta \times M_\theta = M_\theta^2$ | Rotation by $2\theta$ |
| $n$-step shift | $M_\theta^n$ | Rotation by $n\theta$ |
| Shift by $a$ then $b$ | $M_{a\theta} \cdot M_{b\theta}$ | $M_{(a+b)\theta}$ |

---
