# LLM-Enhanced Multi-Armed Bandits: Core Methods Explained

## What is a Multi-Armed Bandit (MAB)?

### The Slot Machine Analogy

```
┌─────────────────────────────────────────────────────┐
│                                                     │
│    Machine 1      Machine 2      Machine 3         │
│       ▼              ▼              ▼               │
│     ┌───┐          ┌───┐          ┌───┐            │
│     │ 🎰│          │ 🎰│          │ 🎰│            │
│     └───┘          └───┘          └───┘            │
│                                                     │
│    Unknown         Unknown         Unknown          │
│    reward          reward           reward          │
│    average         average          average         │
│                                                     │
└─────────────────────────────────────────────────────┘

Goal: Play machines to maximize total reward over time
Challenge: Don't know which machine is best
Strategy needed: Balance trying new machines (exploration)
                 vs. using the best-known machine (exploitation)
```

### The Problem in Real Applications

**Example 1: Recommending articles to users**
- Each article = one arm/machine
- Reward = 1 if user clicks, 0 if they don't
- Goal: Show articles users want to read

**Example 2: Choosing ads to display**
- Each ad = one arm/machine
- Reward = money earned from ad
- Goal: Maximize advertising revenue

---

## The Traditional Approach: Direct LLM Arm Selection

### How It Works

```
┌──────────────────────────────────────────────────────┐
│  Step 1: Show LLM the history                        │
│  ┌────────────────────────────────────┐              │
│  │ Previously tried:                  │              │
│  │ - Article A → User clicked         │              │
│  │ - Article B → User didn't click    │              │
│  │ - Article C → User clicked         │              │
│  └────────────────────────────────────┘              │
│                    ↓                                  │
│  Step 2: Ask LLM to choose next arm                  │
│  ┌────────────────────────────────────┐              │
│  │ "Which article should we show      │              │
│  │  next: A, B, C, or D?"             │              │
│  └────────────────────────────────────┘              │
│                    ↓                                  │
│  Step 3: LLM responds                                │
│  ┌────────────────────────────────────┐              │
│  │ "Show Article A"                   │              │
│  └────────────────────────────────────┘              │
└──────────────────────────────────────────────────────┘
```

### The Problem

**LLMs struggle to explore efficiently.**

They often:
- Pick the same option repeatedly
- Fail to try new options enough
- Don't balance exploration vs. exploitation well

---

## The New Approach: LLM-Enhanced Classical Algorithms

### Key Insight

```
┌─────────────────────────────────────────────────────┐
│                                                     │
│  OLD WAY:  LLM does everything                     │
│            ├─ Decides which arm to pull            │
│            └─ Manages exploration vs exploitation  │
│                                                     │
│            ↓ Often performs poorly                 │
│                                                     │
├─────────────────────────────────────────────────────┤
│                                                     │
│  NEW WAY:  Divide responsibilities                 │
│                                                     │
│            Classical Algorithm                      │
│            ├─ Manages exploration vs exploitation  │
│            └─ Decides which arm to pull            │
│                                                     │
│            LLM                                      │
│            └─ Predicts rewards (what it's good at) │
│                                                     │
│            ↓ Works much better                     │
│                                                     │
└─────────────────────────────────────────────────────┘
```

LLMs are excellent at prediction but poor at exploration.
Use them for what they do best.

---

## Algorithm 1: TS-LLM (Thompson Sampling with LLM)

### Overview

TS-LLM uses random sampling to balance exploration and exploitation.

### Visual Flow

```
┌───────────────────────────────────────────────────────┐
│ ITERATION t                                           │
├───────────────────────────────────────────────────────┤
│                                                       │
│  Step 1: For each arm, ask LLM to predict reward     │
│                                                       │
│  ┌─────────────────────────────────────────────┐     │
│  │ History shown to LLM:                       │     │
│  │  Arm 1 features: [0.2, 0.5, 0.8, 0.1]      │     │
│  │         reward: 0.7                         │     │
│  │  Arm 2 features: [0.1, 0.3, 0.2, 0.9]      │     │
│  │         reward: 0.3                         │     │
│  │  ...                                        │     │
│  └─────────────────────────────────────────────┘     │
│                      ↓                                │
│  ┌─────────────────────────────────────────────┐     │
│  │ Query for Arm 1:                            │     │
│  │  "Given features [0.2, 0.5, 0.8, 0.1],     │     │
│  │   predict the reward"                       │     │
│  │                                             │     │
│  │ LLM prediction: 0.68 (with randomness)     │     │
│  └─────────────────────────────────────────────┘     │
│                      ↓                                │
│  Repeat for all arms → Get predicted rewards         │
│                                                       │
│  Step 2: Pull arm with highest predicted reward      │
│                                                       │
│  Step 3: Observe actual reward                       │
│                                                       │
│  Step 4: Add to history, repeat                      │
│                                                       │
└───────────────────────────────────────────────────────┘
```

### Temperature Control

**Temperature** = How random the LLM's predictions are

```
High temperature (1.5)          Low temperature (0.1)
     ↓                                  ↓
More random                       Less random
More exploration                  More exploitation


┌─────────────────────────────────────────────────┐
│  Early iterations                               │
│  Use HIGH temperature                           │
│  → Predictions vary more                        │
│  → Try different arms                           │
│  → EXPLORATION                                  │
└─────────────────────────────────────────────────┘
                    ↓
                  Time
                    ↓
┌─────────────────────────────────────────────────┐
│  Later iterations                               │
│  Use LOW temperature                            │
│  → Predictions more consistent                  │
│  → Focus on best arms                           │
│  → EXPLOITATION                                 │
└─────────────────────────────────────────────────┘
```

The temperature **decays** (decreases) over time.
This creates automatic transition from exploration to exploitation.

### Key Formula

For each arm i at time t, the LLM predicts:

**r̂(t,i) = LLM(D(t-1), x(i))**

Where:
- **r̂(t,i)** = predicted reward for arm i at time t
- **LLM** = the large language model prediction function
- **D(t-1)** = history of all past arm selections and rewards up to time t-1
- **x(i)** = feature vector describing arm i (e.g., article keywords, ad properties)

The arm selected is:

**i(t) = argmax(i) r̂(t,i)**

Meaning: choose the arm i that has the maximum predicted reward.

---

## Algorithm 2: RO-LLM (Regression Oracle-based MAB with LLM)

### Overview

RO-LLM uses deterministic predictions plus explicit randomization for exploration.

### Visual Flow

```
┌───────────────────────────────────────────────────────┐
│ ITERATION t                                           │
├───────────────────────────────────────────────────────┤
│                                                       │
│  Step 1: LLM predicts loss for each arm              │
│          (temperature = 0, no randomness)             │
│                                                       │
│  ┌─────────────────────────────────────────────┐     │
│  │ Arm 1 predicted loss: 0.32                 │     │
│  │ Arm 2 predicted loss: 0.15  ← Best        │     │
│  │ Arm 3 predicted loss: 0.41                 │     │
│  │ Arm 4 predicted loss: 0.28                 │     │
│  └─────────────────────────────────────────────┘     │
│                      ↓                                │
│  Step 2: Find arm with smallest predicted loss       │
│          Call it j(t)                                │
│                                                       │
│  Step 3: Calculate sampling probability for each arm │
│                                                       │
│  ┌─────────────────────────────────────────────┐     │
│  │ For arms NOT equal to j(t):                │     │
│  │                                             │     │
│  │   p(t,i) = 1 / [μ + γ(loss(i) - loss(j))] │     │
│  │                                             │     │
│  │ Higher loss → Lower probability            │     │
│  │                                             │     │
│  │ For arm j(t):                              │     │
│  │   p(t,j) = 1 - sum of all other p's       │     │
│  └─────────────────────────────────────────────┘     │
│                      ↓                                │
│  Step 4: Randomly sample arm based on p(t,i)        │
│          (explicit exploration)                       │
│                                                       │
│  Step 5: Observe loss, add to history                │
│                                                       │
└───────────────────────────────────────────────────────┘
```

### Key Difference from TS-LLM

```
┌─────────────────────────────────────────────────────┐
│ TS-LLM                                              │
│ ├─ Exploration via LLM randomness (temperature)    │
│ └─ Simple: pick arm with max predicted reward      │
└─────────────────────────────────────────────────────┘

┌─────────────────────────────────────────────────────┐
│ RO-LLM                                              │
│ ├─ LLM makes deterministic predictions             │
│ └─ Exploration via probability distribution        │
└─────────────────────────────────────────────────────┘
```

### Key Formula

Sampling probability for arm i (when i ≠ j(t)):

**p(t,i) = 1 / [μ + γ(ℓ̂(t,i) - ℓ̂(t,j))]**

Where:
- **p(t,i)** = probability of selecting arm i at time t
- **μ** = small constant to prevent division by zero
- **γ** = exploration parameter (larger γ = more exploitation)
- **ℓ̂(t,i)** = LLM's predicted loss for arm i at time t
- **ℓ̂(t,j)** = LLM's predicted loss for best arm j(t) at time t
- **j(t)** = arm with smallest predicted loss: j(t) = argmin(i) ℓ̂(t,i)

For the best arm j(t):

**p(t,j) = 1 - Σ(i≠j) p(t,i)**

This ensures all probabilities sum to 1.

---

## Algorithm 3: TS-LLM-DB (Dueling Bandits)

### What Are Dueling Bandits?

Instead of getting a numerical reward, you get preference feedback.

```
Standard MAB:
┌──────────────────────────────────┐
│ Pull Arm A                       │
│ Get reward: 0.7                  │
│ Know exact value                 │
└──────────────────────────────────┘

Dueling Bandits:
┌──────────────────────────────────┐
│ Pull Arm A and Arm B             │
│ Get feedback: A preferred over B │
│ Only know relative preference    │
└──────────────────────────────────┘
```

**Example:** Comparing two website designs
- Show user both designs
- User picks which they prefer
- No numerical score, just preference

### Visual Flow

```
┌───────────────────────────────────────────────────────┐
│ ITERATION t                                           │
├───────────────────────────────────────────────────────┤
│                                                       │
│  Step 1: SELECT FIRST ARM                            │
│                                                       │
│  For each arm i:                                     │
│  ┌─────────────────────────────────────────────┐     │
│  │ Sample N random arms (e.g., N=15)          │     │
│  │ Ask LLM: "Probability arm i beats arm j?"  │     │
│  │ Average across all N samples               │     │
│  │ → Get score r̂(t,i)                        │     │
│  └─────────────────────────────────────────────┘     │
│                      ↓                                │
│  Choose arm with highest score as first arm          │
│  i(t,1) = argmax(i) r̂(t,i)                          │
│                                                       │
│  Step 2: SELECT SECOND ARM                           │
│                                                       │
│  For each remaining arm j:                           │
│  ┌─────────────────────────────────────────────┐     │
│  │ Ask LLM: "Probability arm j beats first    │     │
│  │           arm i(t,1)?"                     │     │
│  │ → Get score p̂(t,j)                        │     │
│  └─────────────────────────────────────────────┘     │
│                      ↓                                │
│  Choose arm with highest score as second arm         │
│  i(t,2) = argmax(j) p̂(t,j)                          │
│                                                       │
│  Step 3: OBSERVE PREFERENCE                          │
│                                                       │
│  ┌─────────────────────────────────────────────┐     │
│  │ Show both arms to user/system               │     │
│  │ Observe: r(t) = 1 if i(t,1) preferred      │     │
│  │              = 0 if i(t,2) preferred        │     │
│  └─────────────────────────────────────────────┘     │
│                                                       │
│  Step 4: Update history with pair and preference     │
│                                                       │
└───────────────────────────────────────────────────────┘
```

### Why Select Two Arms This Way?

```
┌─────────────────────────────────────────────────────┐
│ First arm i(t,1):                                   │
│ ├─ Selected to have highest average win rate       │
│ ├─ GREEDY choice (exploitation)                    │
│ └─ This is the arm we recommend as "best"          │
└─────────────────────────────────────────────────────┘
                         ↓
┌─────────────────────────────────────────────────────┐
│ Second arm i(t,2):                                  │
│ ├─ Selected to likely beat the first arm           │
│ ├─ Encourages trying different arms                │
│ └─ Provides EXPLORATION                            │
└─────────────────────────────────────────────────────┘
```

### Key Formula

**Borda function** (estimated in Step 1):

**f̂(borda)(x(i)) ≈ (1/N) Σ(n=1 to N) P̂(x(i) ≻ x(j(n)))**

Where:
- **f̂(borda)(x(i))** = estimated Borda score for arm i
- **N** = number of randomly sampled arms for comparison
- **j(n)** = the n-th randomly sampled arm
- **P̂(x(i) ≻ x(j(n)))** = LLM's predicted probability that arm i is preferred over arm j(n)
- **x(i)** = feature vector of arm i

The Borda function measures: "What's the probability this arm beats a random arm?"

**Preference probability** (Bradley-Terry-Luce model):

**P(x(i) ≻ x(j)) = 1 / (1 + e^(-10[f(x(i)) - f(x(j))]))**

Where:
- **P(x(i) ≻ x(j))** = probability arm i is preferred over arm j
- **e** = Euler's number (≈2.718)
- **f(x(i))** = latent reward function value for arm i
- **f(x(j))** = latent reward function value for arm j
- **10** = scaling constant to reduce noise

This formula converts reward differences into preference probabilities.

---

## Comparison Table

```
┌──────────────┬────────────┬────────────┬─────────────┐
│ Algorithm    │ LLM Role   │ LLM Temp   │ Exploration │
├──────────────┼────────────┼────────────┼─────────────┤
│ TS-LLM       │ Predict    │ Decaying   │ Via LLM     │
│              │ rewards    │ (1.5→0.1)  │ randomness  │
├──────────────┼────────────┼────────────┼─────────────┤
│ RO-LLM       │ Predict    │ Fixed      │ Via explicit│
│              │ losses     │ (0)        │ probability │
├──────────────┼────────────┼────────────┼─────────────┤
│ TS-LLM-DB    │ Predict    │ Decaying   │ Via LLM     │
│              │ preferences│ (high→low) │ randomness  │
└──────────────┴────────────┴────────────┴─────────────┘
```

---

## Performance Results Summary

### Synthetic Experiments

```
Cumulative Regret (lower is better)
     │
  50 ├─────────────────────────── Baseline (Direct LLM)
     │                        ╱╱╱╱
     │                    ╱╱╱╱
  30 │                ╱╱╱╱
     │            ╱╱╱╱
     │        ╱╱╱╱       ─────── TS-LLM
  10 │    ╱╱╱╱       ───
     │╱╱╱╱       ───
   0 └────────────────────────────────────
     0        50       100   Iterations
```

**TS-LLM and RO-LLM outperform direct LLM selection consistently.**

### Real-World Text Experiments

**OneShotWikiLinks** (entity recognition - arms have semantic meaning)

```
TS-LLM performance ≈ Baseline performance

Why? LLM can use semantic knowledge to pick good arms directly.
```

**AmazonCat-13K** (item tags - arms are just numbers)

```
Cumulative Reward (higher is better)
     │
  15 │                        ───────── TS-LLM
     │                    ────
     │                ────
  10 │            ────
     │        ────
     │    ────
   5 │────                ╱╱╱╱╱
     │               ╱╱╱╱╱          Baseline
   0 └────────────────────────────────────
     0        50       100   Iterations
```

**TS-LLM significantly outperforms direct LLM selection.**

Why? Arms lack semantic meaning, so exploration is crucial.

---

## Key Takeaway

```
┌─────────────────────────────────────────────────────┐
│                                                     │
│  When to use LLM-enhanced algorithms:              │
│                                                     │
│  ✓ Arms lack clear semantic meaning                │
│  ✓ Need efficient exploration                      │
│  ✓ Want consistent, reliable performance           │
│                                                     │
│  When direct LLM selection works:                  │
│                                                     │
│  ✓ Arms have rich semantic information             │
│  ✓ LLM has strong prior knowledge                  │
│  ✓ Task is more about recognition than learning    │
│                                                     │
└─────────────────────────────────────────────────────┘
```

**Core principle:** Use LLMs for prediction (their strength), not exploration (their weakness).

---

## Glossary

**Arm:** One choice/option in the multi-armed bandit problem

**Exploitation:** Using what you know to get the best reward

**Exploration:** Trying new things to learn more

**Feature vector:** List of numbers describing an arm's properties

**Regret:** Difference between reward from optimal arm and reward from selected arm

**Temperature:** Parameter controlling randomness in LLM outputs

**Loss:** Negative of reward (lower loss = better)

**Preference feedback:** Information about which of two options is better (not how much better)

**Borda score:** Measure of how often an arm beats random other arms

---

*Document created for accessibility - prioritizing visual structure and clarity*
