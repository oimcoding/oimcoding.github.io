# Essay Cheatsheet — Quizzes 2, 3, 5, 6, 7, 8

---

# QUIZ 2 — Tokenization & BPE

---

## Q21. Manual BPE Execution

**Given:** Word list with counts, character-level tokens.
**Find:** First $N$ merges and their frequencies.

### Formula

$$\text{pair\_freq}(a, b) = \sum_{\text{words containing } ab} \text{word\_count}$$

### Steps

| # | Do | Example |
|---|---|---|
| 1 | Split all words into characters | "hug" x5 → `h u g` x5 |
| 2 | Count **every** adjacent pair across all words | `(u,g)`: 5+3=**8** |
| 3 | Merge highest-freq pair → new token | `u g → ug` |
| 4 | **Recount** pairs (neighbors changed!) | `(h,ug)`:5, `(u,n)`:6... |
| 5 | Repeat from step 3 | merge `(u,n)` freq=6 |

```
Before merge 1:   h u g (x5)  p u g (x3)  p u n (x2)  b u n (x4)
                   ↑ count all adjacent pairs
Pair counts:  (h,u)=5  (u,g)=8★  (p,u)=5  (u,n)=6  (b,u)=4

After merge 1 (u,g → ug):
                   h ug (x5)  p ug (x3)  p u n (x2)  b u n (x4)
New pairs:    (h,ug)=5  (p,ug)=3  (p,u)=2  (u,n)=6★  (b,u)=4
```

**Trap:** Always recount after each merge — neighbors change.

---

## Q22. Unigram Probability / Viterbi Selection

**Given:** Token probabilities, a word, two segmentations.
**Find:** Which segmentation Viterbi picks.

### Formula

$$\text{Score}(\text{segmentation}) = \prod_i P(\text{token}_i)$$

$$\text{Viterbi picks:} \quad \arg\max \prod_i P(\text{token}_i)$$

### Steps

| # | Do |
|---|---|
| 1 | List each segmentation's tokens |
| 2 | Multiply probabilities for each segmentation |
| 3 | Higher product wins |

```
"damage" → ["dam","age"] vs ["damage"]

  ["dam","age"]:  0.03 × 0.02 = 0.0006  ← WINS
  ["damage"]:     0.0005

  Viterbi picks: ["dam","age"]
```

---

## Q23. BBPE Untrained Tokenizer Output

**Given:** A character/emoji, its UTF-8 bytes, untrained BBPE.
**Find:** Output tokens.

### Formula

$$\text{Base vocab} = \{0\text{x}00 \ldots 0\text{x}FF\} \quad (256 \text{ tokens})$$

$$\text{No merges} \implies \text{each byte} = \text{one token}$$

### Steps

| # | Do |
|---|---|
| 1 | UTF-8 encode the input → list of bytes |
| 2 | Each byte is in base vocab (0–255) |
| 3 | No merges learned → output = individual bytes |

```
🐶 → UTF-8 → [F0, 9F, 90, B6] → 4 tokens

Trained tokenizer:   [F0 9F 90 B6] → 1 token (merged)
Untrained tokenizer: [F0] [9F] [90] [B6] → 4 tokens
```

---

## Q24. Regex Pre-tokenization / Contraction Splitting

**Given:** Text with contractions, GPT-2 regex pattern.
**Find:** How the regex splits, why it's linguistically better.

### Pattern

```
GPT-2 regex priority:  's | 't | 're | 've | 'm  →  then words, numbers, punct
```

### Steps

| # | Do |
|---|---|
| 1 | Match contraction suffixes first (`'m`, `'t`, etc.) |
| 2 | Then match remaining word/number/punct chunks |
| 3 | Explain: suffix = one morpheme (meaningful unit) |

```
"I'm going"

  Good:  ["I", "'m", " going"]     ← "'m" = morpheme for "am"
  Bad:   ["I", "'", "m", " going"] ← "'" and "m" meaningless alone
```

---

## Q25. BPE vs WordPiece Merge Criterion

**Given:** Conceptual comparison.
**Find:** The fundamental difference in optimization goals.

### Formulas

$$\text{BPE:} \quad \text{score}(a,b) = \text{count}(ab)$$

$$\text{WordPiece:} \quad \text{score}(a,b) = \frac{\text{count}(ab)}{\text{count}(a) \times \text{count}(b)}$$

$\text{count}(ab)$ = how many times $a$ and $b$ appear next to each other
$\text{count}(a)$ = how many times $a$ appears total (solo)

```
           BPE asks:         "Most FREQUENT pair?"     → raw count
           WordPiece asks:   "Most SURPRISING pair?"   → PMI

  ("t","h"):  count=10000,  count(t)=50000, count(h)=40000
  ("x","q"):  count=100,    count(x)=120,   count(q)=110

  BPE:       10000 > 100           → picks (t,h)
  WordPiece: 10000/(50k×40k)=5e-6
             100/(120×110)=0.0076  → picks (x,q)  ← more surprising
```

---
---

# QUIZ 3 — Embeddings & Positional Encoding

---

## Q21. Parameter Efficiency / Weight Tying

**Given:** $V$ (vocab size), $d$ (embedding dim).
**Find:** Embedding params, savings from weight tying.

### Formulas

$$W_E \in \mathbb{R}^{V \times d} \quad \Rightarrow \quad |W_E| = V \times d$$

$W_E$ = input embedding matrix
$V$ = vocabulary size (number of tokens)
$d$ = $d_\text{model}$ (embedding dimension)

$$W_U \in \mathbb{R}^{d \times V} \quad \text{(output unembedding, same size transposed)}$$

$$\text{Weight tying:} \quad W_U = W_E^T \quad \Rightarrow \quad \text{saves } V \times d \text{ params}$$

### Steps

| # | Do |
|---|---|
| 1 | Params in $W_E$ = $V \times d$ |
| 2 | Without tying: total = $2 \times V \times d$ (embed + unembed) |
| 3 | With tying: total = $V \times d$, saved = $V \times d$ |

```
V=50000, d=768:

  W_E = 50000 × 768 = 38.4M params
  W_U = 50000 × 768 = 38.4M params (separate)

  Without tying:  76.8M
  With tying:     38.4M  → saved 38.4M (50%)
```

---

## Q22. Hypercube Center-to-Corner Distance

**Given:** Dimension $d$, unit hypercube $[-0.5, 0.5]^d$.
**Find:** Distance center→corner, ratio vs center→face.

### Formulas

$$d_\text{face} = 0.5 \quad \text{(always, any dimension)}$$

$$d_\text{corner} = \frac{\sqrt{d}}{2}$$

$d$ = number of dimensions

$$\text{Ratio} = \frac{d_\text{corner}}{d_\text{face}} = \sqrt{d}$$

### Steps

| # | Do |
|---|---|
| 1 | $d_\text{corner} = \sqrt{d} \;/\; 2$ |
| 2 | $d_\text{face} = 0.5$ |
| 3 | Ratio $= \sqrt{d}$ |

```
d = 100:

  Center → Face:   0.5
  Center → Corner: √100 / 2 = 5.0
  Ratio: 5.0 / 0.5 = 10×
```

---

## Q23. Sliding Window / Stride Sample Count

**Given:** $T$ (total tokens), $L$ (context length), $S$ (stride).
**Find:** Number of samples, efficiency ratio.

### Formula

$$N = \left\lfloor \frac{T - L}{S} \right\rfloor + 1$$

$T$ = total tokens in corpus
$L$ = window/context length
$S$ = stride (step between windows)

### Steps

| # | Do |
|---|---|
| 1 | Compute $N$ for each stride value |
| 2 | Ratio = $N_{S=1} \;/\; N_{S=L}$ |
| 3 | Note: same unique tokens either way → high overlap = redundancy |

```
T=50000, L=512:

  S=512 (no overlap):  N = ⌊(50000-512)/512⌋ + 1 = 96+1 = 97
  S=1   (max overlap): N = ⌊(50000-512)/1⌋ + 1 = 49489

  Ratio: 49489/97 ≈ 510×
  But: 510× more samples ≠ 510× better (99.8% redundancy per sample)
```

---

## Q24. Sinusoidal PE Wavelength / Denominator

**Given:** $d_\text{model}$, dimension index $i$.
**Find:** Denominator value, frequency interpretation.

### Formulas

$$\text{denominator}(i) = 10000^{2i/d_\text{model}}$$

$$\text{wavelength}(i) = 2\pi \times 10000^{2i/d_\text{model}}$$

$i$ = dimension index (0 to $d/2 - 1$)
$d_\text{model}$ = model dimension

### Steps

| # | Do |
|---|---|
| 1 | Exponent $= 2i / d_\text{model}$ |
| 2 | Denominator $= 10000^{\text{exponent}}$ |
| 3 | Compare to $i=0$: denom$(0)=1$ → ratio = denom$(i)$ times slower |

```
d_model=512, i=256:

  exponent = 2×256/512 = 1
  denominator = 10000^1 = 10000
  wavelength = 2π × 10000 ≈ 62,832 positions

  vs i=0: denominator=1, wavelength=2π ≈ 6.28
  → 10,000× slower
```

---

## Q25. Rotation Matrix Composition Proof

**Given:** $M_\theta$ (2x2 rotation matrix), prove $M_\theta^2 = M_{2\theta}$.
**Find:** Multiply $M_\theta \times M_\theta$, apply double-angle identities.

### Key Identities

$$\cos(2\theta) = \cos^2\theta - \sin^2\theta$$
$$\sin(2\theta) = 2\sin\theta\cos\theta$$

### Steps

| # | Do |
|---|---|
| 1 | Write $M_\theta = \begin{pmatrix} c & s \\ -s & c \end{pmatrix}$ where $c=\cos\theta, s=\sin\theta$ |
| 2 | Multiply: top-left $= c^2 - s^2$, top-right $= 2sc$ |
| 3 | Bottom-left $= -2sc$, bottom-right $= c^2 - s^2$ |
| 4 | Substitute double-angle: $c^2-s^2 = \cos2\theta$, $2sc = \sin2\theta$ |
| 5 | Result = $M_{2\theta}$ $\blacksquare$ |

```
M_θ × M_θ:

  ┌ cos²θ-sin²θ    2sinθcosθ  ┐     ┌ cos(2θ)    sin(2θ)  ┐
  │                            │  =  │                      │ = M_{2θ}  ✓
  └ -2sinθcosθ   cos²θ-sin²θ  ┘     └ -sin(2θ)   cos(2θ)  ┘

  Generalization: M_θ^n = M_{nθ}
```

---
---

# QUIZ 5 — Model Architecture & Scaling

---

## Q21. KV Cache Memory per Token

**Given:** $d_\text{model}$, $H_\text{kv}$ (KV heads), $L$ (layers), $b$ (bytes per element).
**Find:** KV cache memory for one token.

### Formula

$$\boxed{\text{KV Cache per token} = 2 \times H_\text{kv} \times d_k \times L \times b}$$

$2$ = one K vector + one V vector
$H_\text{kv}$ = number of Key/Value heads (GQA: fewer than query heads)
$d_k = d_\text{model} / H_\text{query}$ = per-head dimension
$L$ = number of transformer layers
$b$ = bytes per element (bfloat16 = 2, float32 = 4)

### Steps

| # | Do |
|---|---|
| 1 | $d_k = d_\text{model} / H_\text{query}$ |
| 2 | Elements per layer $= 2 \times H_\text{kv} \times d_k$ |
| 3 | Total elements $= \text{step 2} \times L$ |
| 4 | Bytes $= \text{step 3} \times b$ |

```
d_model=4096, H_query=32, H_kv=8, L=32, b=2 (bf16):

  d_k = 4096/32 = 128
  per layer = 2 × 8 × 128 = 2048 elements
  all layers = 2048 × 32 = 65536 elements
  bytes = 65536 × 2 = 131,072 bytes = 128 KB
```

---

## Q22. Parallel Decoder Block Code

**Given:** `self.input_norm`, `self.attention`, `self.feed_forward`.
**Find:** `forward()` method for parallel (not sequential) block.

### Formula

$$\text{out} = x + \text{Attn}(\text{Norm}(x)) + \text{FFN}(\text{Norm}(x))$$

### Steps + Code

```python
def forward(self, x, freqs_cis, mask):
    x_norm = self.input_norm(x)                     # 1 shared norm
    attn_out = self.attention(x_norm, freqs_cis, mask)  # branch A
    ffn_out  = self.feed_forward(x_norm)                # branch B (parallel)
    return x + attn_out + ffn_out                       # 1 residual add
```

```
         x
    ┌────┴────┐
    │    RMSNorm
    │    ┌──┴──┐
    │  Attn   FFN      ← parallel (no dependency)
    │    └──┬──┘
    └───→ (+) (+)      ← single combined residual
         out

  Sequential: 2 norms, 2 residuals, FFN waits for Attn
  Parallel:   1 norm,  1 residual, both run simultaneously
```

---

## Q23. SwiGLU Parameter Scaling

*Same as Quiz 4 Q23 — see Quiz 4 cheatsheet or use this:*

### Formula

$$P_\text{ReLU} = 2 \times d \times d_\text{ffn} = 8d^2 \quad \text{(when } d_\text{ffn}=4d\text{)}$$

$$P_\text{SwiGLU} = 3 \times d \times d_\text{hidden}$$

$d = d_\text{model}$, $r$ = target ratio (e.g. 1.10 for 10% more)

$$\boxed{d_\text{hidden} = \frac{8r}{3} \times d}$$

| # | Do |
|---|---|
| 1 | $P_\text{ReLU} = 8d^2$ |
| 2 | $P_\text{target} = r \times 8d^2$ |
| 3 | Set $3d \cdot d_\text{hidden} = P_\text{target}$ |
| 4 | Solve: $d_\text{hidden} = 8r/(3) \times d$ |

---

## Q24. RoPE Dot Product Derivation

**Given:** $q=[q_1,q_2]$ at position $m$, $k=[k_1,k_2]$ at position $n$, angle $\theta$.
**Find:** Prove dot product depends only on $m-n$.

### Key Identity

$$\cos A\cos B + \sin A\sin B = \cos(A - B)$$

### Steps

| # | Do |
|---|---|
| 1 | Apply $R_{m\theta}$ to $q$ → $[q_1\cos m\theta - q_2\sin m\theta,\; q_1\sin m\theta + q_2\cos m\theta]$ |
| 2 | Apply $R_{n\theta}$ to $k$ → same form with $n$ |
| 3 | Dot product → expand 4 terms |
| 4 | Group by $q_ik_j$ coefficients |
| 5 | Apply trig identity → every term becomes $f(m-n)$ |

### Result

$$\boxed{(R_m q)^T(R_n k) = (q_1k_1+q_2k_2)\cos((m{-}n)\theta) + (q_1k_2-q_2k_1)\sin((m{-}n)\theta)}$$

Only $m-n$ appears → **relative position only**. $\blacksquare$

---

## Q25. Causal Mask: $-\infty$ vs $\times 0$

*Same concept as Quiz 4 Q25 — use this generalized recipe:*

### Step 1: Write Softmax

$$P(z_i) = \frac{e^{z_i}}{\sum_j e^{z_j}}$$

### Step 2: Trace both methods

| Method | Pre-softmax | Exponentiate | Softmax weight | Leak? |
|--------|-------------|-------------|----------------|-------|
| Add $-\infty$ | $-\infty$ | $e^{-\infty}=0$ | $0/\sum = 0$ | **No** |
| Multiply $\times 0$ | $0$ | $e^0 = 1$ | $1/\sum > 0$ | **Yes** |

### Step 3: State consequence

```
×0 → e⁰=1 → P>0 → model sees future → "cheats" during training
                                       → breaks at inference (no future exists)
```

---
---

# QUIZ 6 — PyTorch Training

---

## Q21. Autoregressive Shift & Reshape

**Given:** Logits shape $(B, S, V)$.
**Find:** Shape after shift + flatten, total values.

### Formulas

$$\text{shift\_logits} = \text{logits}[:, :-1, :] \quad \Rightarrow \quad (B,\; S{-}1,\; V)$$

$B$ = batch size, $S$ = sequence length, $V$ = vocab size

$$\text{flatten} \Rightarrow (B \times (S{-}1),\; V)$$

$$\text{Total values} = B \times (S{-}1) \times V$$

### Steps

| # | Do |
|---|---|
| 1 | Drop last position: $S \to S-1$ |
| 2 | Flatten batch+seq: $B \times (S-1)$ |
| 3 | Total = flattened_dim $\times V$ |

```
(8, 1024, 32000)
      │ shift [:, :-1, :]
(8, 1023, 32000)
      │ flatten dims 0,1
(8184, 32000)
      │ total elements
8184 × 32000 = 261,888,000
```

---

## Q22. LogSumExp Trick + Cross-Entropy

**Given:** Logits $x = [z_1, z_2, \ldots]$, target index.
**Find:** Shifting constant $c$, softmax, CE loss.

### Formulas

$$c = \max(x)$$

$$\text{Softmax}(x_i) = \frac{e^{x_i - c}}{\sum_j e^{x_j - c}}$$

$$\mathcal{L} = -\ln P(\text{target})$$

$c$ = LogSumExp shifting constant (prevents overflow)

### Steps

| # | Do |
|---|---|
| 1 | $c = \max(\text{logits})$ |
| 2 | Subtract $c$ from all logits |
| 3 | Compute softmax of shifted logits |
| 4 | CE loss $= -\ln(\text{softmax of target index})$ |

```
x = [1000, 1000, 1000], target = index 1

  c = 1000
  shifted = [0, 0, 0]
  softmax = [1/3, 1/3, 1/3]
  loss = -ln(1/3) = ln(3)
```

---

## Q23. Perplexity from Token Probabilities

**Given:** Probabilities for each correct token.
**Find:** Average loss, perplexity.

### Formulas

$$\mathcal{L}_i = -\ln P(x_i)$$

$P(x_i)$ = probability model assigned to the $i$-th correct token

$$\bar{\mathcal{L}} = \frac{1}{N}\sum_{i=1}^N \mathcal{L}_i$$

$$\text{PPL} = e^{\bar{\mathcal{L}}}$$

$N$ = number of tokens, PPL = perplexity

### Steps

| # | Do |
|---|---|
| 1 | $\mathcal{L}_i = -\ln(P_i)$ for each token |
| 2 | Average: $\bar{\mathcal{L}} = \text{sum} / N$ |
| 3 | PPL $= e^{\bar{\mathcal{L}}}$ |

```
P₁=0.5, P₂=0.125:

  L₁ = -ln(0.5) = ln(2)
  L₂ = -ln(0.125) = ln(8)
  avg = (ln2 + ln8)/2 = ln(16)/2 = ln(4)
  PPL = e^ln(4) = 4
```

**Shortcut:** $\text{PPL} = e^{\bar{\mathcal{L}}} = \left(\prod_i \frac{1}{P_i}\right)^{1/N}$ (geometric mean of inverse probabilities)

---

## Q24. AMP VRAM Footprint

**Given:** $P$ (parameter count), AMP components.
**Find:** Total VRAM in GB.

### Formula

$$\text{Total} = P \times (b_\text{fp16\_wts} + b_\text{fp16\_grad} + b_\text{fp32\_master})$$

$$= P \times (2 + 2 + 4) = P \times 8 \text{ bytes}$$

$P$ = number of parameters
$b$ = bytes per element per component (FP16=2, FP32=4)

### Steps

| # | Do | Bytes/param |
|---|---|---|
| 1 | FP16 weights | $P \times 2$ |
| 2 | FP16 gradients | $P \times 2$ |
| 3 | FP32 master copy | $P \times 4$ |
| 4 | Sum → total bytes | $P \times 8$ |
| 5 | Convert: GB = bytes $/ 10^9$ | |

```
P = 10B:

  FP16 weights:    10B × 2 = 20 GB
  FP16 gradients:  10B × 2 = 20 GB
  FP32 master:     10B × 4 = 40 GB
  ─────────────────────────────────
  Total:                     80 GB
```

---

## Q25. Gradient Accumulation Step Count

**Given:** Total batches, `ACCUM_STEPS`, zero-indexed loop.
**Find:** How many `optimizer.step()` calls, fate of remainder.

### Formula

$$\text{steps} = \left\lfloor \frac{\text{total\_batches}}{\text{ACCUM\_STEPS}} \right\rfloor$$

$$\text{remainder} = \text{total\_batches} \mod \text{ACCUM\_STEPS}$$

### Steps

| # | Do |
|---|---|
| 1 | Integer divide: batches / ACCUM_STEPS |
| 2 | Remainder = batches mod ACCUM_STEPS |
| 3 | Remainder batches: gradients computed but **never applied** (wasted) |

```
100 batches, ACCUM_STEPS=8:

  optimizer.step() calls: ⌊100/8⌋ = 12
  remainder: 100 mod 8 = 4 batches
  → last 4 batches' gradients are accumulated but never stepped → LOST

  Triggers at i+1 = 8, 16, 24, ..., 96  (12 times)
  Batches 97-100: no trigger → orphaned gradients
```

---
---

# QUIZ 7 — LoRA, Decoding, Quantization

---

## Q21. Beam Search vs Greedy Decoding

**Given:** Initial word probs, conditional probs, beam width $B$.
**Find:** Cumulative probs, surviving beams, greedy path.

### Formula

$$P(\text{path}) = P(w_1) \times P(w_2 \mid w_1) \times \ldots$$

### Steps

| # | Do |
|---|---|
| 1 | For each first word kept in beam, multiply by each next-word prob |
| 2 | Sort all expanded paths by cumulative probability |
| 3 | Keep top $B$ paths |
| 4 | Greedy = pick argmax at each step independently |
| 5 | Compare: did greedy find the highest cumulative path? |

```
B=2, first words: "The"(0.6), "A"(0.3), "An"(0.1)

  "The"→"cat": 0.6×0.4 = 0.24
  "The"→"dog": 0.6×0.3 = 0.18
  "A"→"tiger": 0.3×0.9 = 0.27 ★ highest
  "A"→"lion":  0.3×0.05= 0.015

  Beam keeps: "A tiger"(0.27), "The cat"(0.24)
  Greedy:     "The"→"cat" = 0.24 (missed the best path!)
```

---

## Q22. Temperature Scaling

**Given:** Two logits $z_a, z_b$; different temperatures $T$.
**Find:** Probability ratio $P_a/P_b$ at each $T$.

### Formula

$$\frac{P_a}{P_b} = e^{(z_a - z_b)/T}$$

$z_a, z_b$ = raw logits
$T$ = temperature (low → sharp, high → flat)

### Steps

| # | Do |
|---|---|
| 1 | Logit diff $= z_a - z_b$ |
| 2 | Divide by $T$ |
| 3 | Exponentiate: $e^{\text{result}}$ |

```
z_a=4.0, z_b=2.0, diff=2.0:

  T=1.0: e^(2/1) = e² ≈ 7.39    (A is 7× more likely)
  T=0.5: e^(2/0.5) = e⁴ ≈ 54.6  (A is 55× more likely!)

  T↓ → ratio↑ → more confident/deterministic
  T↑ → ratio→1 → more uniform/random
```

---

## Q23. LoRA Parameter Reduction

**Given:** Weight matrix $d_\text{out} \times d_\text{in}$, LoRA rank $r$.
**Find:** Full FT params, LoRA params, % reduction.

### Formulas

$$P_\text{full} = d_\text{out} \times d_\text{in}$$

$$P_\text{LoRA} = d_\text{out} \times r + r \times d_\text{in}$$

$r$ = LoRA rank (small, e.g. 4, 8, 16)
$A \in \mathbb{R}^{d_\text{out} \times r}$, $B \in \mathbb{R}^{r \times d_\text{in}}$

$$\text{Reduction} = \left(1 - \frac{P_\text{LoRA}}{P_\text{full}}\right) \times 100\%$$

### Steps

| # | Do |
|---|---|
| 1 | Full = $d_\text{out} \times d_\text{in}$ |
| 2 | LoRA = $r(d_\text{out} + d_\text{in})$ |
| 3 | Reduction = $(1 - \text{LoRA}/\text{Full}) \times 100\%$ |

```
W: 1024×1024, r=4:

  Full:  1024 × 1024 = 1,048,576
  LoRA:  1024×4 + 4×1024 = 8,192
  Reduction: (1 - 8192/1048576) × 100% = 99.22%
```

---

## Q24. LoRA Approximation Error (SVD / Eckart-Young)

**Given:** Singular values $\sigma_1, \sigma_2, \ldots, \sigma_n$; rank $r$.
**Find:** Squared error $\|A - A_r\|_F^2$, energy retained.

### Formula

$$\boxed{\text{Error}^2 = \sum_{i=r+1}^{n} \sigma_i^2} \quad \text{(sum of DISCARDED singular values squared)}$$

$$\text{Total energy} = \sum_{i=1}^n \sigma_i^2$$

$$\text{Energy retained} = \frac{\sum_{i=1}^r \sigma_i^2}{\text{Total energy}}$$

$\sigma_i$ = $i$-th singular value (sorted largest first)
$r$ = approximation rank (LoRA rank)

### Steps

| # | Do |
|---|---|
| 1 | Keep first $r$ singular values |
| 2 | Error$^2$ = sum of squares of the **rest** |
| 3 | Energy retained = (kept squares) / (total squares) |

```
σ = [10, 8, 2, 1],  r=2:

  Keep: 10, 8     Discard: 2, 1
  Error² = 2² + 1² = 5
  Total energy = 100+64+4+1 = 169
  Retained = (100+64)/169 = 97.0%  → "Strong" intrinsic dimension
```

---

## Q25. Double Quantization Overhead

**Given:** Block size, bits for scales, second-level block size.
**Find:** Bits-per-parameter overhead at each level.

### Formula

$$\text{overhead (bits/param)} = \frac{\text{bits for scaling factor}}{\text{weights per block}}$$

### Steps

| # | Do |
|---|---|
| 1 | **Single Q:** overhead = scale\_bits / block\_size |
| 2 | **Double Q Level 1:** requantize scales to fewer bits → new overhead = new\_bits / block\_size |
| 3 | **Double Q Level 2:** second-level scale covers (block\_size × L2\_block\_size) weights |
| 4 | Level 2 overhead = L2\_scale\_bits / (block\_size × L2\_block\_size) |
| 5 | Total = Level 1 + Level 2 |

```
Block=128, FP16 scales (16-bit), L2 block=256, L2 scale=FP32:

  Single Q:  16/128 = 0.125 bits/param

  Double Q:
    Level 1: 4/128 = 0.03125 bits/param     (scales quantized to 4-bit)
    Level 2: 32/(128×256) = 32/32768 ≈ 0.001 bits/param
    Total: 0.03125 + 0.001 ≈ 0.032 bits/param

  Savings: 0.125 → 0.032 = 74% overhead reduction
```

---
---

# QUIZ 8 — RAG & Vector Databases

---

## Q21. Cross-Encoder Re-Ranking Code

**Given:** `user_query`, `initial_docs` list, cross-encoder model.
**Find:** Fill in 3 blanks.

### Pattern (always the same)

```python
# 1. Build [query, doc] pairs
inputs = [[user_query, doc.page_content] for doc in docs]

# 2. Score all pairs
logits = cross_encoder.predict(inputs)

# 3. Sort descending
sorted_indices = np.argsort(logits)[::-1]
```

```
Step 1: List comprehension → [[query, doc_text], ...]
Step 2: .predict() → array of scores
Step 3: argsort()[::-1] → indices highest-first
```

---

## Q22. Brute-Force k-NN Scaling

**Given:** FLOPS at $N_1$ documents, new count $N_2$.
**Find:** FLOPS at $N_2$.

### Formula

$$\text{FLOPS} \propto N \quad \text{(linear)}$$

$$\text{FLOPS}_2 = \text{FLOPS}_1 \times \frac{N_2}{N_1}$$

$N$ = number of database vectors
$d$ = embedding dimension (held constant)

### Steps

| # | Do |
|---|---|
| 1 | Ratio $= N_2 / N_1$ |
| 2 | New FLOPS $= \text{old FLOPS} \times \text{ratio}$ |

```
N₁=1B → 1.5T FLOPS
N₂=2B → 2/1 × 1.5T = 3.0T FLOPS
```

---

## Q23. Two-Stage Retrieval Latency

**Given:** Bi-encoder time, cross-encoder time per pair, candidate count $k$.
**Find:** Total pipeline time, naive cross-encoder time.

### Formulas

$$t_\text{two-stage} = t_\text{bi-enc} + k \times t_\text{cross-enc}$$

$t_\text{bi-enc}$ = bi-encoder search time (fast, ANN)
$k$ = number of candidates retrieved
$t_\text{cross-enc}$ = time per query-doc pair (slow, cross-attention)

$$t_\text{naive} = N \times t_\text{cross-enc}$$

$N$ = total database size

### Steps

| # | Do |
|---|---|
| 1 | Two-stage = bi-enc + ($k$ × cross-enc per pair) |
| 2 | Naive = $N$ × cross-enc per pair |
| 3 | Convert units (ms → s → hours) |

```
bi-enc=10ms, cross-enc=25ms/pair, k=50, N=1M:

  Two-stage: 10 + 50×25 = 1,260 ms ≈ 1.26 sec
  Naive: 1M × 25 = 25,000,000 ms ≈ 6.94 hours
  Speedup: ~19,841×
```

---

## Q24. Product Quantization (PQ) Compression

**Given:** Vector dim $D$, float type, number of subvectors $M$, centroids $K$.
**Find:** Compressed size, compression ratio.

### Formulas

$$\text{Original size} = D \times b_\text{float}$$

$D$ = vector dimensions, $b_\text{float}$ = bytes per float (float32=4)

$$\text{Bits per subvector ID} = \lceil \log_2 K \rceil$$

$M$ = number of subvectors, $K$ = centroids per codebook

$$\text{Compressed size} = M \times \lceil \log_2 K \rceil / 8 \text{ bytes}$$

$$\text{Ratio} = \frac{\text{Original}}{\text{Compressed}}$$

### Steps

| # | Do |
|---|---|
| 1 | Original = $D \times$ bytes\_per\_float |
| 2 | Bits per ID = $\log_2 K$ |
| 3 | Compressed = $M \times$ (bits per ID / 8) bytes |
| 4 | Ratio = Original / Compressed |

```
D=1536, float32, M=96 subvectors, K=256 centroids:

  Original: 1536 × 4 = 6,144 bytes
  Bits/ID: log₂(256) = 8 bits = 1 byte
  Compressed: 96 × 1 = 96 bytes
  Ratio: 6144/96 = 64×
```

---

## Q25. Chunking with Overlap — Chunk Count

**Given:** $T$ (doc tokens), chunk size $C$, overlap $O$.
**Find:** Total number of chunks.

### Formulas

$$\text{stride} = C - O$$

$C$ = chunk\_size, $O$ = chunk\_overlap

$$\text{chunks} = 1 + \left\lceil \frac{T - C}{\text{stride}} \right\rceil$$

### Steps

| # | Do |
|---|---|
| 1 | stride $= C - O$ |
| 2 | After first chunk: remaining $= T - C$ |
| 3 | Additional chunks $= \lceil \text{remaining} / \text{stride} \rceil$ |
| 4 | Total $= 1 + \text{additional}$ |

```
T=3200, C=500, O=50:

  stride = 500 - 50 = 450
  remaining = 3200 - 500 = 2700
  additional = ⌈2700/450⌉ = 6
  total = 1 + 6 = 7 chunks

  Verify: last chunk starts at 6×450=2700, ends at 2700+500=3200 ✓
```

```
Chunk layout:
  [0─────500]
       [450─────950]
            [900────1400]
                 [1350───1850]
                      [1800───2300]
                           [2250───2750]
                                [2700───3200]
  7 chunks, each 500 tokens, 50 overlap
```

---

# Quick-Reference Formula Wall

| Quiz | # | Topic | Core Formula |
|------|---|-------|-------------|
| 2 | 21 | BPE | pair\_freq = $\sum$ word\_counts containing pair |
| 2 | 22 | Unigram/Viterbi | $\arg\max \prod P(\text{token}_i)$ |
| 2 | 23 | BBPE untrained | 256 base bytes → each byte = 1 token |
| 2 | 24 | Regex contraction | `'m`, `'t`, `'re` matched as single morphemes |
| 2 | 25 | BPE vs WordPiece | BPE: count$(ab)$ vs WP: count$(ab)$/[count$(a)$×count$(b)$] |
| 3 | 21 | Weight tying | Saved = $V \times d$ |
| 3 | 22 | Hypercube dist | $d_\text{corner} = \sqrt{d}/2$, ratio $= \sqrt{d}$ |
| 3 | 23 | Sliding window | $N = \lfloor(T-L)/S\rfloor + 1$ |
| 3 | 24 | PE wavelength | $10000^{2i/d}$ |
| 3 | 25 | Rotation proof | $M_\theta^2 = M_{2\theta}$ via double-angle |
| 5 | 21 | KV cache | $2 \times H_\text{kv} \times d_k \times L \times b$ |
| 5 | 22 | Parallel block | `out = x + attn(norm(x)) + ffn(norm(x))` |
| 5 | 23 | SwiGLU sizing | $d_h = (8r/3) \times d$ |
| 5 | 24 | RoPE proof | dot product $= f(m-n)$ via $\cos(A-B)$ identity |
| 5 | 25 | Causal mask | $e^{-\infty}=0$ (correct) vs $e^0=1$ (leaks) |
| 6 | 21 | Shift+flatten | $(B(S{-}1), V)$ total $= B(S{-}1)V$ |
| 6 | 22 | LogSumExp | $c=\max(x)$, shift, softmax, $-\ln P$ |
| 6 | 23 | Perplexity | PPL $= e^{\text{avg}(-\ln P_i)}$ |
| 6 | 24 | AMP VRAM | $P \times 8$ bytes (2+2+4) |
| 6 | 25 | Grad accum | steps $= \lfloor B/A \rfloor$, remainder lost |
| 7 | 21 | Beam search | cumulative $= \prod P$, keep top-$B$ |
| 7 | 22 | Temperature | ratio $= e^{(z_a-z_b)/T}$ |
| 7 | 23 | LoRA params | $r(d_\text{out}+d_\text{in})$ |
| 7 | 24 | SVD error | $\sum_{i>r} \sigma_i^2$ |
| 7 | 25 | Double quant | bits/param $= \text{scale\_bits}/\text{block\_size}$ per level |
| 8 | 21 | Cross-encoder | `[[q, doc] for doc in docs]` → `.predict()` → `argsort[::-1]` |
| 8 | 22 | k-NN scaling | FLOPS $\propto N$ (linear) |
| 8 | 23 | Two-stage latency | $t_\text{bi} + k \times t_\text{cross}$ |
| 8 | 24 | PQ compression | $M \times \lceil\log_2 K\rceil/8$ bytes |
| 8 | 25 | Chunking | $1 + \lceil(T-C)/(C-O)\rceil$ |
