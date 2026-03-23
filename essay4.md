# Quiz 4 — Essay Questions Cheatsheet

> **Purpose:** Generalized step-by-step recipes. Plug in any numbers, any format — same method works.

---

## 1. Scaled Dot-Product Attention Score

**What you're given:** Two vectors $Q$ and $K$, each of dimension $d_k$.
**What you find:** The pre-softmax attention score (a single scalar).

**Formula:**

$$\text{score} = \frac{Q \cdot K^T}{\sqrt{d_k}}$$

**Steps:**

| Step | Action | General Form |
|------|--------|-------------|
| 1 | Multiply element-wise | $Q_i \times K_i$ for each $i$ |
| 2 | Sum all products | $\text{dot} = \sum_{i=1}^{d_k} Q_i K_i$ |
| 3 | Compute the scaling factor | $\sqrt{d_k}$ |
| 4 | Divide | $\text{score} = \text{dot} \;/\; \sqrt{d_k}$ |

**Quick-check:** The score should be a moderate number (roughly $-3$ to $+5$ for typical inputs). If your answer is huge, you probably forgot to divide.

---

## 2. RoPE Matrix Rotation

**What you're given:** A 2D vector $x = [x_1, x_2]$, a position $m$, and a base angle $\theta$.
**What you find:** The rotated vector $[x'_1, x'_2]$.

**Formula:**

$$\begin{pmatrix} x'_1 \\ x'_2 \end{pmatrix} = \begin{pmatrix} \cos(m\theta) & -\sin(m\theta) \\ \sin(m\theta) & \cos(m\theta) \end{pmatrix} \begin{pmatrix} x_1 \\ x_2 \end{pmatrix}$$

**Steps:**

| Step | Action | General Form |
|------|--------|-------------|
| 1 | Compute the total angle | $\phi = m \times \theta$ |
| 2 | Look up / compute trig values | $\cos\phi$, $\sin\phi$ |
| 3 | Compute $x'_1$ | $x'_1 = x_1 \cos\phi - x_2 \sin\phi$ |
| 4 | Compute $x'_2$ | $x'_2 = x_1 \sin\phi + x_2 \cos\phi$ |

**Quick-check:** Magnitude is preserved. Verify $\sqrt{x_1^2 + x_2^2} = \sqrt{x'^2_1 + x'^2_2}$. If they differ, you made an arithmetic error.

**Common trig values to memorize:**

| Angle | $\cos$ | $\sin$ |
|-------|--------|--------|
| $30°$ ($\pi/6$) | $\sqrt{3}/2$ | $1/2$ |
| $45°$ ($\pi/4$) | $\sqrt{2}/2$ | $\sqrt{2}/2$ |
| $60°$ ($\pi/3$) | $1/2$ | $\sqrt{3}/2$ |
| $90°$ ($\pi/2$) | $0$ | $1$ |

---

## 3. SwiGLU Parameter Scaling

**What you're given:** $d_\text{model}$, a target parameter ratio $r$ (e.g., 1.10 for "10% more").
**What you find:** The SwiGLU hidden dimension $d_\text{hidden}$.

**Formulas — count parameters first:**

$$\text{ReLU FFN params} = 2 \times d_\text{model} \times d_\text{ffn}$$

(Standard: $d_\text{ffn} = 4 \, d_\text{model}$, so params $= 8 \, d_\text{model}^2$)

$$\text{SwiGLU params} = 3 \times d_\text{model} \times d_\text{hidden}$$

(Three matrices: Gate, Up, Down)

**Steps:**

| Step | Action | General Form |
|------|--------|-------------|
| 1 | Count ReLU FFN params | $P_\text{ReLU} = 2 \cdot d_\text{model} \cdot d_\text{ffn}$ |
| 2 | Apply the target ratio | $P_\text{target} = r \times P_\text{ReLU}$ |
| 3 | Set SwiGLU params equal to target | $3 \cdot d_\text{model} \cdot d_\text{hidden} = P_\text{target}$ |
| 4 | Solve for $d_\text{hidden}$ | $d_\text{hidden} = \dfrac{P_\text{target}}{3 \cdot d_\text{model}}$ |

**General closed-form (when $d_\text{ffn} = 4d_\text{model}$):**

$$\boxed{d_\text{hidden} = \frac{8r}{3}\;d_\text{model}}$$

**Quick-check:** For $r = 1$ (equal params), $d_\text{hidden} \approx 2.67 \, d_\text{model}$, which is *less* than $4 \, d_\text{model}$ because 3 matrices share the budget that 2 matrices had.

---

## 4. Sinusoidal Positional Encoding Wavelength

**What you're given:** $d_\text{model}$ and a dimension index $i$.
**What you find:** The denominator (frequency term) and what it means.

**Formula:**

$$\text{denominator}(i) = 10000^{\,2i\,/\,d_\text{model}}$$

$$\text{wavelength}(i) = 2\pi \times 10000^{\,2i\,/\,d_\text{model}}$$

**Steps:**

| Step | Action | General Form |
|------|--------|-------------|
| 1 | Compute the exponent | $e = 2i \;/\; d_\text{model}$ |
| 2 | Compute the denominator | $10000^e$ |
| 3 | (If asked) Compute wavelength | $\lambda = 2\pi \times 10000^e$ |
| 4 | Interpret | Higher $i$ $\Rightarrow$ larger denominator $\Rightarrow$ slower sine wave $\Rightarrow$ coarser position info |

**Interpretation template:**

| Compare | $i = 0$ | $i = i_\text{given}$ |
|---------|---------|----------------------|
| Exponent | $0$ | $2i/d_\text{model}$ |
| Denominator | $1$ | $10000^{2i/d_\text{model}}$ |
| Wavelength | $2\pi$ (fastest) | $2\pi \times 10000^{2i/d_\text{model}}$ |
| Speed | Highest frequency | $10000^{2i/d_\text{model}} \times$ slower |

**Quick-check:** At $i = 0$ the denominator must be $1$. At $i = d_\text{model}/2$ the denominator must be $10000$. Use these as sanity checks.

---

## 5. Causal Mask — Why $-\infty$ Not $\times\,0$

**What you're given:** The softmax function and two masking strategies.
**What you explain:** Why additive $-\infty$ masking is correct and multiplicative $0$ masking leaks information.

**Step 1 — Write the Softmax definition:**

$$P(z_i) = \frac{e^{z_i}}{\displaystyle\sum_j e^{z_j}}$$

**Step 2 — Trace each masking method through softmax:**

| Step | Add $-\infty$ | Multiply by $0$ |
|------|---------------|-----------------|
| Mask the score | $z_\text{future} \leftarrow z + (-\infty) = -\infty$ | $z_\text{future} \leftarrow z \times 0 = 0$ |
| Exponentiate | $e^{-\infty} = 0$ | $e^{0} = 1$ |
| Softmax weight | $\dfrac{0}{\sum} = 0$ (blocked) | $\dfrac{1}{\sum} > 0$ (leaks!) |

**Step 3 — State the consequence of leakage:**

- **Training:** The model "cheats" by reading future tokens, learning a shortcut instead of true prediction.
- **Inference:** No future tokens exist, so the model's learned strategy breaks $\Rightarrow$ degraded generation quality.

**Answer skeleton (fill in for any similar question):**

1. Define softmax: $P(z_i) = e^{z_i} / \sum_j e^{z_j}$
2. Show $e^{-\infty} = 0$ $\Rightarrow$ zero attention $\Rightarrow$ no leakage
3. Show $e^{0} = 1 > 0$ $\Rightarrow$ positive attention $\Rightarrow$ information leakage
4. Explain the train/inference mismatch caused by leakage

---

## At-a-Glance Formula Card

| # | Topic | Core Formula |
|---|-------|-------------|
| 1 | Attention Score | $\dfrac{\sum Q_i K_i}{\sqrt{d_k}}$ |
| 2 | RoPE Rotation | $x' = R_{m\theta}\,x$ where $R$ is 2D rotation matrix |
| 3 | SwiGLU Sizing | $d_\text{hidden} = \dfrac{8r}{3}\,d_\text{model}$ (when $d_\text{ffn}=4d$) |
| 4 | PE Wavelength | $10000^{2i/d_\text{model}}$ |
| 5 | Causal Mask | Add $-\infty$ so $e^{-\infty}=0$; never multiply by $0$ since $e^0=1>0$ |
