# Quiz 7: Fine-tuning and Parameter Efficiency — Tutor Answers (Q1–Q25)

---

# Quiz 7, Question 1 — Why LoRA Uses an Optimizer Instead of SVD

> **Why does the practical implementation of LoRA use an optimizer to learn the A and B adapter matrices instead of directly calculating the Singular Value Decomposition (SVD) of the weight update?**
>
> **B. Because calculating the SVD of massive matrices (e.g., 4096 x 4096) at every training step would be computationally prohibitive and incredibly slow.**

---

## Sub-questions

### What is LoRA (Low-Rank Adaptation)?

$$W_\text{new} = W_\text{frozen} + \Delta W \approx W_\text{frozen} + A \cdot B$$

A fine-tuning method that freezes the original weight matrix $W$ and learns a low-rank update $\Delta W = AB$, where $A \in \mathbb{R}^{d \times r}$ and $B \in \mathbb{R}^{r \times d}$, with rank $r \ll d$.

### What is Singular Value Decomposition (SVD)?

$$M = U \Sigma V^T$$

Decomposes any matrix $M$ into three matrices: $U$ (left singular vectors), $\Sigma$ (diagonal of singular values), $V^T$ (right singular vectors). The best rank-$r$ approximation is obtained by keeping only the top $r$ singular values.

### What is the computational cost of SVD?

$$\text{SVD of } M \in \mathbb{R}^{m \times n}: \quad O(\min(m^2 n, \, mn^2))$$

For a $4096 \times 4096$ weight matrix:

$$O(4096^3) \approx 6.87 \times 10^{10} \text{ operations}$$

**Numeric example:** At $10^{12}$ FLOPS (GPU throughput), one SVD of a $4096 \times 4096$ matrix takes $\sim 0.07$ seconds. A Transformer with 64 attention layers × 4 weight matrices = 256 SVDs per step → $\sim 18$ seconds per training step (vs. milliseconds for a normal forward/backward).

### What does the optimizer do instead?

$$A \leftarrow A - \alpha \frac{\partial \mathcal{L}}{\partial A}, \quad B \leftarrow B - \alpha \frac{\partial \mathcal{L}}{\partial B}$$

Standard gradient descent learns $A$ and $B$ jointly during training. The gradients flow through the normal backpropagation chain — no extra decomposition required. Cost is folded into the existing training loop.

---

## Main answer

**B. Because calculating SVD of massive matrices at every training step would be computationally prohibitive and incredibly slow.**

```
SVD approach (impractical):

  Every training step:
  ┌─────────────┐    ┌──────────────┐    ┌──────────┐
  │ Compute ΔW  │───▶│ SVD(ΔW)      │───▶│ Truncate │
  │ (4096×4096) │    │ O(d³) ≈ 10¹⁰ │    │ to rank r│
  └─────────────┘    └──────────────┘    └──────────┘
                      ↑ BOTTLENECK
                      prohibitively slow per step


Optimizer approach (LoRA, practical):

  Every training step:
  ┌──────────┐    ┌──────────┐    ┌──────────┐
  │ Forward  │───▶│ Backward │───▶│ Update   │
  │ W + AB   │    │ ∂L/∂A,   │    │ A, B via │
  │          │    │ ∂L/∂B    │    │ optimizer│
  └──────────┘    └──────────┘    └──────────┘
                   folded into normal training
```

| Approach | Cost per Step | Practical? | Quality |
|---|---|---|---|
| Direct SVD | $O(d^3)$ per matrix, per step | No | Optimal rank-$r$ |
| Optimizer (LoRA) | Same as normal training | Yes | Near-optimal (learned) |

---
---

# Quiz 7, Question 2 — Greedy Decoding Drawback

> **What is a major drawback of using Greedy Decoding for text generation?**
>
> **C. It often leads to repetitive, predictable loops and lacks creativity.**

---

## Sub-questions

### What is Greedy Decoding?

$$x_t = \arg\max_{v \in V} P(v \mid x_{<t})$$

At every step, select the single token with the highest probability. No randomness, no exploration.

**Numeric example:** Probabilities $[0.4, 0.35, 0.25]$ for tokens $[\text{the}, \text{a}, \text{an}]$ → always picks "the."

### What is a decoding strategy?

The algorithm that converts a probability distribution over the vocabulary into an actual token selection at each generation step. Different strategies trade off quality, diversity, and coherence.

### What causes repetitive loops?

Once the model produces a sequence like "the cat sat on the," the context now strongly predicts "cat sat on the" again — creating a self-reinforcing cycle. Greedy decoding has no mechanism to break out because it always picks the maximum.

---

## Main answer

**C. It often leads to repetitive, predictable loops and lacks creativity.**

```
Greedy decoding loop trap:

  "The cat sat on the" ──▶ P("cat") = highest
                              │
  "cat sat on the cat" ──▶ P("sat") = highest
                              │
  "sat on the cat sat" ──▶ P("on") = highest
                              │
           ... infinite loop ...
```

Greedy decoding is **deterministic**: same input → same output every time. No randomness means no diversity.

$$\text{Greedy: } x_t = \arg\max P(v \mid x_{<t}) \quad \text{(always same choice)}$$

$$\text{Sampling: } x_t \sim P(v \mid x_{<t}) \quad \text{(varied, creative output)}$$

| Property | Greedy Decoding | Sampling-based |
|---|---|---|
| Selection | Always $\arg\max$ | Probabilistic |
| Deterministic | Yes | No |
| Repetition risk | High (self-reinforcing loops) | Low |
| Creativity | None | Tunable via temperature |
| Use case | Short, factual answers | Creative writing, dialogue |

---
---

# Quiz 7, Question 3 — Temperature Below 1.0

> **When using probabilistic sampling, how does a Temperature value less than 1.0 (T < 1.0) affect the output?**
>
> **B. It sharpens the distribution, making the model more confident and deterministic.**

---

## Sub-questions

### What is Temperature in text generation?

$$P(x_i) = \frac{e^{z_i / T}}{\sum_j e^{z_j / T}}$$

A scalar $T > 0$ that divides the logits before Softmax. Controls the "spread" of the probability distribution.

### What happens when $T < 1$?

Dividing by $T < 1$ is equivalent to **multiplying** the logits by $1/T > 1$. This amplifies the differences between logits:

$$\frac{z_i}{T} > z_i \quad \text{when } T < 1$$

Larger differences after exponentiation → the highest-probability token dominates even more.

**Numeric example:** Logits $z = [2.0, 1.0]$, $T = 0.5$:

$$z / T = [4.0, 2.0] \quad \Rightarrow \quad P = \frac{[e^4, e^2]}{e^4 + e^2} = \frac{[54.6, 7.39]}{62.0} = [0.881, 0.119]$$

Compare with $T = 1.0$: $P = [0.731, 0.269]$. The distribution became sharper.

### What happens when $T > 1$?

Dividing by $T > 1$ shrinks the logit differences → distribution flattens → more uniform → more random output.

### What happens as $T \to 0$?

$$\lim_{T \to 0} P(x_i) = \begin{cases} 1 & \text{if } i = \arg\max(z) \\ 0 & \text{otherwise} \end{cases}$$

Converges to greedy decoding (all probability mass on the top token).

---

## Main answer

**B. It sharpens the distribution, making the model more confident and deterministic.**

```
Effect of temperature on distribution:

  P(x)
  ↑
  │  ╻                        T → 0 (greedy, all mass on top token)
  │  ┃
  │  ┃╻
  │  ┃┃
  │  ┃┃╻                      T = 0.5 (sharp)
  │ ╻┃┃┃
  │ ┃┃┃┃╻                     T = 1.0 (normal)
  │ ┃┃┃┃┃╻
  │ ┃┃┃┃┃┃╻╻                  T = 2.0 (flat, random)
  └─┸┸┸┸┸┸┸┸──────▶ tokens (sorted by probability)
```

$$T < 1: \quad \text{sharpen} \quad \xrightarrow{} \quad \text{confident, deterministic}$$
$$T = 1: \quad \text{unchanged} \quad \xrightarrow{} \quad \text{model's native distribution}$$
$$T > 1: \quad \text{flatten} \quad \xrightarrow{} \quad \text{random, creative}$$

| Temperature | Effect on Logits | Distribution Shape | Behavior |
|---|---|---|---|
| $T \to 0$ | $z/T \to \pm\infty$ | One-hot (greedy) | Deterministic |
| $T = 0.5$ | Doubled | Sharp | Confident |
| $T = 1.0$ | Unchanged | Normal | Balanced |
| $T = 2.0$ | Halved | Flat | Creative/random |
| $T \to \infty$ | $z/T \to 0$ | Uniform | Pure random |

---
---

# Quiz 7, Question 4 — Temperature Approaching Infinity

> **According to the mathematical ratio test, what happens to the relative probability of two tokens as the Temperature (T) approaches infinity?**
>
> **B. The probability ratio approaches 1:1, making all tokens equally likely.**

---

## Sub-questions

### What is the probability ratio of two tokens?

$$\frac{P(x_a)}{P(x_b)} = \frac{e^{z_a/T}}{e^{z_b/T}} = e^{(z_a - z_b)/T}$$

The ratio depends only on the logit difference $(z_a - z_b)$ and the temperature $T$.

### What happens to $e^{(z_a - z_b)/T}$ as $T \to \infty$?

$$\lim_{T \to \infty} \frac{z_a - z_b}{T} = 0 \quad \Rightarrow \quad \lim_{T \to \infty} e^{(z_a - z_b)/T} = e^0 = 1$$

The exponent shrinks to zero regardless of the logit difference. The ratio converges to 1.

**Numeric example:** $z_a = 10, z_b = 2$:

- $T = 1$: ratio $= e^{8} \approx 2981$ (token A overwhelmingly dominant)
- $T = 10$: ratio $= e^{0.8} \approx 2.23$
- $T = 100$: ratio $= e^{0.08} \approx 1.08$
- $T = 1000$: ratio $= e^{0.008} \approx 1.008$ (nearly equal)

---

## Main answer

**B. The probability ratio approaches 1:1, making all tokens equally likely.**

$$\lim_{T \to \infty} \frac{P(x_a)}{P(x_b)} = \lim_{T \to \infty} e^{(z_a - z_b)/T} = e^0 = 1$$

```
Probability ratio vs Temperature (z_a - z_b = 8):

  Ratio
  P(a)/P(b)
   ↑
  2981│●                              T = 1
      │
      │
   2.2│    ●                          T = 10
      │
   1.1│         ●────●────●────●──    T → ∞ (converges to 1)
   1.0│─ ─ ─ ─ ─ ─ ─ ─ ─ ─ ─ ─ ─ ─  equal probability line
      └───────────────────────────▶ T
      1    10   100  1000  ∞
```

As $T \to \infty$, all logit differences are divided into insignificance → uniform distribution over entire vocabulary → pure random sampling.

| $T$ | Exponent $(z_a-z_b)/T$ | Ratio $P(a)/P(b)$ | Behavior |
|---|---|---|---|
| 1 | 8.0 | 2981 | Token A dominates |
| 10 | 0.8 | 2.23 | A slightly favored |
| 100 | 0.08 | 1.08 | Nearly equal |
| $\infty$ | 0 | **1.0** | **Uniform** |

---
---

# Quiz 7, Question 5 — Top-K Sampling

> **How does Top-K sampling prevent the generation of gibberish?**
>
> **B. By applying a hard vertical cutoff and discarding all but the top K probability tokens.**

---

## Sub-questions

### What is the "long tail" problem in sampling?

A vocabulary of $V = 32{,}000$ tokens has thousands of low-probability tokens. When sampling from the full distribution, there is a non-trivial cumulative chance of selecting a nonsensical token from the tail.

### What is Top-K sampling?

$$\text{Top-K: } P'(x_i) = \begin{cases} \frac{P(x_i)}{\sum_{j \in \text{top-K}} P(x_j)} & \text{if } x_i \in \text{top-K} \\ 0 & \text{otherwise} \end{cases}$$

1. Sort tokens by probability.
2. Keep only the $K$ highest-probability tokens.
3. Set all other probabilities to 0.
4. Renormalize the remaining $K$ probabilities to sum to 1.

**Numeric example:** $K=3$, original distribution over 5 tokens: $[0.4, 0.3, 0.15, 0.1, 0.05]$.

Top-3: $[0.4, 0.3, 0.15]$, renormalized: $[0.471, 0.353, 0.176]$. Tokens 4 and 5 are excluded.

---

## Main answer

**B. By applying a hard vertical cutoff and discarding all but the top K probability tokens.**

```
Top-K filtering (K=3):

  P(x)
   ↑
  0.4│ ██                          ─┐
  0.3│ ██ ██                        │ KEPT (top 3)
  0.15│ ██ ██ ██                     ─┘
  0.1│ ██ ██ ██ ░░  ←── cutoff ──── DISCARDED
  0.05│ ██ ██ ██ ░░ ░░               DISCARDED
     └──┴──┴──┴──┴──▶ tokens (sorted)
      t1  t2  t3  t4  t5
```

After filtering, sampling only occurs over the $K$ most likely tokens → gibberish from the tail is impossible.

```
Pipeline:

  Full distribution (V=32000 tokens)
       │
       ▼  Sort by probability
  Sorted distribution
       │
       ▼  Keep top K, zero out rest
  Truncated distribution (K tokens)
       │
       ▼  Renormalize to sum = 1
  Final distribution
       │
       ▼  Sample
  Output token (guaranteed reasonable)
```

| Aspect | Full Sampling | Top-K Sampling |
|---|---|---|
| Candidate tokens | All $V$ | Only $K$ |
| Tail risk | High (gibberish possible) | Zero |
| Cutoff type | None | Fixed count ($K$) |
| Adapts to confidence | — | No (always $K$ tokens) |

---
---

# Quiz 7, Question 6 — Top-p (Nucleus) Sampling vs Top-K

> **What is the primary advantage of Top-p (Nucleus) Sampling over Top-K Sampling?**
>
> **A. It uses a dynamic cutoff that adjusts based on the model's confidence, rather than a fixed number of tokens.**

---

## Sub-questions

### What is Top-p (Nucleus) Sampling?

$$\text{Top-p: smallest set } S \text{ such that } \sum_{x_i \in S} P(x_i) \geq p$$

Instead of keeping a fixed count of $K$ tokens, keep the smallest set whose cumulative probability reaches threshold $p$ (e.g., $p = 0.9$).

### Why is a fixed K problematic?

When the model is very confident (one token has $P = 0.95$), $K = 50$ still considers 49 unlikely tokens — adding noise. When the model is uncertain (many tokens around $P = 0.02$), $K = 50$ may exclude viable options.

**Numeric example — confident model:** $P = [0.95, 0.02, 0.01, \dots]$
- Top-K ($K=50$): keeps 50 tokens (49 are near-zero noise)
- Top-p ($p=0.9$): keeps **1** token (0.95 > 0.9)

**Numeric example — uncertain model:** $P = [0.05, 0.04, 0.04, \dots]$
- Top-K ($K=50$): keeps exactly 50
- Top-p ($p=0.9$): keeps **~22** tokens (whatever reaches 90%)

---

## Main answer

**A. It uses a dynamic cutoff that adjusts based on the model's confidence, rather than a fixed number of tokens.**

```
Top-K (fixed K=3, regardless of confidence):

  Confident:              Uncertain:
  P(x)                   P(x)
  0.9│ ██                 0.05│ ██ ██ ██
     │ ██                     │ ██ ██ ██
  0.05│ ██ ██ ██               │ ██ ██ ██ ░░ ░░ ...
     └──┴──┴──▶              └──┴──┴──┴──┴──▶
      always 3                 always 3 (misses good tokens!)


Top-p (dynamic, p=0.9):

  Confident:              Uncertain:
  P(x)                   P(x)
  0.9│ ██  ← p reached!  0.05│ ██ ██ ██ ██ ██ ██ ██ ... ██
     │ ██                     │ (keeps ~22 tokens to reach p=0.9)
     └──▶                    └──────────────────────────▶
      1 token                  many tokens
```

$$\text{Top-K: } |S| = K \quad \text{(constant)}$$
$$\text{Top-p: } |S| = \text{variable, determined by } \sum P \geq p$$

| Property | Top-K | Top-p (Nucleus) |
|---|---|---|
| Cutoff mechanism | Fixed count $K$ | Cumulative probability $p$ |
| Adapts to confidence | No | Yes |
| Confident model ($P_\max = 0.95$) | Keeps $K$ tokens (noisy) | Keeps ~1 token (focused) |
| Uncertain model ($P_\max = 0.05$) | Keeps $K$ tokens (may miss good ones) | Keeps many tokens (explores) |
| Hyperparameter | $K$ (integer) | $p$ (float, e.g., 0.9) |

---
---

# Quiz 7, Question 7 — Beam Search Uses Log-Likelihoods

> **Why does Beam Search use log-likelihoods instead of multiplying raw probabilities?**
>
> **B. To prevent floating-point underflow caused by multiplying many small numbers.**

---

## Sub-questions

### What is Beam Search?

Maintains $B$ candidate sequences ("beams") and scores each by its cumulative probability:

$$\text{Score}(x_1, \dots, x_t) = P(x_1) \cdot P(x_2 \mid x_1) \cdot \ldots \cdot P(x_t \mid x_{<t})$$

At each step, expand all beams, score, keep top $B$.

### What is floating-point underflow?

$$\prod_{t=1}^{T} P(x_t) \quad \text{where each } P(x_t) \in (0, 1)$$

Multiplying hundreds of probabilities (each < 1) produces astronomically small numbers that fall below the minimum representable float ($\sim 10^{-308}$ for FP64, $\sim 10^{-38}$ for FP32), rounding to exactly 0.

**Numeric example:** If each token probability $\approx 0.1$, after 40 tokens: $0.1^{40} = 10^{-40}$ — underflows in FP32.

### What is the log-likelihood solution?

$$\log P(x_1, \dots, x_t) = \sum_{t=1}^{T} \log P(x_t \mid x_{<t})$$

Log converts multiplication to addition. Sums of negative numbers stay in a representable range.

**Numeric example:** $\log(0.1) = -2.3$, after 40 tokens: $40 \times (-2.3) = -92$ — perfectly representable.

---

## Main answer

**B. To prevent floating-point underflow caused by multiplying many small numbers.**

```
Multiplication approach (underflows):

  P(x₁) × P(x₂) × ... × P(x₄₀)
  = 0.1  × 0.1   × ... × 0.1
  = 10⁻⁴⁰  → underflows to 0 in FP32!


Log-likelihood approach (safe):

  log P(x₁) + log P(x₂) + ... + log P(x₄₀)
  = -2.3    + -2.3      + ... + -2.3
  = -92  → perfectly fine!
```

$$\text{Products: } \prod P \to 0 \quad \text{(underflow)}$$
$$\text{Log sums: } \sum \log P \to \text{finite negative number} \quad \text{(safe)}$$

| Property | Raw Probabilities | Log-Likelihoods |
|---|---|---|
| Operation | Multiply | Add |
| Range after 40 tokens | $\sim 10^{-40}$ | $\sim -92$ |
| Underflow risk | Yes (FP32 limit $\sim 10^{-38}$) | No |
| Comparison validity | Equivalent | Equivalent ($\log$ is monotonic) |
| Implementation | `score *= p` | `score += log(p)` |

---
---

# Quiz 7, Question 8 — Instruction Tuning

> **In the context of LLM training, what is the primary goal of Instruction Tuning?**
>
> **B. To learn a specific behavior by mapping instructions to precise, desired outputs.**

---

## Sub-questions

### What is pre-training?

$$\mathcal{L}_\text{pretrain} = -\sum_t \log P(x_t \mid x_{<t})$$

Unsupervised next-token prediction on massive text corpora. The model learns general language patterns but has no concept of following instructions — it just continues text.

### What is fine-tuning?

$$\theta_\text{fine} = \theta_\text{pretrain} - \alpha \nabla \mathcal{L}_\text{task}$$

Updating pre-trained weights on a smaller, task-specific dataset. Adapts the model's behavior toward a specific goal.

### What is Instruction Tuning specifically?

Fine-tuning on datasets of (instruction, response) pairs:

```
Input:  "Summarize the following article: [article text]"
Output: "[concise summary]"
```

The model learns the pattern: given an instruction in a specific format → produce the expected response.

---

## Main answer

**B. To learn a specific behavior by mapping instructions to precise, desired outputs.**

```
LLM training pipeline:

  Pre-training                    Instruction Tuning
  ┌──────────────────────┐       ┌──────────────────────────┐
  │ Internet text (TB)   │──────▶│ (instruction, response)  │
  │ Next-token prediction│       │ pairs (thousands)        │
  │                      │       │                          │
  │ Output: text         │       │ Output: follows          │
  │ completion engine    │       │ instructions             │
  └──────────────────────┘       └──────────────────────────┘
  "The capital of France"        "What is the capital of France?"
  → "is known for..."           → "The capital of France is Paris."
```

$$\text{Pre-trained: } P(\text{next token} \mid \text{context})$$
$$\text{Instruction-tuned: } P(\text{answer} \mid \text{instruction})$$

| Aspect | Pre-trained LLM | Instruction-tuned LLM |
|---|---|---|
| Training data | Raw text | (instruction, response) pairs |
| Behavior | Autocomplete | Follow instructions |
| Example output | Continues the sentence | Answers the question |
| Alignment | None | Aligned with user intent |

---
---

# Quiz 7, Question 9 — VRAM for 7B Parameters in FP16

> **Approximately how much VRAM is required strictly to load the weights of a 7-billion parameter model in FP16 precision?**
>
> **C. 14 GB**

---

## Sub-questions

### What is VRAM?

Video Random Access Memory — the GPU's dedicated memory. Model weights must reside in VRAM for GPU computation.

### What is FP16?

$$\text{FP16} = 16 \text{ bits} = 2 \text{ bytes per parameter}$$

### How to calculate weight memory?

$$\text{Memory} = \text{Parameters} \times \text{Bytes per Parameter}$$

---

## Main answer

**C. 14 GB**

$$7 \times 10^9 \;\text{params} \times 2 \;\text{bytes/param} = 14 \times 10^9 \;\text{bytes} = 14 \;\text{GB}$$

```
Quick calculation:

  7B params × 2 bytes/param (FP16) = 14 GB
```

| Precision | Bytes/Param | 7B Model |
|---|---|---|
| FP32 | 4 | 28 GB |
| **FP16 / BF16** | **2** | **14 GB** |
| INT8 | 1 | 7 GB |
| INT4 / NF4 | 0.5 | 3.5 GB |

---
---

# Quiz 7, Question 10 — Intrinsic Dimension Hypothesis

> **What does the "Intrinsic Dimension Hypothesis" suggest about fine-tuning LLMs?**
>
> **B. Task-specific fine-tuning resides in a low-dimensional subspace within the vast parameter space.**

---

## Sub-questions

### What is the parameter space of an LLM?

$$\theta \in \mathbb{R}^{d}, \quad d \sim 10^9 \text{ to } 10^{12}$$

The full set of trainable parameters. A 7B model lives in a $7 \times 10^9$-dimensional space.

### What is an "intrinsic dimension"?

The minimum number of dimensions needed to capture the task-relevant variation during fine-tuning. If the intrinsic dimension is $k \ll d$, then the weight updates $\Delta\theta$ during fine-tuning effectively lie on a $k$-dimensional submanifold of the full $d$-dimensional space.

### Why does this matter for LoRA?

If fine-tuning only needs a low-rank subspace, we can parameterize $\Delta W$ as a rank-$r$ matrix ($r \ll d$) without losing performance. This is exactly what LoRA exploits.

$$\Delta W = A \cdot B, \quad A \in \mathbb{R}^{d \times r}, \; B \in \mathbb{R}^{r \times d}, \quad r \ll d$$

---

## Main answer

**B. Task-specific fine-tuning resides in a low-dimensional subspace within the vast parameter space.**

```
Visualization of intrinsic dimension:

  Full parameter space (d = 7 billion dimensions)
  ┌─────────────────────────────────────────┐
  │                                         │
  │         ╱─────────────╲                 │
  │        ╱  Low-rank     ╲                │
  │       │   subspace      │               │
  │       │  (r dimensions) │  ← fine-tuning│
  │       │   r << d        │    lives HERE │
  │        ╲               ╱                │
  │         ╲─────────────╱                 │
  │                                         │
  └─────────────────────────────────────────┘
```

$$\text{Full FT: update all } d \text{ parameters}$$
$$\text{LoRA: update only } r \text{ directions (intrinsic subspace)}$$

| Concept | Full Fine-tuning | LoRA (exploiting Intrinsic Dimension) |
|---|---|---|
| Trainable params | All $d$ ($\sim 10^9$) | $2 \times d \times r$ ($\sim 10^4$ to $10^6$) |
| Subspace | Full $d$-dimensional | Low-rank $r$-dimensional |
| Hypothesis | — | Sufficient for task adaptation |
| Memory | Huge | Tiny |

---
---

# Quiz 7, Question 11 — How LoRA Reduces Trainable Parameters

> **How does LoRA (Low-Rank Adaptation) reduce the number of trainable parameters?**
>
> **B. By approximating the large weight update matrix (delta W) as the product of two much smaller matrices (A x B).**

---

## Sub-questions

### What is the full weight update matrix?

$$\Delta W \in \mathbb{R}^{d_\text{out} \times d_\text{in}}$$

In standard fine-tuning, every element of $\Delta W$ is trainable. For a $d \times d$ matrix, that is $d^2$ parameters.

**Numeric example:** $d = 4096$ → $\Delta W$ has $4096^2 = 16{,}777{,}216$ trainable parameters.

### What is a low-rank factorization?

$$\Delta W = A \cdot B, \quad A \in \mathbb{R}^{d \times r}, \; B \in \mathbb{R}^{r \times d}$$

Express the $d \times d$ matrix as a product of two thin matrices with a shared inner dimension $r \ll d$.

**Parameters:** $d \cdot r + r \cdot d = 2dr$.

**Numeric example:** $d = 4096, r = 4$: $2 \times 4096 \times 4 = 32{,}768$ (vs. $16{,}777{,}216$).

### What does "rank" mean?

$$\text{rank}(\Delta W) = r$$

The maximum number of linearly independent rows (or columns). A rank-$r$ matrix can be expressed as a sum of $r$ outer products. Low rank means the update is constrained to a $r$-dimensional subspace.

---

## Main answer

**B. By approximating $\Delta W$ as the product of two much smaller matrices $A \times B$.**

$$W_\text{new} = W_\text{frozen} + \underbrace{A \cdot B}_{\Delta W}$$

```
Full fine-tuning:                    LoRA:

  ΔW (d × d)                        A (d × r)    B (r × d)
  ┌────────────────┐                ┌──┐          ┌────────────────┐
  │                │                │  │          │                │
  │  d² parameters │                │  │ r        │  r             │
  │                │                │  │          │                │
  │  d = 4096:     │                │  │          └────────────────┘
  │  16,777,216    │                └──┘
  └────────────────┘                d×r + r×d = 2dr parameters
                                     d = 4096, r = 4: 32,768
```

$$\text{Reduction} = 1 - \frac{2dr}{d^2} = 1 - \frac{2r}{d}$$

**Numeric example:** $d = 4096, r = 4$:

$$\text{Reduction} = 1 - \frac{2 \times 4}{4096} = 1 - \frac{8}{4096} = 1 - 0.00195 = 99.8\%$$

| Approach | Trainable Parameters | For $d=4096, r=4$ |
|---|---|---|
| Full FT | $d^2$ | 16,777,216 |
| **LoRA** | $2dr$ | **32,768** |
| **Reduction** | $1 - 2r/d$ | **99.8%** |

---
---

# Quiz 7, Question 12 — The Sigma Matrix in SVD

> **In Singular Value Decomposition (A = U $\Sigma$ V$^T$), what does the diagonal $\Sigma$ matrix represent?**
>
> **B. The scale/importance of the information, sorted from largest to smallest.**

---

## Sub-questions

### What is SVD?

$$A = U \Sigma V^T$$

- $U \in \mathbb{R}^{m \times m}$: left singular vectors (orthonormal columns)
- $\Sigma \in \mathbb{R}^{m \times n}$: diagonal matrix of singular values $\sigma_1 \geq \sigma_2 \geq \dots \geq 0$
- $V^T \in \mathbb{R}^{n \times n}$: right singular vectors (orthonormal rows)

### What are singular values?

$$\Sigma = \text{diag}(\sigma_1, \sigma_2, \dots, \sigma_{\min(m,n)})$$

Non-negative scalars arranged in descending order. Each $\sigma_i$ quantifies how much "information" or "variance" the $i$-th component contributes to the matrix.

**Numeric example:** $\Sigma = \text{diag}(10, 8, 2, 1)$ — the first two components carry most of the information.

### How do singular values relate to low-rank approximation?

The best rank-$r$ approximation keeps the top $r$ singular values and discards the rest. The Eckart-Young-Mirsky theorem guarantees this is optimal:

$$\|A - A_r\|_F^2 = \sum_{i=r+1}^{n} \sigma_i^2$$

---

## Main answer

**B. The scale/importance of the information, sorted from largest to smallest.**

```
SVD decomposition:

  A    =    U      ×     Σ          ×    Vᵀ
 (m×n)   (m×m)        (m×n)           (n×n)
                    ┌              ┐
                    │ σ₁  0  0  0 │   σ₁ ≥ σ₂ ≥ σ₃ ≥ σ₄
                    │ 0  σ₂  0  0 │
                    │ 0   0 σ₃  0 │   ← importance decreasing
                    │ 0   0  0 σ₄ │
                    └              ┘
```

```
Information distribution (σ = [10, 8, 2, 1]):

  σᵢ
   ↑
  10│ ██
   8│ ██ ██
   │ ██ ██
   2│ ██ ██ ██
   1│ ██ ██ ██ ██
   └──┴──┴──┴──▶ component
     σ₁  σ₂  σ₃  σ₄

  Top 2 capture: (10²+8²)/(10²+8²+2²+1²) = 164/169 ≈ 97%
```

| Component of SVD | Symbol | Role |
|---|---|---|
| $U$ | Left singular vectors | Rotation / direction in output space |
| $\Sigma$ | **Singular values** | **Scale / importance (descending)** |
| $V^T$ | Right singular vectors | Rotation / direction in input space |

---
---

# Quiz 7, Question 13 — LoRA Matrix Initialization

> **In a practical PyTorch implementation of LoRA, how are the A_lora and B_lora matrices initialized?**
>
> **C. A_lora is initialized with random tiny numbers, and B_lora is initialized to zero.**

---

## Sub-questions

### Why does initialization matter for LoRA?

$$\Delta W = A \cdot B$$

At the start of training, the adapter should produce $\Delta W = 0$ so the model begins with its pre-trained behavior. This ensures LoRA starts from a known-good state and doesn't corrupt the frozen weights.

### Why is B initialized to zero?

$$B = 0 \implies \Delta W = A \cdot 0 = 0 \quad \forall \; A$$

Regardless of what $A$ contains, the product is zero. This guarantees $\Delta W = 0$ at initialization.

### Why is A initialized with small random values (not also zero)?

If both $A = 0$ and $B = 0$, gradients would be zero for both:

$$\frac{\partial \mathcal{L}}{\partial A} = \frac{\partial \mathcal{L}}{\partial (\Delta W)} \cdot B^T = \frac{\partial \mathcal{L}}{\partial (\Delta W)} \cdot 0 = 0$$

No parameter would ever update — training is dead. By initializing $A$ with small random values, gradients flow through $B$ and training can begin:

$$\frac{\partial \mathcal{L}}{\partial B} = A^T \cdot \frac{\partial \mathcal{L}}{\partial (\Delta W)} \neq 0 \quad \text{(because } A \neq 0\text{)}$$

---

## Main answer

**C. A_lora is initialized with random tiny numbers, and B_lora is initialized to zero.**

```python
# Typical LoRA initialization:
A_lora = nn.Parameter(torch.randn(d, r) * 0.01)  # small random
B_lora = nn.Parameter(torch.zeros(r, d))           # zero
```

```
Initialization logic:

  A = small random    B = zeros
       │                  │
       ▼                  ▼
  ΔW = A × B = A × 0 = 0  ← model starts with pretrained behavior

  After first backward:
  ∂L/∂B = Aᵀ × ∂L/∂(ΔW) ≠ 0  ← B starts learning
  ∂L/∂A = ∂L/∂(ΔW) × Bᵀ ≈ 0  ← A starts learning once B ≠ 0
```

| Matrix | Initialization | Why |
|---|---|---|
| $A$ | Small random (e.g., $\mathcal{N}(0, 0.01)$) | Enables gradient flow to $B$ |
| $B$ | Zero | Guarantees $\Delta W = 0$ at start |
| Both zero? | Dead training ($\nabla = 0$ for both) | Not viable |
| Both random? | $\Delta W \neq 0$ at start (corrupts model) | Not desirable |

---
---

# Quiz 7, Question 14 — Alpha Scaling Factor in LoRA

> **What is the purpose of the Alpha ($\alpha$) scaling factor in LoRA?**
>
> **B. To control the magnitude of the low-rank update relative to the pretrained weights (scaling factor = $\alpha/r$).**

---

## Sub-questions

### What is the full LoRA forward pass with alpha?

$$h = W_\text{frozen} \cdot x + \frac{\alpha}{r} \cdot (A \cdot B) \cdot x$$

The adapter output is scaled by $\alpha / r$ before being added to the frozen weight output.

### What is $r$?

The rank of the LoRA adapter — the inner dimension of matrices $A$ and $B$. Typical values: $r \in \{4, 8, 16, 32, 64\}$.

### Why divide by $r$?

When you increase $r$, the adapter has more capacity and the magnitude of $AB$ grows. Dividing by $r$ normalizes this so that changing $r$ doesn't require retuning the learning rate.

### What does $\alpha$ control?

A hyperparameter (typically $\alpha = 16$ or $\alpha = 32$) that scales how much the adapter contributes relative to the frozen weights. Larger $\alpha$ → larger update → more deviation from the pre-trained model.

**Numeric example:** $\alpha = 16, r = 4$ → scaling factor $= 16/4 = 4$. The adapter output is amplified $4\times$.

---

## Main answer

**B. To control the magnitude of the low-rank update relative to the pretrained weights (scaling factor = $\alpha/r$).**

$$W_\text{effective} = W_\text{frozen} + \frac{\alpha}{r} \cdot A \cdot B$$

```
LoRA scaling:

  Input x
    │
    ├────────────────────────▶ W_frozen · x  (pretrained path)
    │                              │
    │                              │
    └──▶ A·B·x ──▶ × (α/r) ──────┤
          adapter      scaling     │
          output       factor      ▼
                               h = W·x + (α/r)·AB·x
```

The $\alpha/r$ factor decouples two decisions:
1. **$r$** controls adapter capacity (how many dimensions)
2. **$\alpha$** controls adapter influence (how much it affects output)

| Parameter | Role | Effect of Increasing |
|---|---|---|
| $r$ | Adapter rank / capacity | More expressive, more params |
| $\alpha$ | Scaling / influence | Larger deviation from pretrained |
| $\alpha / r$ | Effective scaling factor | Controls update magnitude |

---
---

# Quiz 7, Question 15 — LoRA Inference Latency

> **What latency overhead is introduced by a deployed LoRA adapter during inference?**
>
> **C. Zero latency, because the A x B matrices are permanently merged into the base weights.**

---

## Sub-questions

### What happens during LoRA training?

$$h = W_\text{frozen} \cdot x + \frac{\alpha}{r} \cdot A \cdot B \cdot x$$

Two separate paths: the frozen weight path and the adapter path. The adapter requires an extra matrix multiplication.

### What happens at deployment (inference)?

$$W_\text{merged} = W_\text{frozen} + \frac{\alpha}{r} \cdot A \cdot B$$

Pre-compute the product $A \cdot B$, scale it, and **add it directly to the frozen weights**. The result is a single weight matrix of the same shape as the original.

$$h = W_\text{merged} \cdot x \quad \text{(identical to standard inference)}$$

### Why is this zero overhead?

After merging, the LoRA adapter no longer exists as a separate entity. The model is architecturally identical to the original — same number of matrices, same shapes, same computation graph. No extra operations at inference time.

---

## Main answer

**C. Zero latency, because the A x B matrices are permanently merged into the base weights.**

```
Training (two paths):

  x ──▶ W_frozen · x ────────────┐
  │                              │
  └──▶ (α/r) · A · B · x ───────┤
       (extra computation)       ▼
                              h = sum


Deployment (merged, single path):

  W_merged = W_frozen + (α/r)·A·B    ← one-time merge

  x ──▶ W_merged · x ──▶ h           ← identical to original model
         (no extra computation)
```

$$\text{Merge: } W_\text{merged} = W + \frac{\alpha}{r} AB \quad \text{(one-time, before deployment)}$$

| Phase | Computation | Latency Overhead |
|---|---|---|
| Training | $Wx + \frac{\alpha}{r}ABx$ (two paths) | Yes (extra matmul) |
| **Inference (merged)** | $W_\text{merged} \cdot x$ (one path) | **Zero** |
| Architecture change | None after merge | Same as original model |

---
---

# Quiz 7, Question 16 — QLoRA Base Weight Data Type

> **What optimized data type does QLoRA use to compress the base model weights?**
>
> **C. NormalFloat4 (NF4)**

---

## Sub-questions

### What is QLoRA?

$$\text{QLoRA} = \text{Quantized base weights} + \text{LoRA adapters (trainable in BF16)}$$

Quantized Low-Rank Adaptation. Compresses the frozen base weights to 4-bit precision, then trains LoRA adapters in higher precision. Enables fine-tuning of large models on consumer GPUs.

### What is quantization?

$$\text{Quantize: FP32/FP16} \rightarrow \text{INT4/NF4}$$

Reducing the precision (number of bits) used to represent each weight value. Fewer bits → less memory, but introduces approximation error.

### Why NF4 instead of standard INT4?

Neural network weights follow an approximately **Gaussian distribution** (centered near zero, few extreme values). Standard INT4 uses equally spaced bins — wasting resolution on the tails where few weights exist. NF4 is designed specifically for this distribution.

---

## Main answer

**C. NormalFloat4 (NF4)**

```
Weight distribution vs quantization bins:

  Standard INT4 (equally spaced):

  frequency
   ↑    ╱╲
   │   ╱  ╲
   │  ╱    ╲
   │ ╱      ╲
   │╱        ╲
   └┼──┼──┼──┼──┼──┼──┼──┼──▶ weight value
    bins are evenly spaced (wasteful at tails)


  NormalFloat4 (optimized for Gaussian):

  frequency
   ↑    ╱╲
   │   ╱  ╲
   │  ╱    ╲
   │ ╱      ╲
   │╱        ╲
   └┼┼┼┼──┼──┼┼┼┼──▶ weight value
    dense near center    sparse at tails
    (where most weights live)
```

NF4 places more quantization bins near zero (where weight density is highest) and fewer bins in the tails (where weights are rare).

| Data Type | Bits | Bin Spacing | Designed For |
|---|---|---|---|
| INT4 | 4 | Equal | General integers |
| **NF4** | **4** | **Unequal (Gaussian-optimized)** | **Neural network weights** |
| FP16 | 16 | Floating-point | General computation |
| BF16 | 16 | Floating-point (wider range) | Training |

---
---

# Quiz 7, Question 17 — NF4 vs Standard INT4

> **How does NormalFloat4 (NF4) differ from standard INT4 quantization?**
>
> **B. It uses unequally spaced, equal-area bins optimized for the Gaussian distribution of weights.**

---

## Sub-questions

### What is INT4 quantization?

$$\text{INT4: } 2^4 = 16 \text{ levels, equally spaced}$$

Maps continuous weight values to 16 evenly distributed discrete levels. Every bin covers the same width of the number line.

### What are "equal-area bins"?

Bins are placed so that an equal proportion of the Gaussian distribution falls into each bin. Since the Gaussian is dense near the mean and sparse in the tails:

$$\int_{b_i}^{b_{i+1}} \mathcal{N}(0, \sigma^2) \, dx = \frac{1}{16} \quad \forall \; i$$

Bins are **narrow near zero** (high density → fine resolution where it matters) and **wide at the tails** (low density → coarse resolution where it doesn't matter).

### Why does this reduce quantization error?

More resolution where more weights exist → smaller rounding errors on average → better preservation of model quality.

---

## Main answer

**B. It uses unequally spaced, equal-area bins optimized for the Gaussian distribution of weights.**

```
INT4 (equal width bins):

  │ bin │ bin │ bin │ bin │ bin │ bin │ bin │ bin │
  ├─────┼─────┼─────┼─────┼─────┼─────┼─────┼─────┤
  -4    -3    -2    -1     0     1     2     3     4

  Many bins wasted on tails (almost no weights there)


NF4 (equal area bins):

  │bin│bin│bn│b│b│b│bn│bin│bin│
  ├───┼───┼──┼─┼─┼─┼──┼───┼───┤
  -4       -1  0  1        4

  Bins dense at center (where Gaussian peaks)
  Bins sparse at tails (where few weights exist)
```

$$\text{INT4 error} > \text{NF4 error}$$

Because NF4 allocates its 16 levels to minimize expected rounding error under the Gaussian assumption.

| Property | INT4 | NF4 |
|---|---|---|
| Number of levels | 16 | 16 |
| Bin spacing | Equal width | Equal area (Gaussian CDF) |
| Resolution near zero | Low | **High** |
| Resolution at tails | Same as center | Low (acceptable) |
| Quantization error | Higher | **Lower** (for neural net weights) |
| Assumption | None | Weights $\sim \mathcal{N}(0, \sigma^2)$ |

---
---

# Quiz 7, Question 18 — QLoRA Double Quantization

> **What is achieved by QLoRA's "Double Quantization" feature?**
>
> **C. It compresses the 32-bit quantization scaling factors (metadata) down to 8-bit, saving additional VRAM.**

---

## Sub-questions

### What are quantization scaling factors?

When quantizing a block of weights to NF4, each block needs a **scaling factor** (and possibly a zero-point) stored in FP32 to enable dequantization:

$$w_\text{dequant} = \text{scale} \times w_\text{quantized}$$

These scaling factors are metadata — overhead on top of the 4-bit weights.

### How much overhead do scaling factors add?

With a block size of 64 weights, each block has one FP32 (32-bit) scaling factor:

$$\text{Overhead} = \frac{32 \text{ bits}}{64 \text{ weights}} = 0.5 \text{ bits/param}$$

This adds 12.5% overhead on top of the 4-bit weights.

### What is Double Quantization?

Quantize the scaling factors themselves — the scaling factors (FP32) are grouped into blocks and quantized down to 8-bit, with their own (second-level) scaling factor.

$$\text{FP32 scales} \xrightarrow{\text{quantize}} \text{8-bit scales} + \text{FP32 second-level scale}$$

---

## Main answer

**C. It compresses the 32-bit quantization scaling factors (metadata) down to 8-bit, saving additional VRAM.**

```
Single Quantization:

  Weights (NF4, 4-bit)          Scaling factors (FP32, 32-bit)
  ┌────────────────────┐       ┌──────────────────────┐
  │ 4 bits per weight  │ + │   │ 32 bits per block    │  ← overhead
  └────────────────────┘       └──────────────────────┘
                               0.5 bits/param overhead


Double Quantization:

  Weights (NF4, 4-bit)          Scaling factors (8-bit)     2nd-level scale (FP32)
  ┌────────────────────┐       ┌───────────────────┐       ┌────────────┐
  │ 4 bits per weight  │ + │   │ 8 bits per block  │ + │   │ tiny       │
  └────────────────────┘       └───────────────────┘       └────────────┘
                               ~0.125 bits/param            ~0.001 bits/param
                               (4× less overhead!)
```

$$\text{Single Q overhead: } \frac{32}{64} = 0.5 \text{ bits/param}$$
$$\text{Double Q overhead: } \frac{8}{64} + \frac{32}{64 \times 256} \approx 0.127 \text{ bits/param}$$

| Level | What is Quantized | From | To | Overhead |
|---|---|---|---|---|
| Level 1 | Weights | FP16 | NF4 (4-bit) | — |
| Level 1 metadata | Scaling factors | FP32 (32-bit) | — | 0.5 bits/param |
| **Level 2** | **Scaling factors** | **FP32 (32-bit)** | **8-bit** | **~0.127 bits/param** |
| **Savings** | — | — | — | **~0.37 bits/param saved** |

---
---

# Quiz 7, Question 19 — QLoRA Compute Precision

> **In the QLoRA architecture, what precision is used to perform the actual forward pass and gradient math in the GPU compute core?**
>
> **C. 16-bit BrainFloat (BF16)**

---

## Sub-questions

### What is BF16 (BrainFloat16)?

$$\text{BF16: } 1 \text{ sign bit} + 8 \text{ exponent bits} + 7 \text{ mantissa bits} = 16 \text{ bits}$$

Same dynamic range as FP32 (8-bit exponent) but with reduced precision (7-bit mantissa vs. 23-bit). Designed by Google Brain for deep learning — wide range is more important than fine precision for training stability.

### Why not compute in 4-bit (NF4)?

4-bit arithmetic would produce enormous rounding errors during matrix multiplications — the accumulated errors from multiplying and adding thousands of low-precision values would corrupt gradients and make training unstable.

### How does QLoRA combine NF4 storage with BF16 compute?

$$\text{NF4 (storage)} \xrightarrow{\text{dequantize on-the-fly}} \text{BF16 (compute)} \xrightarrow{\text{result}} \text{BF16}$$

Weights are **stored** in NF4 to save VRAM but **dequantized to BF16** just before computation. The forward pass, loss, and gradients are all computed in BF16.

---

## Main answer

**C. 16-bit BrainFloat (BF16)**

```
QLoRA data flow:

  Storage (VRAM)              Compute (GPU cores)
  ┌──────────────┐           ┌──────────────────┐
  │ Base weights │──dequant─▶│ Forward pass     │
  │ NF4 (4-bit)  │   to BF16 │ in BF16          │
  └──────────────┘           │                  │
                              │ Backward pass    │
  ┌──────────────┐           │ in BF16          │
  │ LoRA A, B    │──────────▶│                  │
  │ BF16 (16-bit)│           │ Gradients: BF16  │
  └──────────────┘           └──────────────────┘
```

$$\text{Memory: } 4\text{-bit (NF4)} \quad | \quad \text{Compute: } 16\text{-bit (BF16)}$$

| Component | Storage Precision | Compute Precision |
|---|---|---|
| Base weights | NF4 (4-bit) | Dequantized to BF16 |
| LoRA adapters | BF16 (16-bit) | BF16 |
| Activations | — | BF16 |
| Gradients | — | BF16 |
| Optimizer states | FP32 (32-bit) | FP32 |

---
---

# Quiz 7, Question 20 — Paged Optimizers in QLoRA

> **What specific problem do "Paged Optimizers" solve in the QLoRA pipeline?**
>
> **B. They prevent Out-Of-Memory (OOM) crashes during gradient calculation spikes by offloading states to CPU RAM.**

---

## Sub-questions

### What are optimizer states?

Adam/AdamW maintain two additional tensors per parameter:

$$m_t \text{ (first moment / momentum)}, \quad v_t \text{ (second moment / RMS)}$$

For a model with $N$ trainable parameters, optimizer states consume $\sim 2N$ additional values (typically in FP32 → $8N$ bytes).

### What causes VRAM spikes during training?

During the backward pass, intermediate activations and gradients temporarily require extra VRAM beyond steady-state usage. These spikes can exceed available VRAM even when the model fits at baseline.

### What is paging (from OS concepts)?

$$\text{VRAM full} \xrightarrow{\text{page out}} \text{CPU RAM} \xrightarrow{\text{page in (when needed)}} \text{VRAM}$$

Borrowed from virtual memory: when GPU memory is full, temporarily move unused data (optimizer states) to CPU RAM. Bring it back when needed for the optimizer step.

---

## Main answer

**B. They prevent Out-Of-Memory (OOM) crashes during gradient calculation spikes by offloading states to CPU RAM.**

```
Without Paged Optimizers:

  VRAM usage
   ↑
   │           ┌──┐
   │  ─────────┤  ├────  ← spike exceeds VRAM
   │           │  │      ← OOM CRASH!
   ├───────────┼──┼──── VRAM limit
   │           │  │
   │  ─────────┘  └────
   └─────────────────────▶ training step
           backward  step


With Paged Optimizers:

  VRAM usage
   ↑
   ├──────────────────── VRAM limit
   │  ─────────┬──┬────  ← spike stays under limit
   │           │  │      ← optimizer states offloaded to CPU
   │  ─────────┘  └────
   └─────────────────────▶ training step

  CPU RAM: [m_t, v_t temporarily stored here during backward]
```

```
Paging flow:

  Before backward:
    VRAM: [weights (NF4) + LoRA (BF16) + optimizer states]
          ↓ nearing limit
    Page out: optimizer states ──▶ CPU RAM

  During backward:
    VRAM: [weights + LoRA + activations + gradients]
          ← room for spike

  Before optimizer.step():
    Page in: CPU RAM ──▶ VRAM ──▶ optimizer states
    optimizer.step() executes normally
```

| Scenario | Without Paging | With Paging |
|---|---|---|
| VRAM spike during backward | OOM crash | Optimizer states offloaded to CPU |
| Performance cost | N/A (crashes) | Small CPU↔GPU transfer overhead |
| Enables training | Only if VRAM is sufficient | Even with tight VRAM budgets |
| Inspiration | — | OS virtual memory paging |

---
---

# Quiz 7, Question 21 — Beam Search vs Greedy Decoding (Essay)

> **Decoding Math (Beam Search vs. Greedy):** A language model is generating text using a Beam Width of $B = 2$.
>
> After the initial prompt, the model predicts the first word probabilities: "The" (60%), "A" (30%), "An" (10%).
>
> For paths extending from "The": "cat" (40%), "dog" (30%). For paths extending from "A": "tiger" (90%), "lion" (5%).
>
> a. Calculate the cumulative probability for all four expanded paths.
> b. Which two paths survive the pruning step?
> c. Which exact sequence would Greedy Search have chosen? Did Greedy find the optimal path?

---

## Sub-questions

### What is Beam Search?

Maintains $B$ candidate sequences. At each step, expand every beam with all possible next tokens, compute cumulative probabilities, keep only the top $B$.

$$\text{Score}(x_1, \dots, x_t) = \prod_{i=1}^{t} P(x_i \mid x_{<i})$$

### What is cumulative probability?

$$P(\text{sequence}) = P(w_1) \times P(w_2 \mid w_1)$$

Joint probability of the entire sequence. Product of each conditional probability.

### What is Greedy Search?

$$x_t = \arg\max P(x_t \mid x_{<t})$$

Pick the single highest-probability token at each step. No backtracking, no alternatives maintained.

---

## Main answer

### Part (a): Cumulative probabilities for all four paths

$$P(\text{"The cat"}) = 0.60 \times 0.40 = 0.24$$

$$P(\text{"The dog"}) = 0.60 \times 0.30 = 0.18$$

$$P(\text{"A tiger"}) = 0.30 \times 0.90 = 0.27$$

$$P(\text{"A lion"}) = 0.30 \times 0.05 = 0.015$$

### Part (b): Top 2 surviving paths (B = 2)

Sorted: "A tiger" (0.27) > "The cat" (0.24) > "The dog" (0.18) > "A lion" (0.015)

**Survivors: "A tiger" (0.27) and "The cat" (0.24)**

### Part (c): Greedy Search comparison

- Step 1: $\arg\max(0.60, 0.30, 0.10) = \text{"The"}$
- Step 2 (from "The"): $\arg\max(0.40, 0.30) = \text{"cat"}$
- Greedy output: **"The cat"** with $P = 0.24$

**Greedy did NOT find the optimal path.** "A tiger" at $P = 0.27$ is superior, but Greedy never explored the "A" branch because it committed to "The" at step 1.

```
Decision tree:

                    [Start]
                   /   |    \
                 The   A     An
                0.60  0.30  0.10
               / \      |  \
            cat  dog  tiger lion
           0.40 0.30  0.90  0.05
             │    │     │     │
           0.24 0.18  0.27  0.015

  Greedy path:  The → cat = 0.24  ← suboptimal
  Beam winner:  A → tiger = 0.27  ← optimal ✓
```

| Path | Cumulative Probability | Beam Survives? | Greedy Selects? |
|---|---|---|---|
| **A tiger** | **0.27** | **Yes (rank 1)** | No (never explored) |
| The cat | 0.24 | Yes (rank 2) | **Yes** |
| The dog | 0.18 | No (pruned) | No |
| A lion | 0.015 | No (pruned) | No |

---
---

# Quiz 7, Question 22 — Temperature Math (Essay)

> **Decoding Math (Temperature):** The ratio of selection probabilities for two tokens ($A$ and $B$) under Softmax with Temperature ($T$) is given by $e^{(z_a - z_b)/T}$. Suppose Token A has a raw logit of 4.0 and Token B has a raw logit of 2.0.
>
> - Calculate the probability ratio ($P_a/P_b$) if Temperature is set to $T = 1.0$.
> - Calculate the probability ratio ($P_a/P_b$) if Temperature is set to $T = 0.5$.
> - Explain how the change in Temperature affected the model's confidence.

---

## Sub-questions

### What is the probability ratio formula?

$$\frac{P_a}{P_b} = \frac{e^{z_a/T}}{e^{z_b/T}} = e^{(z_a - z_b)/T}$$

Depends only on the logit difference and $T$.

### What is the logit difference here?

$$z_a - z_b = 4.0 - 2.0 = 2.0$$

---

## Main answer

### Part 1: $T = 1.0$

$$\frac{P_a}{P_b} = e^{2.0 / 1.0} = e^2 \approx 7.39$$

Token A is approximately **7.39 times** more likely than Token B.

### Part 2: $T = 0.5$

$$\frac{P_a}{P_b} = e^{2.0 / 0.5} = e^4 \approx 54.60$$

Token A is approximately **54.60 times** more likely than Token B.

### Part 3: Explanation

Lowering the temperature from 1.0 to 0.5 **exponentially amplified** the logit distance between the tokens. The effective logit gap went from $2.0$ to $4.0$ (doubled), causing the probability ratio to jump from $\sim 7.4\times$ to $\sim 54.6\times$.

$$T \downarrow \implies \frac{z_a - z_b}{T} \uparrow \implies e^{(\cdot)} \uparrow \implies \text{model becomes hyper-confident}$$

```
Probability ratio vs Temperature:

  P(A)/P(B)
   ↑
  54.6│          ●  T = 0.5 (hyper-confident)
      │
      │
      │
   7.4│                    ●  T = 1.0 (normal)
      │
   2.7│                              ●  T = 2.0
   1.0│─ ─ ─ ─ ─ ─ ─ ─ ─ ─ ─ ─ ─ ─ ─ ─ ─ ─  T → ∞
      └───────────────────────────────────────▶ T
     0.5        1.0              2.0        ∞
```

| Temperature | Exponent $(2.0/T)$ | Ratio $P_a/P_b$ | Behavior |
|---|---|---|---|
| $T = 0.5$ | $4.0$ | $e^4 \approx 54.60$ | Hyper-confident |
| $T = 1.0$ | $2.0$ | $e^2 \approx 7.39$ | Normal |
| $T = 2.0$ | $1.0$ | $e^1 \approx 2.72$ | Less confident |
| $T \to \infty$ | $0$ | $e^0 = 1.0$ | Equal probability |

---
---

# Quiz 7, Question 23 — LoRA Parameter Reduction Math (Essay)

> **Fine-Tuning Math (LoRA Parameter Reduction):** Consider a single Transformer attention layer containing a frozen weight matrix $W$ of dimensions $1024 \times 1024$.
>
> - Calculate the exact number of trainable parameters required if we perform Standard Full Fine-Tuning on this layer.
> - Calculate the exact number of trainable parameters required if we apply LoRA with a rank of $r = 4$.
> - What is the percentage reduction in trainable parameters?

---

## Sub-questions

### What is full fine-tuning parameter count?

$$\text{Full FT params} = d_\text{out} \times d_\text{in}$$

Every element of the weight matrix is trainable.

### What is LoRA parameter count?

$$\text{LoRA params} = d_\text{out} \times r + r \times d_\text{in}$$

Matrix $A \in \mathbb{R}^{d_\text{out} \times r}$ plus matrix $B \in \mathbb{R}^{r \times d_\text{in}}$.

### What is percentage reduction?

$$\text{Reduction} = \left(1 - \frac{\text{LoRA params}}{\text{Full FT params}}\right) \times 100\%$$

---

## Main answer

### Part 1: Standard Full Fine-Tuning

$$1024 \times 1024 = 1{,}048{,}576 \text{ trainable parameters}$$

### Part 2: LoRA with $r = 4$

$$A \in \mathbb{R}^{1024 \times 4}: \quad 1024 \times 4 = 4{,}096 \text{ parameters}$$
$$B \in \mathbb{R}^{4 \times 1024}: \quad 4 \times 1024 = 4{,}096 \text{ parameters}$$
$$\text{Total} = 4{,}096 + 4{,}096 = 8{,}192 \text{ trainable parameters}$$

### Part 3: Percentage reduction

$$\text{Reduction} = \left(1 - \frac{8{,}192}{1{,}048{,}576}\right) \times 100\% = (1 - 0.00781) \times 100\% \approx 99.22\%$$

```
Parameter comparison:

  Full FT:    W (1024 × 1024) = 1,048,576 params
              ┌──────────────────────────────┐
              │██████████████████████████████│
              │██████████████████████████████│
              │██████████████████████████████│  1024 rows
              │██████████████████████████████│
              └──────────────────────────────┘
                       1024 columns

  LoRA:       A (1024 × 4) + B (4 × 1024) = 8,192 params
              ┌──┐    ┌──────────────────────────────┐
              │██│    │████████████████████████████████│  4 rows
              │██│    └──────────────────────────────┘
              │██│ 1024        1024 columns
              │██│ rows
              └──┘
             4 cols
```

$$\frac{8{,}192}{1{,}048{,}576} = 0.0078 = 0.78\% \quad \text{(only 0.78\% of original parameters!)}$$

| Method | Trainable Parameters | % of Full FT |
|---|---|---|
| Full Fine-Tuning | 1,048,576 | 100% |
| **LoRA ($r=4$)** | **8,192** | **0.78%** |
| **Reduction** | — | **99.22%** |

---
---

# Quiz 7, Question 24 — LoRA Approximation Error via SVD (Essay)

> **Linear Algebra (LoRA Approximation Error):** A theoretical weight update matrix $\Delta W$ has exactly four singular values derived from an SVD: $\sigma_1 = 10$, $\sigma_2 = 8$, $\sigma_3 = 2$, $\sigma_4 = 1$.
>
> a. According to the Eckart-Young-Mirsky theorem, calculate the exact squared approximation error ($Error^2$) if we apply LoRA with rank $r = 2$.
> b. Calculate the squared approximation error ($Error^2$) if we apply LoRA with rank $r = 3$.
> c. Does this task represent a "Strong" or "Weak" Intrinsic Dimension Hypothesis? Explain why.

---

## Sub-questions

### What is the Eckart-Young-Mirsky theorem?

$$\|A - A_r\|_F^2 = \sum_{i=r+1}^{n} \sigma_i^2$$

The optimal rank-$r$ approximation error (in Frobenius norm squared) equals the sum of squares of the **discarded** singular values.

### What is the Frobenius norm?

$$\|M\|_F = \sqrt{\sum_{i,j} M_{i,j}^2}$$

The "total energy" of a matrix. The squared Frobenius norm is the sum of squares of all entries.

### What does "discarded" mean?

When approximating with rank $r$, keep $\sigma_1, \dots, \sigma_r$ and set $\sigma_{r+1}, \dots, \sigma_n = 0$. The error comes entirely from the zeroed-out singular values.

---

## Main answer

### Part (a): Rank $r = 2$

Keep $\sigma_1 = 10, \sigma_2 = 8$. Discard $\sigma_3 = 2, \sigma_4 = 1$.

$$Error^2 = \sigma_3^2 + \sigma_4^2 = 2^2 + 1^2 = 4 + 1 = 5$$

### Part (b): Rank $r = 3$

Keep $\sigma_1 = 10, \sigma_2 = 8, \sigma_3 = 2$. Discard $\sigma_4 = 1$.

$$Error^2 = \sigma_4^2 = 1^2 = 1$$

### Part (c): Strong Intrinsic Dimension Hypothesis

**Strong hypothesis.** The vast majority of the "information" (energy) is concentrated in the first two singular values:

$$\text{Total energy} = \sigma_1^2 + \sigma_2^2 + \sigma_3^2 + \sigma_4^2 = 100 + 64 + 4 + 1 = 169$$

$$\text{Top-2 energy} = 100 + 64 = 164 \quad \rightarrow \quad \frac{164}{169} \approx 97.0\%$$

Truncating the tail (rank 2) captures 97% of the total energy with only $\frac{5}{169} \approx 3\%$ error. The intrinsic dimension is low — the information is compressible.

```
Singular value spectrum:

  σᵢ²
   ↑
  100│ ██                           σ₁² = 100
   64│ ██ ██                        σ₂² = 64
      │ ██ ██
    4│ ██ ██ ██                     σ₃² = 4
    1│ ██ ██ ██ ██                  σ₄² = 1
     └──┴──┴──┴──▶ component
      σ₁  σ₂  σ₃  σ₄
           │        │
     ← KEPT (97%) → ← DISCARDED (3%) →
```

| Rank $r$ | Kept $\sigma$ | Discarded $\sigma$ | $Error^2$ | % Energy Retained |
|---|---|---|---|---|
| 2 | 10, 8 | 2, 1 | **5** | 97.0% |
| 3 | 10, 8, 2 | 1 | **1** | 99.4% |
| 4 (full) | all | none | 0 | 100% |

---
---

# Quiz 7, Question 25 — Double Quantization Math (Essay)

> **Systems Engineering (Double Quantization Math):** An AI research team is modifying the QLoRA architecture. Instead of the standard block size, they group their 4-bit base model weights into blocks of **128 weights**, and each block shares a **16-bit (FP16)** scaling factor.
>
> a. Calculate the exact memory overhead in bits-per-parameter for the scaling factors under this single quantization setup.
> b. The team decides to apply Double Quantization. They quantize the 16-bit scaling factors down to **4-bit** values. What is the new primary overhead in bits-per-parameter?
> c. To achieve this double quantization, the team groups the new 4-bit scaling factors into blocks of **256**, sharing a single **32-bit** second-level scaling factor. Calculate the secondary overhead and the **total** bits-per-parameter overhead for this entire Double Quantization setup.

---

## Sub-questions

### What is a quantization scaling factor?

$$w_\text{dequant} = s \times w_\text{quantized}$$

A per-block constant $s$ that maps quantized integer values back to their approximate floating-point originals. Stored alongside the quantized weights as metadata.

### What is "bits-per-parameter" overhead?

$$\text{Overhead (bits/param)} = \frac{\text{bits for scaling factor}}{\text{weights in the block}}$$

The amortized cost of the scaling factor spread across all weights in its block.

### What is Double Quantization?

Quantize the scaling factors themselves. The first-level scaling factors (originally FP16) are quantized to a lower precision (e.g., 4-bit), grouped into blocks, and each block shares a second-level scaling factor (FP32).

---

## Main answer

### Part (a): Single Quantization Overhead

$$\text{Overhead} = \frac{16 \text{ bits (FP16 scale)}}{128 \text{ weights}} = 0.125 \text{ bits/param}$$

### Part (b): Double Quantization — Primary Overhead

The 16-bit scaling factors are quantized to 4-bit:

$$\text{Primary overhead} = \frac{4 \text{ bits}}{128 \text{ weights}} = 0.03125 \text{ bits/param}$$

### Part (c): Secondary Overhead and Total

The 4-bit scaling factors are grouped into blocks of 256. Each group of 256 scaling factors shares one 32-bit second-level scaling factor. Each scaling factor represents a block of 128 weights, so each second-level factor covers $128 \times 256 = 32{,}768$ weights.

$$\text{Secondary overhead} = \frac{32 \text{ bits}}{128 \times 256 \text{ weights}} = \frac{32}{32{,}768} \approx 0.000976 \text{ bits/param}$$

$$\text{Total overhead} = 0.03125 + 0.000976 \approx 0.0322 \text{ bits/param}$$

```
Single Quantization overhead breakdown:

  128 weights ←──── 1 FP16 scale (16 bits)
  overhead = 16/128 = 0.125 bits/param


Double Quantization overhead breakdown:

  Level 1: 128 weights ←──── 1 quantized scale (4 bits)
           overhead = 4/128 = 0.03125 bits/param

  Level 2: 256 quantized scales ←──── 1 FP32 scale (32 bits)
           = 256 × 128 = 32,768 weights per FP32 scale
           overhead = 32/32,768 ≈ 0.000976 bits/param

  Total = 0.03125 + 0.000976 ≈ 0.0322 bits/param
```

```
Overhead reduction:

  Single Q:  0.125    bits/param
  Double Q:  0.0322   bits/param
  Savings:   0.0928   bits/param (74% reduction in overhead!)
```

| Component | Bits | Amortized Over | Bits/Param |
|---|---|---|---|
| **Single Q:** FP16 scale | 16 | 128 weights | 0.125 |
| **Double Q Level 1:** 4-bit scale | 4 | 128 weights | 0.03125 |
| **Double Q Level 2:** FP32 scale | 32 | 32,768 weights | 0.000976 |
| **Double Q Total** | — | — | **≈ 0.0322** |
