# Quiz 5: Transformer Architecture — Tutor Answers (Q1–Q25)

---

# Quiz 5, Question 1 — Scaled Dot-Product Attention

> **In the standard Attention mechanism, what is the primary reason for dividing the dot product of the Query and Key matrices by $\sqrt{d_k}$?**
>
> **B. To normalize the variance of the dot products, preventing the softmax function from entering regions with vanishing gradients.**

---

## Sub-questions

### What is a dot product between Query and Key?

$$\text{score}(q, k) = q \cdot k = \sum_{i=1}^{d_k} q_i \, k_i$$

$q$ = query vector for one token; $k$ = key vector for another token; $d_k$ = dimensionality of both vectors.

**Numeric example:** $q = [1, 2, 3]$, $k = [4, 5, 6]$, $d_k = 3$. Score $= 1 \cdot 4 + 2 \cdot 5 + 3 \cdot 6 = 32$.

### What is the variance of a dot product?

Assume $q_i$ and $k_i$ are independent with mean 0 and variance 1.

$$\text{Var}(q \cdot k) = \text{Var}\!\left(\sum_{i=1}^{d_k} q_i k_i\right) = d_k$$

Each product $q_i k_i$ contributes variance 1, and there are $d_k$ terms. So the dot product's variance **grows linearly** with $d_k$.

**Numeric example:** $d_k = 128 \implies \text{Var}(q \cdot k) = 128$, $\text{std} = \sqrt{128} \approx 11.3$. Scores can easily reach $\pm 30$.

### What happens to Softmax with large inputs?

$$\text{Softmax}(z_i) = \frac{e^{z_i}}{\sum_j e^{z_j}}$$

When one $z_i$ dominates (e.g., $z = [30, 1, 2]$):

$$e^{30} \approx 10^{13} \gg e^1, e^2 \implies \text{Softmax} \approx [1.0, 0.0, 0.0]$$

Output saturates to a one-hot vector. The gradient of Softmax at saturation:

$$\frac{\partial \text{Softmax}(z_i)}{\partial z_j} = p_i(\delta_{ij} - p_j) \approx 0 \quad \text{when } p_i \approx 1 \text{ or } 0$$

**Numeric example:** $p = [0.999, 0.0005, 0.0005]$ → $\frac{\partial p_1}{\partial z_1} = 0.999 \cdot 0.001 = 0.000999 \approx 0$. Gradient vanishes.

### What is the vanishing gradient problem?

$$\Delta\theta = -\alpha \frac{\partial \mathcal{L}}{\partial \theta}$$

When $\frac{\partial \mathcal{L}}{\partial \theta} \to 0$, the weight update $\Delta\theta \to 0$. Training stalls — the model stops learning.

---

## Main answer

**B. To normalize the variance of the dot products, preventing the softmax function from entering regions with vanishing gradients.**

The scaled dot-product attention divides by $\sqrt{d_k}$ to restore unit variance:

$$\text{Attention}(Q, K, V) = \text{Softmax}\!\left(\frac{QK^T}{\sqrt{d_k}}\right)V$$

$$\text{Var}\!\left(\frac{q \cdot k}{\sqrt{d_k}}\right) = \frac{\text{Var}(q \cdot k)}{d_k} = \frac{d_k}{d_k} = 1$$

```
Effect of scaling on Softmax input distribution:

WITHOUT scaling (d_k = 128):                WITH scaling (÷ √128 ≈ 11.3):

  Softmax                                     Softmax
  input                                       input
  freq.                                       freq.
   │                                           │
   │                                           │       ┌───┐
   │                                           │     ┌─┤   ├─┐
   │        ┌─┐                                │   ┌─┤ │   │ ├─┐
   │   ┌────┤ ├────┐                           │ ┌─┤ │ │   │ │ ├─┐
   └───┴────┴─┴────┴──── score                 └─┴─┴─┴─┴───┴─┴─┴─┴── score
  -30   -10   0   10  30                       -3  -1  0   1   3
     std ≈ 11.3                                   std ≈ 1.0
  → Softmax saturates                          → Softmax stays in
  → gradients vanish                             gradient-rich region
```

```
Causal chain (logic flow):

  Large d_k ──▶ Var(QKᵀ) = d_k ──▶ Large scores ──▶ Softmax saturates
       │                                                     │
       ▼                                                     ▼
  ÷ √d_k ──▶ Var = 1 ──▶ Moderate scores ──▶ Healthy gradients ──▶ Training works
```

| Property | Without $\sqrt{d_k}$ scaling | With $\sqrt{d_k}$ scaling |
|---|---|---|
| Score variance | $d_k$ (grows with dim) | $1$ (constant) |
| Typical score magnitude | $\pm 3\sqrt{d_k}$ | $\pm 3$ |
| Softmax output | Near one-hot | Distributed |
| Gradient magnitude | $\approx 0$ (vanishing) | Healthy, nonzero |
| Training stability | Unstable | Stable |

---
---

# Quiz 5, Question 2 — Pre-Norm Residual Connection

> **Which of the following equations accurately represents the "Pre-Norm" residual connection used in modern architectures like Llama 3?**
>
> **C. $x_{t+1} = x_t + \text{Sublayer}(\text{Norm}(x_t))$**

---

## Sub-questions

### What is a residual connection?

$$x_{t+1} = x_t + F(x_t)$$

$x_t$ = input to the sublayer; $F(x_t)$ = sublayer transformation (e.g., self-attention or FFN). The $+ \, x_t$ term is the "skip connection" — it adds the original input back, allowing gradients to bypass the sublayer.

**Numeric example:** $x_t = [1.0, 2.0]$, $F(x_t) = [0.3, -0.1]$ → $x_{t+1} = [1.3, 1.9]$.

### What is Layer Normalization?

$$\text{Norm}(x) = \gamma \cdot \frac{x - \mu}{\sqrt{\sigma^2 + \epsilon}} + \beta$$

$\mu = \frac{1}{d}\sum_i x_i$ (mean), $\sigma^2 = \frac{1}{d}\sum_i (x_i - \mu)^2$ (variance), $\gamma, \beta$ = learned scale and shift, $\epsilon$ = small constant for numerical stability.

Stabilizes activations to zero mean and unit variance before scaling.

### What is Post-Norm vs Pre-Norm?

**Post-Norm** (original Transformer, Vaswani 2017):

$$x_{t+1} = \text{Norm}(x_t + \text{Sublayer}(x_t))$$

Normalization applied **after** the residual addition.

**Pre-Norm** (GPT-2, Llama 3):

$$x_{t+1} = x_t + \text{Sublayer}(\text{Norm}(x_t))$$

Normalization applied **before** the sublayer, **inside** the residual branch.

---

## Main answer

**C. $x_{t+1} = x_t + \text{Sublayer}(\text{Norm}(x_t))$**

```
Pre-Norm architecture (Llama 3):

       x_t ─────────────────────────┐
        │                           │ (skip / residual)
        ▼                           │
  ┌───────────┐                     │
  │   Norm    │                     │
  └─────┬─────┘                     │
        ▼                           │
  ┌───────────┐                     │
  │ Sublayer  │                     │
  │ (Attn/FFN)│                     │
  └─────┬─────┘                     │
        ▼                           │
       (+) ◄────────────────────────┘
        │
        ▼
      x_{t+1}
```

```
Post-Norm architecture (original Transformer):

       x_t ─────────────────────────┐
        │                           │ (skip / residual)
        ▼                           │
  ┌───────────┐                     │
  │ Sublayer  │                     │
  │ (Attn/FFN)│                     │
  └─────┬─────┘                     │
        ▼                           │
       (+) ◄────────────────────────┘
        │
        ▼
  ┌───────────┐
  │   Norm    │
  └─────┬─────┘
        ▼
      x_{t+1}
```

Pre-Norm places normalization inside the residual branch, so the gradient through the skip connection is an unmodified identity:

$$\frac{\partial x_{t+1}}{\partial x_t} = 1 + \frac{\partial \text{Sublayer}(\text{Norm}(x_t))}{\partial x_t}$$

The leading $1$ ensures gradients never vanish through the skip path, regardless of depth.

### Why each wrong option fails

| Option | Formula | Problem |
|---|---|---|
| A | $\text{Norm}(x_t + \text{Sublayer}(x_t))$ | Post-Norm. Norm wraps the sum. |
| B | $\text{Sublayer}(\text{Norm}(x_t))$ | No residual connection. Skip path missing. |
| **C** | $x_t + \text{Sublayer}(\text{Norm}(x_t))$ | **Correct. Pre-Norm.** |
| D | $x_t + \text{Norm}(\text{Sublayer}(x_t))$ | Norm is on sublayer output, not input. Not standard Pre-Norm or Post-Norm. |

| Property | Post-Norm (A) | Pre-Norm (C) |
|---|---|---|
| Norm placement | After residual add | Before sublayer |
| Gradient through skip | Passes through Norm | Clean identity ($+1$) |
| Training stability | Requires warmup | Stable without warmup |
| Used in | Original Transformer | GPT-2, Llama 3, PaLM |

---
---

# Quiz 5, Question 3 — RMSNorm vs LayerNorm

> **How does Root Mean Square Normalization (RMSNorm) differ fundamentally from standard Layer Normalization (LayerNorm)?**
>
> **B. RMSNorm omits the mean-centering operation, calculating variance strictly relative to zero.**

---

## Sub-questions

### What is Layer Normalization (LayerNorm)?

$$\text{LayerNorm}(x) = \gamma \cdot \frac{x - \mu}{\sqrt{\sigma^2 + \epsilon}} + \beta$$

Step 1 — compute mean: $\mu = \frac{1}{d}\sum_{i=1}^{d} x_i$

Step 2 — compute variance: $\sigma^2 = \frac{1}{d}\sum_{i=1}^{d}(x_i - \mu)^2$

Step 3 — normalize: subtract mean (center), divide by std (scale).

**Numeric example:** $x = [2, 4, 6]$, $d = 3$.

$\mu = 4$, $\sigma^2 = \frac{(2-4)^2 + (4-4)^2 + (6-4)^2}{3} = \frac{8}{3} \approx 2.67$, $\sigma \approx 1.63$.

$\text{LayerNorm}(x) \approx [-1.22, 0, 1.22]$ (before $\gamma, \beta$).

### What is RMSNorm?

$$\text{RMSNorm}(x) = \gamma \cdot \frac{x}{\text{RMS}(x)}, \qquad \text{RMS}(x) = \sqrt{\frac{1}{d}\sum_{i=1}^{d} x_i^2}$$

No mean subtraction. No $\beta$ shift. Variance is computed relative to **zero**, not relative to the mean.

**Numeric example:** $x = [2, 4, 6]$, $d = 3$.

$\text{RMS}(x) = \sqrt{\frac{4 + 16 + 36}{3}} = \sqrt{18.67} \approx 4.32$.

$\text{RMSNorm}(x) \approx [0.463, 0.926, 1.389]$ (before $\gamma$).

### What does "mean-centering" do?

$$x' = x - \mu$$

Shifts the distribution so the mean becomes 0. This is the step RMSNorm **skips**.

Without mean-centering, the normalization only rescales by magnitude, preserving the original offset of the data.

---

## Main answer

**B. RMSNorm omits the mean-centering operation, calculating variance strictly relative to zero.**

```
LayerNorm computation:                    RMSNorm computation:

  x = [2, 4, 6]                           x = [2, 4, 6]
       │                                       │
       ▼                                       │ (no mean step)
  ┌──────────┐                                 │
  │ μ = Σx/d │ = 4                             │
  └────┬─────┘                                 │
       ▼                                       │
  ┌──────────────┐                             ▼
  │ x' = x - μ  │ = [-2, 0, 2]          ┌────────────────┐
  └────┬─────────┘                       │ RMS = √(Σx²/d) │ = 4.32
       ▼                                 └──────┬─────────┘
  ┌──────────────────┐                          ▼
  │ σ² = Σ(x')²/d   │ = 2.67            ┌──────────────┐
  └────┬─────────────┘                   │ x / RMS      │ = [0.46, 0.93, 1.39]
       ▼                                 └──────┬───────┘
  ┌──────────────┐                              ▼
  │ x' / σ      │ = [-1.22, 0, 1.22]      γ · (x / RMS)
  └────┬─────────┘
       ▼
  γ · (x'/σ) + β
```

The key difference: LayerNorm computes variance around the mean ($\sigma^2$), RMSNorm computes the root mean square around zero.

$$\text{LayerNorm: } \sigma^2 = \frac{1}{d}\sum(x_i - \mu)^2 \qquad \text{RMSNorm: } \text{RMS}^2 = \frac{1}{d}\sum x_i^2$$

By the bias-variance decomposition:

$$\text{RMS}^2 = \sigma^2 + \mu^2$$

RMSNorm "variance" includes the mean as part of the magnitude. When $\mu \approx 0$, both methods produce nearly identical results.

| Property | LayerNorm | RMSNorm |
|---|---|---|
| Mean centering ($x - \mu$) | Yes | **No** |
| Learned bias ($\beta$) | Yes | No |
| Denominator | $\sqrt{\frac{1}{d}\sum(x_i - \mu)^2}$ | $\sqrt{\frac{1}{d}\sum x_i^2}$ |
| Learned parameters | $\gamma, \beta$ ($2d$ params) | $\gamma$ ($d$ params) |
| Compute cost | 2 passes (mean, then variance) | 1 pass (sum of squares) |
| Used in | Original Transformer, BERT | Llama 3, T5, PaLM |
| Empirical quality | Baseline | Equivalent or better |

---
---

# Quiz 5, Question 4 — SwiGLU Activation and Weight Matrices

> **Llama 3 replaces the standard ReLU or GELU activation functions with the SwiGLU activation. How many distinct weight matrices are required in a standard SwiGLU FFN block compared to a standard ReLU FFN block?**
>
> **C. SwiGLU requires 3 matrices; standard requires 2.**

---

## Sub-questions

### What is the standard ReLU FFN block?

$$\text{FFN}_{\text{ReLU}}(x) = W_2 \, \text{ReLU}(W_1 x + b_1) + b_2$$

$W_1 \in \mathbb{R}^{d_\text{ff} \times d}$ = up-projection (expands dimension). $W_2 \in \mathbb{R}^{d \times d_\text{ff}}$ = down-projection (compresses back). **2 weight matrices.**

$\text{ReLU}(z) = \max(0, z)$.

**Numeric example:** $d = 4$, $d_\text{ff} = 16$. $W_1$: $16 \times 4 = 64$ params. $W_2$: $4 \times 16 = 64$ params. Total: 128 params (ignoring biases).

### What is a Gated Linear Unit (GLU)?

$$\text{GLU}(x) = (W_1 x) \otimes \sigma(W_\text{gate} \, x)$$

$\otimes$ = element-wise multiplication; $\sigma$ = sigmoid gate. The gate controls **how much** of each dimension passes through — a learned, input-dependent filter.

**Two** linear projections are needed just for the gating: $W_1$ (value path) and $W_\text{gate}$ (gate path).

### What is SwiGLU?

$$\text{SwiGLU}(x) = (\text{Swish}(W_\text{gate} \, x)) \otimes (W_1 x)$$

$$\text{Swish}(z) = z \cdot \sigma(z), \qquad \sigma(z) = \frac{1}{1 + e^{-z}}$$

Replaces the sigmoid gate with the Swish activation. Swish is smooth and non-monotonic — allows small negative values to pass, unlike ReLU.

**Numeric example:** $z = -0.5 \implies \text{Swish}(-0.5) = -0.5 \cdot \sigma(-0.5) = -0.5 \cdot 0.378 = -0.189$. Nonzero output for negative input.

### What is the full SwiGLU FFN block?

$$\text{FFN}_{\text{SwiGLU}}(x) = W_2 \left[\, \text{Swish}(W_\text{gate} \, x) \otimes (W_1 x) \,\right]$$

Three distinct weight matrices:

$$W_\text{gate} \in \mathbb{R}^{d_\text{ff} \times d}, \quad W_1 \in \mathbb{R}^{d_\text{ff} \times d}, \quad W_2 \in \mathbb{R}^{d \times d_\text{ff}}$$

---

## Main answer

**C. SwiGLU requires 3 matrices; standard requires 2.**

```
Standard ReLU FFN (2 matrices):

  x ──▶ [ W₁ ] ──▶ ReLU ──▶ [ W₂ ] ──▶ output
         d→d_ff              d_ff→d

  Matrices: W₁, W₂


SwiGLU FFN (3 matrices):

              ┌──▶ [ W_gate ] ──▶ Swish(·) ──┐
              │      d→d_ff                    │
  x ──────────┤                               (⊗) ──▶ [ W₂ ] ──▶ output
              │                                │       d_ff→d
              └──▶ [ W₁ ] ────────────────────┘
                    d→d_ff

  Matrices: W_gate, W₁, W₂
```

Parameter count comparison (ignoring biases, Llama 3 uses none):

$$\text{ReLU params} = d \cdot d_\text{ff} + d_\text{ff} \cdot d = 2 \, d \, d_\text{ff}$$

$$\text{SwiGLU params} = 3 \, d \, d_\text{ff}$$

**Numeric example:** $d = 4096$, $d_\text{ff} = 11008$ (Llama 3 7B).

$\text{ReLU} = 2 \times 4096 \times 11008 \approx 90.2\text{M params}$.

$\text{SwiGLU} = 3 \times 4096 \times 11008 \approx 135.3\text{M params}$.

To keep total parameter count equal, Llama 3 reduces $d_\text{ff}$ by factor $\frac{2}{3}$:

$$d_\text{ff}^{\text{SwiGLU}} = \frac{2}{3} d_\text{ff}^{\text{ReLU}} \implies 3 \cdot d \cdot \frac{2}{3} d_\text{ff} = 2 \, d \, d_\text{ff}$$

| Property | ReLU FFN | SwiGLU FFN |
|---|---|---|
| Weight matrices | 2 ($W_1, W_2$) | 3 ($W_\text{gate}, W_1, W_2$) |
| Activation | $\max(0, z)$ | $z \cdot \sigma(z)$ (Swish) |
| Gating mechanism | None | Element-wise product with gate |
| Negative inputs | Zeroed out | Small negatives pass through |
| $d_\text{ff}$ adjustment | Baseline | Reduced by $\frac{2}{3}$ for param parity |
| Used in | Original Transformer | Llama 3, PaLM, Gemma |

---
---

# Quiz 5, Question 5 — KV Cache Memory Bandwidth Bottleneck

> **During autoregressive text generation, which Transformer component is primarily responsible for the "memory bandwidth bottleneck"?**
>
> **C. Loading the Key-Value (KV) cache for all previous tokens from GPU memory.**

---

## Sub-questions

### What is autoregressive generation?

$$P(x_1, x_2, \dots, x_T) = \prod_{t=1}^{T} P(x_t \mid x_1, \dots, x_{t-1})$$

Tokens are generated one at a time, left to right. At step $t$, the model computes attention over all $t-1$ previous tokens to produce token $t$.

### What is the KV cache?

During generation, each new token needs the Keys and Values of **all** previous tokens to compute attention. Re-computing them every step would be $O(T^2)$ in FLOPs.

The KV cache stores previously computed $K$ and $V$ tensors so they are reused:

$$K_{\text{cache}} \in \mathbb{R}^{L \times T \times d_k}, \qquad V_{\text{cache}} \in \mathbb{R}^{L \times T \times d_v}$$

$L$ = number of layers, $T$ = sequence length so far, $d_k = d_v$ = head dimension.

**Numeric example:** Llama 3 8B: $L = 32$, $d_\text{model} = 4096$, $n_\text{heads} = 32$, FP16, $T = 4096$.

$$\text{KV cache} = 2 \times 32 \times 4096 \times 4096 \times 2 \text{ bytes} = 2 \text{ GB}$$

### What is memory bandwidth?

$$\text{Bandwidth} = \frac{\text{bytes transferred}}{\text{time}} \quad (\text{GB/s})$$

Rate at which data moves between GPU HBM (high-bandwidth memory) and the GPU compute cores (SMs). A100: $\sim 2$ TB/s. H100: $\sim 3.35$ TB/s.

### Why is generation "memory-bound" not "compute-bound"?

At each decoding step, the new token's query $q_t \in \mathbb{R}^{1 \times d_k}$ is multiplied against the entire $K_\text{cache} \in \mathbb{R}^{T \times d_k}$:

$$\text{FLOPs per step} = O(T \cdot d_k) \quad \text{(one vector-matrix multiply)}$$

$$\text{Bytes loaded per step} = O(L \cdot T \cdot d_k \cdot \text{bytes\_per\_param})$$

The arithmetic intensity (FLOPs / byte) is extremely low — roughly 1 FLOP per byte loaded. GPU compute units sit idle waiting for data to arrive from HBM.

---

## Main answer

**C. Loading the Key-Value (KV) cache for all previous tokens from GPU memory.**

```
Autoregressive decode step t:

  GPU HBM (slow)                         GPU SMs (fast, idle)
  ┌──────────────────┐                   ┌──────────────┐
  │  KV Cache        │ ═══load═══════▶   │              │
  │  Layer 1: K,V    │  2 GB for 4K seq  │  q_t · K^T   │
  │  Layer 2: K,V    │                   │  = attention  │
  │  ...             │  bottleneck:      │  scores       │
  │  Layer 32: K,V   │  bandwidth-       │              │
  │                  │  limited           │  Result:     │
  │  ≈ 2 GB total    │                   │  1 new token │
  └──────────────────┘                   └──────────────┘

  Time = bytes / bandwidth               Compute utilization: LOW
       = 2 GB / 2 TB/s                   (waiting for data)
       = 1 ms per step
```

```
Arithmetic intensity comparison:

  Training (batch of sequences):
    FLOPs:  O(B · T² · d)  = massive
    Bytes:  O(params)       = fixed
    Ratio:  HIGH → compute-bound ✓

  Inference (1 token at a time):
    FLOPs:  O(T · d)       = small
    Bytes:  O(L · T · d)   = large (KV cache reload)
    Ratio:  LOW → memory-bound ✗ ← BOTTLENECK
```

The KV cache must be **re-read from HBM every decoding step**, for every layer. As sequence length $T$ grows, the cache grows linearly, and the memory bandwidth wall dominates latency.

| Component | Data size per step | Compute per step | Bottleneck? |
|---|---|---|---|
| Model weights | Fixed ($\sim$16 GB for 8B) | $O(d^2)$ per layer | Shared across steps |
| KV cache load | $O(L \cdot T \cdot d)$ — **grows** | $O(T \cdot d)$ per layer | **Yes — bandwidth-limited** |
| New token embedding | $O(d)$ — tiny | Negligible | No |
| Softmax + output | $O(V)$ — vocab size | $O(V \cdot d)$ | Secondary |

---
---

# Quiz 5, Question 6 — Grouped-Query Attention (GQA)

> **To mitigate the KV cache memory bottleneck, Llama 3 implements Grouped-Query Attention (GQA). Which of the following best describes GQA?**
>
> **C. The Query heads are divided into groups, and each group shares a single Key head and Value head.**

---

## Sub-questions

### What is Multi-Head Attention (MHA)?

$$\text{MHA}(Q, K, V) = \text{Concat}(\text{head}_1, \dots, \text{head}_h) \, W^O$$

$$\text{head}_i = \text{Attention}(Q W_i^Q, \, K W_i^K, \, V W_i^V)$$

Each head $i$ has its **own** $W_i^Q$, $W_i^K$, $W_i^V$. For $h$ heads: $h$ Query projections, $h$ Key projections, $h$ Value projections.

**KV cache size in MHA:**

$$\text{KV}_{\text{MHA}} = 2 \times L \times h \times T \times d_k \times \text{bytes}$$

**Numeric example:** $h = 32$, $L = 32$, $T = 4096$, $d_k = 128$, FP16 → $2 \times 32 \times 32 \times 4096 \times 128 \times 2 = 2$ GB.

### What is Multi-Query Attention (MQA)?

$$\text{All } h \text{ query heads share a single K and V: } K = X W^K, \; V = X W^V$$

Only 1 Key head and 1 Value head, shared across all $h$ Query heads. KV cache shrinks by factor $h$:

$$\text{KV}_{\text{MQA}} = 2 \times L \times 1 \times T \times d_k \times \text{bytes} = \frac{\text{KV}_{\text{MHA}}}{h}$$

**Numeric example:** $\frac{2 \text{ GB}}{32} = 64$ MB. Massive reduction, but quality degrades — too much sharing.

### What is Grouped-Query Attention (GQA)?

$$h_Q \text{ query heads} \div g \text{ groups} = \frac{h_Q}{g} \text{ query heads per group}$$

Each group shares **one** K head and **one** V head. Total KV heads = $g$.

$$\text{KV}_{\text{GQA}} = 2 \times L \times g \times T \times d_k \times \text{bytes}$$

GQA is a middle ground: $g = h$ → MHA; $g = 1$ → MQA.

**Numeric example:** Llama 3 8B: $h_Q = 32$, $g = 8$. Each group: 4 query heads share 1 KV head.

$\text{KV}_\text{GQA} = 2 \times 32 \times 8 \times 4096 \times 128 \times 2 = 512$ MB (vs 2 GB for MHA).

---

## Main answer

**C. The Query heads are divided into groups, and each group shares a single Key head and Value head.**

```
Multi-Head Attention (MHA) — h=8, 8 KV heads:

  Q heads:  Q₁  Q₂  Q₃  Q₄  Q₅  Q₆  Q₇  Q₈
             │   │   │   │   │   │   │   │
  K heads:  K₁  K₂  K₃  K₄  K₅  K₆  K₇  K₈
  V heads:  V₁  V₂  V₃  V₄  V₅  V₆  V₇  V₈

  KV cache: 8 × (K + V) per layer


Grouped-Query Attention (GQA) — h=8, g=2 groups:

  Q heads:  Q₁  Q₂  Q₃  Q₄ │ Q₅  Q₆  Q₇  Q₈
             │   │   │   │  │  │   │   │   │
  K heads:  └───┴───┴───K₁─┘  └───┴───┴───K₂─┘
  V heads:  └───┴───┴───V₁─┘  └───┴───┴───V₂─┘

  KV cache: 2 × (K + V) per layer  →  4× smaller


Multi-Query Attention (MQA) — h=8, g=1:

  Q heads:  Q₁  Q₂  Q₃  Q₄  Q₅  Q₆  Q₇  Q₈
             │   │   │   │   │   │   │   │
  K heads:  └───┴───┴───┴───┴───┴───┴───K₁─┘
  V heads:  └───┴───┴───┴───┴───┴───┴───V₁─┘

  KV cache: 1 × (K + V) per layer  →  8× smaller
```

$$\text{KV cache reduction factor} = \frac{h}{g}$$

| Property | MHA ($g = h$) | GQA ($1 < g < h$) | MQA ($g = 1$) |
|---|---|---|---|
| KV heads | $h$ | $g$ | $1$ |
| KV cache size | $2Lhd_kT$ | $2Lgd_kT$ | $2Ld_kT$ |
| Cache reduction | $1\times$ | $\frac{h}{g}\times$ | $h\times$ |
| Quality | Best | Near-MHA | Degraded |
| Inference speed | Slowest | Fast | Fastest |
| Used in | BERT, GPT-2 | **Llama 3**, Gemma | PaLM, Falcon |

---
---

# Quiz 5, Question 7 — Rotary Positional Embeddings (RoPE)

> **Llama 3 utilizes Rotary Positional Embeddings (RoPE). Instead of adding a positional vector to the token embedding, how does RoPE inject positional information?**
>
> **A. By applying a rotation matrix to the Query and Key vectors in the complex plane before the dot product.**

---

## Sub-questions

### What are positional embeddings and why are they needed?

Self-attention is **permutation-invariant**: $\text{Attention}(\{x_1, x_2, x_3\}) = \text{Attention}(\{x_3, x_1, x_2\})$. Without positional information, "dog bites man" = "man bites dog."

Positional embeddings inject token order into the representation.

### What are absolute (additive) positional embeddings?

$$x'_t = x_t + p_t$$

$p_t \in \mathbb{R}^d$ = fixed or learned vector for position $t$. Added directly to the token embedding before attention.

Problem: the positional signal decays through layers. Attention score $q_i^T k_j$ mixes content and position in a complex, entangled way.

### What is a rotation in 2D?

$$R(\theta) = \begin{pmatrix} \cos\theta & -\sin\theta \\ \sin\theta & \cos\theta \end{pmatrix}$$

Rotates a vector by angle $\theta$ without changing its magnitude: $\|R(\theta) \, v\| = \|v\|$.

**Numeric example:** $v = [1, 0]$, $\theta = 90°$:

$$R(90°) \begin{pmatrix}1\\0\end{pmatrix} = \begin{pmatrix}0 \cdot 1 - 1 \cdot 0 \\ 1 \cdot 1 + 0 \cdot 0\end{pmatrix} = \begin{pmatrix}0\\1\end{pmatrix}$$

### How does rotation encode position?

For position $m$, RoPE rotates each pair of dimensions $(2i, 2i+1)$ by angle $m \cdot \theta_i$:

$$\theta_i = \frac{1}{10000^{2i/d}}$$

$m$ = token position (0, 1, 2, ...); $i$ = dimension pair index (0, 1, ..., $d/2 - 1$). Low-frequency rotations for high dimensions, high-frequency for low dimensions.

**Numeric example:** $d = 4$, position $m = 3$:

$\theta_0 = \frac{1}{10000^{0/4}} = 1.0$, $\theta_1 = \frac{1}{10000^{2/4}} = 0.01$

Pair 0 rotated by $3 \times 1.0 = 3.0$ rad. Pair 1 rotated by $3 \times 0.01 = 0.03$ rad.

### Why does RoPE make the dot product depend on relative position?

$$q_m = R(m\theta) \, \tilde{q}, \qquad k_n = R(n\theta) \, \tilde{k}$$

$$q_m^T k_n = \tilde{q}^T R(m\theta)^T R(n\theta) \, \tilde{k} = \tilde{q}^T R((n - m)\theta) \, \tilde{k}$$

The dot product depends only on $n - m$ (relative distance), not on absolute positions $m$ and $n$ individually. This is because $R(\alpha)^T R(\beta) = R(\beta - \alpha)$.

---

## Main answer

**A. By applying a rotation matrix to the Query and Key vectors in the complex plane before the dot product.**

```
Additive positional embedding (original Transformer):

  token emb ──▶ (+) ◄── pos vector p_t ──▶ into attention
                 │
                 ▼
               x'_t = x_t + p_t
  Position added ONCE, before layer 1.
  Signal fades through layers.


RoPE (Llama 3):

  Q, K computed ──▶ Rotate by position ──▶ into dot product

  For each 2D pair (q_{2i}, q_{2i+1}) at position m:

  ┌                       ┐   ┌         ┐     ┌                  ┐
  │ cos(mθᵢ)  -sin(mθᵢ)  │   │ q_{2i}  │     │ q'_{2i}         │
  │                       │ × │         │  =  │                  │
  │ sin(mθᵢ)   cos(mθᵢ)  │   │ q_{2i+1}│     │ q'_{2i+1}       │
  └                       ┘   └         ┘     └                  ┘

  Applied at EVERY layer, to Q and K only (not V).
```

The full rotation matrix for a $d$-dimensional vector is block-diagonal:

$$R_m = \begin{pmatrix} R(m\theta_0) & & \\ & R(m\theta_1) & \\ & & \ddots \\ & & & R(m\theta_{d/2-1}) \end{pmatrix}$$

Each $2 \times 2$ block operates on one dimension pair with its own frequency $\theta_i$.

```
Relative position emerges from the dot product:

  q_m^T k_n  =  q̃^T  R(mθ)^T  R(nθ)  k̃
                       \_____________/
                        R((n-m)θ)

  Only the DIFFERENCE (n - m) matters.

  Position 5 attending to position 3:
    rotation = R((3-5)θ) = R(-2θ)

  Position 100 attending to position 98:
    rotation = R((98-100)θ) = R(-2θ)   ← same!
```

| Property | Additive (Sinusoidal / Learned) | RoPE (Rotary) |
|---|---|---|
| Mechanism | $x' = x + p$ | $q' = R(m\theta) \, q$, $k' = R(m\theta) \, k$ |
| Applied to | Token embedding | Q and K vectors only |
| When applied | Once, before layer 1 | Every layer, before dot product |
| Position type encoded | Absolute | **Relative** (via dot product) |
| Magnitude change | Yes ($\|x + p\| \neq \|x\|$) | No ($\|R \, q\| = \|q\|$) |
| Extrapolation to longer $T$ | Poor | Better (with NTK scaling) |
| Used in | Original Transformer, BERT, GPT-2 | **Llama 3**, Gemma, Mistral |

---
---

# Quiz 5, Question 8 — Causal Attention Mask Shape

> **What is the mathematical shape of the causal attention mask applied during the training of a decoder-only LLM?**
>
> **B. A lower triangular matrix (allowed) with upper triangular elements set to −∞ (masked).**

---

## Sub-questions

### What is an attention score matrix?

$$S = \frac{QK^\top}{\sqrt{d_k}}, \quad S \in \mathbb{R}^{N \times N}$$

$Q$ = query matrix, $K$ = key matrix, $d_k$ = head dimension, $N$ = sequence length.

**Numeric example:** For $N = 4$ tokens, $S$ is a $4 \times 4$ matrix where $S_{ij}$ = how much token $i$ attends to token $j$.

### What does "causal" mean in this context?

Token $i$ may only attend to tokens $j \leq i$ (current and past positions).

$$\text{Allowed}(i,j) = \begin{cases} \text{yes} & \text{if } j \leq i \\ \text{no} & \text{if } j > i \end{cases}$$

**Numeric example:** Token 3 can attend to tokens {1, 2, 3} but not token 4.

### What is a lower triangular matrix?

$$L_{ij} = \begin{cases} a_{ij} & \text{if } j \leq i \\ 0 & \text{if } j > i \end{cases}$$

**Numeric example ($4 \times 4$):**

$$L = \begin{bmatrix} a_{11} & 0 & 0 & 0 \\ a_{21} & a_{22} & 0 & 0 \\ a_{31} & a_{32} & a_{33} & 0 \\ a_{41} & a_{42} & a_{43} & a_{44} \end{bmatrix}$$

### Why use $-\infty$ instead of $0$ for masking?

$$\text{softmax}(z_i) = \frac{e^{z_i}}{\sum_j e^{z_j}}$$

$$e^{-\infty} = 0 \implies \text{softmax}(-\infty) = 0$$

Setting masked positions to $0$ would give $e^{0} = 1$, which is nonzero attention — information leaks.

---

## Main answer

**B. A lower triangular matrix (allowed) with upper triangular elements set to −∞ (masked).**

```
Causal Mask M (N=5):

         Key position j
         1     2     3     4     5
    1 [  0    -inf  -inf  -inf  -inf ]
Q   2 [  0     0    -inf  -inf  -inf ]
u   3 [  0     0     0    -inf  -inf ]
e   4 [  0     0     0     0    -inf ]
r   5 [  0     0     0     0     0   ]
y
i       ^^^^^^^^^^^^         ^^^^^^^^^^^
        Lower triangle:      Upper triangle:
        keep (add 0)         block (add -inf)
```

$$S_{\text{masked}} = S + M, \quad M_{ij} = \begin{cases} 0 & j \leq i \\ -\infty & j > i \end{cases}$$

$$A = \text{softmax}(S_{\text{masked}})$$

```
Information Flow (causal):

  t1 -----> t1
  t2 -----> t1, t2
  t3 -----> t1, t2, t3
  t4 -----> t1, t2, t3, t4

  Future tokens NEVER contribute to current output.
```

| Property | Upper triangular mask ($-\infty$) | Zero mask ($0$) | No mask |
|---|---|---|---|
| $e^{\text{mask value}}$ | $e^{-\infty} = 0$ | $e^{0} = 1$ | N/A |
| Future attention weight | Exactly $0$ | Nonzero (leaks) | Full (bidirectional) |
| Use case | Decoder (GPT, Llama) | Incorrect | Encoder (BERT) |
| Autoregressive valid | Yes | No | No |

---
---

# Quiz 5, Question 9 — Attention Head Dimension

> **In a Transformer with a hidden dimension $d_{\text{model}} = 4096$ and 32 attention heads, what is the dimension $d_k$ of each individual attention head?**
>
> **B. 128**

---

## Sub-questions

### What is $d_{\text{model}}$?

$$d_{\text{model}} = \text{total hidden dimension of the Transformer}$$

The width of every residual-stream vector.

**Numeric example:** $d_{\text{model}} = 4096$ means each token is a vector of 4096 floats.

### What is multi-head attention?

$$\text{MultiHead}(Q, K, V) = \text{Concat}(\text{head}_1, \dots, \text{head}_H) W^O$$

$H$ = number of heads. Each head operates on a subspace of dimension $d_k$.

### What is $d_k$?

$$d_k = \frac{d_{\text{model}}}{H}$$

$H$ = number of attention heads.

**Numeric example:** $d_k = \frac{4096}{32} = 128$.

---

## Main answer

**B. 128**

$$d_k = \frac{d_{\text{model}}}{H} = \frac{4096}{32} = 128$$

```
d_model = 4096
|<------------------------------- 4096 ------------------------------>|
+--------+--------+--------+--------+-----  ...  -----+--------+
| head 1 | head 2 | head 3 | head 4 |                 | head 32|
|  128   |  128   |  128   |  128   |                 |  128   |
+--------+--------+--------+--------+-----  ...  -----+--------+
   d_k      d_k      d_k      d_k                        d_k

32 heads x 128 dims = 4096
```

Each head computes:

$$\text{head}_i = \text{softmax}\!\left(\frac{Q_i K_i^\top}{\sqrt{d_k}}\right) V_i, \quad Q_i \in \mathbb{R}^{N \times 128}, \; K_i \in \mathbb{R}^{N \times 128}$$

| Parameter | Symbol | Value |
|---|---|---|
| Hidden dimension | $d_{\text{model}}$ | 4096 |
| Number of heads | $H$ | 32 |
| Per-head dimension | $d_k = d_v$ | $4096 / 32 = 128$ |
| $Q$ projection per head | $W_i^Q$ | $\mathbb{R}^{4096 \times 128}$ |
| Attention score matrix per head | $QK^\top / \sqrt{d_k}$ | $\mathbb{R}^{N \times N}$ |

---
---

# Quiz 5, Question 10 — SiLU (Swish) Activation

> **Which equation represents the SiLU (Swish) activation function, which forms the basis of the SwiGLU layer?**
>
> **(A)** $f(x) = \max(0, x)$ **(B)** $f(x) = x \cdot \sigma(x)$ **(C)** $f(x) = \tanh(x)$ **(D)** $f(x) = \ln(1 + e^x)$
>
> **B. $f(x) = x \cdot \sigma(x)$**

---

## Sub-questions

### What is the sigmoid function $\sigma(x)$?

$$\sigma(x) = \frac{1}{1 + e^{-x}}$$

Maps any real number to $(0, 1)$.

**Numeric example:** $\sigma(0) = \frac{1}{1 + 1} = 0.5$, $\quad \sigma(2) \approx 0.88$.

### What is SiLU / Swish?

$$\text{SiLU}(x) = x \cdot \sigma(x) = \frac{x}{1 + e^{-x}}$$

Self-gated activation: the input $x$ gates itself through its own sigmoid.

**Numeric example:** $\text{SiLU}(2) = 2 \times 0.88 = 1.76$, $\quad \text{SiLU}(-1) = -1 \times 0.27 = -0.27$.

### What are the other options?

$$\text{ReLU}(x) = \max(0, x) \quad \text{(Option A)}$$
$$\tanh(x) = \frac{e^x - e^{-x}}{e^x + e^{-x}} \quad \text{(Option C)}$$
$$\text{Softplus}(x) = \ln(1 + e^x) \quad \text{(Option D)}$$

---

## Main answer

**B. $f(x) = x \cdot \sigma(x)$**

```
Activation function shapes (x from -4 to 4):

  SiLU/Swish: f(x) = x * sigma(x)
  y
  4 |                                    /
  3 |                                  /
  2 |                               /
  1 |                           . '
  0 |_ _ _ _ _ _ _ _ _ . ' ' '
 -1 |          . _ . '
    |_ _ _ _.'_ _ _ _ _ _ _ _ _ _ _ _ _  x
   -4  -3  -2  -1   0   1   2   3   4

  Key: smooth, non-monotonic dip below 0 near x ~ -1
       Minimum ~ -0.28 at x ~ -1.28

  ReLU: f(x) = max(0, x)
  y
  4 |                                    /
  2 |                                /
  0 |_________________ ____________/
    |                 |
   -4                 0              4   x
  Key: flat zero for x<0, linear for x>0, non-smooth at 0
```

$$\frac{d}{dx}\text{SiLU}(x) = \sigma(x) + x \cdot \sigma(x)(1 - \sigma(x)) = \sigma(x)(1 + x(1 - \sigma(x)))$$

Derivative is smooth everywhere (unlike ReLU which has undefined derivative at 0).

| Activation | Formula | Smooth | Allows negative output | Zero gradient region |
|---|---|---|---|---|
| ReLU (A) | $\max(0, x)$ | No (kink at 0) | No | $x < 0$ (dead neurons) |
| **SiLU/Swish (B)** | $x \cdot \sigma(x)$ | **Yes** | **Yes (small)** | **None** |
| tanh (C) | $\frac{e^x - e^{-x}}{e^x + e^{-x}}$ | Yes | Yes | Saturates at $|x| \gg 0$ |
| Softplus (D) | $\ln(1 + e^x)$ | Yes | No | None |

---
---

# Quiz 5, Question 11 — RoPE Relative Position Property

> **Rotary Positional Embeddings (RoPE) apply a specific rotation to the Query and Key vectors. What mathematical property of this rotation makes RoPE highly effective at capturing positional information?**
>
> **A. The dot product of two rotated vectors depends only on the relative angle (distance) between them.**

---

## Sub-questions

### What is a rotation matrix in 2D?

$$R(\theta) = \begin{bmatrix} \cos\theta & -\sin\theta \\ \sin\theta & \cos\theta \end{bmatrix}$$

Rotates a 2D vector by angle $\theta$ without changing its magnitude.

**Numeric example:** Rotating $\begin{bmatrix}1\\0\end{bmatrix}$ by $\theta = 90°$: $R(90°)\begin{bmatrix}1\\0\end{bmatrix} = \begin{bmatrix}0\\1\end{bmatrix}$.

### How does RoPE encode position $m$?

$$\tilde{q}_m = R(\theta_m) \, q, \quad \tilde{k}_n = R(\theta_n) \, k$$

$\theta_m = m \cdot \theta_{\text{base}}$ where $m$ is the token position and $\theta_{\text{base}}$ is a frequency derived from $d_k$.

For a $d_k$-dimensional vector, RoPE applies rotations in $d_k / 2$ independent 2D planes, each with a different frequency:

$$\theta_i = \frac{m}{10000^{2i/d_k}}, \quad i = 0, 1, \dots, \frac{d_k}{2} - 1$$

**Numeric example:** Position $m = 3$, plane $i = 0$: $\theta_0 = \frac{3}{10000^0} = 3$ radians.

### What is the key dot-product property?

$$\tilde{q}_m^\top \tilde{k}_n = (R(\theta_m) q)^\top (R(\theta_n) k) = q^\top R(\theta_m)^\top R(\theta_n) \, k$$

Since $R(\alpha)^\top = R(-\alpha)$ and $R(-\alpha)R(\beta) = R(\beta - \alpha)$:

$$\tilde{q}_m^\top \tilde{k}_n = q^\top R(\theta_n - \theta_m) \, k = q^\top R((n - m)\theta_{\text{base}}) \, k$$

The result depends on $(n - m)$, the **relative distance**, not the absolute positions $m$ or $n$.

---

## Main answer

**A. The dot product of two rotated vectors depends only on the relative angle (distance) between them.**

```
RoPE rotation mechanism (2D plane):

     Position m=2          Position n=5          Relative = n-m = 3
     rotate by 2*theta     rotate by 5*theta

        q_rot                 k_rot
         \  2*theta            \  5*theta
          \ |                   \ |
           \|                    \|
    --------+-------->    --------+-------->
            |                     |

    dot(q_rot, k_rot) = dot(q, R((5-2)*theta) * k)
                       = dot(q, R(3*theta) * k)
                         ^^^^^^^^^^^^^^^^^^^^
                         Only relative distance 3 matters!
```

$$\boxed{\langle \tilde{q}_m, \tilde{k}_n \rangle = q^\top R\big((n-m)\,\theta_{\text{base}}\big) \, k}$$

```
Positional encoding comparison:

  Sinusoidal (original):  added to embeddings, fixed
       x' = x + PE(pos)

  Learned (GPT-2):  added to embeddings, trained
       x' = x + E_pos[pos]

  RoPE:  multiplied into Q,K via rotation, no additive term
       q' = R(pos) * q
```

| Property | Sinusoidal PE | Learned PE | RoPE |
|---|---|---|---|
| Applied to | Embeddings (additive) | Embeddings (additive) | Q, K only (multiplicative) |
| Relative position | Implicit only | Not inherent | **Explicit via dot product** |
| Extrapolation | Limited | None (fixed table) | **Decays gracefully** |
| Parameter cost | 0 | $N_{\max} \times d_{\text{model}}$ | 0 |
| Key math property | $PE(m+k)$ is linear combo of $PE(m)$ | None | $q_m^\top k_n = f(n-m)$ |

---
---

# Quiz 5, Question 12 — Attention Score Matrix Shape

> **In a standard self-attention layer for a sequence of length $N$, what is the shape of the resulting attention score matrix (pre-softmax), assuming $H$ heads?**
>
> **(A)** $H \times N \times d_k$ **(B)** $H \times N \times N$ **(C)** $N \times d_{\text{model}} \times d_{\text{model}}$ **(D)** $H \times d_k \times d_k$
>
> **B. $H \times N \times N$**

---

## Sub-questions

### How are Q, K, V computed per head?

$$Q_h = X W_h^Q, \quad K_h = X W_h^K, \quad V_h = X W_h^V$$

$X \in \mathbb{R}^{N \times d_{\text{model}}}$, $\quad W_h^Q \in \mathbb{R}^{d_{\text{model}} \times d_k}$.

Result: $Q_h, K_h, V_h \in \mathbb{R}^{N \times d_k}$.

### What is the attention score computation?

$$S_h = \frac{Q_h K_h^\top}{\sqrt{d_k}}$$

$Q_h \in \mathbb{R}^{N \times d_k}$, $\quad K_h^\top \in \mathbb{R}^{d_k \times N}$.

$$\underbrace{(N \times d_k)}_Q \times \underbrace{(d_k \times N)}_{K^\top} = \underbrace{(N \times N)}_S$$

**Numeric example:** $N = 512$, $d_k = 128$: $(512 \times 128)(128 \times 512) = 512 \times 512$.

### Why stack across $H$ heads?

All $H$ heads compute in parallel, so the full tensor is $H$ copies of $N \times N$:

$$S \in \mathbb{R}^{H \times N \times N}$$

---

## Main answer

**B. $H \times N \times N$**

```
Tensor shape flow through multi-head attention:

  Input X: [N, d_model]
       |
       | Split into H heads (via projection)
       v
  Q_h: [H, N, d_k]     K_h: [H, N, d_k]     V_h: [H, N, d_k]
       |                     |
       |   matmul(Q, K^T)   |
       +----------+----------+
                  |
                  v
  Scores S: [H, N, N]      <--- THIS IS THE ANSWER
                  |
                  | softmax (+ mask)
                  v
  Weights A: [H, N, N]
                  |
                  | matmul(A, V)
                  v
  Output: [H, N, d_k]
                  |
                  | concat + W^O
                  v
  Final: [N, d_model]
```

$$S_{ij}^{(h)} = \frac{1}{\sqrt{d_k}} \sum_{l=1}^{d_k} Q_{il}^{(h)} K_{jl}^{(h)}$$

$S_{ij}^{(h)}$ = how much token $i$ attends to token $j$ in head $h$.

| Option | Shape | Interpretation | Correct? |
|---|---|---|---|
| A | $H \times N \times d_k$ | This is the shape of $Q$, $K$, or $V$ | No |
| **B** | $H \times N \times N$ | **Score matrix: every token pair, per head** | **Yes** |
| C | $N \times d_{\text{model}} \times d_{\text{model}}$ | No standard operation produces this | No |
| D | $H \times d_k \times d_k$ | Would require $Q^\top K$ not $Q K^\top$ | No |

---
---

# Quiz 5, Question 13 — LlamaRMSNorm Implementation

> **In the PyTorch implementation of LlamaRMSNorm, which of the following snippets correctly calculates the reciprocal square root of the variance while maintaining numerical stability?**
>
> **(A)** `variance = x.pow(2).mean(-1, keepdim=True); x_norm = x * torch.rsqrt(variance + self.eps)`
> **(B)** `variance = x.mean(-1, keepdim=True).pow(2); x_norm = x / torch.sqrt(variance + self.eps)`
> **(C)** `std = torch.std(x, dim=-1, keepdim=True); x_norm = (x - x.mean()) / (std + self.eps)`
> **(D)** `variance = x.var(-1, keepdim=True); x_norm = x * torch.rsqrt(variance)`
>
> **A.**

---

## Sub-questions

### What is RMSNorm?

$$\text{RMSNorm}(x) = \frac{x}{\text{RMS}(x)} \cdot \gamma$$

$$\text{RMS}(x) = \sqrt{\frac{1}{d}\sum_{i=1}^{d} x_i^2}$$

$\gamma$ = learnable scale parameter, $d$ = dimension of $x$.

Unlike LayerNorm, RMSNorm does **not** subtract the mean (no centering).

**Numeric example:** $x = [1, 2, 3]$, $d=3$: $\text{RMS} = \sqrt{\frac{1+4+9}{3}} = \sqrt{4.67} \approx 2.16$.

### What is `torch.rsqrt`?

$$\texttt{torch.rsqrt}(v) = \frac{1}{\sqrt{v}}$$

Reciprocal square root — one fused operation, numerically faster than separate `1/sqrt`.

**Numeric example:** `torch.rsqrt(4.0) = 0.5`.

### Why add `self.eps`?

$$\frac{1}{\sqrt{v + \epsilon}} \quad \text{prevents division by zero when } v = 0$$

$\epsilon$ is typically $10^{-5}$ or $10^{-6}$.

### Why is Option B wrong?

Option B computes $(\text{mean}(x))^2$ — the square of the mean, not the mean of the squares.

$$(\text{mean}(x))^2 \neq \text{mean}(x^2)$$

**Numeric example:** $x = [1, 3]$: $\text{mean}(x)^2 = 2^2 = 4$, but $\text{mean}(x^2) = \frac{1+9}{2} = 5$.

### Why is Option C wrong?

Option C subtracts the mean and uses `std` — this is **LayerNorm**, not RMSNorm.

### Why is Option D wrong?

Option D uses `.var()` (which subtracts the mean internally) and omits $\epsilon$ — both incorrect for RMSNorm.

---

## Main answer

**A. `variance = x.pow(2).mean(-1, keepdim=True); x_norm = x * torch.rsqrt(variance + self.eps)`**

```
RMSNorm computation flow:

  x = [x_1, x_2, ..., x_d]
        |
        v
  x.pow(2) = [x_1^2, x_2^2, ..., x_d^2]
        |
        v
  .mean(-1) = (1/d) * sum(x_i^2) = variance       <-- mean of squares
        |
        v
  variance + eps                                    <-- numerical stability
        |
        v
  torch.rsqrt(variance + eps) = 1/sqrt(var + eps)  <-- fused reciprocal sqrt
        |
        v
  x * rsqrt(variance + eps) = x_norm               <-- normalize
        |
        v
  x_norm * self.weight                              <-- scale by gamma
```

$$\text{variance} = \frac{1}{d}\sum_{i=1}^{d} x_i^2 \quad \longleftarrow \texttt{x.pow(2).mean(-1)}$$

$$x_{\text{norm}} = x \cdot \frac{1}{\sqrt{\text{variance} + \epsilon}} \quad \longleftarrow \texttt{x * torch.rsqrt(variance + self.eps)}$$

| Option | Operation | Problem |
|---|---|---|
| **A** | `x.pow(2).mean()` then `rsqrt(var + eps)` | **Correct RMSNorm** |
| B | `x.mean().pow(2)` then `sqrt` | Wrong order: $(\bar{x})^2 \neq \overline{x^2}$ |
| C | `std` and mean subtraction | This is LayerNorm, not RMSNorm |
| D | `.var()` and no $\epsilon$ | `.var()` subtracts mean; division by zero risk |

---
---

# Quiz 5, Question 14 — SwiGLU Forward Pass in Llama 3

> **The Llama 3 Feed-Forward block uses the SwiGLU activation. Given the linear layers `gate_proj`, `up_proj`, and `down_proj`, which PyTorch line correctly implements the forward pass of this block?**
>
> **C. `return self.down_proj(F.silu(self.gate_proj(x)) * self.up_proj(x))`**

---

## Sub-questions

### What is a Gated Linear Unit (GLU)?

$$\text{GLU}(x) = (\sigma(xW_{\text{gate}} + b_{\text{gate}})) \odot (xW_{\text{up}} + b_{\text{up}})$$

$\odot$ = element-wise multiplication. One branch produces a gate (values in $[0,1]$), the other produces candidate values.

**Numeric example:** gate output $= [0.9, 0.1, 0.8]$, up output $= [2.0, 3.0, 1.0]$, result $= [1.8, 0.3, 0.8]$.

### What is SwiGLU?

Replace $\sigma$ (sigmoid gate) with SiLU/Swish:

$$\text{SwiGLU}(x) = \text{SiLU}(xW_{\text{gate}}) \odot (xW_{\text{up}})$$

$$\text{SiLU}(z) = z \cdot \sigma(z)$$

### What are the three projections?

$$\texttt{gate\_proj}: \mathbb{R}^{d_{\text{model}}} \to \mathbb{R}^{d_{\text{ff}}} \quad (W_{\text{gate}})$$
$$\texttt{up\_proj}: \mathbb{R}^{d_{\text{model}}} \to \mathbb{R}^{d_{\text{ff}}} \quad (W_{\text{up}})$$
$$\texttt{down\_proj}: \mathbb{R}^{d_{\text{ff}}} \to \mathbb{R}^{d_{\text{model}}} \quad (W_{\text{down}})$$

**Numeric example (Llama 3 8B):** $d_{\text{model}} = 4096$, $d_{\text{ff}} = 14336$.

### Why does SwiGLU use 3 matrices instead of the standard FFN's 2?

Standard FFN: $\text{FFN}(x) = W_2 \cdot \text{ReLU}(W_1 x)$ — two projections.

SwiGLU adds a third ($W_{\text{up}}$) because gating requires two parallel branches before element-wise multiplication.

$$\text{Params}_{\text{FFN}} = 2 \cdot d_{\text{model}} \cdot d_{\text{ff}}$$
$$\text{Params}_{\text{SwiGLU}} = 3 \cdot d_{\text{model}} \cdot d_{\text{ff}}$$

To keep parameter count comparable, Llama uses $d_{\text{ff}} = \frac{2}{3} \cdot 4d_{\text{model}}$ (rounded to a multiple of 256).

---

## Main answer

**C. `return self.down_proj(F.silu(self.gate_proj(x)) * self.up_proj(x))`**

```
SwiGLU Forward Pass:

                          x (d_model)
                         / \
                        /   \
                       v     v
              gate_proj(x)  up_proj(x)
              (d_ff)        (d_ff)
                  |             |
                  v             |
              F.silu()          |
              (d_ff)            |
                  |             |
                  +------*------+     <-- element-wise multiply
                         |
                         v
                    (d_ff)
                         |
                    down_proj()
                         |
                         v
                   output (d_model)
```

$$\text{FFN}_{\text{SwiGLU}}(x) = W_{\text{down}} \Big( \text{SiLU}(W_{\text{gate}} \, x) \odot (W_{\text{up}} \, x) \Big)$$

Step-by-step with shapes ($d_{\text{model}} = 4096$, $d_{\text{ff}} = 14336$):

$$x \in \mathbb{R}^{4096} \xrightarrow{W_{\text{gate}}} \mathbb{R}^{14336} \xrightarrow{\text{SiLU}} \mathbb{R}^{14336}$$
$$x \in \mathbb{R}^{4096} \xrightarrow{W_{\text{up}}} \mathbb{R}^{14336}$$
$$\mathbb{R}^{14336} \odot \mathbb{R}^{14336} = \mathbb{R}^{14336} \xrightarrow{W_{\text{down}}} \mathbb{R}^{4096}$$

| Component | Standard FFN (GPT-2) | SwiGLU FFN (Llama 3) |
|---|---|---|
| Up projection | 1 matrix ($W_1$) | 2 matrices ($W_{\text{gate}}$, $W_{\text{up}}$) |
| Activation | $\text{ReLU}(W_1 x)$ or $\text{GELU}(W_1 x)$ | $\text{SiLU}(W_{\text{gate}} x) \odot W_{\text{up}} x$ |
| Down projection | $W_2$ | $W_{\text{down}}$ |
| Total linear layers | 2 | 3 |
| Gating mechanism | None | Element-wise multiply with `up_proj` |
| PyTorch forward | `W2(relu(W1(x)))` | `down(silu(gate(x)) * up(x))` |

---
---

# Quiz 5, Question 15 — SwiGLU Gating Mechanism

> **The SwiGLU activation utilizes a gating mechanism. What is the primary mathematical operation that applies this "gate" to the main feature branch?**
>
> **C. Element-wise multiplication (Hadamard product)**

---

## Sub-questions

### What is SwiGLU?

$$\text{SwiGLU}(\mathbf{x}) = \bigl(\text{Swish}(\mathbf{x} W_{\text{gate}}) \bigr) \odot \bigl(\mathbf{x} W_{\text{up}}\bigr)$$

A gated activation function combining Swish with a Gated Linear Unit; used in Llama, PaLM, and most modern FFN blocks.

### What is Swish?

$$\text{Swish}(z) = z \cdot \sigma(z), \quad \sigma(z) = \frac{1}{1 + e^{-z}}$$

Smooth, non-monotonic activation that allows small negative signals through.

**Numeric example:** $z = 1.0 \Rightarrow \sigma(1.0) = 0.731 \Rightarrow \text{Swish}(1.0) = 1.0 \times 0.731 = 0.731$

### What is a Gated Linear Unit (GLU)?

$$\text{GLU}(\mathbf{x}) = \sigma(\mathbf{x} W_{\text{gate}}) \odot (\mathbf{x} W_{\text{up}})$$

Two parallel linear projections; one passes through a gate function, then the two branches merge via element-wise multiplication.

### What is the Hadamard product ($\odot$)?

$$(\mathbf{a} \odot \mathbf{b})_i = a_i \cdot b_i$$

Element-wise (component-by-component) multiplication of two tensors of identical shape.

**Numeric example:** $[0.7, -0.1, 0.9] \odot [2.0, 3.0, 1.0] = [1.4, -0.3, 0.9]$

---

## Main answer

**C. Element-wise multiplication (Hadamard product)**

```
SwiGLU Forward Pass
====================

              x  (input, shape [B, T, d])
             / \
            /   \
     x @ W_gate  x @ W_up
        |            |
    Swish(.)      identity
        |            |
     [gate]      [feature]
        \           /
         \         /
          \       /
       gate (*) feature       <--- Hadamard product (element-wise)
              |
          SwiGLU output
              |
          x @ W_down
              |
           FFN output
```

$$\text{FFN}_{\text{SwiGLU}}(\mathbf{x}) = \bigl(\text{Swish}(\mathbf{x} W_{\text{gate}}) \odot \mathbf{x} W_{\text{up}}\bigr) W_{\text{down}}$$

The gate branch produces per-element scaling factors in $(\approx -0.28, +\infty)$; the Hadamard product lets each gate element independently amplify or suppress the corresponding feature element.

| Operation | Symbol | What it does | Shape change |
|---|---|---|---|
| Matrix multiply | $AB$ | Linear projection | $[n,d] \times [d,h] \to [n,h]$ |
| Hadamard product | $A \odot B$ | Element-wise gating | $[n,h] \odot [n,h] \to [n,h]$ |
| Dot product | $\mathbf{a}^\top \mathbf{b}$ | Scalar similarity | $[d] \cdot [d] \to [1]$ |
| Concatenation | $[A ; B]$ | Stack features | $[n,h] + [n,h] \to [n,2h]$ |

Only the Hadamard product preserves dimensionality while allowing per-element gating control --- the defining operation of SwiGLU.

---
---

# Quiz 5, Question 16 — Causal Mask Application in PyTorch

> **During the forward pass of the attention mechanism, we must apply the causal mask to the attention scores before the softmax. If `attn_weights` contains the raw dot products, which PyTorch operation correctly applies the mask?**
>
> **B. `attn_weights = attn_weights.masked_fill(causal_mask == 0, float('-inf'))`**

---

## Sub-questions

### What are raw attention scores (attn_weights)?

$$S = \frac{Q K^\top}{\sqrt{d_k}}, \quad S \in \mathbb{R}^{T \times T}$$

$S_{ij}$ = scaled dot-product similarity between query at position $i$ and key at position $j$.

**Numeric example:** $d_k = 64 \Rightarrow \sqrt{d_k} = 8$; if $q_i \cdot k_j = 16$, then $S_{ij} = 16/8 = 2.0$

### What is the causal mask?

$$M_{ij} = \begin{cases} 1 & \text{if } j \le i \\[4pt] 0 & \text{if } j > i \end{cases}$$

Lower-triangular binary matrix that prevents position $i$ from attending to any future position $j > i$.

```
Causal mask M (T=4):

     key j ->  0   1   2   3
query i
   0        [  1   0   0   0 ]
   1        [  1   1   0   0 ]
   2        [  1   1   1   0 ]
   3        [  1   1   1   1 ]
```

### What does `masked_fill` do?

$$\text{masked\_fill}(S, \text{condition}, v)_{ij} = \begin{cases} v & \text{if condition}_{ij} \text{ is True} \\ S_{ij} & \text{otherwise} \end{cases}$$

Replaces selected entries in-place without changing tensor shape.

### Why $-\infty$ and not 0?

$$\text{softmax}(z_i) = \frac{e^{z_i}}{\sum_j e^{z_j}}, \quad e^{-\infty} = 0$$

Setting masked positions to $-\infty$ guarantees exactly zero probability after softmax; setting to $0$ would still leave $e^0 = 1$, leaking attention to future tokens.

**Numeric example:** $\text{softmax}([2.0, -\infty, 1.0]) = \frac{[e^2, 0, e^1]}{e^2 + 0 + e^1} = \frac{[7.39, 0, 2.72]}{10.11} = [0.731, 0, 0.269]$

---

## Main answer

**B. `attn_weights = attn_weights.masked_fill(causal_mask == 0, float('-inf'))`**

```
Causal Masking Pipeline
========================

  Q @ K^T / sqrt(d_k)          Raw scores S (T x T)
        |
        v
  +--------------------------+
  | S =  2.0  1.5  0.8  3.1 |
  |      0.3  2.7  1.2  0.5 |
  |      1.1  0.9  3.0  2.2 |
  |      0.7  1.8  0.4  2.9 |
  +--------------------------+
        |
        v  masked_fill(causal_mask == 0, -inf)
  +--------------------------+
  | S =  2.0 -inf -inf -inf |
  |      0.3  2.7 -inf -inf |
  |      1.1  0.9  3.0 -inf |
  |      0.7  1.8  0.4  2.9 |
  +--------------------------+
        |
        v  softmax(dim=-1)
  +--------------------------+
  | P = 1.00  0     0     0  |
  |     0.08  0.92  0     0  |
  |     0.09  0.07  0.84  0  |
  |     0.07  0.22  0.05  0.66|
  +--------------------------+
        |
        v
     P @ V   (context vectors)
```

| Candidate | Code | Correct? | Why |
|---|---|---|---|
| A | `attn_weights[causal_mask == 0] = 0` | No | $e^0 = 1 \neq 0$; future tokens get nonzero attention |
| **B** | `attn_weights.masked_fill(causal_mask == 0, float('-inf'))` | **Yes** | $e^{-\infty} = 0$; future tokens get exactly zero probability |
| C | `attn_weights = attn_weights * causal_mask` | No | Multiplies by 0, but $\text{softmax}(0) \neq 0$ |
| D | `attn_weights.masked_fill(causal_mask == 1, float('-inf'))` | No | Masks past tokens instead of future; inverts causality |

---
---

# Quiz 5, Question 17 — RoPE Complex Tensor Reshaping

> **When applying Rotary Positional Embeddings (RoPE), we often manipulate the Query and Key tensors as complex numbers. Which PyTorch function is commonly used to reshape the last dimension of a real-valued tensor of shape `[..., d]` into a complex tensor of shape `[..., d//2]`?**
>
> **B. `torch.view_as_complex(x.float().reshape(*x.shape[:-1], -1, 2))`**

---

## Sub-questions

### What is RoPE?

$$\text{RoPE}(\mathbf{q}, m) = \mathbf{q} \cdot e^{im\theta}, \quad \theta_k = 10000^{-2k/d}$$

$m$ = token position, $k$ = dimension pair index, $d$ = head dimension.

Encodes absolute position into queries/keys such that their dot product depends only on relative distance.

**Numeric example:** $d=4, k=0 \Rightarrow \theta_0 = 10000^{0/4} = 1.0$; at position $m=3$: rotation angle $= 3 \times 1.0 = 3.0$ radians.

### What is a complex number representation of a 2D rotation?

$$z = a + bi, \quad z \cdot e^{i\phi} = (a\cos\phi - b\sin\phi) + (a\sin\phi + b\cos\phi)i$$

Multiplying by $e^{i\phi}$ rotates the 2D vector $(a, b)$ by angle $\phi$.

**Numeric example:** $z = 1 + 2i$, $\phi = \pi/2$: $z \cdot e^{i\pi/2} = (1)(0) - (2)(1) + [(1)(1) + (2)(0)]i = -2 + 1i$

### What does `torch.view_as_complex` require?

The input must be a float32 tensor whose last dimension is exactly 2, representing $[\text{real}, \text{imag}]$ pairs.

$$\mathbb{R}^{[\ldots, n, 2]} \xrightarrow{\text{view\_as\_complex}} \mathbb{C}^{[\ldots, n]}$$

### Why reshape `[..., d]` to `[..., d//2, 2]` first?

$$[q_0, q_1, q_2, q_3, \ldots, q_{d-1}] \to [(q_0, q_1), (q_2, q_3), \ldots, (q_{d-2}, q_{d-1})]$$

Each consecutive pair $(q_{2k}, q_{2k+1})$ becomes one complex number $q_{2k} + q_{2k+1}i$, yielding $d/2$ complex values --- one per rotation frequency $\theta_k$.

---

## Main answer

**B. `torch.view_as_complex(x.float().reshape(*x.shape[:-1], -1, 2))`**

```
RoPE Tensor Reshaping Pipeline
================================

Input x: shape [B, T, n_heads, d]       (real, e.g. d=64)
              |
              v  .float()                (ensure float32)
         [B, T, n_heads, 64]
              |
              v  .reshape(*x.shape[:-1], -1, 2)
         [B, T, n_heads, 32, 2]          (pair up consecutive dims)
              |                           dim=-1 is [real, imag]
              v  torch.view_as_complex(.)
         [B, T, n_heads, 32]             (complex64, d//2 values)
              |
              v  * freqs_cis             (complex multiply = 2D rotation)
         [B, T, n_heads, 32]             (rotated complex)
              |
              v  torch.view_as_real(.)
         [B, T, n_heads, 32, 2]
              |
              v  .reshape(*original_shape)
         [B, T, n_heads, 64]             (back to real)
```

$$\text{freqs\_cis}[m, k] = e^{im\theta_k} = \cos(m\theta_k) + i\sin(m\theta_k)$$

| Candidate | Function | Input shape req. | Output | Correct for RoPE? |
|---|---|---|---|---|
| A | `torch.view_as_real` | Complex tensor | Real with trailing dim 2 | No (goes wrong direction) |
| **B** | `torch.view_as_complex(x...reshape(..., -1, 2))` | Real `[..., n, 2]` | Complex `[..., n]` | **Yes** |
| C | `torch.complex(real, imag)` | Two separate tensors | Complex | Requires manual split |
| D | `x.to(torch.cfloat)` | Any real tensor | Complex with imag=0 | No (discards half the dims) |

---
---

# Quiz 5, Question 18 — Multi-Head Attention Output Aggregation

> **In a multi-head attention block, after the V vectors are multiplied by the attention probabilities, the outputs of all heads are:**
>
> **C. Concatenated along the hidden dimension and multiplied by an output projection matrix $W_O$.**

---

## Sub-questions

### What is multi-head attention?

$$\text{MultiHead}(Q, K, V) = \text{Concat}(\text{head}_1, \ldots, \text{head}_h) \, W_O$$

$$\text{head}_i = \text{Attention}(Q W_i^Q, K W_i^K, V W_i^V)$$

Each head operates on a $d_k = d_{\text{model}} / h$ dimensional subspace independently.

**Numeric example:** $d_{\text{model}} = 512, h = 8 \Rightarrow d_k = 64$ per head.

### What is the per-head attention output?

$$\text{head}_i = \text{softmax}\!\left(\frac{Q_i K_i^\top}{\sqrt{d_k}}\right) V_i \in \mathbb{R}^{T \times d_k}$$

Each head produces a $[T, d_k]$ context matrix.

### What does concatenation along the hidden dimension mean?

$$\text{Concat}(\text{head}_1, \ldots, \text{head}_h) \in \mathbb{R}^{T \times (h \cdot d_k)} = \mathbb{R}^{T \times d_{\text{model}}}$$

Stack all head outputs side-by-side along the last axis, recovering the full model dimension.

### What is $W_O$?

$$W_O \in \mathbb{R}^{d_{\text{model}} \times d_{\text{model}}}$$

Learned output projection that mixes information across heads.

**Numeric example:** $W_O \in \mathbb{R}^{512 \times 512}$ has $512^2 = 262{,}144$ parameters.

---

## Main answer

**C. Concatenated along the hidden dimension and multiplied by an output projection matrix $W_O$.**

```
Multi-Head Attention (h=4, d_model=512, d_k=128)
==================================================

         Q, K, V  [B, T, 512]
            |
     +------+------+------+------+
     |      |      |      |      |
   W_1^Q  W_2^Q  W_3^Q  W_4^Q   (and W^K, W^V per head)
     |      |      |      |
  head_1  head_2  head_3  head_4
 [B,T,128][B,T,128][B,T,128][B,T,128]
     |      |      |      |
     +------+------+------+
            |
      Concat(dim=-1)
       [B, T, 512]
            |
         @ W_O  [512 x 512]
            |
     MHA output
       [B, T, 512]
```

$$\text{MHA}(Q,K,V) = \underbrace{[\text{head}_1 \| \text{head}_2 \| \cdots \| \text{head}_h]}_{T \times d_{\text{model}}} \cdot \underbrace{W_O}_{d_{\text{model}} \times d_{\text{model}}}$$

| Option | Operation | Correct? | Problem |
|---|---|---|---|
| A | Averaged across heads | No | Loses per-head specialization; not the standard formulation |
| B | Summed element-wise | No | Collapses distinct subspace information |
| **C** | **Concat + $W_O$ projection** | **Yes** | Preserves all head outputs; $W_O$ learns cross-head mixing |
| D | Concatenated only (no $W_O$) | No | Without $W_O$, no inter-head interaction occurs |

---
---

# Quiz 5, Question 19 — Timing of Causal Mask in Decoder Attention

> **In the Multi-Head Attention block of a decoder, when exactly is the causal mask mathematically applied?**
>
> **D. After multiplying Queries and Keys, but before the Softmax function.**

---

## Sub-questions

### What is the full scaled dot-product attention formula?

$$\text{Attention}(Q, K, V) = \text{softmax}\!\left(\frac{QK^\top}{\sqrt{d_k}} + M_{\text{causal}}\right) V$$

where $M_{\text{causal}}$ has $0$ at allowed positions and $-\infty$ at forbidden (future) positions.

### What are the sequential steps in the attention forward pass?

$$\text{Step 1: } S = QK^\top$$
$$\text{Step 2: } S = S \,/\, \sqrt{d_k}$$
$$\text{Step 3: } S = S + M_{\text{causal}} \quad (\text{or } \texttt{masked\_fill})$$
$$\text{Step 4: } P = \text{softmax}(S)$$
$$\text{Step 5: } O = P \, V$$

### Why must masking occur before softmax?

$$\text{softmax}(-\infty) = 0 \quad \text{(exact zero probability)}$$

If masking occurs after softmax, the probabilities have already been normalized over all positions including future tokens; zeroing them out would leave a distribution that no longer sums to 1 and would have already used future-token information in the denominator.

---

## Main answer

**D. After multiplying Queries and Keys, but before the Softmax function.**

```
Decoder Attention Forward Pass — Step-by-Step
===============================================

  Q [B,h,T,d_k]    K [B,h,T,d_k]
       \              /
        \            /
    S = Q @ K^T              Step 1: Raw dot products
     [B, h, T, T]
           |
           v
    S = S / sqrt(d_k)        Step 2: Scale
           |
           v
  +---------------------+
  | S = S.masked_fill(  |    Step 3: CAUSAL MASK  <--- HERE
  |   mask==0, -inf)    |
  +---------------------+
           |
           v
    P = softmax(S, dim=-1)   Step 4: Normalize to probabilities
     [B, h, T, T]            (masked positions become exactly 0)
           |
           v
    O = P @ V                Step 5: Weighted sum of values
     [B, h, T, d_k]
```

| Option | Timing | Correct? | Consequence if used |
|---|---|---|---|
| A | Before $QK^\top$ | No | Cannot mask; scores don't exist yet |
| B | After softmax | No | Probabilities no longer sum to 1; information already leaked into normalization denominator |
| C | After $P \cdot V$ | No | Future information already mixed into output vectors |
| **D** | **After $QK^\top$, before softmax** | **Yes** | $-\infty$ entries become exact 0 after softmax; distribution is correct and causal |

---
---

# Quiz 5, Question 20 — Grouped Query Attention (GQA) Head Ratio

> **If a Llama 3 model uses 32 Query heads and 8 Key/Value heads (GQA), how many Query heads share a single Key/Value head pair?**
>
> **B. 4**

---

## Sub-questions

### What is Grouped Query Attention (GQA)?

$$\text{GQA group size} = G = \frac{n_{\text{Q heads}}}{n_{\text{KV heads}}}$$

GQA partitions query heads into groups; each group shares one Key head and one Value head, reducing KV cache memory and compute while retaining most of multi-head attention's expressiveness.

**Numeric example:** $n_Q = 32, n_{KV} = 8 \Rightarrow G = 32/8 = 4$

### How does GQA relate to MHA and MQA?

$$\text{MHA: } n_{KV} = n_Q \quad (G = 1)$$
$$\text{GQA: } 1 < n_{KV} < n_Q \quad (1 < G < n_Q)$$
$$\text{MQA: } n_{KV} = 1 \quad (G = n_Q)$$

MHA = every query head has its own KV pair. MQA = all query heads share one KV pair. GQA is the interpolation.

### What is the KV cache memory savings from GQA?

$$\text{KV cache size} \propto 2 \times n_{KV} \times d_k \times T \times \text{layers}$$

$2$ = one K tensor + one V tensor, $T$ = sequence length.

**Numeric example (Llama 3 8B):** $n_{KV} = 8$ vs MHA $n_{KV} = 32 \Rightarrow$ KV cache reduced by $\frac{8}{32} = 0.25\times$ (4$\times$ savings).

---

## Main answer

**B. 4**

$$G = \frac{n_Q}{n_{KV}} = \frac{32}{8} = 4$$

Each KV head pair serves exactly 4 query heads.

```
GQA Layout: 32 Query Heads, 8 KV Heads (G=4)
==============================================

KV Head 0        KV Head 1        KV Head 2    ...  KV Head 7
  |                |                |                  |
  +--+--+--+      +--+--+--+      +--+--+--+         +--+--+--+
  |  |  |  |      |  |  |  |      |  |  |  |         |  |  |  |
 Q0 Q1 Q2 Q3    Q4 Q5 Q6 Q7    Q8 Q9 Q10 Q11     Q28 Q29 Q30 Q31

Group 0          Group 1          Group 2      ...  Group 7


KV Cache per layer: 2 * 8 * d_k * T      (instead of 2 * 32 * d_k * T)
Savings factor:     32 / 8 = 4x
```

| Attention Type | $n_Q$ | $n_{KV}$ | $G$ | KV Cache (relative) | Quality |
|---|---|---|---|---|---|
| MHA | 32 | 32 | 1 | $1.00\times$ (baseline) | Highest |
| **GQA (Llama 3)** | **32** | **8** | **4** | $0.25\times$ | Near-MHA |
| MQA | 32 | 1 | 32 | $0.03\times$ | Lower |

---
---

# Quiz 5, Question 21 — KV Cache Memory Computation (Essay)

> **Assume you are generating text autoregressively using a model with the following specifications: Hidden dimension ($d_\text{model}$) = 4096, Number of Layers ($L$) = 32, Number of Attention Heads ($H_\text{query}$) = 32, Grouped-Query Attention with $H_\text{kv}$ = 8, Data type: bfloat16 (2 bytes per parameter). Calculate the exact GPU memory (in Bytes or Megabytes) required to store the KV Cache for a single token across all layers. Show your work.**

---

## Sub-questions

### What is the KV Cache?

During autoregressive generation, the decoder recomputes attention at every step. The KV cache stores previously computed Key and Value projections so they are not recomputed.

```
Step 1: compute K1,V1 for token 1        → cache: [K1,V1]
Step 2: compute K2,V2 for token 2        → cache: [K1,V1 | K2,V2]
Step 3: compute K3,V3 for token 3        → cache: [K1,V1 | K2,V2 | K3,V3]
         ↑ only Q3 is freshly computed;
           K1,K2,V1,V2 are read from cache
```

### What is Grouped-Query Attention (GQA)?

Standard Multi-Head Attention (MHA) uses $H$ independent K and V heads. GQA shares K/V heads across groups of query heads to reduce memory.

$$\text{Group size} = \frac{H_\text{query}}{H_\text{kv}}$$

**Numeric example:** $H_\text{query} = 32,\; H_\text{kv} = 8 \;\Rightarrow\; \text{group size} = 32/8 = 4$ query heads share each KV head.

```
MHA (32 KV heads):       GQA (8 KV heads):

Q1 → K1,V1               Q1 ─┐
Q2 → K2,V2               Q2 ─┼→ K1,V1
Q3 → K3,V3               Q3 ─┤
...                       Q4 ─┘
Q32→ K32,V32              ...
                          Q29─┐
  32 KV pairs stored      Q30─┼→ K8,V8
                          Q31─┤
                          Q32─┘

                           8 KV pairs stored
```

### What is the head dimension $d_k$?

$$d_k = \frac{d_\text{model}}{H_\text{query}}$$

**Numeric example:** $d_k = 4096 / 32 = 128$

Each K or V vector per head has dimensionality 128.

### What is bfloat16?

$$\text{bfloat16: } 1 \text{ sign bit} + 8 \text{ exponent bits} + 7 \text{ mantissa bits} = 16 \text{ bits} = 2 \text{ bytes}$$

**Numeric example:** Storing one scalar costs 2 bytes. A vector of length 128 costs $128 \times 2 = 256$ bytes.

---

## Main answer

### Step 1 — Elements per KV head per layer for one token

Each head stores one K vector and one V vector, each of dimension $d_k$.

$$\text{Elements per head per layer} = 2 \times d_k = 2 \times 128 = 256$$

$2$ = one for K, one for V.

### Step 2 — Elements across all KV heads per layer

$$\text{Elements per layer} = 2 \times H_\text{kv} \times d_k = 2 \times 8 \times 128 = 2{,}048$$

### Step 3 — Elements across all layers

$$\text{Total elements} = 2 \times H_\text{kv} \times d_k \times L = 2 \times 8 \times 128 \times 32 = 65{,}536$$

### Step 4 — Convert to bytes

$$\text{Memory (bytes)} = \text{Total elements} \times \text{bytes per element}$$

$$= 2 \times H_\text{kv} \times d_k \times L \times b$$

where $b$ = bytes per parameter = 2 (bfloat16).

$$= 2 \times 8 \times 128 \times 32 \times 2 = 131{,}072 \text{ bytes}$$

### Step 5 — Convert units

$$131{,}072 \text{ bytes} = \frac{131{,}072}{1{,}024} = 128 \text{ KB} \approx 0.125 \text{ MB}$$

### Full formula (general form)

$$\boxed{\text{KV Cache per token} = 2 \times H_\text{kv} \times d_k \times L \times b}$$

### Verification with MHA baseline

If this were standard MHA ($H_\text{kv} = 32$):

$$2 \times 32 \times 128 \times 32 \times 2 = 524{,}288 \text{ bytes} = 512 \text{ KB}$$

GQA reduction factor: $512 / 128 = 4\times$ memory savings (matches group size).

| Symbol | Value | Meaning |
|---|---|---|
| $d_\text{model}$ | 4096 | Hidden dimension |
| $H_\text{query}$ | 32 | Query heads |
| $H_\text{kv}$ | 8 | Key/Value heads (GQA) |
| $d_k$ | 128 | Per-head dimension |
| $L$ | 32 | Transformer layers |
| $b$ | 2 | Bytes per bfloat16 element |
| **Result** | **131,072 bytes (128 KB)** | **KV cache per token** |

---
---

# Quiz 5, Question 22 — Architecture to Code: The Parallel Decoder Block (Essay)

> **You are researching a high-throughput variant of the standard Llama 3 architecture, inspired by models like PaLM and GPT-J. Instead of the traditional sequential Pre-Norm routing, this variant calculates the Attention and Feed-Forward Network branches in parallel using a shared normalized input. The `__init__` method has already instantiated: `self.input_norm` (RMSNorm), `self.attention` (GQA; takes x, freqs_cis, mask), `self.feed_forward` (SwiGLU FFN; takes x). Write the complete `forward(self, x, freqs_cis, mask)` method.**

---

## Sub-questions

### What is the Sequential (Standard) Pre-Norm Decoder Block?

```
x ─────────────────────────────────────┐
│                                      │ (residual 1)
├→ RMSNorm₁ → Attention ──────────────⊕→ h
                                           │
h ─────────────────────────────────────┐
│                                      │ (residual 2)
├→ RMSNorm₂ → FFN ────────────────────⊕→ out
```

Two norms, two residual additions, sequential.

$$\text{Sequential: } h = x + \text{Attn}(\text{Norm}_1(x)), \quad \text{out} = h + \text{FFN}(\text{Norm}_2(h))$$

### What is the Parallel Decoder Block?

```
              ┌→ Attention(x_norm) ──┐
x → RMSNorm ─┤                      ├→ (+) → (+) → out
              └→ FFN(x_norm) ────────┘    ↑
                                          │
x ────────────────────────────────────────┘
              (residual)
```

One norm, one residual addition, parallel branches.

$$\text{Parallel: } \text{out} = x + \text{Attn}(\text{Norm}(x)) + \text{FFN}(\text{Norm}(x))$$

### What is RMSNorm?

$$\text{RMSNorm}(x) = \frac{x}{\text{RMS}(x)} \cdot \gamma, \quad \text{RMS}(x) = \sqrt{\frac{1}{d}\sum_{i=1}^{d} x_i^2}$$

$\gamma$ = learnable scale parameter, $d$ = dimension.

**Numeric example:** $x = [1, -1, 1, -1] \Rightarrow \text{RMS} = \sqrt{(1+1+1+1)/4} = 1 \Rightarrow \text{RMSNorm}(x) = [1, -1, 1, -1] \cdot \gamma$

### Why parallel instead of sequential?

Parallel execution fuses two serial operations into one GPU kernel pass, reducing latency at the cost of a minor quality trade-off.

---

## Main answer

### The Code

```python
def forward(self, x, freqs_cis, mask):
    # Step 1: Save residual
    residual = x

    # Step 2: Single shared normalization
    x_norm = self.input_norm(x)

    # Step 3: Parallel branches (same normalized input)
    attn_out = self.attention(x_norm, freqs_cis, mask)
    ffn_out = self.feed_forward(x_norm)

    # Step 4: Combined residual addition
    out = residual + attn_out + ffn_out

    return out
```

### Data Flow Diagram

```
                    x  (input tensor, shape: [B, T, d_model])
                    │
            ┌───────┴───────┐
            │               │
            │     input_norm (RMSNorm)
            │               │
            │           x_norm
            │         ┌─────┴─────┐
            │         │           │
            │    attention      feed_forward
            │    (GQA)          (SwiGLU)
            │         │           │
            │      attn_out    ffn_out
            │         │           │
            └────→  ( + )  ←─────┘
                      │
                     out  (shape: [B, T, d_model])
```

### Three Contract Requirements Verified

| Requirement | How Satisfied | Line |
|---|---|---|
| Shared normalization | `x_norm` computed once, passed to both branches | `x_norm = self.input_norm(x)` |
| Parallel execution | Both branches read `x_norm` independently (no data dependency between them) | `attn_out = ...` / `ffn_out = ...` |
| Combined residual | Single addition of residual + both branch outputs | `out = residual + attn_out + ffn_out` |

### Sequential vs Parallel Comparison

| Property | Sequential (Llama 3) | Parallel (PaLM / GPT-J) |
|---|---|---|
| Norm layers per block | 2 (`RMSNorm₁`, `RMSNorm₂`) | 1 (`input_norm`) |
| Residual additions | 2 (after Attn, after FFN) | 1 (combined) |
| FFN input | Output of attention sublayer | Normalized original input |
| Serial dependency | FFN waits for Attn | None (both run simultaneously) |
| Throughput | Lower (two serial steps) | Higher (one fused step) |
| Quality | Slightly better | Slightly worse at small scale, comparable at large scale |

---
---

# Quiz 5, Question 23 — SwiGLU Parameter Scaling (Essay)

> **In the original Transformer, the FFN hidden dimension ($d_\text{ffn}$) is traditionally set to $4 \times d_\text{model}$. Llama 3 uses a SwiGLU FFN, which introduces an additional gating matrix. Suppose you want the SwiGLU FFN block to contain exactly 10 percent more parameters than a standard ReLU FFN block. Calculate the exact algebraic scaling factor required for the SwiGLU hidden dimension ($d_\text{hidden}$) in terms of $d_\text{model}$.**

---

## Sub-questions

### What is the Standard ReLU FFN?

$$\text{FFN}(x) = \text{ReLU}(xW_1 + b_1)W_2 + b_2$$

Two weight matrices (ignoring biases for parameter counting):

$$W_1 \in \mathbb{R}^{d_\text{model} \times d_\text{ffn}}, \quad W_2 \in \mathbb{R}^{d_\text{ffn} \times d_\text{model}}$$

where $d_\text{ffn} = 4 \, d_\text{model}$.

$$P_\text{ReLU} = d_\text{model} \times d_\text{ffn} + d_\text{ffn} \times d_\text{model} = 2 \, d_\text{model} \, d_\text{ffn}$$

**Numeric example:** $d_\text{model} = 4096 \Rightarrow d_\text{ffn} = 16384 \Rightarrow P_\text{ReLU} = 2 \times 4096 \times 16384 = 134{,}217{,}728$

### What is SwiGLU?

$$\text{SwiGLU}(x) = (\text{Swish}(xW_\text{gate}) \odot xW_\text{up}) \, W_\text{down}$$

$\odot$ = element-wise multiplication. Three weight matrices:

$$W_\text{gate} \in \mathbb{R}^{d_\text{model} \times d_\text{hidden}}, \quad W_\text{up} \in \mathbb{R}^{d_\text{model} \times d_\text{hidden}}, \quad W_\text{down} \in \mathbb{R}^{d_\text{hidden} \times d_\text{model}}$$

$$P_\text{SwiGLU} = 3 \, d_\text{model} \, d_\text{hidden}$$

**Numeric example:** If $d_\text{hidden} = d_\text{ffn} = 16384$: $P_\text{SwiGLU} = 3 \times 4096 \times 16384 = 201{,}326{,}592$ (50% more than ReLU FFN).

### What is the Swish activation?

$$\text{Swish}(x) = x \cdot \sigma(\beta x), \quad \sigma(z) = \frac{1}{1+e^{-z}}$$

$\sigma$ = sigmoid, $\beta$ = learnable or fixed to 1. Smooth, non-monotonic alternative to ReLU.

**Numeric example:** $\text{Swish}(1.0) = 1.0 \times \sigma(1.0) = 1.0 \times 0.731 = 0.731$

---

## Main answer

### Step 1 — Standard ReLU FFN parameter count

$$P_\text{ReLU} = 2 \, d_\text{model} \times 4 \, d_\text{model} = 8 \, d_\text{model}^2$$

### Step 2 — Target parameter count (10% more)

$$P_\text{target} = 1.10 \times P_\text{ReLU} = 1.10 \times 8 \, d_\text{model}^2 = 8.8 \, d_\text{model}^2$$

### Step 3 — SwiGLU parameter count (3 matrices)

$$P_\text{SwiGLU} = 3 \, d_\text{model} \, d_\text{hidden}$$

### Step 4 — Equate and solve for $d_\text{hidden}$

$$3 \, d_\text{model} \, d_\text{hidden} = 8.8 \, d_\text{model}^2$$

Divide both sides by $3 \, d_\text{model}$:

$$d_\text{hidden} = \frac{8.8}{3} \, d_\text{model}$$

Convert to exact fraction:

$$\frac{8.8}{3} = \frac{88}{30} = \frac{44}{15}$$

$$\boxed{d_\text{hidden} = \frac{44}{15} \, d_\text{model} \approx 2.9\overline{3} \, d_\text{model}}$$

### Step 5 — Numeric verification

$$d_\text{model} = 4096$$

$$d_\text{hidden} = \frac{44}{15} \times 4096 = \frac{180{,}224}{15} = 12{,}014.9\overline{3}$$

Round to $d_\text{hidden} = 12{,}015$ (in practice, rounded to a multiple of 128 or 256 for GPU efficiency).

$$P_\text{SwiGLU} = 3 \times 4096 \times 12{,}015 = 147{,}701{,}760$$

$$P_\text{ReLU} = 8 \times 4096^2 = 134{,}217{,}728$$

$$\text{Ratio} = \frac{147{,}701{,}760}{134{,}217{,}728} \approx 1.1005 \approx 110\% \;\checkmark$$

### Parameter Scaling Diagram

```
Standard ReLU FFN:          SwiGLU FFN:

x ──→ [W1: d×4d] ──→ ReLU    x ──→ [W_gate: d×d_h] ──→ Swish ──┐
      [W2: 4d×d] ──→ out           [W_up:   d×d_h] ─────────────⊙──→ [W_down: d_h×d] → out
                               
  2 matrices                   3 matrices
  8d² params                   3d·(44/15)d = 8.8d² params
```

| Property | ReLU FFN | SwiGLU FFN (10% budget) |
|---|---|---|
| Weight matrices | 2 ($W_1, W_2$) | 3 ($W_\text{gate}, W_\text{up}, W_\text{down}$) |
| Hidden dim | $4 \, d_\text{model}$ | $\frac{44}{15} \, d_\text{model} \approx 2.933 \, d_\text{model}$ |
| Parameters | $8 \, d_\text{model}^2$ | $8.8 \, d_\text{model}^2$ |
| Activation | $\text{ReLU}(z) = \max(0,z)$ | $\text{Swish}(z) \odot \text{gate}$ |
| Scaling factor | $4\times$ | $\frac{44}{15}\times \approx 2.933\times$ |

---
---

# Quiz 5, Question 24 — RoPE Dot Product Derivation (Essay)

> **Let a 2D query vector be $\mathbf{q} = [q_1, q_2]$ at position $m$, and a 2D key vector be $\mathbf{k} = [k_1, k_2]$ at position $n$. The RoPE rotation matrix for position $t$ and angle $\theta$ is $R_{t,\theta} = \begin{bmatrix} \cos(t\theta) & -\sin(t\theta) \\ \sin(t\theta) & \cos(t\theta) \end{bmatrix}$. Apply $R_{m,\theta}$ to $\mathbf{q}$ and $R_{n,\theta}$ to $\mathbf{k}$. Compute the dot product and prove algebraically that the final attention score is strictly a function of the relative distance $(m - n)$.**

---

## Sub-questions

### What is RoPE (Rotary Position Embedding)?

RoPE encodes position by rotating query/key vectors in 2D subspaces, so the dot product depends only on relative position.

$$R_{t,\theta} = \begin{bmatrix} \cos(t\theta) & -\sin(t\theta) \\ \sin(t\theta) & \cos(t\theta) \end{bmatrix}$$

$t$ = token position, $\theta$ = base frequency angle.

**Numeric example:** $t = 3, \theta = \pi/4 \Rightarrow t\theta = 3\pi/4 \Rightarrow R_{3,\pi/4} = \begin{bmatrix} -0.707 & -0.707 \\ 0.707 & -0.707 \end{bmatrix}$

### What is a rotation matrix?

$$R_\alpha = \begin{bmatrix} \cos\alpha & -\sin\alpha \\ \sin\alpha & \cos\alpha \end{bmatrix}, \quad R_\alpha^T R_\alpha = I, \quad \det(R_\alpha) = 1$$

Rotation matrices are orthogonal: they preserve vector norms and angles.

**Key property:** $R_\alpha^T = R_{-\alpha}$ (transpose = inverse rotation).

### What trigonometric identities are needed?

$$\cos(A)\cos(B) + \sin(A)\sin(B) = \cos(A - B)$$

$$\sin(A)\cos(B) - \cos(A)\sin(B) = \sin(A - B)$$

---

## Main answer

### Step 1 — Apply rotation to $\mathbf{q}$ at position $m$

$$R_{m,\theta}\mathbf{q} = \begin{bmatrix} \cos(m\theta) & -\sin(m\theta) \\ \sin(m\theta) & \cos(m\theta) \end{bmatrix} \begin{bmatrix} q_1 \\ q_2 \end{bmatrix}$$

$$= \begin{bmatrix} q_1\cos(m\theta) - q_2\sin(m\theta) \\ q_1\sin(m\theta) + q_2\cos(m\theta) \end{bmatrix}$$

### Step 2 — Apply rotation to $\mathbf{k}$ at position $n$

$$R_{n,\theta}\mathbf{k} = \begin{bmatrix} k_1\cos(n\theta) - k_2\sin(n\theta) \\ k_1\sin(n\theta) + k_2\cos(n\theta) \end{bmatrix}$$

### Step 3 — Compute dot product

$$(R_{m,\theta}\mathbf{q})^T(R_{n,\theta}\mathbf{k})$$

$$= \bigl[q_1\cos(m\theta) - q_2\sin(m\theta)\bigr]\bigl[k_1\cos(n\theta) - k_2\sin(n\theta)\bigr]$$

$$+ \bigl[q_1\sin(m\theta) + q_2\cos(m\theta)\bigr]\bigl[k_1\sin(n\theta) + k_2\cos(n\theta)\bigr]$$

### Step 4 — Expand all terms

**First product:**

$$q_1 k_1 \cos(m\theta)\cos(n\theta) - q_1 k_2 \cos(m\theta)\sin(n\theta) - q_2 k_1 \sin(m\theta)\cos(n\theta) + q_2 k_2 \sin(m\theta)\sin(n\theta)$$

**Second product:**

$$q_1 k_1 \sin(m\theta)\sin(n\theta) + q_1 k_2 \sin(m\theta)\cos(n\theta) + q_2 k_1 \cos(m\theta)\sin(n\theta) + q_2 k_2 \cos(m\theta)\cos(n\theta)$$

### Step 5 — Collect by coefficient pairs

**$q_1 k_1$ terms:**

$$q_1 k_1 \bigl[\cos(m\theta)\cos(n\theta) + \sin(m\theta)\sin(n\theta)\bigr] = q_1 k_1 \cos\bigl((m-n)\theta\bigr)$$

**$q_2 k_2$ terms:**

$$q_2 k_2 \bigl[\sin(m\theta)\sin(n\theta) + \cos(m\theta)\cos(n\theta)\bigr] = q_2 k_2 \cos\bigl((m-n)\theta\bigr)$$

**$q_1 k_2$ terms:**

$$q_1 k_2 \bigl[-\cos(m\theta)\sin(n\theta) + \sin(m\theta)\cos(n\theta)\bigr] = q_1 k_2 \sin\bigl((m-n)\theta\bigr)$$

**$q_2 k_1$ terms:**

$$q_2 k_1 \bigl[-\sin(m\theta)\cos(n\theta) + \cos(m\theta)\sin(n\theta)\bigr] = -q_2 k_1 \sin\bigl((m-n)\theta\bigr)$$

### Step 6 — Final grouped result

$$\boxed{(R_{m,\theta}\mathbf{q})^T(R_{n,\theta}\mathbf{k}) = \underbrace{(q_1 k_1 + q_2 k_2)}_{\mathbf{q} \cdot \mathbf{k}} \cos\bigl((m-n)\theta\bigr) + \underbrace{(q_1 k_2 - q_2 k_1)}_{\text{cross term}} \sin\bigl((m-n)\theta\bigr)}$$

### Step 7 — Proof of relative-position dependence

Positions $m$ and $n$ appear **only** inside $\cos((m-n)\theta)$ and $\sin((m-n)\theta)$. Define $\Delta = m - n$:

$$\text{score}(\mathbf{q}, \mathbf{k}, \Delta) = (\mathbf{q} \cdot \mathbf{k})\cos(\Delta\theta) + (q_1 k_2 - q_2 k_1)\sin(\Delta\theta)$$

No absolute position $m$ or $n$ survives independently. $\blacksquare$

### Geometric Interpretation (ASCII)

```
        ↑ dim 2
        │      q (at position m)           k (at position n)
        │     ╱  rotated by mθ            ╱  rotated by nθ
        │    ╱                           ╱
        │   ╱  ← angle = mθ            ╱ ← angle = nθ
        │  ╱                           ╱
        │ ╱                           ╱
  ──────┼──────────────────────────────→ dim 1
        │
        │  The dot product between the two rotated vectors
        │  depends on the DIFFERENCE of their rotation angles:
        │
        │     angle between them = mθ − nθ = (m−n)θ
        │
        │  ⟹ dot product = f(m−n), not f(m) or f(n) separately
```

### Summary

| Component | Expression | Depends on |
|---|---|---|
| Rotated query | $R_{m,\theta}\mathbf{q}$ | Absolute position $m$ |
| Rotated key | $R_{n,\theta}\mathbf{k}$ | Absolute position $n$ |
| Dot product (attention score) | $(q_1k_1+q_2k_2)\cos((m-n)\theta) + (q_1k_2-q_2k_1)\sin((m-n)\theta)$ | **Relative position $m-n$ only** |
| Trig identity used | $\cos A\cos B + \sin A\sin B = \cos(A-B)$ | — |

---
---

# Quiz 5, Question 25 — The Causal Mask and Information Leakage (Essay)

> **In decoder-only Transformers, a causal mask is applied to the raw attention scores (the $QK^T$ matrix) before the softmax operation. First, write the mathematical definition of the Softmax function. Second, explain why we add $-\infty$ instead of multiplying by 0. What would mathematically happen to the probability distribution if we multiplied by 0 before the softmax, and how would that affect training?**

---

## Sub-questions

### What is the Softmax function?

$$\text{Softmax}(z_i) = \frac{e^{z_i}}{\displaystyle\sum_{j=1}^{N} e^{z_j}}$$

$z_i$ = raw logit (pre-softmax score) for position $i$, $N$ = sequence length.

**Properties:** Output $\in (0, 1)$ for all inputs, $\sum_i \text{Softmax}(z_i) = 1$.

**Numeric example:** $\mathbf{z} = [2, 1, 0] \Rightarrow e^{\mathbf{z}} = [7.389, 2.718, 1.000] \Rightarrow \text{sum} = 11.107 \Rightarrow \text{Softmax} = [0.665, 0.245, 0.090]$

### What is the causal (autoregressive) constraint?

Token at position $i$ must attend **only** to tokens at positions $\leq i$ (past and self). No future information.

$$\text{Mask}_{ij} = \begin{cases} 0 & \text{if } j \leq i \quad \text{(allowed)} \\ -\infty & \text{if } j > i \quad \text{(blocked)} \end{cases}$$

```
         Keys:  t1   t2   t3   t4
Queries:
  t1          [ 0   -∞   -∞   -∞ ]
  t2          [ 0    0   -∞   -∞ ]
  t3          [ 0    0    0   -∞ ]
  t4          [ 0    0    0    0  ]
```

$0$ = no change to score, $-\infty$ = score sent to negative infinity.

### What is $e^{-\infty}$?

$$\lim_{x \to -\infty} e^x = 0$$

**Numeric example:** $e^{-10} = 0.0000454$, $e^{-100} \approx 10^{-44}$, $e^{-\infty} = 0$ exactly.

### What is $e^0$?

$$e^0 = 1$$

This is the source of the problem when multiplying by 0 instead of adding $-\infty$.

---

## Main answer

### Part 1 — Softmax Definition

$$\boxed{P(z_i) = \frac{e^{z_i}}{\displaystyle\sum_{j=1}^{N} e^{z_j}}}$$

### Part 2 — Why add $-\infty$ instead of multiply by 0

#### Method A: Correct approach (add $-\infty$)

$$z_i^{\text{masked}} = z_i + (-\infty) = -\infty$$

$$\text{Softmax}(-\infty) = \frac{e^{-\infty}}{\sum_j e^{z_j^{\text{masked}}}} = \frac{0}{\sum_j e^{z_j^{\text{masked}}}} = 0$$

Future token receives **exactly zero** attention probability.

**Numeric example:**

$$\mathbf{z} = [3.0, \; 1.5, \; \underbrace{2.0}_{\text{future}}]$$

After adding $-\infty$ to position 3:

$$\mathbf{z}^{\text{masked}} = [3.0, \; 1.5, \; -\infty]$$

$$e^{\mathbf{z}^{\text{masked}}} = [20.09, \; 4.48, \; 0] \quad \text{sum} = 24.57$$

$$\text{Softmax} = [0.818, \; 0.182, \; \mathbf{0.000}] \quad \checkmark$$

#### Method B: Incorrect approach (multiply by 0)

$$z_i^{\text{masked}} = z_i \times 0 = 0$$

$$\text{Softmax}(0) = \frac{e^{0}}{\sum_j e^{z_j^{\text{masked}}}} = \frac{1}{\sum_j e^{z_j^{\text{masked}}}} \neq 0$$

Future token receives a **positive, non-zero** attention probability.

**Numeric example:**

$$\mathbf{z} = [3.0, \; 1.5, \; \underbrace{2.0}_{\text{future}}]$$

After multiplying position 3 by 0:

$$\mathbf{z}^{\text{masked}} = [3.0, \; 1.5, \; 0]$$

$$e^{\mathbf{z}^{\text{masked}}} = [20.09, \; 4.48, \; 1.00] \quad \text{sum} = 25.57$$

$$\text{Softmax} = [0.786, \; 0.175, \; \mathbf{0.039}] \quad \boldsymbol{\times}$$

The future token gets 3.9% of attention -- information leakage.

### Flowchart: Causal Failure Mode

```
Multiply-by-0 masking
━━━━━━━━━━━━━━━━━━━━━

raw score z=2.0 ──→ multiply by 0 ──→ z=0 ──→ exp(0)=1 ──→ Softmax > 0
                                                                    │
                                                          ┌─────────▼─────────┐
                                                          │ INFORMATION LEAK  │
                                                          │ Model attends to  │
                                                          │ future tokens     │
                                                          └─────────┬─────────┘
                                                                    │
                                                          ┌─────────▼─────────┐
                                                          │ TRAINING FAILURE  │
                                                          │ Model learns to   │
                                                          │ "cheat" using     │
                                                          │ future context    │
                                                          └───────────────────┘

Add-negative-infinity masking
━━━━━━━━━━━━━━━━━━━━━━━━━━━━

raw score z=2.0 ──→ add −∞ ──→ z=−∞ ──→ exp(−∞)=0 ──→ Softmax = 0
                                                                │
                                                      ┌─────────▼─────────┐
                                                      │  CORRECT: zero    │
                                                      │  attention to     │
                                                      │  future tokens    │
                                                      └───────────────────┘
```

### Training Impact of Multiply-by-0

$$\text{Context vector: } c_i = \sum_{j=1}^{N} \alpha_{ij} V_j$$

If $\alpha_{ij} > 0$ for future $j > i$, then $c_i$ contains future information $V_j$.

During training (teacher forcing), the model sees the correct answer at every position. With leaky masking, the model learns to extract future tokens from the attention weights, achieving artificially low training loss. At inference (autoregressive, no future tokens available), performance collapses.

### Summary Table

| Property | Add $-\infty$ (Correct) | Multiply by 0 (Incorrect) |
|---|---|---|
| Pre-softmax value for masked position | $-\infty$ | $0$ |
| $e^{\text{(masked value)}}$ | $e^{-\infty} = 0$ | $e^{0} = 1$ |
| Softmax output | Exactly $0$ | Positive value $> 0$ |
| Attention to future tokens | None | Non-zero (information leakage) |
| Autoregressive property | Preserved | Violated |
| Training behavior | Correct learning signal | Model "cheats" with future info |
| Train-test gap | None | Severe (low train loss, high test loss) |

---
---
