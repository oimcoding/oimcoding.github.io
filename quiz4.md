# Quiz 4: Attention Mechanisms — Tutor Answers (Q1–Q25)

---

# Quiz 4, Question 1 — RNN Computational Bottleneck

> **The primary computational bottleneck in Recurrent Neural Networks (RNNs) during training is:**
>
> **B. Sequential dependency (h_t depends on h_t-1), preventing parallelization.**

---

## Sub-questions

### What is a Recurrent Neural Network (RNN)?

$$h_t = f(W_h h_{t-1} + W_x x_t + b)$$

A neural network that processes sequences one step at a time. Each hidden state $h_t$ depends on the previous hidden state $h_{t-1}$ and the current input $x_t$.

### What is parallelization?

Processing multiple computations simultaneously on hardware (e.g., GPU cores). Requires that computations are **independent** — no data dependency between them.

### Why does sequential dependency prevent parallelization?

$$h_1 \rightarrow h_2 \rightarrow h_3 \rightarrow \dots \rightarrow h_T$$

To compute $h_t$, the model must first finish computing $h_{t-1}$. For a sequence of length $T$, all $T$ steps must execute **one after another**. GPU parallelism is wasted — only one step runs at a time.

---

## Main answer

**B. Sequential dependency ($h_t$ depends on $h_{t-1}$), preventing parallelization.**

```
RNN sequential computation:

  x₁    x₂    x₃    x₄    x₅
   ↓     ↓     ↓     ↓     ↓
  h₁ ──▶ h₂ ──▶ h₃ ──▶ h₄ ──▶ h₅
   │      │      │      │      │
  wait   wait   wait   wait   done

  Time: O(T) sequential steps — cannot parallelize


Transformer parallel computation:

  x₁    x₂    x₃    x₄    x₅
   ↓     ↓     ↓     ↓     ↓
  h₁    h₂    h₃    h₄    h₅   (all computed simultaneously)

  Time: O(1) parallel steps
```

| Property | RNN | Transformer |
|---|---|---|
| Dependency | $h_t$ depends on $h_{t-1}$ | All positions independent |
| Parallelizable | No (sequential) | Yes (fully parallel) |
| Training time scaling | $O(T)$ sequential steps | $O(1)$ parallel steps |
| GPU utilization | Low | High |

---
---

# Quiz 4, Question 2 — RNN Information Bottleneck

> **In the context of RNNs, the "Information Bottleneck" refers to:**
>
> **B. Compressing an entire source sentence into a single fixed-size context vector.**

---

## Sub-questions

### What is a context vector?

$$c = h_T$$

In sequence-to-sequence RNNs, the encoder reads the entire input and produces a single final hidden state $h_T$. This one vector $c$ must encode **everything** the decoder needs.

**Numeric example:** A 100-word sentence compressed into a 512-dimensional vector — 100 words worth of meaning squeezed into 512 numbers.

### Why is this a bottleneck?

$$\text{Information in sentence} \gg \text{Capacity of } c \in \mathbb{R}^{d}$$

A fixed-size vector has finite capacity. Longer sentences force more information through the same narrow channel. Early tokens in long sequences are "forgotten" as they get overwritten by later processing.

### How does attention solve this?

$$c_t = \sum_{i=1}^{T} \alpha_{t,i} \cdot h_i$$

Instead of one fixed vector, attention creates a **different** context vector $c_t$ for each decoder step, as a weighted sum over **all** encoder hidden states. No information is discarded.

---

## Main answer

**B. Compressing an entire source sentence into a single fixed-size context vector.**

```
Information Bottleneck:

  Encoder:  h₁ ──▶ h₂ ──▶ h₃ ──▶ ... ──▶ h_T
                                            │
                                    c = h_T  (SINGLE vector)
                                            │
  Decoder:                                  ▼
            All information must fit through this one vector

  Problem: 100-word sentence → 512-dim vector → information loss


Attention (the fix):

  Encoder:  h₁    h₂    h₃    ...    h_T
             ↑     ↑     ↑            ↑
  Decoder:   α₁    α₂    α₃          α_T   (weighted access to ALL states)
             └─────┴─────┴────────────┘
                        c_t = Σ αᵢhᵢ
```

| Property | Fixed Context Vector | Attention |
|---|---|---|
| Context | Single $h_T$ | Weighted sum of all $h_i$ |
| Capacity | Fixed $d$-dimensional | Scales with sequence length |
| Long sentences | Information loss | Full access |
| What solved it | — | Bahdanau Attention (2014) |

---
---

# Quiz 4, Question 3 — LSTM Cell State

> **Which architectural feature allows LSTMs to mitigate the vanishing gradient problem better than standard RNNs?**
>
> **B. The "Cell State" (C_t) acting as a gradient highway.**

---

## Sub-questions

### What is the vanishing gradient problem?

$$\frac{\partial \mathcal{L}}{\partial h_1} = \frac{\partial \mathcal{L}}{\partial h_T} \cdot \prod_{t=2}^{T} \frac{\partial h_t}{\partial h_{t-1}}$$

In standard RNNs, gradients are multiplied through $T$ time steps. If each factor $\frac{\partial h_t}{\partial h_{t-1}} < 1$, the product shrinks exponentially toward zero. Early tokens receive no gradient signal.

**Numeric example:** Factor $= 0.9$, $T = 100$: $0.9^{100} \approx 2.66 \times 10^{-5}$ — gradient nearly vanishes.

### What is the LSTM Cell State?

$$C_t = f_t \odot C_{t-1} + i_t \odot \tilde{C}_t$$

A separate memory channel that flows through time via **element-wise** operations (multiply by forget gate $f_t$, add new info $i_t \odot \tilde{C}_t$). No matrix multiplication — just gated addition.

### Why does addition preserve gradients?

$$\frac{\partial C_t}{\partial C_{t-1}} = f_t$$

The gradient through the cell state is simply the forget gate value $f_t \in (0, 1)$. When $f_t \approx 1$, the gradient passes through **undiminished** — like a highway with no toll.

---

## Main answer

**B. The "Cell State" ($C_t$) acting as a gradient highway.**

```
Standard RNN gradient flow (vanishing):

  ∂L/∂h₁ = ∂L/∂h_T × W × W × W × ... × W
                        ↑   ↑   ↑         ↑
                    T multiplications by W
                    → exponential shrinkage


LSTM Cell State gradient flow (highway):

  C₁ ──(×f₂ + new)──▶ C₂ ──(×f₃ + new)──▶ C₃ ──▶ ... ──▶ C_T

  ∂C_T/∂C₁ = f_T × f_{T-1} × ... × f₂
              ↑
         when f ≈ 1: gradient ≈ 1 (preserved!)
```

| Property | Standard RNN | LSTM |
|---|---|---|
| Gradient path | Through $W$ matrices | Through cell state (additive) |
| Gradient factor per step | $W$ (matrix multiply) | $f_t$ (scalar gate) |
| Long-range gradients | Vanish exponentially | Preserved via highway |
| Memory mechanism | Hidden state only | Cell state + hidden state |

---
---

# Quiz 4, Question 4 — Self-Attention Complexity

> **What is the computational complexity of Self-Attention with respect to sequence length N?**
>
> **B. O(N^2)**

---

## Sub-questions

### What is Self-Attention?

$$\text{Attention}(Q, K, V) = \text{Softmax}\!\left(\frac{QK^T}{\sqrt{d_k}}\right) V$$

Every token computes a similarity score with **every other token** via the dot product $QK^T$.

### What is the shape of $QK^T$?

$$Q \in \mathbb{R}^{N \times d_k}, \quad K \in \mathbb{R}^{N \times d_k}$$

$$QK^T \in \mathbb{R}^{N \times N}$$

The result is an $N \times N$ attention score matrix. Each of the $N^2$ entries requires a dot product of dimension $d_k$.

### Total computation?

$$\text{FLOPs} = O(N^2 \cdot d_k)$$

Since $d_k$ is a fixed constant (e.g., 64), the complexity **with respect to $N$** is $O(N^2)$.

---

## Main answer

**B. $O(N^2)$**

```
Attention score matrix (N × N):

         Key tokens
         k₁  k₂  k₃  ...  k_N
  q₁  [  ·   ·   ·        ·  ]
  q₂  [  ·   ·   ·        ·  ]     N × N entries
  q₃  [  ·   ·   ·        ·  ]     each is a dot product
  ...
  q_N  [  ·   ·   ·        ·  ]

  Total operations: N² dot products
```

**Numeric example:** $N = 4096$ tokens → $4096^2 = 16{,}777{,}216$ attention scores.

$$\text{Double the sequence: } N = 8192 \implies N^2 = 67{,}108{,}864 \quad (4\times \text{more})$$

| Sequence Length $N$ | Attention Scores ($N^2$) | Relative Cost |
|---|---|---|
| 512 | 262,144 | 1x |
| 1024 | 1,048,576 | 4x |
| 4096 | 16,777,216 | 64x |
| 8192 | 67,108,864 | 256x |

---
---

# Quiz 4, Question 5 — Query Vector in Search Engine Analogy

> **In the "Search Engine" analogy for Attention, the Query vector (Q) represents:**
>
> **C. The current token looking for relevant information.**

---

## Sub-questions

### What are Q, K, V in attention?

$$Q = XW_Q, \quad K = XW_K, \quad V = XW_V$$

Three linear projections of the input. Each serves a different role in the attention mechanism.

### What is the search engine analogy?

- **Query (Q)**: The search term you type — "what am I looking for?"
- **Key (K)**: The index/title of each document — "what does this document contain?"
- **Value (V)**: The actual content of each document — "what information do I return?"

### How does the mechanism work?

$$\text{score}(q_i, k_j) = \frac{q_i \cdot k_j}{\sqrt{d_k}}$$

The query $q_i$ (current token's question) is compared against all keys $k_j$ (every token's label). High score = high relevance. The output is a weighted sum of values.

---

## Main answer

**C. The current token looking for relevant information.**

```
Search engine analogy:

  Query: "What information do I need?"     → Q (current token)
  Keys:  "What does each token offer?"     → K (all tokens)
  Values: "The actual content to retrieve"  → V (all tokens)

  Process:
  ┌──────────┐     ┌──────────┐     ┌──────────┐
  │  Query   │────▶│  Match   │────▶│  Return  │
  │  "cat"   │     │  Q · Kᵀ  │     │  Σ αᵢVᵢ │
  │          │     │  scores   │     │  content │
  └──────────┘     └──────────┘     └──────────┘
```

| Role | Analogy | Function | Shape |
|---|---|---|---|
| **Query (Q)** | **Search term** | **"What am I looking for?"** | $(N, d_k)$ |
| Key (K) | Document title | "What do I contain?" | $(N, d_k)$ |
| Value (V) | Document content | "Here is my information" | $(N, d_v)$ |

---
---

# Quiz 4, Question 6 — Scaled Dot-Product: Why Divide by $\sqrt{d_k}$

> **In the equation Attention(Q, K, V) = softmax(Q K^T / sqrt(d_k)) * V, why do we divide by sqrt(d_k)?**
>
> **C. To prevent dot products from getting too large, causing vanishing softmax gradients.**

---

## Sub-questions

### What happens without scaling?

$$q \cdot k = \sum_{i=1}^{d_k} q_i \cdot k_i$$

If $q_i, k_i \sim \mathcal{N}(0, 1)$, the dot product is a sum of $d_k$ independent products. By the CLT:

$$\text{Var}(q \cdot k) = d_k$$

$$\text{Std}(q \cdot k) = \sqrt{d_k}$$

**Numeric example:** $d_k = 64$ → dot products have standard deviation $\sqrt{64} = 8$. Values like $\pm 16$ are common.

### What happens when softmax receives large inputs?

$$\text{Softmax}([20, 0, 0]) = [1.000, 0.000, 0.000]$$

Large inputs push softmax toward a **one-hot** distribution. Gradients approach zero:

$$\frac{\partial \text{Softmax}_i}{\partial z_j} = p_i(\delta_{ij} - p_j) \approx 0 \quad \text{when } p_i \approx 0 \text{ or } 1$$

### What does dividing by $\sqrt{d_k}$ achieve?

$$\text{Var}\!\left(\frac{q \cdot k}{\sqrt{d_k}}\right) = \frac{d_k}{d_k} = 1$$

Normalizes the variance of dot products to 1, keeping softmax inputs in a moderate range where gradients are healthy.

---

## Main answer

**C. To prevent dot products from getting too large, causing vanishing softmax gradients.**

```
Without scaling (d_k = 64):

  dot products ~ N(0, 64)    → values like ±16 common
       │
       ▼
  Softmax([16, -8, 2]) ≈ [1.0, 0.0, 0.0]  → one-hot
       │
       ▼
  ∂Softmax/∂z ≈ 0  → vanishing gradients → training stalls


With scaling:

  (dot products) / √64 ~ N(0, 1)    → values like ±2
       │
       ▼
  Softmax([2, -1, 0.25]) ≈ [0.66, 0.03, 0.11]  → soft distribution
       │
       ▼
  ∂Softmax/∂z ≠ 0  → healthy gradients → training works
```

$$\frac{QK^T}{\sqrt{d_k}}: \quad \text{Var} = \frac{d_k}{d_k} = 1 \quad \text{(controlled range)}$$

| Scenario | Dot product variance | Softmax output | Gradients |
|---|---|---|---|
| No scaling | $d_k$ (large) | Near one-hot | Vanishing |
| **Scaled by $\sqrt{d_k}$** | **1** | **Soft distribution** | **Healthy** |

---
---

# Quiz 4, Question 7 — Causal Masking

> **What happens mathematically during "Causal Masking" in a Transformer Decoder?**
>
> **B. Future tokens are set to negative infinity before the Softmax step.**

---

## Sub-questions

### What is causal masking?

$$\text{Mask}_{i,j} = \begin{cases} 0 & \text{if } j \leq i \\ -\infty & \text{if } j > i \end{cases}$$

A matrix added to attention scores that blocks token $i$ from attending to any future token $j > i$. Enforces the autoregressive constraint.

### Why $-\infty$ and not $0$?

$$e^{-\infty} = 0 \implies \text{Softmax}(-\infty) = 0$$

Setting the raw score to $-\infty$ guarantees **exactly zero** attention weight after softmax. The future token is completely invisible.

### What if we multiplied by $0$ instead?

$$\text{score} \times 0 = 0 \implies e^0 = 1 \implies \text{Softmax}(0) > 0$$

Multiplying by zero sets the pre-softmax value to $0$, but $e^0 = 1$, so the future token would still receive **positive** attention weight — information leaks.

---

## Main answer

**B. Future tokens are set to $-\infty$ before the Softmax step.**

```
Score matrix after causal masking (4 tokens):

         j=0    j=1    j=2    j=3
  i=0 [  s₀₀   -∞     -∞     -∞  ]   ← sees only itself
  i=1 [  s₁₀   s₁₁   -∞     -∞  ]   ← sees 0,1
  i=2 [  s₂₀   s₂₁   s₂₂   -∞  ]   ← sees 0,1,2
  i=3 [  s₃₀   s₃₁   s₃₂   s₃₃ ]   ← sees all

After Softmax: all -∞ positions → 0 (zero attention)
```

$$\text{Masked\_Attention} = \text{Softmax}\!\left(\frac{QK^T}{\sqrt{d_k}} + \text{Mask}\right) V$$

| Masking method | Pre-softmax value | Post-softmax weight | Information leak? |
|---|---|---|---|
| Add $-\infty$ | $-\infty$ | $e^{-\infty} = 0$ | No |
| Multiply by $0$ | $0$ | $e^0 = 1 > 0$ | Yes |

---
---

# Quiz 4, Question 8 — Output Dimension of Attention

> **What determines the output dimension of the Scaled Dot-Product Attention mechanism?**
>
> **B. The dimension of the Value vectors (d_v).**

---

## Sub-questions

### What is the attention output formula?

$$\text{Output} = \text{Softmax}\!\left(\frac{QK^T}{\sqrt{d_k}}\right) V$$

The attention weights (shape $N \times N$) multiply the value matrix $V$.

### What are the matrix dimensions?

$$\underbrace{\text{Softmax}(\cdots)}_{N \times N} \times \underbrace{V}_{N \times d_v} = \underbrace{\text{Output}}_{N \times d_v}$$

The attention weights are $N \times N$. $V$ is $N \times d_v$. Matrix multiplication yields $N \times d_v$.

### Why doesn't $d_k$ determine the output?

$d_k$ determines the size of the dot product in $QK^T$ but cancels out in the softmax — the resulting attention weights are always $N \times N$ regardless of $d_k$. The final output dimension is determined solely by $V$.

---

## Main answer

**B. The dimension of the Value vectors ($d_v$).**

```
Dimension flow through attention:

  Q (N × d_k)  ×  Kᵀ (d_k × N)  =  Scores (N × N)
                                         │
                                    Softmax (N × N)
                                         │
                                    × V (N × d_v)
                                         │
                                    Output (N × d_v)
                                                ↑
                                           determined by d_v
```

| Matrix | Shape | Role in output dimension |
|---|---|---|
| $Q$ | $N \times d_k$ | Used for scoring, not output |
| $K$ | $N \times d_k$ | Used for scoring, not output |
| Attention weights | $N \times N$ | Weights over tokens |
| **$V$** | $N \times d_v$ | **Determines output dimension** |
| **Output** | $N \times d_v$ | **Inherits $d_v$ from $V$** |

---
---

# Quiz 4, Question 9 — $QK^T$ Operation

> **Which of the following best describes the operation Q * K^T?**
>
> **B. Calculating a similarity score between every pair of tokens.**

---

## Sub-questions

### What is a dot product between two vectors?

$$q_i \cdot k_j = \sum_{l=1}^{d_k} q_{i,l} \cdot k_{j,l} = \|q_i\| \|k_j\| \cos\theta$$

Measures how aligned two vectors are. Higher value → more similar directions → more relevant.

### What does the full $QK^T$ matrix contain?

$$[QK^T]_{i,j} = q_i \cdot k_j$$

Entry $(i, j)$ is the similarity score between token $i$'s query and token $j$'s key. The full matrix is $N \times N$ — **every pair** of tokens is compared.

---

## Main answer

**B. Calculating a similarity score between every pair of tokens.**

```
QKᵀ matrix (4-token sequence: "The cat sat down"):

              K: The   cat   sat   down
  Q: The     [ 3.2   1.1   0.5   0.8 ]
  Q: cat     [ 1.0   2.8   1.9   0.3 ]
  Q: sat     [ 0.7   2.1   2.5   1.4 ]
  Q: down    [ 0.9   0.4   1.6   3.0 ]

  Each entry = dot product = similarity score
  N² total scores for N tokens
```

$$QK^T \in \mathbb{R}^{N \times N}: \quad \text{all-pairs similarity matrix}$$

| Entry $(i,j)$ | Meaning |
|---|---|
| High score | Token $i$ finds token $j$ highly relevant |
| Low score | Token $i$ finds token $j$ irrelevant |
| Diagonal $(i,i)$ | Token's self-similarity |

---
---

# Quiz 4, Question 10 — Feed-Forward Network Purpose

> **What is the purpose of the Feed-Forward Network (FFN) that follows the Attention layer?**
>
> **B. To process information individually for each token and add non-linearity.**

---

## Sub-questions

### What is the FFN in a Transformer?

$$\text{FFN}(x) = W_2 \cdot \text{ReLU}(W_1 x + b_1) + b_2$$

A two-layer MLP applied **independently** to each token's representation. $W_1 \in \mathbb{R}^{d_\text{model} \times d_{ff}}$, $W_2 \in \mathbb{R}^{d_{ff} \times d_\text{model}}$, typically $d_{ff} = 4 \times d_\text{model}$.

### Why is non-linearity needed?

Attention is a **weighted sum** (linear operation). Without non-linearity, stacking attention layers would collapse into a single linear transformation. The ReLU (or SwiGLU) in the FFN introduces non-linear capacity.

### What does "individually for each token" mean?

$$\text{FFN}(x_i) \quad \text{for each } i \in \{1, \dots, N\}$$

The same FFN weights are applied to each token's vector independently. No cross-token interaction — that was attention's job. The FFN transforms each token's representation in isolation.

---

## Main answer

**B. To process information individually for each token and add non-linearity.**

```
Transformer block — division of labor:

  Input tokens
       │
       ▼
  ┌─────────────────────┐
  │   Self-Attention     │  ← Cross-token: "who should I attend to?"
  │   (linear mixing)   │     Mixes information BETWEEN tokens
  └─────────────────────┘
       │
       ▼
  ┌─────────────────────┐
  │   Feed-Forward Net   │  ← Per-token: "how should I transform?"
  │   (non-linear)       │     Processes each token INDEPENDENTLY
  │   W₂·ReLU(W₁x+b₁)+b₂│
  └─────────────────────┘
       │
       ▼
  Output tokens
```

| Property | Self-Attention | FFN |
|---|---|---|
| Scope | Cross-token (all pairs) | Per-token (independent) |
| Linearity | Linear (weighted sum) | Non-linear (ReLU/SwiGLU) |
| Role | Information routing | Feature transformation |
| Parameters | $W_Q, W_K, W_V, W_O$ | $W_1, W_2, b_1, b_2$ |

---
---

# Quiz 4, Question 11 — Why Transformers Need Positional Encodings

> **Why do Transformers need Positional Encodings, unlike RNNs?**
>
> **A. Transformers are invariant to sequence order without them.**

---

## Sub-questions

### What is permutation invariance?

$$\text{Attention}(\{x_1, x_2, x_3\}) = \text{Attention}(\{x_3, x_1, x_2\})$$

Self-attention computes pairwise dot products between all tokens. Reordering the input produces the same attention weights (just permuted). The mechanism treats the input as a **set**, not a **sequence**.

### How do RNNs encode order?

$$h_t = f(h_{t-1}, x_t)$$

RNNs process tokens left-to-right. Position is encoded implicitly by the sequential computation — $x_1$ is always processed first.

### What are Positional Encodings?

$$\text{input}_i = \text{embedding}(x_i) + \text{PE}(i)$$

A position-dependent vector added to each token's embedding. Breaks the permutation invariance by making each position's representation unique.

---

## Main answer

**A. Transformers are invariant to sequence order without them.**

```
Without positional encodings:

  "dog bites man" → Attention → same result as
  "man bites dog" → Attention → (just permuted outputs)

  The model cannot distinguish word order!


With positional encodings:

  "dog" + PE(0), "bites" + PE(1), "man" + PE(2)
  "man" + PE(0), "bites" + PE(1), "dog" + PE(2)

  Now position 0 ≠ position 2 → order is encoded
```

| Property | RNN | Transformer (no PE) | Transformer (with PE) |
|---|---|---|---|
| Order encoding | Implicit (sequential) | None (set operation) | Explicit (PE vectors) |
| "dog bites man" vs "man bites dog" | Different $h_T$ | Identical output | Different output |
| Permutation invariant | No | Yes | No |

---
---

# Quiz 4, Question 12 — Absolute Positional Encoding

> **In "Absolute" Positional Encoding (Sinusoidal), a token's position is represented by:**
>
> **C. A fixed vector of sines and cosines added to the word embedding.**

---

## Sub-questions

### What is the sinusoidal positional encoding formula?

$$\text{PE}(\text{pos}, 2i) = \sin\!\left(\frac{\text{pos}}{10000^{2i/d_\text{model}}}\right)$$

$$\text{PE}(\text{pos}, 2i+1) = \cos\!\left(\frac{\text{pos}}{10000^{2i/d_\text{model}}}\right)$$

Even dimensions use sine, odd dimensions use cosine. Each dimension oscillates at a different frequency.

### What does "fixed" mean?

The PE vectors are computed from a formula — no learnable parameters. Position 42 always gets the same vector regardless of the input text. Deterministic and precomputed.

### How is it combined with the word embedding?

$$x_i = \text{Embedding}(w_i) + \text{PE}(i)$$

Element-wise addition. The token embedding and positional encoding must have the same dimension $d_\text{model}$.

---

## Main answer

**C. A fixed vector of sines and cosines added to the word embedding.**

```
Sinusoidal PE construction for position pos:

  dim 0: sin(pos / 10000^(0/d))       ← high frequency
  dim 1: cos(pos / 10000^(0/d))
  dim 2: sin(pos / 10000^(2/d))
  dim 3: cos(pos / 10000^(2/d))
  ...
  dim d-2: sin(pos / 10000^((d-2)/d))  ← low frequency
  dim d-1: cos(pos / 10000^((d-2)/d))


Combining with embedding:

  ┌──────────────┐   ┌──────────────┐   ┌──────────────────┐
  │ Embedding(w)  │ + │   PE(pos)    │ = │ Position-aware    │
  │ semantic info │   │ position info│   │ representation    │
  └──────────────┘   └──────────────┘   └──────────────────┘
```

| Property | Sinusoidal PE |
|---|---|
| Learned? | No (formula-based) |
| Unique per position? | Yes |
| Dimension | Same as $d_\text{model}$ |
| How combined | Element-wise addition |
| Extrapolation | Possible (formula works for any position) |

---
---

# Quiz 4, Question 13 — Relative vs Absolute Positional Encoding

> **Which method focuses on "How far are you?" rather than "Where am I?"?**
>
> **B. Relative Positional Encoding.**

---

## Sub-questions

### What is absolute positional encoding?

$$\text{PE}(i) \quad \text{— depends only on position } i$$

Assigns a unique vector to each absolute position. "Token at position 5" always gets $\text{PE}(5)$.

### What is relative positional encoding?

$$\text{bias}(i - j) \quad \text{— depends on distance between positions}$$

Instead of encoding where each token is, it encodes the **distance** between any two tokens. The attention score between positions $i$ and $j$ gets a bias based on $|i - j|$.

### Why is "how far" better than "where"?

The phrase "the cat" has the same internal relationship whether it starts at position 0 or position 500. Relative encoding captures this — the bias for distance 1 is the same regardless of absolute position.

---

## Main answer

**B. Relative Positional Encoding.**

```
Absolute: "Where am I?"

  Position:  0     1     2     3     4
  PE:       PE(0) PE(1) PE(2) PE(3) PE(4)

  "the cat" at positions 0,1 gets PE(0),PE(1)
  "the cat" at positions 3,4 gets PE(3),PE(4)  ← different!


Relative: "How far are you?"

  Distance:  ..., -2, -1, 0, +1, +2, ...
  Bias:      b(-2) b(-1) b(0) b(+1) b(+2)

  "the cat" → distance = 1 → same bias b(1) regardless of position ✓
```

| Approach | Encodes | Example |
|---|---|---|
| Absolute | Position $i$ | "I am at position 5" |
| **Relative** | **Distance $i - j$** | **"You are 3 tokens away"** |

---
---

# Quiz 4, Question 14 — T5 Logarithmic Bucketing

> **In T5's Relative Positional Bias, "Logarithmic Bucketing" is used to:**
>
> **B. Group far-away tokens into larger "buckets" to save parameters.**

---

## Sub-questions

### What is T5's relative positional bias?

$$\text{score}(i, j) = q_i \cdot k_j + b(i - j)$$

T5 adds a learned scalar bias $b$ to each attention score based on the relative distance $i - j$. Each distinct distance needs its own learned parameter.

### Why is naive relative encoding expensive?

If the max sequence length is $T = 4096$, relative distances range from $-(T-1)$ to $+(T-1)$: $2T - 1 = 8191$ distinct values, each needing a learned bias parameter.

### What is logarithmic bucketing?

For small distances (e.g., 0–15), each distance gets its own bucket (fine-grained). For large distances (e.g., 16–4096), many distances share the same bucket (coarse-grained), grouped logarithmically.

$$\text{bucket}(d) = \begin{cases} d & \text{if } d \leq 15 \\ 15 + \lfloor\log_2(d) \cdot k\rfloor & \text{if } d > 15 \end{cases}$$

---

## Main answer

**B. Group far-away tokens into larger "buckets" to save parameters.**

```
Logarithmic bucketing:

  Distance:   0  1  2  3  ...  15  16-31  32-63  64-127  128-255  ...
  Bucket:     0  1  2  3  ...  15   16     17     18      19      ...
              ├── fine-grained ──┤  ├──── coarse-grained (log scale) ────┤

  Nearby tokens: precise distance matters → unique bucket each
  Far tokens: exact distance matters less → share buckets
```

$$\text{Naive: } 2T - 1 \text{ parameters} \quad \text{vs} \quad \text{Bucketed: } \sim 32 \text{ parameters}$$

| Distance range | Bucket granularity | Rationale |
|---|---|---|
| 0–15 | One bucket per distance | Nearby token relationships are precise |
| 16+ | Logarithmic groups | Far-away tokens: "roughly 100 away" suffices |
| Parameters saved | $\sim 8000 \rightarrow \sim 32$ | Massive reduction |

---
---

# Quiz 4, Question 15 — RoPE Mechanism

> **Rotary Positional Embedding (RoPE) applies position information via:**
>
> **B. Matrix multiplication (Rotation) (x * R).**

---

## Sub-questions

### What is RoPE?

$$q_m = R_m \cdot q, \quad k_n = R_n \cdot k$$

Instead of **adding** a positional vector (like sinusoidal PE), RoPE **rotates** the query and key vectors by an angle proportional to their position. Position is encoded in the angle of rotation.

### What is the rotation matrix $R$?

$$R_{t,\theta} = \begin{pmatrix} \cos(t\theta) & -\sin(t\theta) \\ \sin(t\theta) & \cos(t\theta) \end{pmatrix}$$

A 2D rotation by angle $t\theta$, where $t$ is the position and $\theta$ is the frequency. Applied to pairs of dimensions in the embedding.

### Why is rotation useful for attention?

$$q_m^T k_n = (R_m q)^T (R_n k) = q^T R_m^T R_n k = q^T R_{n-m} k$$

The dot product between rotated $q$ and $k$ depends only on the **relative** distance $m - n$. Absolute positions cancel out — RoPE achieves relative encoding through rotation.

---

## Main answer

**B. Matrix multiplication (Rotation) ($x \cdot R$).**

```
RoPE mechanism:

  ┌────────────┐     ┌──────────────┐     ┌────────────────┐
  │ q (query)   │  ×  │ R_m (rotation │  =  │ q_rotated      │
  │ raw vector  │     │  at pos m)   │     │ position-aware │
  └────────────┘     └──────────────┘     └────────────────┘

  Applied to pairs of dimensions:
  [q₁, q₂] → rotate by θ
  [q₃, q₄] → rotate by θ/10000^(2/d)
  [q₅, q₆] → rotate by θ/10000^(4/d)
  ...
```

| PE Method | How position is applied | Type |
|---|---|---|
| Sinusoidal | Addition: $x + \text{PE}$ | Absolute |
| Learned | Addition: $x + \text{PE}$ | Absolute |
| T5 Bias | Bias added to score | Relative |
| **RoPE** | **Rotation: $R \cdot x$** | **Relative (via rotation)** |

---
---

# Quiz 4, Question 16 — RoPE Geometric Interpretation

> **In the geometric interpretation of RoPE, rotating a vector changes its:**
>
> **B. Orientation (Angle).**

---

## Sub-questions

### What properties does a vector have geometrically?

$$x \in \mathbb{R}^d: \quad \text{magnitude} = \|x\|, \quad \text{direction/orientation} = \frac{x}{\|x\|}$$

A vector is fully described by its length (magnitude) and direction (angle/orientation in space).

### What does a rotation matrix preserve?

$$\|Rx\| = \|x\|$$

Rotation matrices are **orthogonal**: they preserve the length (norm) of vectors. Only the direction changes.

### What does rotation change?

$$\text{angle}(Rx) \neq \text{angle}(x)$$

The vector is rotated to point in a new direction. In 2D, this is a simple angular shift. In higher dimensions, RoPE rotates pairs of dimensions independently.

---

## Main answer

**B. Orientation (Angle).**

```
2D rotation example:

     ↑ y                      ↑ y
     │    /  x (original)     │  / x_rotated
     │   /                    │ /
     │  / angle α             │/ angle α + θ
     │ /                      │
     └──────────▶ x           └──────────▶ x

  Magnitude: unchanged (||Rx|| = ||x||)
  Angle: shifted by θ (orientation changed)
```

| Property | Before rotation | After rotation |
|---|---|---|
| Magnitude $\|x\|$ | $r$ | $r$ (preserved) |
| **Orientation (angle)** | $\alpha$ | $\alpha + \theta$ **(changed)** |
| Information content | Same | Same (just reoriented) |

---
---

# Quiz 4, Question 17 — Needle in a Haystack Test

> **The "Needle in a Haystack" test measures a model's ability to:**
>
> **C. Retrieve a specific piece of information hidden in a massive context window.**

---

## Sub-questions

### What is a context window?

$$\text{context window} = \text{max tokens the model can process at once}$$

Modern LLMs have context windows of 4K–128K+ tokens. The model must attend to all tokens in this window.

### What is the test setup?

A specific fact (the "needle") is inserted at a random position inside a large body of irrelevant text (the "haystack"). The model is then asked to recall that fact.

**Example:** Insert "The secret code is 7429" at position 50K in a 100K-token document, then ask "What is the secret code?"

### What does it measure?

Whether the model can actually attend to and retrieve information at **any** position in its context window, not just the beginning or end.

---

## Main answer

**C. Retrieve a specific piece of information hidden in a massive context window.**

```
Needle in a Haystack test:

  ┌──────────────────────────────────────────────────┐
  │  irrelevant text ... irrelevant text ...          │
  │  irrelevant text ... irrelevant text ...          │
  │  irrelevant text ... "The secret code is 7429"   │ ← needle
  │  irrelevant text ... irrelevant text ...          │
  │  irrelevant text ... irrelevant text ...          │
  └──────────────────────────────────────────────────┘
                          haystack (100K tokens)

  Question: "What is the secret code?"
  Expected: "7429"
```

| Aspect | Description |
|---|---|
| Needle | A specific fact embedded in text |
| Haystack | Large volume of irrelevant context |
| Measures | Long-range information retrieval |
| Failure mode | Model ignores or forgets mid-context information ("lost in the middle") |

---
---

# Quiz 4, Question 18 — Extrapolation Failure

> **"Extrapolation" failure occurs when:**
>
> **B. A model encounters a sequence length longer than training.**

---

## Sub-questions

### What is extrapolation in the context of positional encodings?

$$\text{Train on } L_\text{train} = 2048 \text{ tokens, test on } L_\text{test} = 8192 \text{ tokens}$$

The model sees positions it has never encountered during training. Position 5000 was never in any training example.

### Why do models fail at unseen positions?

- **Learned PE**: Positions beyond $L_\text{train}$ have no learned embeddings — undefined behavior.
- **Sinusoidal PE**: Mathematically defined for any position, but attention patterns were only trained on shorter distances.
- **RoPE**: Rotation angles for large positions produce unfamiliar patterns the model hasn't learned to interpret.

---

## Main answer

**B. A model encounters a sequence length longer than training.**

```
Extrapolation failure:

  Training:  positions 0 ──────────────── 2048
             [  well-learned territory    ]

  Inference: positions 0 ──────────────── 2048 ──────── 8192
             [  well-learned territory    ][ UNKNOWN !! ]
                                            ↑
                                     extrapolation zone
                                     → performance degrades
```

| PE Method | Extrapolation ability |
|---|---|
| Learned PE | Cannot (undefined beyond training length) |
| Sinusoidal | Formula works but attention hasn't seen it |
| RoPE (raw) | Degrades — unfamiliar rotation angles |
| RoPE + PI | Improved — interpolates into known range |
| ALiBi | Better — linear bias generalizes more naturally |

---
---

# Quiz 4, Question 19 — Linear Interpolation (PI)

> **To extend context length using "Linear Interpolation" (PI), we effectively:**
>
> **A. Squeeze the new, longer indices back into the original trained range.**

---

## Sub-questions

### What is Position Interpolation (PI)?

$$\text{PE}_\text{PI}(\text{pos}) = \text{PE}\!\left(\frac{\text{pos} \cdot L_\text{train}}{L_\text{new}}\right)$$

Scale down all position indices so they fit within the original training range $[0, L_\text{train}]$. A model trained on 2048 positions can handle 8192 by mapping $[0, 8192] \rightarrow [0, 2048]$.

### What is the scaling factor?

$$\text{scale} = \frac{L_\text{train}}{L_\text{new}} = \frac{2048}{8192} = 0.25$$

Every position is multiplied by 0.25: position 8000 becomes position $8000 \times 0.25 = 2000$ — within the trained range.

### What is the tradeoff?

Positions are now more densely packed. The model must distinguish positions that are closer together than it saw during training. Brief fine-tuning on the new length restores resolution.

---

## Main answer

**A. Squeeze the new, longer indices back into the original trained range.**

```
Without interpolation (extrapolation):

  Trained range:  [0 ─────── 2048]
  New positions:  [0 ─────── 2048 ──────── 8192]
                                   ↑ outside trained range → failure


With Position Interpolation:

  New positions:  [0 ─────────────────────── 8192]
                           │ scale by 0.25
                           ▼
  Mapped to:      [0 ─────── 2048]  ← fits in trained range ✓
                   ↑ more densely packed
```

$$\text{pos}_\text{mapped} = \text{pos} \times \frac{L_\text{train}}{L_\text{new}}$$

| Approach | Position 8000 maps to | In trained range? |
|---|---|---|
| Extrapolation | 8000 (unchanged) | No ($> 2048$) |
| **Interpolation** | $8000 \times 0.25 = 2000$ | **Yes** |

---
---

# Quiz 4, Question 20 — NTK-Aware Scaling Downside

> **What is the main downside of NTK-Aware scaling without the "YaRN" fix?**
>
> **B. It causes the attention scores (logits) to become too small, confusing the Softmax.**

---

## Sub-questions

### What is NTK-Aware scaling?

Instead of uniformly compressing all position indices (like PI), NTK-Aware scaling modifies the **base frequency** of RoPE:

$$\theta'_i = \theta_i \cdot \alpha^{-2i/d}$$

This changes the wavelengths of different dimensions non-uniformly — high frequencies are adjusted more than low frequencies.

### What goes wrong without YaRN?

The modified frequencies change the magnitude of attention scores. When rotations become too fine-grained, the dot product $q_m^T k_n$ produces smaller values. Smaller attention logits → flatter softmax distributions → the model cannot distinguish relevant from irrelevant tokens.

### What is YaRN?

"Yet another RoPE extensioN" — applies a temperature correction and interpolation thresholds to fix the attention score magnitude problem.

---

## Main answer

**B. It causes the attention scores (logits) to become too small, confusing the Softmax.**

```
NTK scaling problem:

  Normal RoPE:     qᵀk → reasonable scores → [0.6, 0.1, 0.3] (clear winner)

  NTK (no fix):    qᵀk → shrunken scores   → [0.34, 0.33, 0.33] (nearly uniform)
                                                 ↑
                                          Softmax can't distinguish tokens
                                          → attention is "confused"

  NTK + YaRN:      qᵀk → corrected scores  → [0.6, 0.1, 0.3] (clear winner ✓)
```

| Method | Attention scores | Softmax behavior | Fix needed? |
|---|---|---|---|
| Position Interpolation | Normal magnitude | Sharp | No |
| NTK-Aware (raw) | Too small | Flat/confused | Yes (YaRN) |
| NTK + YaRN | Corrected | Sharp | No |

---
---

# Quiz 4, Question 21 — Single-Head Attention Calculation (Essay)

> **Single-Head Attention Calculation:** Consider a single attention head with dimension $d_k = 4$. Given:
> $Q = [1, 0, 2, 1], \quad K = [2, 0, 1, 1]$
>
> Calculate the Scaled Dot-Product Attention Score (scalar) before the Softmax is applied. (Recall formula: $\frac{Q \cdot K^T}{\sqrt{d_k}}$).

---

## Sub-questions

### What is a dot product?

$$Q \cdot K = \sum_{i=1}^{d_k} Q_i \cdot K_i$$

Element-wise multiply, then sum.

### What is the scaling factor?

$$\sqrt{d_k} = \sqrt{4} = 2$$

---

## Main answer

### Step 1: Compute the dot product

$$Q \cdot K = (1 \times 2) + (0 \times 0) + (2 \times 1) + (1 \times 1)$$

$$= 2 + 0 + 2 + 1 = 5$$

### Step 2: Apply scaling

$$\frac{Q \cdot K}{\sqrt{d_k}} = \frac{5}{\sqrt{4}} = \frac{5}{2} = 2.5$$

```
Computation:

  Q = [1, 0, 2, 1]
  K = [2, 0, 1, 1]
       ↓  ↓  ↓  ↓
  products: [2, 0, 2, 1]  → sum = 5
                                │
                           ÷ √4 = 2
                                │
                           Result: 2.5
```

| Step | Computation | Result |
|---|---|---|
| Dot product | $1(2) + 0(0) + 2(1) + 1(1)$ | 5 |
| Scaling factor | $\sqrt{d_k} = \sqrt{4}$ | 2 |
| **Scaled score** | $5 / 2$ | **2.5** |

---
---

# Quiz 4, Question 22 — RoPE Matrix Rotation (Essay)

> **RoPE Matrix Rotation:** You are given a 2D embedding vector $x = [1, \sqrt{3}]$. Apply Rotary Positional Embedding (RoPE) for position $m = 1$ with a base rotation angle $\theta = 30°$ ($\pi/6$). Calculate the new coordinates $[x'_1, x'_2]$.

---

## Sub-questions

### What is the RoPE rotation matrix?

$$R_\theta = \begin{pmatrix} \cos\theta & -\sin\theta \\ \sin\theta & \cos\theta \end{pmatrix}$$

### What are $\cos(30°)$ and $\sin(30°)$?

$$\cos(30°) = \frac{\sqrt{3}}{2}, \quad \sin(30°) = \frac{1}{2}$$

### How is the rotation applied?

$$\begin{pmatrix} x'_1 \\ x'_2 \end{pmatrix} = \begin{pmatrix} \cos\theta & -\sin\theta \\ \sin\theta & \cos\theta \end{pmatrix} \begin{pmatrix} x_1 \\ x_2 \end{pmatrix}$$

---

## Main answer

### Step 1: Set up the matrix multiplication

$$\begin{pmatrix} x'_1 \\ x'_2 \end{pmatrix} = \begin{pmatrix} \frac{\sqrt{3}}{2} & -\frac{1}{2} \\ \frac{1}{2} & \frac{\sqrt{3}}{2} \end{pmatrix} \begin{pmatrix} 1 \\ \sqrt{3} \end{pmatrix}$$

### Step 2: Compute $x'_1$

$$x'_1 = 1 \cdot \frac{\sqrt{3}}{2} - \sqrt{3} \cdot \frac{1}{2} = \frac{\sqrt{3}}{2} - \frac{\sqrt{3}}{2} = 0$$

### Step 3: Compute $x'_2$

$$x'_2 = 1 \cdot \frac{1}{2} + \sqrt{3} \cdot \frac{\sqrt{3}}{2} = \frac{1}{2} + \frac{3}{2} = 2$$

### Result: $[x'_1, x'_2] = [0, 2]$

```
Geometric interpretation:

     ↑ y
   2 ┤       ● x' = [0, 2]   (after rotation)
     │      ╱
     │     ╱  30° rotation
  √3 ┤   ● x = [1, √3]       (before rotation)
     │  ╱
     │ ╱
     │╱
     └────┼────────▶ x
          1

  ||x|| = √(1² + (√3)²) = √4 = 2
  ||x'|| = √(0² + 2²) = 2             ← magnitude preserved ✓
  Angle shifted by 30°                 ← orientation changed ✓
```

| Component | Formula | Value |
|---|---|---|
| $x'_1$ | $1 \cdot \frac{\sqrt{3}}{2} - \sqrt{3} \cdot \frac{1}{2}$ | $0$ |
| $x'_2$ | $1 \cdot \frac{1}{2} + \sqrt{3} \cdot \frac{\sqrt{3}}{2}$ | $2$ |
| Magnitude before | $\sqrt{1 + 3}$ | $2$ |
| Magnitude after | $\sqrt{0 + 4}$ | $2$ (preserved) |

---
---

# Quiz 4, Question 23 — SwiGLU Parameter Scaling (Essay)

> **SwiGLU Parameter Scaling:** In the original Transformer, the FFN hidden dimension ($d_{ffn}$) is traditionally set to $4 \times d_\text{model}$. Llama 3 uses a SwiGLU FFN, which introduces an additional gating matrix. Suppose you want the SwiGLU FFN block to contain exactly **10 percent** more parameters than a standard ReLU FFN block. Calculate the exact algebraic scaling factor for the SwiGLU hidden dimension ($d_\text{hidden}$) in terms of $d_\text{model}$.

---

## Sub-questions

### What is the standard ReLU FFN?

$$\text{FFN}(x) = W_2 \cdot \text{ReLU}(W_1 x + b_1) + b_2$$

Two weight matrices:
- $W_1 \in \mathbb{R}^{d_\text{model} \times 4d_\text{model}}$
- $W_2 \in \mathbb{R}^{4d_\text{model} \times d_\text{model}}$

$$\text{Params}_\text{ReLU} = 2 \times d_\text{model} \times 4d_\text{model} = 8d_\text{model}^2$$

### What is SwiGLU?

$$\text{SwiGLU}(x) = (\text{Swish}(W_\text{gate} \cdot x) \odot (W_\text{up} \cdot x)) \cdot W_\text{down}$$

Three weight matrices (Gate, Up, Down), each of size $d_\text{model} \times d_\text{hidden}$:

$$\text{Params}_\text{SwiGLU} = 3 \times d_\text{model} \times d_\text{hidden} = 3d_\text{model} \cdot d_\text{hidden}$$

### What does "10% more parameters" mean?

$$\text{Params}_\text{SwiGLU} = 1.10 \times \text{Params}_\text{ReLU}$$

$$3d_\text{model} \cdot d_\text{hidden} = 1.10 \times 8d_\text{model}^2 = 8.8d_\text{model}^2$$

---

## Main answer

### Solve for $d_\text{hidden}$

$$3d_\text{model} \cdot d_\text{hidden} = 8.8d_\text{model}^2$$

$$d_\text{hidden} = \frac{8.8d_\text{model}^2}{3d_\text{model}} = \frac{8.8}{3} d_\text{model} = \frac{44}{15} d_\text{model}$$

$$d_\text{hidden} \approx 2.933 \, d_\text{model}$$

```
Parameter count comparison:

  Standard ReLU FFN:
    2 matrices × (d_model × 4·d_model) = 8·d²_model

  Target (10% more):
    1.10 × 8·d²_model = 8.8·d²_model

  SwiGLU FFN:
    3 matrices × (d_model × d_hidden) = 3·d_model·d_hidden

  Solving:
    3·d_model·d_hidden = 8.8·d²_model
    d_hidden = (8.8/3)·d_model = (44/15)·d_model ≈ 2.933·d_model
```

| FFN Type | Matrices | Hidden dim | Total params |
|---|---|---|---|
| ReLU (standard) | 2 | $4d_\text{model}$ | $8d_\text{model}^2$ |
| SwiGLU (10% more) | 3 | $\frac{44}{15}d_\text{model}$ | $8.8d_\text{model}^2$ |

---
---

# Quiz 4, Question 24 — Positional Wavelength Calculation (Essay)

> **Positional Wavelength Calculation:** The wavelength of the sinusoidal encoding for a given dimension index $i$ is determined by the denominator term: $10000^{2i/d_\text{model}}$. For a model with $d_\text{model} = 512$:
>
> (a) Calculate the denominator value for dimension index $i = 256$ (the midpoint).
> (b) Briefly explain what this value implies about the "speed" (frequency) of the sine wave at this dimension compared to index $i = 0$.

---

## Sub-questions

### What is the sinusoidal wavelength formula?

$$\text{wavelength}_i = 2\pi \times 10000^{2i/d_\text{model}}$$

The denominator $10000^{2i/d_\text{model}}$ controls how fast the sine/cosine oscillates at dimension $i$.

### What happens at $i = 0$?

$$10000^{2 \cdot 0 / 512} = 10000^0 = 1$$

Wavelength $= 2\pi \cdot 1 = 2\pi$. The sine wave completes a full cycle every $2\pi \approx 6.28$ positions — **highest frequency** (fastest oscillation).

### What is a geometric progression of wavelengths?

Low $i$ → small denominator → short wavelength → high frequency (fast changes).
High $i$ → large denominator → long wavelength → low frequency (slow changes).

---

## Main answer

### Part (a): Denominator at $i = 256$

$$10000^{2 \times 256 / 512} = 10000^{512/512} = 10000^1 = 10000$$

### Part (b): Interpretation

$$\text{At } i = 0: \quad \text{denominator} = 1 \quad \rightarrow \text{wavelength} = 2\pi$$

$$\text{At } i = 256: \quad \text{denominator} = 10000 \quad \rightarrow \text{wavelength} = 2\pi \times 10000 \approx 62{,}832$$

The sine wave at dimension 256 is **10,000 times slower** than at dimension 0. It completes one full cycle every $\sim 62{,}832$ positions — capturing only very **global** positional patterns.

```
Frequency spectrum across dimensions:

  dim 0:   ∿∿∿∿∿∿∿∿∿∿∿∿∿∿∿∿  (fast oscillation, wavelength = 2π)
  dim 64:  ∿∿∿∿∿∿∿∿            (medium-fast)
  dim 128: ∿∿∿∿                  (medium)
  dim 256: ∿∿                    (slow, wavelength = 2π × 10000)
  dim 511: ∿                     (slowest, wavelength = 2π × 10000)

  Low dims → fine-grained position (nearby tokens differ)
  High dims → coarse position (only distant tokens differ)
```

| Dimension $i$ | Denominator $10000^{2i/512}$ | Wavelength | Role |
|---|---|---|---|
| 0 | 1 | $2\pi$ | Fine-grained (local position) |
| 128 | 100 | $200\pi$ | Medium |
| **256** | **10,000** | **$20{,}000\pi$** | **Coarse (global position)** |

---
---

# Quiz 4, Question 25 — Causal Mask and Information Leakage (Essay)

> **The Causal Mask and Information Leakage:** In decoder-only Transformers, a causal mask is applied to the raw attention scores ($Q \cdot K^T$) before the softmax operation.
>
> First, write the Softmax definition. Second, explain why we add $-\infty$ instead of multiplying by 0. What would happen to the probability distribution if we multiplied by 0 before the softmax?

---

## Sub-questions

### What is the Softmax function?

$$P(z_i) = \frac{e^{z_i}}{\sum_j e^{z_j}}$$

Converts raw scores $z$ into a probability distribution. Every output is positive, and they sum to 1.

### What does $e^{-\infty}$ equal?

$$e^{-\infty} = 0$$

Guarantees the numerator for that position is exactly zero → attention weight = 0.

### What does $e^{0}$ equal?

$$e^{0} = 1$$

A pre-softmax value of 0 maps to a **positive** numerator of 1 → the position gets nonzero attention weight.

---

## Main answer

### Softmax Definition

$$P(z_i) = \frac{\exp(z_i)}{\sum_j \exp(z_j)}$$

### Why add $-\infty$ (not multiply by 0)?

**Adding $-\infty$:**

$$z_\text{masked} = -\infty \implies e^{-\infty} = 0 \implies P(\text{future token}) = \frac{0}{\sum_j e^{z_j}} = 0$$

The future token receives **exactly zero** attention. No information leaks.

**Multiplying by 0 (the mistake):**

$$z_\text{masked} = z \times 0 = 0 \implies e^{0} = 1 \implies P(\text{future token}) = \frac{1}{\sum_j e^{z_j}} > 0$$

The future token receives **positive** attention weight! The model would "see" future tokens and incorporate their information — destroying the autoregressive property.

```
Correct masking (add -∞):

  Scores:  [2.0, 1.5, -∞, -∞]
  exp:     [7.4, 4.5,  0,  0]
  Softmax: [0.62, 0.38, 0, 0]   ← future tokens = exactly 0 ✓


Wrong masking (multiply by 0):

  Scores:  [2.0, 1.5, 0, 0]
  exp:     [7.4, 4.5, 1, 1]
  Softmax: [0.53, 0.32, 0.07, 0.07]  ← future tokens > 0 ✗ LEAKAGE!
```

```
Consequence of information leakage:

  multiply by 0 ──▶ e⁰ = 1 ──▶ P(future) > 0 ──▶ model sees future
       ↓
  During training: model "cheats" by reading ahead
  During generation: no future exists → mismatch → garbage output
```

| Masking method | Pre-softmax | Post-softmax | Information leak? |
|---|---|---|---|
| **Add $-\infty$** | $-\infty$ | $\frac{e^{-\infty}}{\sum} = 0$ | **No** |
| Multiply by 0 | $0$ | $\frac{e^0}{\sum} = \frac{1}{\sum} > 0$ | Yes |

---
