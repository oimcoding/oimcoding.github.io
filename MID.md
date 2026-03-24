# Statistics & Machine Learning Answer Sheet
## Visual Reference Guide for Complete Beginners

---

## TABLE OF CONTENTS

1. [Question 1: Bias-Variance Decomposition](#part-1-bias-variance-decomposition)
2. [Question 2: K-Nearest Neighbors](#part-2-k-nearest-neighbors)
3. [Question 3: Ridge Regression with SVD](#part-3-ridge-regression-with-svd)
4. [Question 4: Equal Singular Values](#part-4-equal-singular-values)
5. [Question 5: Multiple Testing](#part-5-multiple-testing)

---

## PART 1: BIAS-VARIANCE DECOMPOSITION

### Original Exam Question

```
Question 1.

For a regression problem with model yi = f*(xi) + δi where E[δi] = 0 and
E[δi²] = σ², derive the bias-variance decomposition of the expected squared
error. Start from the basic expression E[(y - ŷf(x))²] and show all steps.
```

### Sub-Questions You Should Be Able to Answer

```
Q1.1: What is a regression problem?
Q1.2: What are all the symbols (y, f*, δ, E, σ²)?
Q1.3: What does each term in the decomposition mean?
Q1.4: How do you derive the bias-variance decomposition?
Q1.5: Why does the cross term equal zero?
Q1.6: What is irreducible error?
```

### Symbol Dictionary (First Appearance)

```
┌──────────┬────────────────────────────────────┬──────────────┐
│ Symbol   │ Full Name                          │ Think of it  │
├──────────┼────────────────────────────────────┼──────────────┤
│ y        │ Observed value (response)          │ Real data    │
│          │                                    │ point        │
├──────────┼────────────────────────────────────┼──────────────┤
│ x        │ Input features (predictors)        │ What you     │
│          │                                    │ know         │
├──────────┼────────────────────────────────────┼──────────────┤
│ f*(x)    │ True function (unknown in reality) │ Perfect      │
│          │ The "star" means "true/optimal"    │ pattern      │
├──────────┼────────────────────────────────────┼──────────────┤
│ δ        │ Delta = Random noise/error         │ Unpredictable│
│          │ (Greek letter "delta")             │ scatter      │
├──────────┼────────────────────────────────────┼──────────────┤
│ E[·]     │ Expectation = Average over many    │ Long-run     │
│          │ datasets/samples                   │ average      │
├──────────┼────────────────────────────────────┼──────────────┤
│ σ²       │ Sigma squared = Variance of noise  │ How spread   │
│          │ (Greek letter "sigma")             │ out noise is │
├──────────┼────────────────────────────────────┼──────────────┤
│ ŷf(x)    │ Estimated function (your model)    │ Your         │
│          │ The "hat" means "estimated"        │ prediction   │
├──────────┼────────────────────────────────────┼──────────────┤
│ (a)²     │ Square of a value                  │ a × a        │
│          │ Used to measure "size" of error    │              │
└──────────┴────────────────────────────────────┴──────────────┘
```

### Core Concept Map

```
REGRESSION PROBLEM:
Given data points (x₁,y₁), (x₂,y₂), ..., (xₙ,yₙ)
Find a function ŷf(x) that predicts y from x

TRUE WORLD          YOUR MODEL          PREDICTION ERROR

y = f*(x) + δ  →   ŷf(x)         →    Error = y - ŷf(x)
│                                              │
│                                              ↓
└─ noise (random)                    Break into 3 parts:

                                     σ² + Bias² + Variance
                                     │    │       │
                                     │    │       └─ How jumpy is your model?
                                     │    └─ How wrong on average?
                                     └─ Noise you can't fix
```

### The Master Formula

```
┌─────────────────────────────────────────────────┐
│                                                 │
│  E[(y - ŷf(x))²] = σ² + Bias² + Variance       │
│     │     │                                     │
│     │     └─ Your prediction                    │
│     └─ Observed value                          │
│                                                 │
│  Total Error = Irreducible + Systematic + Random│
│                                                 │
└─────────────────────────────────────────────────┘

WHERE:
  σ² = E[δ²] = Average squared noise
  Bias² = (f*(x) - E[ŷf(x)])² = How far off on average
  Variance = E[(E[ŷf(x)] - ŷf(x))²] = How much predictions vary
```

### Step-by-Step Derivation Template

**GIVEN FACTS:**
```
1. y = f*(x) + δ           (true model)
2. E[δ] = 0                (noise averages to zero)
3. E[δ²] = σ²              (noise variance)
4. δ independent of ŷf(x)  (noise independent of model)
```

**STEP 1:** Start with expected squared error
```
E[(y - ŷf(x))²]
  │   │   │
  │   │   └─ ŷf(x) = your prediction
  │   └─ y = observed value
  └─ E[·] = expectation (average over many datasets)
```

**STEP 2:** Add and subtract f*(x) (the true function)
```
Why? To separate what we can't control (noise) from what we can (model)

E[(y - ŷf(x))²] = E[(y - f*(x) + f*(x) - ŷf(x))²]
                      └─── a ────┘ └──── b ────┘

This is a mathematical trick: a + b where a = (y - f*) and b = (f* - ŷf)
```

**STEP 3:** Expand square using (a + b)² = a² + b² + 2ab
```
= E[(y - f*(x))²] + E[(f*(x) - ŷf(x))²] + 2E[(y - f*(x))(f*(x) - ŷf(x))]
   └──── Term 1 ───┘   └──── Term 2 ────┘   └────── Cross term ──────┘
```

**STEP 4:** Simplify cross term using y - f*(x) = δ
```
Cross term = 2E[δ(f*(x) - ŷf(x))]
           = 2E[δ] · E[f*(x) - ŷf(x)]    (independence)
           = 2 · 0 · E[f*(x) - ŷf(x)]    (E[δ] = 0)
           = 0

The cross term vanishes!
```

**STEP 5:** Substitute back
```
E[(y - ŷf(x))²] = E[(y - f*(x))²] + E[(f*(x) - ŷf(x))²]
                = E[δ²] + E[(f*(x) - ŷf(x))²]
                = σ² + E[(f*(x) - ŷf(x))²]
                  │
                  └─ Irreducible error (can't reduce this!)
```

**STEP 6:** Decompose second term: Add/subtract E[ŷf(x)]
```
E[(f*(x) - ŷf(x))²] = E[(f*(x) - E[ŷf(x)] + E[ŷf(x)] - ŷf(x))²]
                          └──── Bias ────┘ └── Deviation ──┘

Same trick as before! Let a = f*(x) - E[ŷf(x)] and b = E[ŷf(x)] - ŷf(x)
```

**STEP 7:** Expand square again
```
= E[(f*(x) - E[ŷf(x)])²] + E[(E[ŷf(x)] - ŷf(x))²]
  + 2E[(f*(x) - E[ŷf(x)])(E[ŷf(x)] - ŷf(x))]
```

**STEP 8:** Cross term is zero because E[E[ŷf(x)] - ŷf(x)] = 0
```
E[E[ŷf(x)] - ŷf(x)] = E[ŷf(x)] - E[ŷf(x)] = 0

So cross term = 2(f*(x) - E[ŷf(x)]) · 0 = 0
```

**STEP 9:** Notice that (f*(x) - E[ŷf(x)])² has no expectation
```
Why? Because f*(x) and E[ŷf(x)] are both constants (not random)
So E[(f*(x) - E[ŷf(x)])²] = (f*(x) - E[ŷf(x)])²
```

**FINAL RESULT:**
```
┌──────────────────────────────────────────────────────────┐
│                                                          │
│  E[(y - ŷf(x))²] = σ² + (f*(x) - E[ŷf(x)])² + Var[ŷf(x)] │
│                    │    └────── Bias² ──────┘  │         │
│                    │                            │         │
│                Irreducible                  Variance      │
│                                                           │
└──────────────────────────────────────────────────────────┘

WHERE:
  σ² = E[δ²] = Irreducible error (noise we can't remove)
  Bias² = (f*(x) - E[ŷf(x)])² = Systematic error (model wrong on average)
  Variance = E[(E[ŷf(x)] - ŷf(x))²] = Random error (model jumps around)
```

### Visual Understanding

```
         y (what you observe)
         │
         ├─ δ (noise) ───────────────→ σ² (irreducible error)
         │                              Can't fix this!
         f*(x) (truth - unknown!)
         │
         ├─ Bias ────────────────────→ Bias² (systematic error)
         │                              Model wrong on average
         E[ŷf(x)] (average prediction)
         │
         ├─ Fluctuation ─────────────→ Variance (random error)
         │                              Model jumps around
         ŷf(x) (single prediction)
```

### Why Cross Terms = 0 (Important!)

```
Cross term: E[δ(f*(x) - ŷf(x))]

Step 1: Use independence
        δ is random noise, independent of our model ŷf(x)
        So: E[δ(f*(x) - ŷf(x))] = E[δ] · (f*(x) - E[ŷf(x)])

Step 2: Use E[δ] = 0
        = 0 · (f*(x) - E[ŷf(x)]) = 0 (check!)

KEY INSIGHT: Noise has zero mean, so it doesn't correlate with model error!
```

---

## PART 2: K-NEAREST NEIGHBORS

### Original Exam Question

```
Question 2.

Consider the true model y = f*(x) + δ, where f*(x) is the true function and
δ is random noise with zero mean (E[δ] = 0) and variance Var[δ] = σ². For the
K-Nearest Neighbors estimator

    ŷfK(x) = (1/K) Σ yℓ
                  ℓ∈Kx

derive the bias, the variance term, and explain how variance depends on K.
```

### Sub-Questions You Should Be Able to Answer

```
Q2.1: What is K-Nearest Neighbors (K-NN)?
Q2.2: What does K represent?
Q2.3: What are all the symbols (ŷfK, Kx, yℓ, xℓ)?
Q2.4: What is the bias of K-NN?
Q2.5: What is the variance of K-NN?
Q2.6: How does K affect bias and variance?
Q2.7: When is K-NN unbiased?
Q2.8: What is the bias-variance tradeoff?
```

### Symbol Dictionary

```
┌──────────┬────────────────────────────────────┬──────────────┐
│ Symbol   │ Full Name                          │ Think of it  │
├──────────┼────────────────────────────────────┼──────────────┤
│ K        │ Number of neighbors to average     │ How many to  │
│          │                                    │ look at      │
├──────────┼────────────────────────────────────┼──────────────┤
│ ŷfK(x)   │ K-NN prediction at point x         │ Your K-NN    │
│          │                                    │ guess        │
├──────────┼────────────────────────────────────┼──────────────┤
│ Kx       │ Set of K nearest neighbors to x    │ K closest    │
│          │ (subscript x means "around x")     │ points       │
├──────────┼────────────────────────────────────┼──────────────┤
│ yℓ       │ Response value of neighbor ℓ       │ Data point   │
│          │ (ℓ is just an index/label)         │ #ℓ           │
├──────────┼────────────────────────────────────┼──────────────┤
│ xℓ       │ Feature values of neighbor ℓ       │ Location of  │
│          │                                    │ point #ℓ     │
├──────────┼────────────────────────────────────┼──────────────┤
│ Σ        │ Summation (Greek "sigma")          │ Add up all   │
│ ℓ∈Kx     │ Sum over all ℓ in set Kx          │ neighbors    │
├──────────┼────────────────────────────────────┼──────────────┤
│ Var[·]   │ Variance = E[(· - E[·])²]         │ How spread   │
│          │                                    │ out values   │
│          │                                    │ are          │
└──────────┴────────────────────────────────────┴──────────────┘
```

### Visual Model

```
What is K-NN?
Given a new point x, find the K closest training points and average them.

         Past training data points
              ●
          ●       ●
              ●
          ●   X  ●  ← Query point x (predict y for this)
              ●       Find K=3 nearest neighbors
          ●       ●
              ●

Distance to x:   Point    Distance
                  ●₁       2.1  ← Closest
                  ●₂       2.3  ← 2nd closest  } These K=3
                  ●₃       2.7  ← 3rd closest  } are in Kx
                  ●₄       4.1
                  ●₅       5.0

K=3: ŷf₃(x) = (y₁ + y₂ + y₃)/3
             Average the 3 closest points
```

### K-NN Formula

```
┌────────────────────────────────────────┐
│                                        │
│  ŷfK(x) = (1/K) Σ yℓ                  │
│                ℓ∈Kx                    │
│           │     │   │                  │
│           │     │   └─ Neighbors       │
│           │     └─ Sum over neighbors  │
│           └─ Divide by K (average)     │
│                                        │
│  "Average response of K neighbors"     │
│                                        │
└────────────────────────────────────────┘
```

### Bias Derivation

**GOAL:** Find E[ŷfK(x)] and compare to f*(x)

**STEP 1:** Start with K-NN formula
```
ŷfK(x) = (1/K) Σ yℓ
               ℓ∈Kx
```

**STEP 2:** Substitute true model yℓ = f*(xℓ) + δℓ
```
What is yℓ? Each training point has yℓ = f*(xℓ) + δℓ
            │                        │          │
            │                        │          └─ Noise for point ℓ
            │                        └─ True function at location xℓ
            └─ Observed response for neighbor ℓ

ŷfK(x) = (1/K) Σ [f*(xℓ) + δℓ]
               ℓ∈Kx
```

**STEP 3:** Split sum using Σ[a + b] = Σa + Σb
```
= (1/K) Σ f*(xℓ) + (1/K) Σ δℓ
        ℓ∈Kx            ℓ∈Kx
  └─ deterministic ─┘   └─ random ─┘
```

**STEP 4:** Take expectation E[·]
```
E[ŷfK(x)] = E[(1/K) Σ f*(xℓ)] + E[(1/K) Σ δℓ]
                     ℓ∈Kx              ℓ∈Kx
```

**STEP 5:** f*(xℓ) is deterministic (not random), E[δℓ] = 0
```
= (1/K) Σ f*(xℓ) + (1/K) Σ E[δℓ]
        ℓ∈Kx            ℓ∈Kx

= (1/K) Σ f*(xℓ) + (1/K) Σ 0
        ℓ∈Kx            ℓ∈Kx

= (1/K) Σ f*(xℓ)
        ℓ∈Kx
```

**STEP 6:** Compute Bias
```
┌────────────────────────────────────────────┐
│                                            │
│  Bias = f*(x) - E[ŷfK(x)]                 │
│                                            │
│       = f*(x) - (1/K) Σ f*(xℓ)            │
│                       ℓ∈Kx                │
│                                            │
│  Interpretation:                           │
│  • If f* constant near x: Bias ≈ 0        │
│  • If f* varies near x: Bias ≠ 0          │
│  • Larger K → more variation → more bias  │
│                                            │
└────────────────────────────────────────────┘
```

### Variance Derivation

**GOAL:** Find Var[ŷfK(x)]

**STEP 1:** Start with K-NN in two parts
```
ŷfK(x) = (1/K) Σ f*(xℓ) + (1/K) Σ δℓ
                ℓ∈Kx            ℓ∈Kx
         └─ deterministic ─┘   └─ random ─┘
```

**STEP 2:** Variance of sum = variance of random part only
```
Var[ŷfK(x)] = Var[(1/K) Σ f*(xℓ) + (1/K) Σ δℓ]
                         ℓ∈Kx            ℓ∈Kx

            = Var[(1/K) Σ δℓ]    (only random part has variance)
                        ℓ∈Kx
```

**STEP 3:** Pull out constant (1/K)²
```
Rule: Var[cX] = c² Var[X]

Var[ŷfK(x)] = (1/K)² Var[Σ δℓ]
                        ℓ∈Kx
```

**STEP 4:** Use independence: Var[Σ] = Σ Var when independent
```
Rule: If X₁, X₂, ..., Xₖ independent, then Var[X₁+X₂+...+Xₖ] = Var[X₁]+...+Var[Xₖ]

Var[Σ δℓ] = Σ Var[δℓ]
  ℓ∈Kx     ℓ∈Kx
```

**STEP 5:** Each δℓ has Var[δℓ] = σ², and there are K terms
```
Σ Var[δℓ] = σ² + σ² + ... + σ²  (K times)
ℓ∈Kx
           = K · σ²
```

**STEP 6:** Substitute back
```
Var[ŷfK(x)] = (1/K)² · K · σ²
            = (1/K²) · K · σ²
            = σ²/K
```

**FINAL RESULT:**
```
┌────────────────────────────────────────┐
│                                        │
│  Var[ŷfK(x)] = σ²/K                   │
│                │   │                   │
│                │   └─ # neighbors      │
│                └─ Noise variance       │
│                                        │
│  KEY INSIGHT:                          │
│  • ↑K (more neighbors) → ↓Variance    │
│  • K=1 → Var = σ² (highest)           │
│  • K=n → Var = σ²/n (lowest)          │
│                                        │
└────────────────────────────────────────┘
```

### K Effect Diagram

```
K=1 (small K)                    K=100 (large K)
│                                │
├─ Bias: LOW                     ├─ Bias: HIGH
│  (follows data closely)        │  (smooth, misses details)
│                                │
├─ Variance: HIGH                ├─ Variance: LOW
│  (σ²/1 = σ²)                   │  (σ²/100)
│  Predictions jump around       │  Predictions stable
│                                │
└─ Overfitting (WARNING)         └─ Underfitting (WARNING)
   Fits noise                       Misses signal


         OPTIMAL K (sweet spot)
            │
            ├─ Bias: MEDIUM
            ├─ Variance: MEDIUM
            └─ Best test error
```

### Formula Summary Chart

```
┌────────────────┬──────────────────────┬───────────────┐
│ Quantity       │ Formula              │ Effect of ↑K  │
├────────────────┼──────────────────────┼───────────────┤
│ Prediction     │ ŷfK(x) = (1/K)Σyℓ   │ More smooth   │
├────────────────┼──────────────────────┼───────────────┤
│ Expectation    │ E[ŷfK] = (1/K)Σf*(xℓ)│ More averaging│
├────────────────┼──────────────────────┼───────────────┤
│ Bias           │ f*(x) - E[ŷfK]      │ Increases ↑   │
├────────────────┼──────────────────────┼───────────────┤
│ Variance       │ σ²/K                 │ Decreases ↓   │
└────────────────┴──────────────────────┴───────────────┘
```

---

## PART 3: RIDGE REGRESSION WITH SVD

### Original Exam Question

```
Question 3.

Using Singular Value Decomposition (SVD), express the ridge regression
estimator in terms of singular components and analyze its properties. Given
SVD: X = UDV^T where:

• U is an n × r orthogonal matrix
• V is a p × r orthogonal matrix
• D is an r × r diagonal matrix with singular values d₁ ≥ d₂ ≥ ... ≥ dr > 0
```

### Sub-Questions You Should Be Able to Answer

```
Q3.1: What is ridge regression?
Q3.2: What is Singular Value Decomposition (SVD)?
Q3.3: What are all the symbols (X, U, D, V, λ, β)?
Q3.4: What are singular values?
Q3.5: What does "orthogonal matrix" mean?
Q3.6: How do you substitute SVD into ridge formula?
Q3.7: What is the shrinkage factor?
Q3.8: Why does ridge help with small singular values?
```

### Symbol Dictionary

```
┌──────────┬────────────────────────────────────┬──────────────┐
│ Symbol   │ Full Name                          │ Think of it  │
├──────────┼────────────────────────────────────┼──────────────┤
│ X        │ Design matrix (n×p)                │ All your     │
│          │ n samples, p features              │ input data   │
├──────────┼────────────────────────────────────┼──────────────┤
│ Y        │ Response vector (n×1)              │ All your     │
│          │                                    │ outputs      │
├──────────┼────────────────────────────────────┼──────────────┤
│ β        │ Beta = coefficient vector (p×1)    │ What you're  │
│          │ (Greek letter "beta")              │ solving for  │
├──────────┼────────────────────────────────────┼──────────────┤
│ β̂        │ Estimated coefficients             │ Your solution│
│          │ ("beta hat")                       │              │
├──────────┼────────────────────────────────────┼──────────────┤
│ λ        │ Lambda = regularization parameter  │ Penalty      │
│          │ (Greek letter "lambda")            │ strength     │
├──────────┼────────────────────────────────────┼──────────────┤
│ I        │ Identity matrix (diagonal of 1's)  │ [1 0]        │
│          │                                    │ [0 1]        │
├──────────┼────────────────────────────────────┼──────────────┤
│ U        │ Left singular vectors (n×r)        │ Row space    │
│          │ Columns are orthonormal            │ directions   │
├──────────┼────────────────────────────────────┼──────────────┤
│ D        │ Diagonal matrix of singular values │ [d₁  0 ]     │
│          │ (r×r)                              │ [ 0 d₂]      │
├──────────┼────────────────────────────────────┼──────────────┤
│ V        │ Right singular vectors (p×r)       │ Column space │
│          │ Columns are orthonormal            │ directions   │
├──────────┼────────────────────────────────────┼──────────────┤
│ r        │ Rank of X (≤ min(n,p))            │ # non-zero   │
│          │                                    │ sing. values │
├──────────┼────────────────────────────────────┼──────────────┤
│ dj       │ j-th singular value (diagonal of D)│ Importance   │
│          │ d₁ ≥ d₂ ≥ ... ≥ dr > 0            │ of direction │
├──────────┼────────────────────────────────────┼──────────────┤
│ uj       │ j-th column of U                   │ Left vector  │
│          │                                    │ #j           │
├──────────┼────────────────────────────────────┼──────────────┤
│ vj       │ j-th column of V                   │ Right vector │
│          │                                    │ #j           │
├──────────┼────────────────────────────────────┼──────────────┤
│ A^T      │ Transpose of matrix A              │ Flip rows/   │
│          │                                    │ columns      │
├──────────┼────────────────────────────────────┼──────────────┤
│ A^(-1)   │ Inverse of matrix A                │ Like 1/A     │
│          │                                    │ for matrices │
└──────────┴────────────────────────────────────┴──────────────┘
```

### What is SVD?

```
SINGULAR VALUE DECOMPOSITION (SVD):
Break any matrix into 3 simple pieces

X = U D V^T
│   │ │ │
│   │ │ └─ Rotation in output space (p×r, orthogonal)
│   │ └─ Scaling (r×r, diagonal with d₁, d₂, ..., dr)
│   └─ Rotation in input space (n×r, orthogonal)
└─ Original data matrix (n×p)

Think of it as: Rotate → Scale → Rotate back
```

### Matrix Dimensions Diagram

```
    X      =    U    ×    D    ×   V^T
  [n×p]       [n×r]    [r×r]     [r×p]

Example: n=100 samples, p=50 features, r=30 rank

    X      =    U    ×    D    ×   V^T
 [100×50]    [100×30]  [30×30]   [30×50]

   Data        Row      Singular   Column
              space     values     space
```

### SVD Visual

```
X (data matrix)    =    U    ×    D    ×    V^T
[n × p]                [n×r]    [r×r]     [r×p]
                         │        │          │
                         │        │          └─ Right singular vectors
                         │        │             (directions in feature space)
                         │        │
                         │        └─ Singular values (d₁, d₂, ..., dr)
                         │           d₁ ≥ d₂ ≥ ... ≥ dr > 0
                         │           (importance of each direction)
                         │
                         └─ Left singular vectors
                            (directions in sample space)

Orthogonal:            Diagonal:           Orthogonal:
U^T U = I              [d₁  0   0]         V^T V = I
(columns               [0   d₂  0]         (columns
 perpendicular)        [0   0  d₃]          perpendicular)
```

### What is "Orthogonal"?

```
ORTHOGONAL MATRIX: Columns are perpendicular (90°) and length 1

Example: U = [u₁ u₂ u₃]

Properties:
  u₁^T u₁ = 1    (length 1)
  u₁^T u₂ = 0    (perpendicular)
  u₁^T u₃ = 0    (perpendicular)
  u₂^T u₃ = 0    (perpendicular)

Key fact: U^T U = I

Visual:
    u₂
     ↑
     │
     │
     └────→ u₁
    90°

All columns point in different directions, length 1
```

### Ridge vs OLS

```
ORDINARY LEAST SQUARES (OLS):
┌─────────────────────────────────┐
│ β̂OLS = (X^T X)^(-1) X^T Y      │
│                                 │
│ Minimize: ||Y - Xβ||²          │
│           (fit data exactly)    │
│                                 │
│ Problem: Unstable when X^T X    │
│          nearly singular        │
└─────────────────────────────────┘

RIDGE REGRESSION:
┌─────────────────────────────────┐
│ β̂ridge = (X^T X + λI)^(-1) X^T Y│
│          └─ penalty ─┘          │
│                                 │
│ Minimize: ||Y - Xβ||² + λ||β||²│
│           └─ fit ──┘   └─penalty│
│                                 │
│ Benefit: Always stable          │
│          (even if X^T X singular)│
└─────────────────────────────────┘

λ = 0:    Ridge = OLS (no penalty)
λ small:  Light penalty (close to OLS)
λ large:  Heavy penalty (shrink toward 0)
λ → ∞:    β̂ridge → 0 (all coefficients → 0)
```

### SVD Substitution Steps

**GOAL:** Express β̂ridge using U, D, V instead of X

**STEP 1:** Write ridge formula
```
β̂ridge = (X^T X + λI)^(-1) X^T Y
```

**STEP 2:** Substitute X = UDV^T
```
What is X^T? If X = UDV^T, then X^T = (V^T)^T D^T U^T = VD^T U^T

β̂ridge = (V D^T U^T U D V^T + λI)^(-1) V D^T U^T Y
          │       │       │
          │       │       └─ From X = UDV^T
          │       └─ X^T = VD^TU^T
          └─ (X^T)(X) = ...
```

**STEP 3:** Use U^T U = I (orthogonality)
```
= (V D^T I D V^T + λI)^(-1) V D^T U^T Y
= (V D^T D V^T + λI)^(-1) V D^T U^T Y

Since D is diagonal, D^T D = D²:
= (V D² V^T + λI)^(-1) V D^T U^T Y
```

**STEP 4:** Factor out V from first term
```
Use: I = V V^T (because V is orthogonal)

λI = λV V^T

So:
= (V D² V^T + λV V^T)^(-1) V D^T U^T Y
= (V(D² + λI)V^T)^(-1) V D^T U^T Y
   └─ Factor V ──┘
```

**STEP 5:** Use matrix inversion rule: (VAV^T)^(-1) = V A^(-1) V^T
```
Why? (VAV^T)(VA^(-1)V^T) = VAA^(-1)V^T = VV^T = I (check!)

β̂ridge = V(D² + λI)^(-1) V^T V D^T U^T Y
```

**STEP 6:** Use V^T V = I
```
β̂ridge = V(D² + λI)^(-1) I D^T U^T Y
       = V(D² + λI)^(-1) D^T U^T Y
```

**STEP 7:** Since D is diagonal, D^T = D
```
β̂ridge = V(D² + λI)^(-1) D U^T Y
```

**STEP 8:** Expand as sum over singular components
```
What is (D² + λI)^(-1) D?

(D² + λI)^(-1) D = [1/(d₁²+λ)    0        0    ] [d₁  0   0 ]
                   [   0      1/(d₂²+λ)    0    ] [0   d₂  0 ]
                   [   0         0      1/(d₃²+λ)] [0   0   d₃]

                 = [d₁/(d₁²+λ)      0           0      ]
                   [    0       d₂/(d₂²+λ)      0      ]
                   [    0           0       d₃/(d₃²+λ) ]

So each component j gets weight dⱼ/(dⱼ² + λ)
```

**FINAL RESULT:**
```
┌────────────────────────────────────────────────┐
│                                                │
│  β̂ridge = Σ (dⱼ/(dⱼ² + λ)) uⱼ^T Y vⱼ         │
│           j=1                                  │
│           └─ shrinkage ──┘  │  │   │          │
│                             │  │   └─ direction│
│                             │  └─ project Y    │
│                             └─ weight          │
│                                                │
│  Shrinkage factor: dⱼ/(dⱼ² + λ) ∈ (0, 1]     │
│                                                │
└────────────────────────────────────────────────┘
```

### Shrinkage Factor Analysis

```
Component j:  Weight = dⱼ/(dⱼ² + λ)

Case 1: Large dⱼ (important direction)
        dⱼ/(dⱼ² + λ) ≈ dⱼ/dⱼ² = 1/dⱼ  (close to OLS weight)
        → Keep most of this component

Case 2: Small dⱼ (unimportant direction)
        dⱼ/(dⱼ² + λ) ≈ dⱼ/λ ≈ 0  (shrink heavily)
        → Discard most of this component

Case 3: λ = 0 (no penalty)
        dⱼ/(dⱼ² + 0) = 1/dⱼ  (OLS)

Case 4: λ → ∞ (infinite penalty)
        dⱼ/(dⱼ² + λ) → 0  (all components → 0)
```

### Comparison: OLS vs Ridge Weights

```
Component j      OLS Weight       Ridge Weight

Direction vⱼ  →  1/dⱼ         →   dⱼ/(dⱼ² + λ)
                    │                  │
                    │                  ↓
                Large dⱼ:         dⱼ/(dⱼ² + λ) ≈ 1/dⱼ
                  Stable            (similar to OLS)
                  1/dⱼ small        Weight ≈ OLS
                    │
                    ↓
                Small dⱼ:         dⱼ/(dⱼ² + λ) ≈ 0
                  UNSTABLE!         (shrunk heavily)
                  1/dⱼ huge!        Weight ≈ 0
                  Explodes!         Controlled (good)

KEY INSIGHT: Ridge shrinks unstable directions more!
```

### Shrinkage Visualization

```
     Weight
       ↑
       │     ╱ OLS: 1/dⱼ (unstable for small dⱼ)
       │    ╱
   100 │   ╱
       │  ╱
    10 │ ╱
       │╱_____ Ridge: dⱼ/(dⱼ²+λ) (controlled)
     1 │
       │
       └──────────────→ dⱼ (singular value)
      0.01  0.1   1   10
      small           large
    (unstable)      (stable)

Ridge prevents explosion when dⱼ small!
```

---

## PART 4: EQUAL SINGULAR VALUES

### Original Exam Question

```
Question 4.

Consider a special case where all singular values of the design matrix X are
equal, i.e., d₁ = d₂ = ⋯ = dr = d.

(i) Derive the expressions for both the OLS estimator and the Ridge regression
    estimator in this case.

(ii) Compare how these estimators differ in terms of the direction and magnitude
     of the coefficient vector β.
```

### Sub-Questions You Should Be Able to Answer

```
Q4.1: What does "equal singular values" mean?
Q4.2: How do you derive OLS in this special case?
Q4.3: How do you derive Ridge in this special case?
Q4.4: What is the relationship between Ridge and OLS?
Q4.5: What is the shrinkage factor?
Q4.6: Do they point in the same direction?
Q4.7: How is this different from the general case?
```

### Symbol Reminder

```
d₁ = d₂ = ... = dr = d  (all singular values equal)
```

### Special Case Setup

```
GENERAL CASE:               SPECIAL CASE:
d₁ = 10                     d₁ = 5
d₂ = 5    (different)       d₂ = 5    (all equal!)
d₃ = 2                      d₃ = 5

WHY SPECIAL?
When all directions equally important, shrinkage is uniform!
```

### OLS Derivation (Equal Singular Values)

**STEP 1:** Start with general OLS formula (from Part 3)
```
β̂OLS = Σ (1/dⱼ) uⱼ^T Y vⱼ
        j=1

This sums over r components (j = 1, 2, ..., r)
```

**STEP 2:** All dⱼ = d (given condition)
```
β̂OLS = Σ (1/d) uⱼ^T Y vⱼ
        j=1

Every term has same 1/d weight!
```

**STEP 3:** Factor out constant 1/d
```
β̂OLS = (1/d) Σ uⱼ^T Y vⱼ
              j=1

Rule: Σ c·aⱼ = c·Σaⱼ when c is constant
```

**STEP 4:** Recognize matrix form: Σ uⱼ^T Y vⱼ = V U^T Y
```
Why? Because:
  u₁^T Y v₁ + u₂^T Y v₂ + ... = [v₁ v₂ ...][u₁^T Y]   = V U^T Y
                                              [u₂^T Y]
                                              [  ...  ]
```

**FINAL RESULT:**
```
┌─────────────────────────────┐
│                             │
│  β̂OLS = (1/d) V U^T Y      │
│         │     │  │  │       │
│         │     │  │  └─ data │
│         │     │  └─ project │
│         │     └─ rotate     │
│         └─ scale by 1/d     │
│                             │
└─────────────────────────────┘
```

### Ridge Derivation (Equal Singular Values)

**STEP 1:** Start with general ridge formula (from Part 3)
```
β̂ridge = Σ (dⱼ/(dⱼ² + λ)) uⱼ^T Y vⱼ
         j=1
```

**STEP 2:** All dⱼ = d
```
β̂ridge = Σ (d/(d² + λ)) uⱼ^T Y vⱼ
         j=1

Every term has same d/(d²+λ) weight!
```

**STEP 3:** Factor out constant d/(d² + λ)
```
β̂ridge = (d/(d² + λ)) Σ uⱼ^T Y vⱼ
                       j=1
```

**STEP 4:** Matrix form
```
β̂ridge = (d/(d² + λ)) V U^T Y
```

**STEP 5:** Compare to OLS (β̂OLS = (1/d) V U^T Y)
```
β̂ridge = (d/(d² + λ)) V U^T Y

        = (d/(d² + λ)) · d · (1/d) V U^T Y

        = (d²/(d² + λ)) · β̂OLS
          └─ shrinkage ──┘
```

**FINAL RESULT:**
```
┌──────────────────────────────────────────────┐
│                                              │
│  β̂ridge = (d²/(d² + λ)) β̂OLS                │
│            └─ shrinkage ──┘                  │
│                                              │
│  Shrinkage factor = d²/(d² + λ)             │
│                                              │
│  Properties:                                 │
│  • Always < 1 (when λ > 0)                  │
│  • Closer to 1 when d large or λ small      │
│  • Closer to 0 when d small or λ large      │
│                                              │
└──────────────────────────────────────────────┘
```

### Shrinkage Factor Behavior

```
Shrinkage factor: s = d²/(d² + λ)

Test values (d = 5):

λ = 0:     s = 25/(25 + 0) = 1.00     (no shrinkage, same as OLS)
λ = 1:     s = 25/(25 + 1) = 0.96     (96% of OLS)
λ = 5:     s = 25/(25 + 5) = 0.83     (83% of OLS)
λ = 25:    s = 25/(25 + 25) = 0.50    (50% of OLS)
λ = 100:   s = 25/(25 + 100) = 0.20   (20% of OLS)
λ → ∞:     s → 0                      (shrinks to 0)

    Shrinkage s
       ↑
     1 │──────╲
       │       ╲
   0.5 │        ╲___
       │            ╲___
       │                ╲___
     0 └─────────────────────→ λ
       0    d²              ∞

Steepest drop near λ = d²
```

### Direction vs Magnitude Comparison

```
         ↑ β₂
         │
         │    β̂OLS (length = L)
         │    ╱
         │   ╱
         │  ╱
         │ ╱ β̂ridge (length = s·L, where s = d²/(d²+λ))
         │╱
         └──────────→ β₁

SAME direction (check)
DIFFERENT magnitude (check)

β̂ridge points same way as β̂OLS, just shorter!

Example (d=5, λ=25, s=0.5):
  β̂OLS = [4, 3]      (length = 5)
  β̂ridge = [2, 1.5]  (length = 2.5 = 0.5 × 5)
```

### Key Differences Table

```
┌──────────────┬─────────────────────┬──────────────────────┐
│ Property     │ General Case        │ Equal Singular Values│
├──────────────┼─────────────────────┼──────────────────────┤
│ Singular     │ d₁ ≠ d₂ ≠ d₃       │ d₁ = d₂ = d₃ = d     │
│ values       │ (different)         │ (all equal)          │
├──────────────┼─────────────────────┼──────────────────────┤
│ Shrinkage    │ dⱼ/(dⱼ²+λ)         │ d/(d²+λ)             │
│ formula      │ varies with j       │ same for all j       │
├──────────────┼─────────────────────┼──────────────────────┤
│ Shrinkage    │ DIFFERENT           │ UNIFORM              │
│ per direction│ Some directions     │ All directions       │
│              │ shrunk more         │ shrunk equally       │
├──────────────┼─────────────────────┼──────────────────────┤
│ Relationship │ No simple relation  │ β̂ridge = s·β̂OLS     │
│ to OLS       │                     │ (scalar multiple)    │
├──────────────┼─────────────────────┼──────────────────────┤
│ Direction    │ DIFFERENT           │ SAME                 │
│ vs OLS       │ Ridge changes       │ Ridge just scales    │
│              │ direction           │                      │
└──────────────┴─────────────────────┴──────────────────────┘
```

---

## PART 5: MULTIPLE TESTING

### Original Exam Question

```
Question 5.

For the Benjamini-Hochberg procedure with p-values p₁, ..., pm: How is the
number of false positives estimated using the uniform distribution of p-values
under the null hypothesis, and how is the False Discovery Proportion (FDP)
estimated for a given cutoff threshold t?

For the Mirror Statistics approach with statistics M₁, ..., Mp: How are false
positives estimated using the symmetry property of mirror statistics under the
null hypothesis, and how is the False Discovery Proportion (FDP) estimated for
a given cutoff threshold t?
```

### Sub-Questions You Should Be Able to Answer

```
Q5.1: What is the multiple testing problem?
Q5.2: What are false positives and true positives?
Q5.3: What are all the symbols (FP, TP, FDP, FDR, m, m₀, m₁)?
Q5.4: What is Benjamini-Hochberg method?
Q5.5: Why are p-values uniform under null?
Q5.6: How do you estimate false positives in BH?
Q5.7: What is Mirror Statistics method?
Q5.8: What is symmetry under null?
Q5.9: How do you estimate false positives in Mirror?
Q5.10: How do the two methods differ?
```

### Symbol Dictionary

```
┌──────────┬────────────────────────────────────┬──────────────┐
│ Symbol   │ Full Name                          │ Think of it  │
├──────────┼────────────────────────────────────┼──────────────┤
│ m        │ Total number of tests/hypotheses   │ How many     │
│          │                                    │ tests total  │
├──────────┼────────────────────────────────────┼──────────────┤
│ m₀       │ Number of true null hypotheses     │ How many are │
│          │ (subscript 0 = null)               │ truly null   │
├──────────┼────────────────────────────────────┼──────────────┤
│ m₁       │ Number of true alternatives        │ How many have│
│          │ (subscript 1 = alternative)        │ real signal  │
│          │ m₁ = m - m₀                        │              │
├──────────┼────────────────────────────────────┼──────────────┤
│ pi       │ p-value for test i                 │ Evidence     │
│          │ ∈ [0,1]                            │ against null │
├──────────┼────────────────────────────────────┼──────────────┤
│ t        │ Threshold for rejection            │ Cutoff value │
│          │ Reject if pi ≤ t                   │              │
├──────────┼────────────────────────────────────┼──────────────┤
│ R        │ Total rejections (# tests rejected)│ How many you │
│          │                                    │ call positive│
├──────────┼────────────────────────────────────┼──────────────┤
│ V        │ False positives (FP)               │ Mistakes     │
│          │ Rejected but null is true          │ (Type I)     │
├──────────┼────────────────────────────────────┼──────────────┤
│ S        │ True positives (TP)                │ Correct      │
│          │ Rejected and alternative is true   │ discoveries  │
│          │ S = R - V                          │              │
├──────────┼────────────────────────────────────┼──────────────┤
│ FDP      │ False Discovery Proportion         │ Fraction of  │
│          │ FDP = V/R                          │ errors       │
├──────────┼────────────────────────────────────┼──────────────┤
│ FDR      │ False Discovery Rate               │ Expected FDP │
│          │ FDR = E[FDP]                       │              │
├──────────┼────────────────────────────────────┼──────────────┤
│ Mi       │ Mirror statistic for test i        │ Test stat    │
│          │                                    │ (symmetric)  │
├──────────┼────────────────────────────────────┼──────────────┤
│ N        │ Negative rejections in Mirror      │ # with Mi<-t │
│          │ N = #{i : Mi < -t}                │              │
├──────────┼────────────────────────────────────┼──────────────┤
│ #{·}     │ "Number of" or "count"             │ How many     │
│          │ #{i : condition} = count items     │ satisfy      │
│          │ where condition is true            │ condition    │
└──────────┴────────────────────────────────────┴──────────────┘
```

### The Multiple Testing Problem

```
PROBLEM: Testing many hypotheses → many false positives by chance!

Example: Drug screening (1000 drugs)
  Truth: 50 drugs work (5%), 950 don't work (95%)
  Test at α = 0.05 level (5% false positive rate per test)

  Expected false positives: 950 × 0.05 = 47.5 ≈ 48
  Expected true positives:  50 (if perfect power)

  So ~48/(48+50) = 49% of rejections are FALSE! (WARNING)

More tests → More chances for false positives!
```

### Visual Example

```
Perform m tests:

Test 1: p₁ = 0.03  [Y] Reject (looks significant)
Test 2: p₂ = 0.45  [N] Keep
Test 3: p₃ = 0.01  [Y] Reject (looks significant)
Test 4: p₄ = 0.89  [N] Keep
Test 5: p₅ = 0.04  [Y] Reject (looks significant)
...
Test m: pₘ = 0.20  [N] Keep

Which rejections are real? Which are false positives?
Need to control False Discovery Rate!
```

### Confusion Matrix

```
                        TRUTH
                   NULL         ALT
                 (no effect)  (real effect)
              ┌───────────────────────────┐
              │                           │
   Reject     │    V          S           │  R
   (call      │   (FP)       (TP)         │  (total
   positive)  │   [BAD]      [GOOD]       │   rejections)
              │                           │
              ├───────────────────────────┤
              │                           │
   Keep       │    TN         FN          │
   (call      │   [GOOD]     [BAD]       │
   negative)  │                           │
              │                           │
              └───────────────────────────┘
                m₀          m₁
              (true       (true
               nulls)      alts)

KEY:
  V = False Positives (Type I errors) - BAD
  S = True Positives (correct discoveries) - GOOD
  TN = True Negatives (correct non-rejections) - GOOD
  FN = False Negatives (Type II errors) - BAD

  R = V + S (total rejections)
  m = m₀ + m₁ (total tests)
```

### Key Metrics

```
┌────────────────────────────────────────────────┐
│                                                │
│  FDP = V/R                                     │
│        │  │                                    │
│        │  └─ Total rejections (what you see)  │
│        └─ False Positives (hidden!)           │
│                                                │
│  "Fraction of rejections that are false"      │
│                                                │
│  Problem: V is unknown! (we don't know truth) │
│                                                │
├────────────────────────────────────────────────┤
│                                                │
│  FDR = E[FDP]                                  │
│                                                │
│  "Expected fraction of false discoveries"      │
│                                                │
│  Goal: Control FDR ≤ α (e.g., α = 0.1 = 10%)  │
│                                                │
└────────────────────────────────────────────────┘
```

---

## METHOD A: BENJAMINI-HOCHBERG (BH)

### Key Idea

```
Under NULL hypothesis: p-value ~ Uniform[0,1]

Use this to estimate # false positives
```

### What is a p-value?

```
p-value = Probability of seeing data this extreme (or more) if null is true

p = 0.01: Very strong evidence against null (1% chance)
p = 0.50: Weak evidence (50% chance - could be random)
p = 0.95: No evidence against null

Small p → Reject null
Large p → Keep null

Typical threshold: α = 0.05 (reject if p ≤ 0.05)
```

### Uniform Distribution Under Null

```
KEY FACT: If null hypothesis is true, p-value ~ Uniform[0,1]

    Density
      │ ▓▓▓▓▓▓▓▓▓▓
    1 │ ▓▓▓▓▓▓▓▓▓▓  Equal probability everywhere
      │ ▓▓▓▓▓▓▓▓▓▓
      │ ▓▓▓▓▓▓▓▓▓▓  P(p ≤ 0.1) = 0.1
      └────────────→ p    P(p ≤ 0.5) = 0.5
      0            1      P(p ≤ t) = t

This means: P(p ≤ t | NULL) = t

For m₀ true nulls: Expected # with p ≤ t is m₀ · t
```

### BH Estimation Logic

**For threshold t:**

```
STEP 1: Count rejections
        R = #{i : pi ≤ t}
        │   │    │   │
        │   │    │   └─ p-value ≤ threshold
        │   │    └─ test i
        │   └─ count how many
        └─ total rejections

STEP 2: Estimate false positives
        E[V] = m₀ · t
               │    │
               │    └─ Probability of p ≤ t under null (uniform!)
               └─ Number of true nulls (unknown)

        If m₀ unknown, use conservative estimate:
        E[V] ≈ m · t  (assumes all are null)

STEP 3: Estimate FDP
        FDP̂ = E[V] / R = (m · t) / R
```

### BH Formula

```
┌──────────────────────────────────────┐
│                                      │
│  FDP̂ = (m · t) / R                  │
│        │   │    │                    │
│        │   │    └─ # rejections      │
│        │   └─ threshold              │
│        └─ total tests                │
│                                      │
│  In words:                           │
│  FDP̂ = (# tests × threshold)        │
│        ─────────────────────        │
│        # rejections                  │
│                                      │
└──────────────────────────────────────┘

Example:
  m = 1000 tests
  t = 0.05
  R = 50 rejections

  FDP̂ = (1000 × 0.05) / 50 = 50/50 = 1.0 = 100%

  Interpretation: All rejections might be false! Don't trust them.
```

### Visual Example

```
Sort p-values: p₍₁₎ ≤ p₍₂₎ ≤ ... ≤ p₍ₘ₎

   p-value
      │
    1 │                               ●
      │                         ●
      │                    ●
  0.5 │              ●
      │         ●
    t │─────●────●──●                    ← threshold t
      │   ●
      │ ●
    0 └───────────────────────────→ rank
      0   1   2   3   4   5   ... m
          └─ R = 3 rejections ─┘

At threshold t:
  R = 3 rejections (p₍₁₎, p₍₂₎, p₍₃₎ ≤ t)
  Expected FP: m · t
  FDP̂ = (m · t) / 3

If m = 100, t = 0.05:
  FDP̂ = (100 × 0.05) / 3 = 5/3 ≈ 1.67

If FDP̂ > 1, set FDP̂ = 1 (can't have >100% false!)
```

---

## METHOD B: MIRROR STATISTICS

### Key Idea

```
Under NULL hypothesis: Statistic M ~ Symmetric around 0

Use negative tail to estimate false positives in positive tail
```

### What are Mirror Statistics?

```
MIRROR STATISTIC: A test statistic with special property

Property: Under NULL, M is symmetric around 0
          P(M > t | NULL) = P(M < -t | NULL)

Examples:
  • t-statistic (mean / SE)
  • z-score
  • Correlation coefficient

         Symmetric around 0
              ↓
    Density
      │      ╱▔▔▔╲
      │     ╱     ╲
      │    ╱       ╲    Under NULL:
      │   ╱    0    ╲   Left = Right
      │  ╱           ╲
      └──────────────────→ M
        -3   0   3

If alternative exists → Pushes distribution right →
```

### Symmetry Under Null

```
KEY FACT: P(M > t | NULL) = P(M < -t | NULL)

    Density (NULL)
      │      ╱▔▔▔╲
      │     ╱     ╲
      │    ╱       ╲
      │   ╱    0    ╲
      │  ╱           ╲
      └──────────────────→ M
        │         0         │
       -t                   t
      ◄──►                ◄──►
   P(M<-t)              P(M>t)
      │                    │
      └──── EQUAL ─────────┘

Area in left tail = Area in right tail
```

### Mirror Estimation Logic

```
IDEA: Use left tail (M < -t) to estimate false positives in right tail (M > t)

STEP 1: Count positive rejections
        R = #{i : Mi > t}
        │   │    │    │
        │   │    │    └─ above threshold
        │   │    └─ statistic for test i
        │   └─ count how many
        └─ total rejections

        These could be TRUE signals or FALSE positives!

STEP 2: Count negative rejections
        N = #{i : Mi < -t}
        │   │    │     │
        │   │    │     └─ below negative threshold
        │   │    └─ statistic for test i
        │   └─ count how many
        └─ negative rejections

        These are ALL false positives! (no true signal makes M < -t)

STEP 3: Use symmetry
        By symmetry: # false positives in M > t ≈ # in M < -t
        So: V ≈ N

STEP 4: Estimate FDP
        FDP̂ = N / R
```

### Mirror Formula

```
┌──────────────────────────────────────┐
│                                      │
│  FDP̂ = N / R                        │
│        │   │                         │
│        │   └─ Positive rejections    │
│        └─ Negative rejections        │
│                                      │
│      = #{i : Mi < -t}                │
│        ───────────────               │
│        #{i : Mi > t}                 │
│                                      │
└──────────────────────────────────────┘

Example:
  R = 15 (Mi > t)
  N = 5 (Mi < -t)

  FDP̂ = 5/15 = 0.33 = 33%

  Interpretation: About 1/3 of positive rejections are false
```

### Visual Example

```
    Distribution of M statistics (with signals)

        Count
          │
          │     ╱▔╲      ╱▔╲
          │    ╱   ╲    ╱   ╲      ← Signals push right
          │   ╱NULL ╲  ╱ SIG ╲
          │  ╱       ╲╱       ╲
          │ ╱                  ╲
          └───────────────────────→ M
           │         0         │
         -t                    t
         ◄─►                  ◄──►
       N = 5                 R = 15
    (all FP)            (TP + FP)

By symmetry of NULL component:
  FP in M > t ≈ N = 5

So:
  FDP̂ = 5/15 ≈ 0.33

Interpretation:
  • 15 total rejections
  • ~5 are false positives (33%)
  • ~10 are true positives (67%)
```

### Why Negative Side = All False Positives?

```
REASONING:

TRUE signals increase M → Push to the right (+)
NULL signals (noise) → Symmetric around 0

So:
  M > t  can be: TRUE signal (pushed right) OR NULL (random)
  M < -t can be: Only NULL (no signal pushes left)

Therefore:
  All M < -t are false positives!

By symmetry:
  # false positives in M > t ≈ # in M < -t
```

---

## COMPARISON: BH vs MIRROR

```
┌─────────────────┬────────────────────┬──────────────────┐
│ Feature         │ Benjamini-Hochberg │ Mirror Statistics│
├─────────────────┼────────────────────┼──────────────────┤
│ Input           │ p-values           │ Test statistics  │
│                 │ (pi ∈ [0,1])       │ (Mi ∈ ℝ)         │
├─────────────────┼────────────────────┼──────────────────┤
│ NULL Property   │ Uniform[0,1]       │ Symmetric at 0   │
│                 │ P(p≤t|NULL) = t    │ P(M>t) = P(M<-t) │
├─────────────────┼────────────────────┼──────────────────┤
│ FP Estimate     │ m · t              │ #{Mi < -t}       │
│                 │ (formula-based)    │ (data-driven)    │
├─────────────────┼────────────────────┼──────────────────┤
│ FDP Formula     │ (m·t) / R          │ N / R            │
├─────────────────┼────────────────────┼──────────────────┤
│ Assumption      │ Independence       │ Exchangeability  │
│                 │ (tests independent)│ (under null)     │
├─────────────────┼────────────────────┼──────────────────┤
│ Advantage       │ Standard, proven   │ Adaptive, uses   │
│                 │ Well-studied       │ actual data      │
├─────────────────┼────────────────────┼──────────────────┤
│ Disadvantage    │ Conservative       │ Needs symmetric  │
│                 │ (uses m not m₀)    │ statistics       │
└─────────────────┴────────────────────┴──────────────────┘
```

---

## QUICK REFERENCE: ALL FORMULAS

### Part 1: Bias-Variance
```
E[(y - ŷf)²] = σ² + (f* - E[ŷf])² + E[(E[ŷf] - ŷf)²]
               └──┘   └──────────┘   └────────────┘
             Noise      Bias²          Variance

WHERE:
  y = f*(x) + δ              (true model)
  E[δ] = 0, E[δ²] = σ²      (noise assumptions)
```

### Part 2: K-NN
```
Prediction:  ŷfK(x) = (1/K) Σ yℓ
                            ℓ∈Kx

Expectation: E[ŷfK] = (1/K) Σ f*(xℓ)
                             ℓ∈Kx

Bias:        Bias = f*(x) - E[ŷfK] = f*(x) - (1/K)Σf*(xℓ)
                                                    ℓ∈Kx

Variance:    Var[ŷfK] = σ²/K

Effect:      ↑K → ↑Bias, ↓Variance
```

### Part 3: Ridge with SVD
```
SVD:         X = UDV^T

Ridge:       β̂ridge = Σ (dⱼ/(dⱼ² + λ)) uⱼ^T Y vⱼ
                      j=1

Shrinkage:   Weight j = dⱼ/(dⱼ² + λ) ∈ (0, 1/dⱼ]

OLS:         β̂OLS = Σ (1/dⱼ) uⱼ^T Y vⱼ  (set λ=0)
                    j=1
```

### Part 4: Equal Singular Values
```
Condition:   d₁ = d₂ = ... = dr = d

OLS:         β̂OLS = (1/d) V U^T Y

Ridge:       β̂ridge = (d/(d²+λ)) V U^T Y
                     = (d²/(d²+λ)) β̂OLS

Shrinkage:   s = d²/(d²+λ) < 1

Direction:   Same as OLS, just scaled by s
```

### Part 5: Multiple Testing
```
Metrics:
  FDP = V/R  (false discoveries / total rejections)
  FDR = E[FDP]

Method A - Benjamini-Hochberg:
  R = #{pi ≤ t}              (count rejections)
  FDP̂ = (m·t) / R            (estimate FDP)

Method B - Mirror Statistics:
  R = #{Mi > t}              (positive rejections)
  N = #{Mi < -t}             (negative rejections)
  FDP̂ = N / R                (estimate FDP)
```

---

## PROBLEM-SOLVING CHECKLIST

### For Derivations:
```
□ Identify goal (bias, variance, estimator)
□ Write starting formula
□ Substitute known relationships
  • y = f* + δ
  • E[δ] = 0
  • E[δ²] = σ²
□ Apply expectation rules
  • E[a + b] = E[a] + E[b]
  • E[constant] = constant
  • E[constant · X] = constant · E[X]
□ Apply variance rules
  • Var[aX] = a² Var[X]
  • Var[X + Y] = Var[X] + Var[Y] (if independent)
□ Use orthogonality when given SVD
  • U^T U = I
  • V^T V = I
□ Check final form makes sense
```

### For Multiple Testing:
```
□ Identify method (BH or Mirror)
□ Count rejections R
□ Estimate false positives
  • BH: m · t
  • Mirror: N
□ Compute FDP̂ = FP / R
□ Interpret (% of rejections that are false)
```

---

## COMMON MISTAKES TO AVOID

```
[X] Forgetting that E[δ] = 0
[X] Not using independence for variance of sums
[X] Confusing 1/dⱼ (OLS) with dⱼ/(dⱼ²+λ) (Ridge)
[X] Thinking Ridge changes direction (only in general case!)
[X] Using FP = m (should be m·t or N)
[X] Forgetting that FDP = V/R (not V/m)
```

---

**END OF ANSWER SHEET**

*This sheet covers all concepts needed to solve ANY problem involving these topics, regardless of variable names or specific values. Follow the templates step-by-step!*
