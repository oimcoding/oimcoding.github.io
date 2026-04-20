# Tutorial: Hierarchical Bayesian Bandits (HierTS)

**Paper:** Hong, Kveton, Zaheer, Ghavamzadeh (2022). *Hierarchical Bayesian Bandits.* AISTATS 2022, PMLR vol. 151.

---

## 1. The problem in one picture

```
                    ┌─────────────────────────────────┐
                    │  The agent faces MANY tasks.    │
                    │  Each task is a mini-bandit.    │
                    └────────────┬────────────────────┘
                                 │
         ┌───────────────────────┼───────────────────────┐
         ▼                       ▼                       ▼
    ┌─────────┐             ┌─────────┐             ┌─────────┐
    │ Task 1  │             │ Task 2  │    ....     │ Task m  │
    │ K arms  │             │ K arms  │             │ K arms  │
    └─────────┘             └─────────┘             └─────────┘

    All tasks are SIMILAR (drawn from a common family).
    Learning about task 1 should help task 2.
```

**Key idea:** Treat the m tasks as related, not independent. Learn a shared "hyperparameter" that links them.

---

## 2. Bandit basics (for readers new to bandits)

### 2.1 What is an arm?

| Term | Meaning |
|---|---|
| Arm `k` | A choice the agent can make |
| Reward `r` | The number observed after pulling |
| Mean `θ_k` | The true average reward (unknown to agent) |
| Pull | One trial of one arm |

### 2.2 Thompson Sampling (TS), the starting algorithm

Think of TS as "guess, play, update."

```
┌──────────────────────────────────────────────┐
│ 1. For each arm k, keep a BELIEF            │
│    about its true mean θ_k.                 │
│    (A probability distribution)             │
├──────────────────────────────────────────────┤
│ 2. To pick an arm:                          │
│    a. SAMPLE a guess from each arm's belief │
│    b. PULL the arm with the highest guess   │
├──────────────────────────────────────────────┤
│ 3. Observe reward. UPDATE that arm's belief │
│    (posterior update).                      │
└──────────────────────────────────────────────┘
```

**Why it works:**

- Arms with WIDE belief (high uncertainty) sometimes sample a huge guess → get explored.
- Arms with TIGHT belief (high confidence) sample near the true value → get exploited.
- Over time, beliefs tighten around true means; exploration drops naturally.

### 2.3 The Beta-Bernoulli pair

If rewards are 0 or 1 (binary), the standard belief is a **Beta distribution**.

```
Beta(α, β)
   α: count of 1s observed + 1  (starts at 1)
   β: count of 0s observed + 1  (starts at 1)

Beta(1, 1) = flat prior (all means equally likely)
Beta(7, 3) = "I've seen 6 wins and 2 losses, so mean ≈ 0.7"
```

After observing reward `r ∈ {0, 1}`:

```
α ← α + r
β ← β + (1 − r)
```

This is conjugate: the posterior stays a Beta distribution. Very efficient.

### 2.4 Fractional Beta updates (for continuous rewards in [0, 1])

If reward `r ∈ [0, 1]` (not binary), use the fractional version:

```
α ← α + r
β ← β + (1 − r)
```

Same formula, but `r` is any real number in [0, 1]. The posterior is still well-behaved.

---

## 3. What this paper adds

### 3.1 The hierarchical model (Paper §2, Figure 1)

```
                     ┌──────────┐
                     │   μ*     │   ← GLOBAL hyperparameter
                     │ (hidden) │      (same across all tasks)
                     └────┬─────┘
                          │ sampled from
                          ▼
          ┌───────────────┼───────────────┐
          ▼               ▼               ▼
      ┌───────┐       ┌───────┐       ┌───────┐
      │ θ_1   │       │ θ_2   │       │ θ_m   │   ← TASK parameters
      └───┬───┘       └───┬───┘       └───┬───┘
          │               │               │
          ▼               ▼               ▼
      [rewards]       [rewards]       [rewards]
```

**Symbols:**

- `m`: number of tasks
- `n`: max pulls per task
- `μ*`: hyperparameter, unknown. Acts as a shared prior center.
- `θ_s`: task-specific parameter for task `s`. Unknown. Drawn from a distribution centered at `μ*`.
- `Y_s,t`: reward at round `t` in task `s`. Noisy observation of the action chosen.
- `Q`: hyper-prior (what the agent believes about `μ*` before any data).

**Equation meaning (Paper eq. (2), Gaussian case):**

```
μ*   ~  N(μ_q, Σ_q)       ← agent's initial belief about the hyperparameter
θ_s  ~  N(μ*, Σ_0)        ← each task's param is a noisy copy of μ*
Y_{s,t}  ~  N(A_{s,t}ᵀ θ_s,  σ²)    ← rewards depend on chosen action A
```

### 3.2 Why this helps

```
INDEPENDENT (standard TS)        HIERARCHICAL (HierTS)
─────────────────────────        ──────────────────────
Task 1 learns alone.             Task 1 data also informs μ*.
Task 2 learns alone.             μ* also informs Task 2's prior.
No sharing.                      Early pulls on any task help all
                                 others via the shared μ*.
```

If the tasks really are similar, HierTS is much faster.
If the tasks are unrelated, HierTS is not worse (hyper-posterior just ignores structure).

### 3.3 The HierTS algorithm (Paper §3, Algorithm 1)

```
Each round t:
┌──────────────────────────────────────────────────┐
│ 1. Sample hyperparameter:                        │
│        μ_t  ~  Q_t    (hyper-posterior)         │
├──────────────────────────────────────────────────┤
│ 2. For each active task s in S_t:                │
│    a. Sample task parameter:                     │
│        θ_{s,t}  ~  P(θ | μ_t, history of s)     │
│    b. Take action:                               │
│        A_{s,t}  =  argmax_a  r(a; θ_{s,t})      │
│    c. Observe reward Y_{s,t}                    │
├──────────────────────────────────────────────────┤
│ 3. Update hyper-posterior Q_{t+1} using all     │
│    observations across all tasks.               │
└──────────────────────────────────────────────────┘
```

**Key difference from standard TS:** Step 1 and Step 3 are new. Everything else is TS.

### 3.4 The key result (Paper Theorem 3, simplified)

For `m` tasks over `n` rounds each:

```
Regret  =  O( m · √(c1 · n) )   +   O( √(c2 · m · n) )
           ────────────────         ─────────────────────
           per-task learning         learning shared μ*
```

**Symbols:**

- `c1`: cost constant for learning per-task parameters. Scales with per-task prior width.
- `c2`: cost constant for learning the hyperparameter. Scales with hyper-prior width.

**Interpretation:** As `m` grows, the shared-learning term (c2) becomes negligible compared to the per-task term (c1 · m). That is, more tasks make the shared learning essentially free.

---

## 4. Runnable Python: HierTS vs independent TS

### 4.1 Setup

```python
import numpy as np
import matplotlib.pyplot as plt

rng_data = np.random.default_rng(42)
```

### 4.2 Generate a problem

```python
def make_tasks(num_tasks: int, hyperparam_mean: float, task_spread: float,
               rng) -> np.ndarray:
    """
    Build a set of related tasks.
    Each task is a 1-arm bandit with true mean drawn near hyperparam_mean.

    num_tasks:       m in the paper
    hyperparam_mean: the true μ* (reveal only for simulation)
    task_spread:     how tightly clustered the task means are
    """
    # clip to [0.05, 0.95] to avoid degenerate Beta draws
    task_means = rng.normal(loc=hyperparam_mean, scale=task_spread, size=num_tasks)
    return np.clip(task_means, 0.05, 0.95)

def bernoulli_pull(mean: float, rng) -> int:
    """Return 1 w.p. `mean`, else 0."""
    return int(rng.uniform() < mean)
```

### 4.3 Standard Thompson Sampling (one Beta per task)

```python
def independent_ts(task_means: np.ndarray, n_rounds_per_task: int,
                   rng) -> np.ndarray:
    """
    Run TS independently on each task.
    Returns per-round regret, shape (total_rounds,).
    """
    num_tasks = len(task_means)
    alphas = np.ones(num_tasks)
    betas = np.ones(num_tasks)
    regrets = []

    for round_idx in range(n_rounds_per_task * num_tasks):
        s = round_idx % num_tasks              # round-robin over tasks

        # sample a guess for task s
        theta_sample = rng.beta(alphas[s], betas[s])

        # (in this toy: one arm = pull the task itself; the "action" is trivial)
        # record regret against the true task mean
        observed = bernoulli_pull(task_means[s], rng)
        regrets.append(task_means[s] - observed)

        # posterior update
        alphas[s] += observed
        betas[s]  += (1 - observed)

    return np.array(regrets)
```

### 4.4 HierTS with a 1-D grid over the hyperparameter

```python
def hierts_grid(task_means: np.ndarray, n_rounds_per_task: int,
                rng, grid_size: int = 100, prior_strength: float = 10.0
                ) -> np.ndarray:
    """
    HierTS with a discretized 1-D grid over the shared hyperparameter μ*.

    grid_size:       number of grid points in [0, 1]
    prior_strength:  how strongly the sampled μ* biases each task's prior
                     (called κ in the text; higher = more cross-task sharing)
    """
    num_tasks = len(task_means)
    alphas = np.ones(num_tasks)
    betas = np.ones(num_tasks)

    # hyper-posterior as a log-unnormalized vector over the grid
    grid = np.linspace(0.01, 0.99, grid_size)
    log_hyper_post = np.zeros(grid_size)  # flat prior over μ*

    regrets = []

    for round_idx in range(n_rounds_per_task * num_tasks):
        s = round_idx % num_tasks

        # 1. SAMPLE μ* from hyper-posterior (over the grid)
        probs = np.exp(log_hyper_post - log_hyper_post.max())
        probs /= probs.sum()
        mu_star = rng.choice(grid, p=probs)

        # 2. SAMPLE θ_s from task posterior, biased by μ*
        #    Use Beta(α + κ·μ*, β + κ·(1-μ*)) as the effective posterior.
        eff_alpha = alphas[s] + prior_strength * mu_star
        eff_beta  = betas[s]  + prior_strength * (1 - mu_star)
        theta_sample = rng.beta(eff_alpha, eff_beta)

        # 3. PULL and observe
        observed = bernoulli_pull(task_means[s], rng)
        regrets.append(task_means[s] - observed)

        # 4. UPDATE task posterior
        alphas[s] += observed
        betas[s]  += (1 - observed)

        # 5. UPDATE hyper-posterior
        #    Likelihood of this reward at each grid value of μ*:
        #    Approximate: P(reward | μ*) = Bernoulli(μ*)  (treat μ* as the
        #    task-population mean; valid when all tasks cluster tightly).
        if observed == 1:
            log_hyper_post = log_hyper_post + np.log(grid + 1e-9)
        else:
            log_hyper_post = log_hyper_post + np.log(1 - grid + 1e-9)

        # normalize numerically so logs don't blow up
        log_hyper_post -= log_hyper_post.max()

    return np.array(regrets)
```

### 4.5 Run the comparison

```python
# Scenario: 10 tasks, true means tightly clustered around 0.3
task_means = make_tasks(num_tasks=10, hyperparam_mean=0.3,
                        task_spread=0.04, rng=rng_data)

trials = 30
rounds_per_task = 200

indep_curves = []
hier_curves  = []

for trial in range(trials):
    r = np.random.default_rng(2000 + trial)
    reg_indep = independent_ts(task_means, rounds_per_task, r)

    r = np.random.default_rng(2000 + trial)  # same seed so noise is controlled
    reg_hier = hierts_grid(task_means, rounds_per_task, r,
                           grid_size=100, prior_strength=10.0)

    indep_curves.append(np.cumsum(reg_indep))
    hier_curves.append(np.cumsum(reg_hier))

indep_curves = np.array(indep_curves)
hier_curves  = np.array(hier_curves)

# Summary
print(f"Final cumulative regret (mean ± std across {trials} trials)")
print(f"  Independent TS: {indep_curves[:, -1].mean():.1f} ± "
      f"{indep_curves[:, -1].std():.1f}")
print(f"  HierTS (grid):  {hier_curves[:, -1].mean():.1f} ± "
      f"{hier_curves[:, -1].std():.1f}")
```

### 4.6 Expected result

If tasks are tightly clustered (`task_spread` = 0.04): **HierTS wins by ~20-40%**.

If you rerun with `task_spread = 0.3` (spread-out tasks): HierTS wins less, sometimes ties.

This matches the paper's claim: structure pays off when structure exists.

### 4.7 Plot (optional)

```python
rounds = np.arange(indep_curves.shape[1])
plt.figure(figsize=(8, 5))
plt.plot(rounds, indep_curves.mean(axis=0), label="Independent TS")
plt.plot(rounds, hier_curves.mean(axis=0),  label="HierTS (grid)")
plt.fill_between(rounds,
                 indep_curves.mean(axis=0) - indep_curves.std(axis=0),
                 indep_curves.mean(axis=0) + indep_curves.std(axis=0),
                 alpha=0.2)
plt.fill_between(rounds,
                 hier_curves.mean(axis=0) - hier_curves.std(axis=0),
                 hier_curves.mean(axis=0) + hier_curves.std(axis=0),
                 alpha=0.2)
plt.xlabel("Round")
plt.ylabel("Cumulative regret")
plt.legend()
plt.title("HierTS beats independent TS when tasks are similar")
plt.show()
```

---

## 5. How to read the paper

### 5.1 Reading order (recommended)

| Step | Section | Focus |
|---|---|---|
| 1 | Abstract + §1 Intro | Setting |
| 2 | §2 Setting (with Figure 1) | Graphical model; the meaning of μ*, θ_s |
| 3 | §3 Algorithm (Algorithm 1) | The 3-step HierTS loop |
| 4 | §4.1 (K-armed Gaussian bandit) | Closed-form posteriors to build intuition |
| 5 | §6.1 Theorem 3 | The main regret bound (just the statement) |
| 6 | §7 Experiments | Empirical comparison to OracleTS and flat TS |

### 5.2 Symbols cheat sheet

| Symbol | Meaning |
|---|---|
| `m` | Number of tasks |
| `n` | Max pulls per task |
| `μ*` | Shared hyperparameter (true, hidden) |
| `μ_q, Σ_q` | Mean and covariance of hyper-prior |
| `θ_s` | Task `s`'s parameter (true, hidden) |
| `Σ_0` | Spread of task parameters around μ* |
| `A_{s,t}` | Action chosen for task `s` at round `t` |
| `Y_{s,t}` | Reward observed |
| `S_t` | Set of active tasks at round `t` |
| `σ²` | Reward noise variance |
| `Q_t` | Hyper-posterior at time `t` (belief over μ*) |
| `H_t` | History of all (action, reward) pairs so far |

### 5.3 Things you can skip on first pass

- Section 5 (proof techniques — total variance decomposition)
- Section 6.3 (concurrent regret bound)
- Section 4.2 (linear bandit version)
- Appendices A-E (all proofs)

---

## 6. Connection to your pipeline

The paper treats "tasks" as related problems sharing a hyperparameter. Your pipeline has exactly this structure:

| Paper concept | Your pipeline |
|---|---|
| Task `s` | Harm category `c` (6 categories total) |
| Task parameter `θ_s` | True jailbreak rate of category `c` |
| Hyperparameter `μ*` | Overall robustness of the target bot |
| `m` (number of tasks) | 6 |
| `n` (pulls per task) | Conversations per category |
| Bayes regret bound | Token savings from cross-category learning |

**When to adopt:** when the 6 categories behave similarly under the target bot (the bot is uniformly robust or uniformly weak). Verify via a calibration pilot: if mean reward per category clusters tightly (low variance of means), HierTS helps.

**When to skip:** when categories are very different (some nearly always break, others never do). The shared hyperparameter then over-smooths and hurts accuracy.

---

## 7. Next-step exercises

1. Run the code above. Vary `task_spread` from 0.02 to 0.3. Plot when HierTS loses its advantage.
2. Change `prior_strength` from 10 to 1, 100. Discuss what each extreme means.
3. Replace the 1-D grid with a Gaussian approximation on `logit(μ*)`. Compare runtime and accuracy.
4. Extend to the contextual case: each pull returns a reward conditional on a feature vector. This is §4.2 of the paper.

---

## 8. Glossary

| Word | Meaning |
|---|---|
| Bayesian model | Framework where beliefs are expressed as probability distributions. |
| Beta distribution | Standard prior/posterior for a probability in [0, 1]. |
| Bayes regret | Regret averaged over the prior (a weaker but practical metric). |
| Closed-form posterior | A posterior you can write down exactly (no sampling needed). |
| Conjugate prior | A prior that keeps the same family after a posterior update. |
| Gaussian distribution | Normal distribution; used for continuous rewards. |
| Graphical model | Picture of how variables depend on each other. |
| Hierarchy | Structure where one variable (hyper) controls others (tasks). |
| Hyper-posterior | Belief about the hyperparameter given all data so far. |
| Hyper-prior | Initial belief about the hyperparameter. |
| Posterior | Updated belief after observing data. |
| Prior | Initial belief before any data. |
| Regret | How much worse the agent did than the best possible choice. |
| Thompson Sampling | Bayesian bandit algorithm based on sampling beliefs. |
