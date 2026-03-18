# Quiz 6: Training and Optimization — Tutor Answers (Q3–Q25)

---

# Quiz 6, Question 3 — Accuracy as a Loss Function

> **Why is Accuracy considered an unsuitable loss function for training neural networks via backpropagation?**
>
> **B. It is a discrete step function that provides zero gradients, offering no directional signal for continuous weight updates.**

---

## Sub-questions

### What is Accuracy?

$$\text{Accuracy} = \frac{\text{correct predictions}}{\text{total predictions}}$$

Discrete metric. Output changes only when a prediction flips across a decision threshold — no partial credit for "almost right."

**Numeric example:** 8 correct out of 10 predictions → $\text{Accuracy} = 0.8$.

### What is a loss function?

$$\mathcal{L}(\hat{y}, y) \rightarrow \text{scalar}$$

Measures error between prediction $\hat{y}$ and true label $y$. Must be **differentiable** so gradients can flow through backpropagation.

### What is backpropagation?

$$\theta \leftarrow \theta - \alpha \frac{\partial \mathcal{L}}{\partial \theta}$$

Chain rule applied layer-by-layer to compute $\frac{\partial \mathcal{L}}{\partial \theta}$. Requires $\frac{\partial \mathcal{L}}{\partial \hat{y}} \neq 0$ everywhere — otherwise the update $\Delta\theta = 0$ and weights never change.

### What is a gradient?

$$\nabla_\theta \mathcal{L} = \left[\frac{\partial \mathcal{L}}{\partial \theta_1}, \frac{\partial \mathcal{L}}{\partial \theta_2}, \dots\right]$$

Vector of partial derivatives indicating the direction and magnitude of steepest ascent. Training moves **opposite** to this direction.

**Numeric example:** If $\nabla_\theta \mathcal{L} = [0.5, -0.3]$ and $\alpha = 0.1$, then $\Delta\theta = -0.1 \cdot [0.5, -0.3] = [-0.05, 0.03]$.

---

## Main answer

**B. It is a discrete step function that provides zero gradients, offering no directional signal for continuous weight updates.**

```
Accuracy as a function of model output ŷ:

  Acc
  1.0 ┤                    ●━━━━━━━━━━━━━
      │                    │
      │                    │  ∂Acc/∂ŷ = 0
      │                    │
  0.0 ┤━━━━━━━━━━━━━━━━━━━━●
      └────────────────────┼────────────── ŷ
                        threshold=0.5
                     ∂Acc/∂ŷ = undefined
```

Accuracy is a **step function**: flat at 0, vertical jump at threshold, flat at 1.

$$\frac{\partial \text{Accuracy}}{\partial \hat{y}} = \begin{cases} 0 & \text{on flat regions} \\ \text{undefined} & \text{at the jump} \end{cases}$$

Since $\frac{\partial \mathcal{L}}{\partial \hat{y}} = 0$ almost everywhere:

$$\Delta\theta = -\alpha \cdot \frac{\partial \mathcal{L}}{\partial \hat{y}} \cdot \frac{\partial \hat{y}}{\partial \theta} = -\alpha \cdot 0 \cdot \frac{\partial \hat{y}}{\partial \theta} = 0$$

Weights never update. Training fails.

```
Why accuracy kills training (logic chain):

  Accuracy ──▶ Step function ──▶ ∂L/∂ŷ = 0 ──▶ Δθ = 0 ──▶ Training fails
```

| Property | Accuracy | Cross-Entropy |
|---|---|---|
| Form | Step function | $-\log(\hat{y})$ |
| Differentiable | No | Yes |
| Gradient signal | 0 or undefined | $-1/\hat{y}$ (always nonzero) |
| Usable for backprop | No | Yes |

---
---

# Quiz 6, Question 4 — Softmax Function Purpose

> **What is the primary mathematical purpose of applying the Softmax function to the linear head's output logits?**
>
> **B. To convert unbounded, unnormalized scores into a valid probability distribution that sums to 1.0.**

---

## Sub-questions

### What are logits?

$$z_i = W \cdot h + b$$

Raw, unbounded output scores from the final linear layer. Range: $(-\infty, +\infty)$. Not probabilities — can be negative, can exceed 1, do not sum to 1.

**Numeric example:** For a 3-word vocabulary, logits might be $z = [2.0, -1.0, 0.5]$.

### What is a probability distribution?

A vector $p$ where:

$$p_i \geq 0 \quad \forall \; i, \qquad \sum_i p_i = 1$$

Each element represents the likelihood of one outcome. All elements non-negative. Total sums to exactly 1.

### What is the Softmax function?

$$\text{Softmax}(z_i) = \frac{e^{z_i}}{\sum_{j} e^{z_j}}$$

Exponentiates each logit (making it positive), then normalizes by the sum (making total = 1).

**Numeric example:** $z = [2.0, -1.0, 0.5]$

$$e^{2.0} = 7.389, \quad e^{-1.0} = 0.368, \quad e^{0.5} = 1.649$$

$$\text{sum} = 7.389 + 0.368 + 1.649 = 9.406$$

$$p = [0.786, 0.039, 0.175] \quad (\text{sums to } 1.0)$$

---

## Main answer

**B. To convert unbounded, unnormalized scores into a valid probability distribution that sums to 1.0.**

```
Transformation pipeline:

┌──────────────┐    ┌──────────────┐    ┌──────────────────┐
│ Linear Head  │───▶│   Softmax    │───▶│   Probability    │
│ z = Wh + b   │    │ eᶻⁱ/Σeᶻʲ   │    │   Distribution   │
│              │    │              │    │                  │
│ z ∈ (-∞,+∞)  │    │ guarantees:  │    │ p ∈ [0,1]        │
│ Σzᵢ ≠ 1      │    │ pᵢ > 0      │    │ Σpᵢ = 1          │
│              │    │ Σpᵢ = 1     │    │                  │
└──────────────┘    └──────────────┘    └──────────────────┘
```

Softmax provides two guarantees:

$$\text{1. Positivity: } e^{z_i} > 0 \;\; \forall \; z_i \implies p_i > 0$$

$$\text{2. Normalization: } \frac{e^{z_i}}{\sum_j e^{z_j}} \implies \sum_i p_i = 1$$

| Property | Raw Logits | After Softmax |
|---|---|---|
| Range | $(-\infty, +\infty)$ | $(0, 1)$ |
| Sum constraint | None | Exactly 1.0 |
| Interpretation | Arbitrary scores | Probabilities |
| Used by | — | Cross-Entropy Loss, sampling |

---
---

# Quiz 6, Question 5 — Causal Masking in Transformers

> **How is the Causal Constraint enforced during the parallel processing of a sequence in a Transformer?**
>
> **B. By applying an upper triangular mask that sets j > i positions to negative infinity before the Softmax operation.**

---

## Sub-questions

### What is the Causal Constraint?

$$P(x_t \mid x_1, x_2, \dots, x_{t-1})$$

Token $t$ can only attend to tokens at positions $\leq t$. No "looking ahead." This mimics autoregressive generation where future tokens do not exist yet.

### What is Self-Attention?

$$\text{Attention}(Q, K, V) = \text{Softmax}\!\left(\frac{QK^T}{\sqrt{d_k}}\right) V$$

Each token queries all other tokens. The score matrix $QK^T$ is of shape $(T \times T)$ where entry $(i, j)$ measures how much token $i$ attends to token $j$.

### What happens when you add $-\infty$ before Softmax?

$$\text{Softmax}(-\infty) = \frac{e^{-\infty}}{\sum e^{z_j}} = \frac{0}{\sum e^{z_j}} = 0$$

Setting a score to $-\infty$ guarantees zero attention weight after Softmax. The model completely ignores that position.

**Numeric example:** Scores $[2.0, -\infty, 1.0]$ → $e^{[2.0, -\infty, 1.0]} = [7.39, 0, 2.72]$ → Softmax $= [0.731, 0, 0.269]$.

---

## Main answer

**B. By applying an upper triangular mask that sets j > i positions to $-\infty$ before the Softmax operation.**

The causal mask is an upper-triangular matrix of $-\infty$ values applied to the attention scores:

$$\text{Mask}_{i,j} = \begin{cases} 0 & \text{if } j \leq i \quad (\text{allowed}) \\ -\infty & \text{if } j > i \quad (\text{blocked}) \end{cases}$$

```
Score matrix after masking (4-token sequence):

         j=0    j=1    j=2    j=3
  i=0 [  s₀₀   -∞     -∞     -∞  ]   ← token 0 sees only itself
  i=1 [  s₁₀   s₁₁   -∞     -∞  ]   ← token 1 sees 0,1
  i=2 [  s₂₀   s₂₁   s₂₂   -∞  ]   ← token 2 sees 0,1,2
  i=3 [  s₃₀   s₃₁   s₃₂   s₃₃ ]   ← token 3 sees all

  After Softmax: all -∞ → 0 (zero attention weight)
```

$$\text{Masked\_Scores} = \frac{QK^T}{\sqrt{d_k}} + \text{Mask}$$

$$\text{Attention} = \text{Softmax}(\text{Masked\_Scores}) \cdot V$$

```
Processing flow:

  QKᵀ/√d ──▶ + Causal Mask ──▶ Softmax ──▶ × V ──▶ Output
              (upper tri = -∞)   (-∞ → 0)
```

| Aspect | Without Mask | With Causal Mask |
|---|---|---|
| Token $i$ attends to | All positions $j \in [0, T)$ | Only $j \leq i$ |
| Future leakage | Yes | No |
| Training validity | Invalid (cheating) | Valid autoregressive |
| Where applied | — | Before Softmax, after $QK^T/\sqrt{d_k}$ |

---
---

# Quiz 6, Question 6 — Logits Tensor Reshaping for Cross-Entropy

> **A language model outputs a logits tensor of shape (Batch, Sequence, Vocab). To calculate F.cross_entropy properly, how must this tensor be reshaped?**
>
> **B. It must be reshaped to a 2D tensor (N, C) where N is the total number of predictions (Batch * Sequence) and C is the vocabulary size.**

---

## Sub-questions

### What is the logits tensor shape (Batch, Sequence, Vocab)?

$$\text{logits} \in \mathbb{R}^{B \times S \times V}$$

- $B$ = batch size (number of sequences processed in parallel)
- $S$ = sequence length (number of tokens per sequence)
- $V$ = vocabulary size (number of possible output tokens)

Each of the $B \times S$ positions has a $V$-dimensional vector of raw scores.

**Numeric example:** $B=2, S=4, V=32000$ → tensor contains $2 \times 4 \times 32000 = 256{,}000$ values.

### What does F.cross_entropy expect?

$$\texttt{F.cross\_entropy(input, target)}$$

- `input`: 2D tensor of shape $(N, C)$ — $N$ predictions, each with $C$ class scores
- `target`: 1D tensor of shape $(N,)$ — integer class labels

PyTorch requires this flat 2D layout. It does not natively handle 3D logits.

### What is flattening / reshaping?

$$\text{reshape}(B, S, V) \rightarrow (B \cdot S, \; V)$$

Collapses the batch and sequence dimensions into a single dimension $N = B \cdot S$, keeping vocab as the class dimension $C = V$.

**Numeric example:** $(2, 4, 32000) \rightarrow (8, 32000)$. Eight independent predictions, each over 32000 classes.

---

## Main answer

**B. Reshaped to (N, C) where N = Batch × Sequence and C = Vocab.**

```
Reshaping pipeline:

  ┌──────────────────┐         ┌────────────────┐
  │ (B, S, V)        │  ────▶  │ (B×S, V)       │
  │ 3D logits        │ flatten │ 2D logits      │
  │                  │         │                │
  │ (2, 4, 32000)    │         │ (8, 32000)     │
  └──────────────────┘         └────────────────┘

  ┌──────────────────┐         ┌────────────────┐
  │ (B, S)           │  ────▶  │ (B×S,)         │
  │ 3D targets       │ flatten │ 1D targets     │
  │                  │         │                │
  │ (2, 4)           │         │ (8,)           │
  └──────────────────┘         └────────────────┘
```

In PyTorch:
```python
logits = logits.view(-1, vocab_size)   # (B*S, V)
targets = targets.view(-1)             # (B*S,)
loss = F.cross_entropy(logits, targets)
```

| Dimension | Before | After | Role |
|---|---|---|---|
| Batch ($B$) | Axis 0 | Merged into $N$ | — |
| Sequence ($S$) | Axis 1 | Merged into $N$ | — |
| Vocab ($V$) | Axis 2 | Axis 1 ($C$) | Class scores |
| $N$ | — | $B \times S$ | Total predictions |

---
---

# Quiz 6, Question 7 — Autoregressive Sequence Shifting

> **To properly align autoregressive predictions with ground truth targets for loss calculation, how must the sequences be shifted?**
>
> **C. Drop the last logit from the predictions and the first token from the targets.**

---

## Sub-questions

### What is autoregressive prediction?

$$P(x_t \mid x_1, \dots, x_{t-1})$$

The model predicts the **next** token given all previous tokens. At position $t$, the model's output is a prediction for position $t+1$.

### Why is shifting needed?

The model at position $i$ predicts token $i+1$. The raw logits and targets are **off by one**:

- Logit at position 0 → predicts token at position 1
- Logit at position 1 → predicts token at position 2
- Logit at position $S-1$ → predicts token at position $S$ (doesn't exist)

The last logit has no corresponding target. The first target has no corresponding prediction.

### What does "drop the last logit" mean?

$$\text{shift\_logits} = \text{logits}[:, :-1, :]$$

Remove the final position's prediction (it predicts a token beyond the sequence).

**Numeric example:** Sequence of 5 tokens → logits shape $(B, 5, V)$ → after shift $(B, 4, V)$.

### What does "drop the first target" mean?

$$\text{shift\_targets} = \text{targets}[:, 1:]$$

Remove the first target (no prediction corresponds to it — position 0's logit predicts position 1).

**Numeric example:** Targets $[A, B, C, D, E]$ → after shift $[B, C, D, E]$.

---

## Main answer

**C. Drop the last logit from the predictions and the first token from the targets.**

```
Alignment before shifting (sequence: "The cat sat on"):

  Position:     0       1       2       3
  Input:       The     cat     sat     on
  Logit[i]:   pred→1  pred→2  pred→3  pred→4 (no target!)
  Target:      The     cat     sat     on
               ↑
         (no prediction for this!)

After shifting:

  shift_logits:   logit[0]  logit[1]  logit[2]
  Predicts:        cat       sat       on
  shift_targets:   cat       sat       on       ✓ aligned!
```

In PyTorch:
```python
shift_logits  = logits[:, :-1, :]     # (B, S-1, V)
shift_targets = targets[:, 1:]        # (B, S-1)
loss = F.cross_entropy(
    shift_logits.view(-1, V),
    shift_targets.view(-1)
)
```

$$\text{Original: } S \text{ positions} \xrightarrow{\text{shift}} S-1 \text{ aligned pairs}$$

| Component | Operation | Result Shape |
|---|---|---|
| Logits | `[:, :-1, :]` (drop last) | $(B, S-1, V)$ |
| Targets | `[:, 1:]` (drop first) | $(B, S-1)$ |
| Alignment | logit$[i]$ ↔ target$[i+1]$ | Prediction matches ground truth |

---
---

# Quiz 6, Question 8 — Numerical Danger of Manual Softmax + Log

> **Why is it numerically dangerous to manually calculate Cross-Entropy Loss by applying torch.softmax followed by torch.log?**
>
> **B. Softmax can produce probabilities extremely close to 0, which causes log(0) to evaluate to negative infinity (NaN), crashing the training.**

---

## Sub-questions

### What is Cross-Entropy Loss?

$$\mathcal{L} = -\sum_i y_i \log(\hat{y}_i)$$

For a single ground-truth class $c$: $\mathcal{L} = -\log(\hat{y}_c)$. Requires computing $\log$ of the predicted probability for the correct class.

### What does torch.softmax produce?

$$\hat{y}_i = \frac{e^{z_i}}{\sum_j e^{z_j}}$$

When one logit $z_k$ is much larger than the others, $\hat{y}_i \approx 0$ for $i \neq k$.

**Numeric example:** $z = [100, 0, 0]$ → $\hat{y} \approx [1.0, \; 3.7 \times 10^{-44}, \; 3.7 \times 10^{-44}]$.

### What happens with log(0)?

$$\log(0) = -\infty$$

In floating-point arithmetic, values below $\sim 10^{-45}$ (FP32 denorm limit) round to exactly 0. Then $\log(0)$ produces `-inf` or `NaN`, which propagates through all subsequent computations.

### What is LogSoftmax / log_softmax?

$$\log \text{Softmax}(z_i) = z_i - \log\!\left(\sum_j e^{z_j}\right)$$

Algebraically equivalent but computed in log-space. Never explicitly computes a near-zero probability, so $\log$ never sees 0.

---

## Main answer

**B. Softmax can produce probabilities extremely close to 0, which causes log(0) to evaluate to negative infinity (NaN), crashing the training.**

```
The danger pipeline:

  logits ──▶ softmax() ──▶ tiny prob ≈ 0 ──▶ log(≈0) ──▶ -inf / NaN ──▶ crash
                             ↓
                        FP32 underflow
```

The safe alternative — `F.cross_entropy` combines LogSoftmax + NLLLoss internally:

$$\mathcal{L} = -z_c + \log\!\left(\sum_j e^{z_j}\right)$$

This is the **LogSumExp trick**: never computes $e^{z_i}$ individually, stays in log-space throughout.

```
Safe pipeline:

  logits ──▶ F.cross_entropy() ──▶ stable loss
              (log-domain math)
              never computes p≈0
```

| Approach | Computation | Risk | Result |
|---|---|---|---|
| `softmax()` → `log()` | Explicit $e^z$ then $\log$ | $\log(0) = -\infty$ | NaN / crash |
| `F.cross_entropy()` | $z_c - \log\sum e^{z_j}$ | None (log-space) | Stable |
| `log_softmax()` + `nll_loss()` | $z_i - \log\sum e^{z_j}$ | None (log-space) | Stable |

---
---

# Quiz 6, Question 9 — Gradient Accumulation Loss Scaling

> **You are implementing Gradient Accumulation to simulate a larger batch size. What critical mathematical operation must be applied to the loss before calling loss.backward()?**
>
> **B. Divide the loss by the number of accumulation steps (ACCUM_STEPS).**

---

## Sub-questions

### What is Gradient Accumulation?

Technique to simulate a large effective batch by splitting it across multiple forward/backward passes. Instead of stepping every batch, accumulate gradients over $K$ mini-batches, then step once.

$$\text{Effective Batch Size} = \text{Mini-Batch Size} \times K$$

**Numeric example:** Mini-batch = 4, $K = 8$ → effective batch = 32.

### Why do we need a larger batch size?

Larger batches produce more stable gradient estimates. For LLMs, batch sizes of 256–2048 are typical, but GPU VRAM limits how many samples fit in a single forward pass.

### Why divide the loss?

PyTorch **accumulates** gradients by default (sums them). After $K$ mini-batches, the accumulated gradient is $K$ times larger than the gradient from a single large batch:

$$\nabla_\text{accumulated} = \sum_{k=1}^{K} \nabla_k$$

To get the correct **mean** gradient (as if processing one large batch):

$$\nabla_\text{correct} = \frac{1}{K} \sum_{k=1}^{K} \nabla_k$$

Dividing the loss by $K$ before `backward()` achieves this because:

$$\nabla\!\left(\frac{\mathcal{L}}{K}\right) = \frac{\nabla \mathcal{L}}{K}$$

---

## Main answer

**B. Divide the loss by the number of accumulation steps (ACCUM_STEPS).**

```python
ACCUM_STEPS = 8
for i, (inputs, targets) in enumerate(dataloader):
    loss = model(inputs, targets)
    loss = loss / ACCUM_STEPS          # ← critical scaling
    loss.backward()                    # gradients accumulate in .grad

    if (i + 1) % ACCUM_STEPS == 0:
        optimizer.step()               # apply averaged gradient
        optimizer.zero_grad()           # reset for next accumulation
```

```
Gradient accumulation flow:

  Batch 1 ──▶ loss/K ──▶ backward ──▶ .grad += ∇₁/K
  Batch 2 ──▶ loss/K ──▶ backward ──▶ .grad += ∇₂/K
     ...
  Batch K ──▶ loss/K ──▶ backward ──▶ .grad += ∇ₖ/K
                                          │
                                          ▼
                                    .grad = (1/K)Σ∇ᵢ  ← correct mean
                                          │
                                    optimizer.step()
                                    optimizer.zero_grad()
```

| Scenario | Gradient after $K$ steps | Correct? |
|---|---|---|
| No division | $\sum_{k=1}^{K} \nabla_k$ (sum, $K\times$ too large) | No |
| Divide loss by $K$ | $\frac{1}{K}\sum_{k=1}^{K} \nabla_k$ (mean) | Yes |

---
---

# Quiz 6, Question 10 — AMP GradScaler

> **When using Automatic Mixed Precision (AMP) in PyTorch, what specific problem does the GradScaler solve?**
>
> **C. It temporarily inflates (scales) the loss before backpropagation to prevent tiny FP16 gradients from underflowing to zero.**

---

## Sub-questions

### What is Automatic Mixed Precision (AMP)?

Training technique that uses **FP16** (16-bit) for most computations (faster, less VRAM) while keeping a **FP32** (32-bit) master copy of weights for parameter updates (numerical accuracy).

### What is FP16?

$$\text{FP16 range: } \pm 65{,}504, \quad \text{smallest positive: } \approx 6 \times 10^{-8}$$

16-bit floating point. Half the memory and faster on modern GPUs, but much narrower range than FP32 ($\approx 10^{-38}$ to $10^{38}$).

### What is gradient underflow?

$$|\nabla_\theta \mathcal{L}| < 6 \times 10^{-8} \implies \nabla_\theta \mathcal{L} \approx 0 \quad \text{(in FP16)}$$

Gradients smaller than FP16's minimum representable value get rounded to exactly zero. The parameter receives no update — training signal is lost.

### What is the GradScaler?

Multiplies the loss by a large scale factor $S$ before `backward()`, then divides the gradients by $S$ before `optimizer.step()`:

$$\text{scaled\_loss} = S \cdot \mathcal{L}$$
$$\nabla_\theta(\text{scaled\_loss}) = S \cdot \nabla_\theta \mathcal{L}$$
$$\text{final gradient} = \frac{S \cdot \nabla_\theta \mathcal{L}}{S} = \nabla_\theta \mathcal{L}$$

---

## Main answer

**C. It temporarily inflates (scales) the loss before backpropagation to prevent tiny FP16 gradients from underflowing to zero.**

```
AMP + GradScaler pipeline:

  Forward (FP16)     Scaled Loss        Backward (FP16)      Unscale         Step (FP32)
  ┌────────────┐    ┌─────────────┐    ┌──────────────┐    ┌───────────┐    ┌───────────┐
  │ ŷ = f(x)   │───▶│ L × S       │───▶│ ∇(S·L) =     │───▶│ ∇L = S·∇L/S│───▶│ θ ← θ-α∇L│
  │ in FP16    │    │ inflate loss│    │ S·∇L         │    │ correct   │    │ in FP32   │
  │            │    │             │    │ fits in FP16!│    │ magnitude │    │           │
  └────────────┘    └─────────────┘    └──────────────┘    └───────────┘    └───────────┘
```

**Numeric example:** Gradient $= 1 \times 10^{-9}$ (underflows to 0 in FP16). With scale $S = 1024$:

$$S \cdot \nabla = 1024 \times 10^{-9} = 1.024 \times 10^{-6} \quad \text{(survives in FP16!)}$$

$$\text{Unscaled: } \frac{1.024 \times 10^{-6}}{1024} = 1 \times 10^{-9} \quad \text{(correct value restored in FP32)}$$

```python
scaler = torch.cuda.amp.GradScaler()
with torch.cuda.amp.autocast():           # FP16 forward
    loss = model(inputs, targets)
scaler.scale(loss).backward()              # scale up → backward in FP16
scaler.step(optimizer)                     # unscale → step in FP32
scaler.update()                            # adjust S dynamically
```

| Without GradScaler | With GradScaler |
|---|---|
| Tiny gradients → FP16 underflow → 0 | Scaled gradients → fit in FP16 |
| Lost training signal | Preserved training signal |
| Training stalls or diverges | Training proceeds normally |

---
---

# Quiz 6, Question 11 — AdamW vs Adam

> **How does the AdamW optimizer improve upon the standard Adam optimizer?**
>
> **C. It decouples weight decay, applying it directly to the weights rather than tangling it with Adam's adaptive gradient steps.**

---

## Sub-questions

### What is weight decay?

$$\theta \leftarrow \theta - \lambda \theta = (1 - \lambda)\theta$$

Shrinks weights toward zero each step by factor $(1 - \lambda)$. Acts as regularization, preventing weights from growing excessively large.

**Numeric example:** $\theta = 2.0, \lambda = 0.01$ → $\theta \leftarrow 2.0 \times 0.99 = 1.98$.

### What is the Adam optimizer?

$$m_t = \beta_1 m_{t-1} + (1-\beta_1) g_t \quad \text{(momentum)}$$
$$v_t = \beta_2 v_{t-1} + (1-\beta_2) g_t^2 \quad \text{(adaptive scaling)}$$
$$\theta \leftarrow \theta - \alpha \frac{\hat{m}_t}{\sqrt{\hat{v}_t} + \epsilon}$$

Combines momentum (exponential moving average of gradients) with per-parameter adaptive learning rates (divides by RMS of past gradients).

### How does Adam handle weight decay (L2)?

In standard Adam, weight decay is implemented as **L2 regularization** — added to the loss:

$$\mathcal{L}_\text{reg} = \mathcal{L} + \frac{\lambda}{2}\|\theta\|^2$$

This means the decay gradient $\lambda\theta$ flows through Adam's adaptive scaling ($v_t$), which **distorts** the intended decay.

### What does "decoupled" mean?

Weight decay is applied **directly to the weights** as a separate step, completely outside Adam's gradient machinery:

$$\theta \leftarrow \theta - \alpha \frac{\hat{m}_t}{\sqrt{\hat{v}_t} + \epsilon} - \alpha \lambda \theta$$

The decay term $\alpha\lambda\theta$ is never touched by momentum or adaptive scaling.

---

## Main answer

**C. It decouples weight decay, applying it directly to the weights rather than tangling it with Adam's adaptive gradient steps.**

```
Adam (coupled):                          AdamW (decoupled):

  loss + λ/2·||θ||²                       loss (no regularization term)
       │                                      │
   gradient = ∇L + λθ                    gradient = ∇L
       │                                      │
   Adam(m, v) processes                   Adam(m, v) processes
   EVERYTHING together                   ONLY the true gradient
       │                                      │
   θ ← θ - α·m̂/(√v̂+ε)                  θ ← θ - α·m̂/(√v̂+ε) - αλθ
                                                              ↑
       decay is distorted                     decay applied separately
       by adaptive scaling                    (clean, undistorted)
```

The problem with coupled decay: Adam's $v_t$ adapts the effective learning rate per parameter. When $\lambda\theta$ is mixed into the gradient, parameters with large $v_t$ get **less** decay, and parameters with small $v_t$ get **more** decay — the opposite of the intended uniform shrinkage.

| Property | Adam + L2 | AdamW |
|---|---|---|
| Decay mechanism | Added to loss → gradient | Applied directly to $\theta$ |
| Passes through $m_t, v_t$ | Yes (distorted) | No (clean) |
| Decay is uniform | No (scaled by $1/\sqrt{v_t}$) | Yes |
| Generalization | Weaker | Stronger |

---
---

# Quiz 6, Question 12 — WSD vs Cosine Decay Learning Rate Schedule

> **What is the primary advantage of the Warmup-Stable-Decay (WSD) learning rate schedule over traditional Cosine Decay?**
>
> **B. WSD allows for infinite or continuous pre-training because it does not require foreknowledge of the exact total number of training tokens.**

---

## Sub-questions

### What is a learning rate schedule?

$$\alpha_t = f(t)$$

A function that adjusts the learning rate $\alpha$ over the course of training. Typically starts low (warmup), peaks, then decreases.

### What is Cosine Decay?

$$\alpha_t = \alpha_{\min} + \frac{1}{2}(\alpha_{\max} - \alpha_{\min})\left(1 + \cos\!\left(\frac{\pi \cdot t}{T}\right)\right)$$

Smoothly decreases $\alpha$ from $\alpha_{\max}$ to $\alpha_{\min}$ following a cosine curve. **Requires $T$** (total training steps) to be known in advance — the cosine period is defined by $T$.

### What is Warmup-Stable-Decay (WSD)?

Three phases, no dependency on total steps:

$$\alpha_t = \begin{cases} \alpha_{\max} \cdot \frac{t}{T_w} & \text{warmup } (t < T_w) \\ \alpha_{\max} & \text{stable } (T_w \leq t < T_s) \\ \text{decay from } \alpha_{\max} & \text{decay (final phase)} \end{cases}$$

The stable phase can run **indefinitely**. Decay is only triggered when you decide to stop.

---

## Main answer

**B. WSD allows for infinite or continuous pre-training because it does not require foreknowledge of the exact total number of training tokens.**

```
Cosine Decay (must know T in advance):

  α
  ↑
  αmax ●╲
       │  ╲
       │    ╲  cosine curve
       │      ╲
  αmin │────────●
       └──────────────────▶ t
       0                  T (must be set at start)


WSD (no T required):

  α
  ↑
       ┌──────────────────────────────┐
  αmax │    STABLE (run forever)      │╲
       │                              │  ╲ decay
       │                              │    ╲  (triggered
  αmin ╱                              │──────● when ready)
      ╱ warmup                        │
     ╱                                │
     └────────────────────────────────┴──▶ t
     Tw                              Ts
```

| Property | Cosine Decay | WSD |
|---|---|---|
| Requires total steps $T$ | Yes (at initialization) | No |
| Continuous pre-training | Difficult (must restart schedule) | Native support |
| Shape | Smooth cosine curve | Three flat/linear phases |
| When to decay | Pre-determined by $T$ | User-triggered anytime |
| Used by | GPT-3, BERT | LLaMA 3, modern LLMs |

---
---

# Quiz 6, Question 13 — Gradient Clipping

> **Gradient Clipping (clip_grad_norm_) acts as a safety harness during training. How does it work?**
>
> **B. It rescales the entire gradient vector if its L2 norm exceeds a specified threshold, preventing massive, erratic weight updates.**

---

## Sub-questions

### What is the L2 norm of a gradient vector?

$$\|\nabla\mathcal{L}\|_2 = \sqrt{\sum_i \left(\frac{\partial \mathcal{L}}{\partial \theta_i}\right)^2}$$

Single scalar measuring the total magnitude of all gradients across all parameters.

**Numeric example:** Gradients $= [3.0, 4.0]$ → $\|g\|_2 = \sqrt{9 + 16} = 5.0$.

### What are exploding gradients?

When $\|\nabla\mathcal{L}\|_2$ becomes extremely large (e.g., $10^4$), the weight update $\Delta\theta = -\alpha \cdot \nabla\mathcal{L}$ overshoots catastrophically, causing loss to spike or produce NaN.

### What does clip_grad_norm_ do?

$$\text{If } \|\nabla\mathcal{L}\|_2 > \tau: \quad \nabla\mathcal{L} \leftarrow \nabla\mathcal{L} \cdot \frac{\tau}{\|\nabla\mathcal{L}\|_2}$$

If the total gradient norm exceeds threshold $\tau$, rescale the entire vector so its norm equals $\tau$. Direction is preserved; only magnitude is capped.

**Numeric example:** $g = [30, 40]$, $\|g\| = 50$, $\tau = 1.0$:

$$g_\text{clipped} = [30, 40] \cdot \frac{1.0}{50} = [0.6, 0.8], \quad \|g_\text{clipped}\| = 1.0$$

---

## Main answer

**B. It rescales the entire gradient vector if its L2 norm exceeds a specified threshold, preventing massive, erratic weight updates.**

```
Gradient clipping decision:

  Compute ‖∇L‖₂
       │
       ├── ‖∇L‖₂ ≤ τ  ──▶  No change (gradients are safe)
       │
       └── ‖∇L‖₂ > τ  ──▶  ∇L ← ∇L × (τ / ‖∇L‖₂)
                              │
                              ▼
                         ‖∇L‖₂ = τ  (magnitude capped, direction preserved)
```

```python
torch.nn.utils.clip_grad_norm_(model.parameters(), max_norm=1.0)
```

This is called **after** `loss.backward()` and **before** `optimizer.step()`:

```
zero_grad ──▶ forward ──▶ loss ──▶ backward ──▶ clip_grad_norm_ ──▶ step
                                                  ↑ safety harness
```

| Scenario | $\|\nabla\|_2$ | Action | Result |
|---|---|---|---|
| Normal gradients | $0.5$ (< $\tau=1.0$) | No change | $0.5$ |
| Exploding gradients | $500$ (> $\tau=1.0$) | Rescale by $1/500$ | $1.0$ |
| Direction | — | Always preserved | Same direction, bounded magnitude |

---
---

# Quiz 6, Question 14 — Perplexity from Cross-Entropy Loss

> **During validation, your model achieves a Cross-Entropy Loss of approximately 1.79. What is the model's approximate Perplexity, and what does it represent?**
>
> **C. Perplexity is approx 6.0; the model's uncertainty is equivalent to choosing randomly among 6 equally likely words.**

---

## Sub-questions

### What is Cross-Entropy Loss?

$$\mathcal{L} = -\frac{1}{N}\sum_{i=1}^{N} \log P(x_i \mid x_{<i})$$

Average negative log-probability of the correct next token. Lower = better.

**Numeric example:** If the model assigns probability $0.167$ to the correct token: $\mathcal{L} = -\ln(0.167) \approx 1.79$.

### What is Perplexity?

$$\text{PPL} = e^{\mathcal{L}}$$

Exponential of the cross-entropy loss. Converts log-probability into an intuitive measure: "how many tokens is the model choosing between, on average?"

### What does $e^{1.79}$ equal?

$$e^{1.79} \approx 5.99 \approx 6.0$$

**Verification:** $\ln(6) = 1.7918 \approx 1.79$ ✓

---

## Main answer

**C. Perplexity is approx 6.0; the model's uncertainty is equivalent to choosing randomly among 6 equally likely words.**

$$\text{PPL} = e^{\mathcal{L}} = e^{1.79} \approx 6.0$$

Interpretation: On average, the model is as uncertain as if it were rolling a **6-sided die** for each next token.

```
Perplexity scale:

  PPL = 1     Perfect (100% confident in correct token)
  PPL = 6     ← this model (like choosing among 6 words)
  PPL = 100   Very uncertain
  PPL = V     Random guessing over full vocabulary V
```

```
Loss → Perplexity conversion:

  L = 1.79 ──▶ PPL = e^1.79 ──▶ ≈ 6.0
                │
                ▼
  "The model's average surprise is equivalent
   to uniformly guessing among 6 words"
```

| Cross-Entropy Loss | Perplexity | Interpretation |
|---|---|---|
| 0 | 1 | Perfect prediction |
| 0.69 | 2 | Coin flip between 2 tokens |
| 1.79 | **6** | **Choosing among 6 tokens** |
| 2.30 | 10 | Choosing among 10 tokens |
| 10.37 | 32000 | Random over full vocab |

---
---

# Quiz 6, Question 15 — Overfitting Diagnosis

> **You are monitoring a real-time training dashboard. The Training Loss is steadily decreasing, but the Validation Loss has begun to trend sharply upwards. What is the most likely diagnosis?**
>
> **B. The model is overfitting; it is memorizing training data and failing to generalize to new data.**

---

## Sub-questions

### What is Training Loss?

$$\mathcal{L}_\text{train} = \frac{1}{|\mathcal{D}_\text{train}|}\sum_{(x,y) \in \mathcal{D}_\text{train}} \mathcal{L}(f_\theta(x), y)$$

Error measured on data the model trains on. Always decreases if the model has enough capacity and learning rate is appropriate.

### What is Validation Loss?

$$\mathcal{L}_\text{val} = \frac{1}{|\mathcal{D}_\text{val}|}\sum_{(x,y) \in \mathcal{D}_\text{val}} \mathcal{L}(f_\theta(x), y)$$

Error measured on **held-out** data the model has never seen during training. Measures generalization ability.

### What is overfitting?

$$\mathcal{L}_\text{train} \downarrow \quad \text{while} \quad \mathcal{L}_\text{val} \uparrow$$

The model memorizes training examples (including noise and idiosyncrasies) instead of learning generalizable patterns. Performance on new data degrades.

### What is the generalization gap?

$$\text{Gap} = \mathcal{L}_\text{val} - \mathcal{L}_\text{train}$$

When this gap widens, the model is overfitting. Healthy training: gap remains small.

---

## Main answer

**B. The model is overfitting; it is memorizing training data and failing to generalize to new data.**

```
Loss curves over training steps:

  Loss
   ↑
   │  ╲
   │   ╲  val loss
   │    ╲___________╱─────────── ↗ rising!
   │                 ╲
   │                  ╲  train loss
   │                   ╲__________ ↘ still falling
   │
   └──────────────────────────────────▶ steps
                      ↑
                  overfitting starts here
                  (divergence point)
```

```
Diagnosis logic chain:

  Train Loss ↓ ──▶ Model fits training data better
       +
  Val Loss ↑   ──▶ Model fails on unseen data
       =
  Overfitting ──▶ Memorization, not generalization
```

**Remedies:**

| Technique | Mechanism |
|---|---|
| Early stopping | Stop training at divergence point |
| Dropout | Randomly disable neurons → prevents co-adaptation |
| Weight decay | Penalize large weights → simpler model |
| More training data | Harder to memorize → forces generalization |
| Data augmentation | Synthetic variety → reduces memorization |

| Scenario | Train Loss | Val Loss | Diagnosis |
|---|---|---|---|
| Healthy training | ↓ | ↓ | Learning |
| **Overfitting** | **↓** | **↑** | **Memorizing** |
| Underfitting | High | High | Insufficient capacity |
| Converged | Flat | Flat | Training complete |

---
---

# Quiz 6, Question 16 — model.eval() vs torch.no_grad()

> **Which of the following correctly distinguishes the roles of model.eval() and torch.no_grad() during model evaluation?**
>
> **C. model.eval() alters the behavior of specific layers (disabling Dropout), while torch.no_grad() turns off the computational graph engine to save VRAM and speed up inference.**

---

## Sub-questions

### What is model.eval()?

$$\text{model.training} \leftarrow \texttt{False}$$

Switches certain layers to inference mode:
- **Dropout**: stops randomly zeroing activations (uses all neurons)
- **BatchNorm**: uses running statistics instead of batch statistics

Does **not** affect gradient computation or memory.

### What is torch.no_grad()?

$$\text{requires\_grad} \leftarrow \texttt{False} \quad \text{(within context)}$$

Context manager that disables the autograd engine. No computational graph is built, so:
- `.grad` attributes are not populated
- VRAM for storing intermediate activations is freed
- Forward pass runs faster

### What is Dropout?

$$h_i = \begin{cases} 0 & \text{with probability } p \\ \frac{h_i}{1-p} & \text{with probability } 1-p \end{cases} \quad \text{(training mode)}$$

During training, randomly zeroes activations to prevent co-adaptation. During eval, all neurons are active — predictions must be deterministic and reproducible.

### What is the computational graph?

DAG of tensor operations built during forward pass. Each node stores intermediate values needed for `backward()`. Consumes significant VRAM. Unnecessary during inference.

---

## Main answer

**C. model.eval() alters layer behavior; torch.no_grad() disables the computational graph engine.**

Both are needed during evaluation, but they do completely different things:

```
model.eval()                         torch.no_grad()
┌──────────────────────┐            ┌──────────────────────┐
│ Changes LAYER BEHAVIOR│            │ Changes AUTOGRAD     │
│                      │            │                      │
│ • Dropout → off      │            │ • No graph built     │
│ • BatchNorm → frozen │            │ • No .grad stored    │
│   running stats      │            │ • Less VRAM          │
│                      │            │ • Faster forward     │
│ Does NOT save memory │            │ Does NOT affect      │
│ Does NOT stop grads  │            │ Dropout or BatchNorm │
└──────────────────────┘            └──────────────────────┘
```

Correct evaluation pattern:

```python
model.eval()                          # switch layer behavior
with torch.no_grad():                 # disable autograd
    outputs = model(val_inputs)
    val_loss = criterion(outputs, val_targets)
```

| Aspect | `model.eval()` | `torch.no_grad()` |
|---|---|---|
| What it controls | Layer behavior | Autograd engine |
| Dropout | Disabled | Unaffected |
| BatchNorm | Uses running stats | Unaffected |
| Computational graph | Still built | Not built |
| VRAM savings | None | Significant |
| Speed improvement | Minimal | Noticeable |
| Both needed? | **Yes** | **Yes** |

---
---

# Quiz 6, Question 17 — .safetensors vs .pt Format

> **According to modern machine learning standards, why is the .safetensors format preferred over the traditional .pt / .pth format for saving model checkpoints?**
>
> **B. .safetensors is a pure data format that prevents arbitrary code execution and enables fast, zero-copy memory loading.**

---

## Sub-questions

### What is a .pt / .pth file?

Python's `pickle` serialization of PyTorch tensors. `pickle` can serialize **arbitrary Python objects**, including executable code. Loading a `.pt` file runs `pickle.load()`, which can execute embedded malicious code.

### What is arbitrary code execution?

A malicious `.pt` file can contain Python code that runs automatically when loaded:

```python
# Loading an untrusted .pt file:
model = torch.load("malicious_model.pt")  # ← could run os.system("rm -rf /")
```

The user has no warning — the code executes during deserialization.

### What is .safetensors?

A pure **data-only** format by Hugging Face. Stores tensors as raw byte arrays with a JSON header describing shapes and dtypes. Cannot contain executable code by design.

### What is zero-copy memory loading?

$$\text{Traditional: disk} \xrightarrow{\text{copy 1}} \text{CPU RAM} \xrightarrow{\text{copy 2}} \text{tensors}$$
$$\text{Zero-copy: disk} \xrightarrow{\text{memory map}} \text{tensors (no copy)}$$

Memory-mapped file access: the OS maps the file directly into the process's address space. No intermediate copies needed.

---

## Main answer

**B. .safetensors is a pure data format that prevents arbitrary code execution and enables fast, zero-copy memory loading.**

```
Security comparison:

  .pt / .pth (pickle-based):
  ┌─────────────────────────────────────┐
  │  Python objects + executable code   │
  │  pickle.load() → runs ANY code     │
  │  ⚠ SECURITY RISK                   │
  └─────────────────────────────────────┘

  .safetensors (data-only):
  ┌─────────────────────────────────────┐
  │  JSON header + raw tensor bytes     │
  │  No executable code possible        │
  │  ✓ SAFE BY DESIGN                   │
  └─────────────────────────────────────┘
```

```
Loading speed comparison:

  .pt:           Read file → Unpickle → Allocate → Copy → Tensors
                 (slow, 2+ copies)

  .safetensors:  Memory-map file → Tensors
                 (fast, zero-copy)
```

| Property | `.pt` / `.pth` | `.safetensors` |
|---|---|---|
| Serialization | Python pickle | Raw bytes + JSON header |
| Can contain code | Yes (dangerous) | No (impossible) |
| Loading mechanism | Deserialize objects | Memory-map bytes |
| Memory copies | 2+ | 0 (zero-copy) |
| Loading speed | Slower | Faster |
| Industry standard | Legacy | Modern (Hugging Face Hub) |

---
---

# Quiz 6, Question 18 — ignore_index for Padded Sequences

> **When calculating loss with padded sequences, what PyTorch mechanism prevents the model from artificially lowering its loss by predicting pad tokens?**
>
> **B. The ignore_index parameter in the CrossEntropyLoss function.**

---

## Sub-questions

### What is padding in sequences?

Batching requires all sequences to have the same length. Shorter sequences are extended with a special `<PAD>` token (typically ID 0 or -100).

**Numeric example:** Batch of 2 sequences, max length 5:

```
Sequence 1: [The, cat, sat, <PAD>, <PAD>]
Sequence 2: [I, love, large, language, models]
```

### Why does padding corrupt the loss?

Without masking, the model learns to predict `<PAD>` tokens. Since padding positions follow a trivial pattern, the model achieves near-zero loss on them — inflating its apparent performance while learning nothing useful.

$$\mathcal{L}_\text{naive} = \frac{1}{N_\text{total}} \sum_\text{all positions} \mathcal{L}_i \quad \text{(includes free pad predictions)}$$

### What does ignore_index do?

$$\mathcal{L} = \frac{1}{N_\text{real}} \sum_{i: \; y_i \neq \texttt{ignore\_index}} \mathcal{L}_i$$

Tells `CrossEntropyLoss` to skip any target position whose label equals `ignore_index`. Those positions contribute zero loss and zero gradient.

---

## Main answer

**B. The ignore_index parameter in the CrossEntropyLoss function.**

```python
criterion = nn.CrossEntropyLoss(ignore_index=-100)  # skip pad positions
```

```
How ignore_index works:

  Targets:     [cat, sat, on, <PAD>, <PAD>]
  Labels:      [ 42,  87, 15, -100,  -100 ]
                 ↓    ↓   ↓     ↓      ↓
  Loss:        [L₁,  L₂, L₃,   0,     0  ]  ← ignored!
                 ↓    ↓   ↓
  Final Loss = (L₁ + L₂ + L₃) / 3     ← averaged over real tokens only
```

```
Without ignore_index:

  Loss includes pad predictions ──▶ Model "cheats" by predicting <PAD>
                                ──▶ Loss appears artificially low
                                ──▶ Gradients are diluted

With ignore_index:

  Pad positions excluded ──▶ Loss reflects only real predictions
                          ──▶ No free reward for trivial predictions
                          ──▶ Gradients come from meaningful tokens
```

| Aspect | Without `ignore_index` | With `ignore_index` |
|---|---|---|
| Pad positions | Contribute to loss | Excluded (zero loss) |
| Denominator | All positions ($N_\text{total}$) | Real tokens only ($N_\text{real}$) |
| Model incentive | Predict `<PAD>` for easy reward | Focus on real content |
| Loss accuracy | Artificially low | True performance |

---
---

# Quiz 6, Question 19 — flatten() vs view() for Tensor Reshaping

> **Why is tensor.flatten() often preferred over tensor.view() when reshaping tensors for the loss function?**
>
> **C. flatten() explicitly squashes specified dimensions and automatically handles memory layout, making it safer and less prone to contiguous-memory RuntimeError.**

---

## Sub-questions

### What is tensor.view()?

$$\text{view}(B, S, V) \rightarrow (B \cdot S, V)$$

Reshapes a tensor **without copying data**. Reinterprets the same memory block with a new shape. Requires the tensor to be **contiguous** in memory — otherwise raises `RuntimeError`.

### What does "contiguous" mean?

A tensor is contiguous when its elements are stored in a single, unbroken block of memory in row-major (C) order. Operations like `transpose()`, `permute()`, or slicing can make a tensor non-contiguous.

**Example of non-contiguous:**
```python
x = torch.randn(3, 4)
y = x.t()          # transpose → non-contiguous
y.view(-1)          # RuntimeError!
```

### What is tensor.flatten()?

$$\text{flatten}(\text{start\_dim}, \text{end\_dim}) \rightarrow \text{collapsed tensor}$$

Squashes specified dimensions together. Internally calls `.contiguous().view()` if needed — handles the memory layout automatically.

```python
x = torch.randn(3, 4)
y = x.t()           # non-contiguous
y.flatten()          # works! (calls .contiguous() internally)
```

---

## Main answer

**C. flatten() is safer because it automatically handles memory layout.**

```
view() failure scenario:

  tensor (B, S, V)
       │
       ├── contiguous? ──▶ YES ──▶ view(-1, V) ✓
       │
       └── contiguous? ──▶ NO  ──▶ view(-1, V) ✗ RuntimeError!


flatten() always works:

  tensor (B, S, V)
       │
       ├── contiguous? ──▶ YES ──▶ flatten(0, 1) ✓
       │
       └── contiguous? ──▶ NO  ──▶ .contiguous() ──▶ flatten(0, 1) ✓
                                    (automatic)
```

```python
# view — explicit, fragile:
logits = logits.view(-1, vocab_size)         # may crash

# flatten — explicit, robust:
logits = logits.flatten(0, 1)                # always works
# squashes dims 0 (Batch) and 1 (Seq) → (B*S, V)
```

| Property | `view()` | `flatten()` |
|---|---|---|
| Requires contiguous memory | Yes (crashes if not) | No (handles automatically) |
| Semantics | Generic reshape (any dims) | Explicit: "collapse these dims" |
| Intent clarity | `view(-1, V)` — what does $-1$ mean? | `flatten(0, 1)` — clearly merges dims 0 and 1 |
| Data copy | Never | Only if non-contiguous |
| Safety | Prone to RuntimeError | Robust |

---
---

# Quiz 6, Question 20 — Model Size in FP16

> **If you are training a 7-Billion parameter language model in standard 16-bit floating-point (FP16) precision, what is the approximate size of the model's weights on disk?**
>
> **B. 14 GB**

---

## Sub-questions

### What is a parameter?

A single trainable scalar value (weight or bias) in the model. A "7-Billion parameter" model has $7 \times 10^9$ individual numbers.

### What is FP16 (16-bit floating point)?

$$\text{FP16: } 16 \text{ bits} = 2 \text{ bytes per value}$$

Half-precision float. Stores sign (1 bit), exponent (5 bits), mantissa (10 bits). Range: $\pm 65{,}504$, precision: $\sim 3$ decimal digits.

### How to calculate model size on disk?

$$\text{Size} = \text{Parameters} \times \text{Bytes per Parameter}$$

---

## Main answer

**B. 14 GB**

$$\text{Size} = 7 \times 10^9 \;\text{params} \times 2 \;\text{bytes/param} = 14 \times 10^9 \;\text{bytes} = 14 \;\text{GB}$$

```
Size calculation:

  7,000,000,000 parameters
        ×
  2 bytes per parameter (FP16)
        =
  14,000,000,000 bytes
        =
  14 GB
```

| Precision | Bytes/Param | Formula | 7B Model Size |
|---|---|---|---|
| FP32 (full) | 4 | $7 \times 10^9 \times 4$ | 28 GB |
| **FP16 (half)** | **2** | $7 \times 10^9 \times 2$ | **14 GB** |
| BF16 | 2 | $7 \times 10^9 \times 2$ | 14 GB |
| INT8 | 1 | $7 \times 10^9 \times 1$ | 7 GB |
| INT4 / NF4 | 0.5 | $7 \times 10^9 \times 0.5$ | 3.5 GB |

**Quick rule of thumb:**

$$\text{Size (GB)} \approx \text{Parameters (B)} \times \text{Bytes per Param}$$

$$7\text{B model in FP16} = 7 \times 2 = 14 \;\text{GB}$$

---
---

# Quiz 6, Question 21 — Autoregressive Shifting & Reshaping (Essay)

> **Autoregressive Shifting & Reshaping:** An input batch has the shape (Batch=8, Sequence=1024). The model outputs a raw logits tensor of shape (8, 1024, 32000). To prepare for CrossEntropyLoss, you apply the standard autoregressive shift trick (dropping the last logit and first target) and then flatten the batch and sequence dimensions.
>
> What is the exact 2D shape of the resulting `shift_logits` tensor, and exactly how many individual raw logit values does it contain?

---

## Sub-questions

### What is the autoregressive shift trick?

$$\text{shift\_logits} = \text{logits}[:, :-1, :] \quad \text{(drop last position)}$$
$$\text{shift\_targets} = \text{targets}[:, 1:] \quad \text{(drop first position)}$$

Aligns predictions with targets: logit at position $i$ predicts the token at position $i+1$.

### What happens to the sequence dimension after shifting?

$$S_\text{new} = S - 1 = 1024 - 1 = 1023$$

One position is lost from the shift. The logits go from $(B, 1024, V)$ to $(B, 1023, V)$.

### What does flattening the batch and sequence dimensions do?

$$\text{flatten}(B, S_\text{new}, V) \rightarrow (B \times S_\text{new}, \; V)$$

Merges the batch and (shifted) sequence dimensions into one dimension $N$.

---

## Main answer

### Step 1: Apply the shift

$$\text{shift\_logits shape} = (8, \; 1024 - 1, \; 32000) = (8, \; 1023, \; 32000)$$

### Step 2: Flatten batch and sequence dimensions

$$N = B \times S_\text{new} = 8 \times 1023 = 8184$$

$$\text{2D shape} = (8184, \; 32000)$$

### Step 3: Count total raw logit values

$$\text{Total values} = 8184 \times 32000 = 261{,}888{,}000$$

```
Transformation pipeline:

  (8, 1024, 32000)                    Original logits
        │
        ▼  shift: drop last position
  (8, 1023, 32000)                    Shifted logits
        │
        ▼  flatten dims 0,1
  (8184, 32000)                       Final 2D shape
        │
        ▼  total elements
  8184 × 32000 = 261,888,000         Individual logit values
```

| Step | Operation | Shape |
|---|---|---|
| Original | Model output | $(8, 1024, 32000)$ |
| Shift | `[:, :-1, :]` | $(8, 1023, 32000)$ |
| Flatten | `flatten(0, 1)` | $(8184, 32000)$ |
| **Total values** | $8184 \times 32000$ | **261,888,000** |

---
---

# Quiz 6, Question 22 — The LogSumExp Trick (Essay)

> **The LogSumExp Trick (Algebraic Stability):** To prevent numerical overflow, PyTorch applies the LogSumExp shifting trick before evaluating Cross-Entropy. Imagine your model outputs raw logits for a tiny 3-word vocabulary: $x = [1000, 1000, 1000]$. The true target token is index 1.
>
> Identify the shifting constant $c$ PyTorch will use. Apply the shift, calculate the Softmax probability for the correct token, and find the final Cross-Entropy Loss value. (Leave your answer in terms of natural logarithms, e.g., $\ln(X)$).

---

## Sub-questions

### What is the LogSumExp trick?

$$c = \max(x_1, x_2, \dots, x_n)$$

Subtract $c$ from all logits before exponentiation. This prevents $e^{1000}$ (overflow to `inf`) while producing mathematically identical results:

$$\text{Softmax}(x_i) = \frac{e^{x_i}}{\sum_j e^{x_j}} = \frac{e^{x_i - c}}{\sum_j e^{x_j - c}}$$

### Why does $e^{1000}$ overflow?

$$e^{1000} \approx 10^{434}$$

FP32 max $\approx 3.4 \times 10^{38}$. The value $e^{1000}$ is astronomically beyond this — produces `inf`, then `inf/inf = NaN`.

### What is Cross-Entropy Loss for a single target?

$$\mathcal{L} = -\log P(y_\text{true})$$

Negative log-probability of the correct class.

---

## Main answer

### Step 1: Identify the shifting constant $c$

$$c = \max(1000, 1000, 1000) = 1000$$

### Step 2: Apply the shift

$$x_i - c: \quad [1000 - 1000, \; 1000 - 1000, \; 1000 - 1000] = [0, 0, 0]$$

### Step 3: Calculate Softmax probability for the correct token (index 1)

$$P(\text{index } 1) = \frac{e^0}{e^0 + e^0 + e^0} = \frac{1}{1 + 1 + 1} = \frac{1}{3}$$

### Step 4: Calculate Cross-Entropy Loss

$$\mathcal{L} = -\ln\!\left(\frac{1}{3}\right) = \ln(3)$$

```
Full computation flow:

  x = [1000, 1000, 1000]
       │
       ▼  c = max(x) = 1000
  x - c = [0, 0, 0]
       │
       ▼  Softmax
  P = [e⁰/3, e⁰/3, e⁰/3] = [1/3, 1/3, 1/3]
       │
       ▼  target = index 1 → P = 1/3
  Loss = -ln(1/3) = ln(3)
```

| Step | Computation | Result |
|---|---|---|
| Shifting constant | $c = \max(x)$ | $1000$ |
| Shifted logits | $x_i - c$ | $[0, 0, 0]$ |
| Softmax (target) | $e^0 / (e^0 + e^0 + e^0)$ | $1/3$ |
| **Cross-Entropy Loss** | $-\ln(1/3)$ | $\ln(3) \approx 1.099$ |

---
---

# Quiz 6, Question 23 — Perplexity from Probabilities (Essay)

> **Perplexity from Probabilities:** During a tiny evaluation run, your model processes a batch of 2 tokens. It assigns a probability of 0.5 to the correct first token, and a probability of 0.125 (1/8) to the correct second token.
>
> Based on the definition of Cross-Entropy as "average surprise," calculate the exact average Loss for this batch. Then, calculate the model's overall Perplexity for this batch.

---

## Sub-questions

### What is Cross-Entropy Loss for a single token?

$$\mathcal{L}_i = -\ln P(x_i)$$

Negative natural log of the probability assigned to the correct token. Higher probability → lower loss.

**Numeric example:** $P = 0.5$ → $\mathcal{L} = -\ln(0.5) = \ln(2) \approx 0.693$.

### What is "average surprise"?

$$\bar{\mathcal{L}} = \frac{1}{N}\sum_{i=1}^{N} \mathcal{L}_i = -\frac{1}{N}\sum_{i=1}^{N} \ln P(x_i)$$

Mean of individual token losses across the batch. "Surprise" = $-\ln P$; the less likely a token, the more surprising.

### What is Perplexity?

$$\text{PPL} = e^{\bar{\mathcal{L}}}$$

Exponential of average loss. Converts log-space loss into "effective vocabulary size" — how many equally likely tokens the model is choosing among on average.

---

## Main answer

### Step 1: Compute individual token losses

$$\mathcal{L}_1 = -\ln(0.5) = \ln(2)$$

$$\mathcal{L}_2 = -\ln(0.125) = -\ln\!\left(\frac{1}{8}\right) = \ln(8)$$

### Step 2: Compute average loss

$$\bar{\mathcal{L}} = \frac{\mathcal{L}_1 + \mathcal{L}_2}{2} = \frac{\ln(2) + \ln(8)}{2}$$

Using log property $\ln(a) + \ln(b) = \ln(ab)$:

$$\bar{\mathcal{L}} = \frac{\ln(2 \times 8)}{2} = \frac{\ln(16)}{2} = \ln(16^{1/2}) = \ln(4)$$

### Step 3: Compute Perplexity

$$\text{PPL} = e^{\bar{\mathcal{L}}} = e^{\ln(4)} = 4$$

```
Computation flow:

  Token 1: P = 0.5   ──▶ Loss₁ = -ln(0.5) = ln(2)
  Token 2: P = 0.125 ──▶ Loss₂ = -ln(0.125) = ln(8)
                              │
                              ▼
  Average Loss = (ln(2) + ln(8)) / 2 = ln(16)/2 = ln(4)
                              │
                              ▼
  Perplexity = e^ln(4) = 4
```

| Step | Formula | Value |
|---|---|---|
| Loss token 1 | $-\ln(0.5)$ | $\ln(2) \approx 0.693$ |
| Loss token 2 | $-\ln(0.125)$ | $\ln(8) \approx 2.079$ |
| **Average Loss** | $\frac{\ln(2) + \ln(8)}{2}$ | $\ln(4) \approx 1.386$ |
| **Perplexity** | $e^{\ln(4)}$ | **4** |

---
---

# Quiz 6, Question 24 — AMP VRAM Footprint (Essay)

> **AMP VRAM Footprint:** You are training a 10-Billion parameter model using Automatic Mixed Precision (AMP). According to standard AMP architecture, you must store the model weights in FP16, the gradients in FP16, and a "master copy" of the weights in FP32.
>
> Calculate the exact combined VRAM footprint in Gigabytes (GB) for these three specific components. (Assume 1 GB = $10^9$ bytes).

---

## Sub-questions

### What is FP16?

$$\text{FP16} = 16 \text{ bits} = 2 \text{ bytes per value}$$

Half-precision floating point. Used for forward/backward passes in AMP for speed and memory savings.

### What is FP32?

$$\text{FP32} = 32 \text{ bits} = 4 \text{ bytes per value}$$

Full-precision floating point. Used for the master copy of weights to maintain numerical accuracy during parameter updates.

### Why does AMP keep a FP32 master copy?

The optimizer update $\theta \leftarrow \theta - \alpha \nabla\mathcal{L}$ involves tiny subtractions. In FP16, $\theta$ and $\alpha \nabla\mathcal{L}$ may differ by less than FP16 precision → the subtraction rounds to zero → weight never changes. The FP32 master copy has enough precision to capture these small updates.

### How to calculate memory per component?

$$\text{Memory} = \text{Parameters} \times \text{Bytes per Parameter}$$

---

## Main answer

### Component 1: FP16 Weights

$$10 \times 10^9 \times 2 \;\text{bytes} = 20 \times 10^9 \;\text{bytes} = 20 \;\text{GB}$$

### Component 2: FP16 Gradients

$$10 \times 10^9 \times 2 \;\text{bytes} = 20 \times 10^9 \;\text{bytes} = 20 \;\text{GB}$$

### Component 3: FP32 Master Weights

$$10 \times 10^9 \times 4 \;\text{bytes} = 40 \times 10^9 \;\text{bytes} = 40 \;\text{GB}$$

### Total VRAM

$$\text{Total} = 20 + 20 + 40 = 80 \;\text{GB}$$

```
AMP memory layout:

  ┌────────────────────────────────────┐
  │           VRAM Breakdown           │
  ├────────────────────────────────────┤
  │  FP16 Weights     │    20 GB      │  10B × 2 bytes
  ├────────────────────────────────────┤
  │  FP16 Gradients   │    20 GB      │  10B × 2 bytes
  ├────────────────────────────────────┤
  │  FP32 Master Copy │    40 GB      │  10B × 4 bytes
  ├────────────────────────────────────┤
  │  TOTAL             │    80 GB      │
  └────────────────────────────────────┘
```

```
AMP data flow:

  FP32 Master (40 GB)
       │
       ▼  cast to FP16
  FP16 Weights (20 GB) ──▶ Forward/Backward ──▶ FP16 Gradients (20 GB)
                                                       │
                                                       ▼  unscale + cast to FP32
                                                  FP32 Master update
                                                  θ ← θ - α·∇L
```

| Component | Precision | Bytes/Param | 10B Model |
|---|---|---|---|
| Weights | FP16 | 2 | 20 GB |
| Gradients | FP16 | 2 | 20 GB |
| Master copy | FP32 | 4 | 40 GB |
| **Total** | — | **8** | **80 GB** |

---
---

# Quiz 6, Question 25 — Gradient Accumulation Loop Logic (Essay)

> **Gradient Accumulation Loop Logic:** You are training with a dataloader that contains exactly 100 batches. You set `ACCUM_STEPS = 8`. In your training loop, you use a zero-indexed enumeration: `for i, (inputs, targets) in enumerate(dataloader):`. Your optimizer step is wrapped in the condition: `if (i + 1) % ACCUM_STEPS == 0:`.
>
> By the time the `for` loop finishes iterating over all 100 batches, exactly how many times has `optimizer.step()` been executed? What happens to the gradients computed from the final 4 batches?

---

## Sub-questions

### What is `enumerate()` with zero-indexing?

$$i \in \{0, 1, 2, \dots, 99\} \quad \text{(100 batches total)}$$

`i` starts at 0, ends at 99. The expression `i + 1` converts to 1-based indexing: $\{1, 2, \dots, 100\}$.

### What does `(i + 1) % ACCUM_STEPS == 0` check?

$$\text{True when } (i+1) \text{ is divisible by } 8$$

Values where condition is True: $i+1 \in \{8, 16, 24, 32, 40, 48, 56, 64, 72, 80, 88, 96\}$

That is $i \in \{7, 15, 23, 31, 39, 47, 55, 63, 71, 79, 87, 95\}$.

### What is integer division with remainder?

$$100 \div 8 = 12 \text{ remainder } 4$$

$$100 = 8 \times 12 + 4$$

12 complete groups of 8, plus 4 leftover batches.

---

## Main answer

### Part A: How many times is `optimizer.step()` executed?

$$\left\lfloor \frac{100}{8} \right\rfloor = 12 \text{ times}$$

The condition `(i + 1) % 8 == 0` is satisfied at $i+1 = 8, 16, 24, \dots, 96$. That is **12** values.

The last trigger is at $i+1 = 96$ (batch index $i = 95$).

### Part B: What happens to the final 4 batches?

Batches $i = 96, 97, 98, 99$ (i.e., $i+1 = 97, 98, 99, 100$) compute gradients via `loss.backward()`, which accumulate into `.grad` attributes. However, the next multiple of 8 would be $i+1 = 104$, which is beyond the loop's range.

**The gradients are computed and accumulated in `.grad`, but `optimizer.step()` is never called for them. Their training signal is lost/wasted for this epoch.**

```
Loop iteration map (100 batches, ACCUM_STEPS=8):

  Batches 0-7:    backward × 8  →  step() #1   ✓
  Batches 8-15:   backward × 8  →  step() #2   ✓
  Batches 16-23:  backward × 8  →  step() #3   ✓
  ...
  Batches 88-95:  backward × 8  →  step() #12  ✓
  Batches 96-99:  backward × 4  →  NO step()   ✗ gradients lost!
                                     │
                                     ▼
                            loop ends at i=99
                            next trigger would be i+1=104
```

```
Timeline:

  i:    0  1  2  3  4  5  6  7  8  ...  95  96  97  98  99
  i+1:  1  2  3  4  5  6  7  8  9  ...  96  97  98  99  100
                                ↑                          ↑
                            step #1                    loop ends
                          (8%8==0)                   (100%8≠0)

  Last step: i+1=96 (step #12)
  Orphaned: i+1 = 97, 98, 99, 100 (4 batches, no step)
```

| Quantity | Value |
|---|---|
| Total batches | 100 |
| ACCUM_STEPS | 8 |
| `optimizer.step()` calls | $\lfloor 100/8 \rfloor = 12$ |
| Remainder batches | $100 \mod 8 = 4$ |
| Fate of remainder | Gradients accumulated but never applied — **wasted** |
