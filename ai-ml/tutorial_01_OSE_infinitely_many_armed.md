# Tutorial: Adaptive Algorithms for Infinitely Many-Armed Bandits (OSE, PROSE)

**Paper:** Pilliat, E. (2025). *Adaptive Algorithms for Infinitely Many-Armed Bandits: A Unified Framework.* arXiv:2510.27319v2.

**Author:** Emmanuel Pilliat (Univ Rennes, Ensai, CNRS, CREST).

---

## 1. The problem in one picture

```
               ┌────────────────────────────────────┐
               │  ∞ arms indexed 1, 2, 3, ...       │
               │  Each arm has unknown mean reward  │
               └──────────────┬─────────────────────┘
                              │ budget t pulls (small vs ∞)
                              ▼
               ┌────────────────────────────────────┐
               │  Agent pulls one arm per step      │
               │  Agent also RECOMMENDS one arm     │
               │  at every step (not only at end)   │
               └──────────────┬─────────────────────┘
                              │ the recommended arm should have a
                              │ HIGH MEAN REWARD, not necessarily
                              │ the best mean reward overall
                              ▼
                 Goal: recommended arm is in top η_t
                       of all arms, for η_t as small as possible
```

**Key idea:** you cannot try every arm. Instead, guarantee that your recommendation is in the TOP FRACTION of arms, and make that fraction shrink over time.

---

## 2. Bandit basics (for readers with no background)

### 2.1 Terms

| Term | Meaning |
|---|---|
| Arm `a` | One of infinitely many choices |
| Pull | One trial of one arm, costs 1 unit of budget |
| Reward `X` | The numeric outcome of a pull (noisy) |
| Mean reward | The long-run average reward of an arm (unknown) |
| Rank `γ(a)` | A value in [0, 1] telling how good arm `a` is: 0 is best, 1 is worst. Every arm gets a rank sampled uniformly. |
| `λ_η` | The quantile function: the mean reward of the rank-η arm. Nonincreasing in η. |
| Budget `t` | Total pulls used so far (the paper uses this as current time) |
| Recommendation `r̂_t` | The arm you would commit to if forced to stop right now |

### 2.2 Why this differs from classical bandits

```
Classical bandit:                   This paper:
─────────────────                   ───────────
K known arms                        Infinitely many arms
Budget >> K                         Budget < K (can be much less)
Goal: maximize TOTAL reward         Goal: maximize FINAL recommendation
      (cumulative regret)                 reward (simple reward)
Every pull counts                   Only the recommendation counts
```

### 2.3 UCB and LCB (two numbers per arm)

Given `N_{a,t}` = number of pulls of arm `a` by time `t`, and `X̄_{a,t}` = empirical mean of those pulls:

**UCB (Upper Confidence Bound), equation (8) in the paper:**
```
UCB_{a,t} = X̄_{a,t} + sqrt( ζ² β_t / N_{a,t} )
```

**Symbols:**
- `X̄_{a,t}`: sample mean of rewards seen for arm `a`
- `ζ²`: noise variance parameter (sub-Gaussian bound)
- `β_t`: tuning parameter, grows slowly with `t` (paper sets `β_t = 6 log(5t/δ)`)
- `N_{a,t}`: number of pulls of arm `a` so far

**Equation meaning:** "Optimistic estimate of arm `a`'s true mean: sample mean plus a confidence bonus that shrinks as you pull more."

**LCB (Lower Confidence Bound):**
```
LCB_{a,t} = X̄_{a,t} - sqrt( ζ² β_t / N_{a,t} )
```

**Equation meaning:** "Pessimistic estimate: sample mean minus confidence bonus."

### 2.4 Convention for unseen arms

- UCB of an unseen arm = `+∞`
- LCB of an unseen arm = `-∞`

This lets both algorithms naturally prefer new arms (because their UCB is `+∞`) when the scope permits.

---

## 3. The model in equations

### 3.1 Reward model, equation (1) of the paper

```
X_{a, s}  =  λ_{γ(a)}  +  ε_{a, s}
```

**Symbols:**
- `X_{a, s}`: reward observed on the `s`-th pull of arm `a`
- `γ(a)`: rank of arm `a`. Drawn once, independently from Uniform[0, 1]. Small γ = good arm.
- `λ_η`: quantile function, η ∈ (0, 1]. Nonincreasing, right-continuous.
- `ε_{a, s}`: noise, independent across arms and pulls, ζ²-sub-Gaussian with mean 0

**Equation meaning:** Arm `a`'s mean reward is `λ_{γ(a)}`, hidden from the agent. Every pull adds random noise.

### 3.2 The quantile function tells you everything

```
Distribution of arm means is characterized by η → λ_η.
To simulate: draw γ ~ Uniform[0, 1], compute mean = λ_γ.
```

Examples the paper analyzes:

| Distribution | Quantile function `λ_η` | Where it lives |
|---|---|---|
| Bernoulli-type | `u` if η ≤ η_0, else 0 | Only 2 values |
| Beta(1, 1/α) | `1 - η^α` | Bounded on [0, 1] |
| Pareto(1/α, 1) | `η^(-α)` | Unbounded on [1, ∞) |

### 3.3 δ-achievable rank sequence (Definition 1 of the paper)

An algorithm producing recommendations `r̂_t` achieves sequence `(η_t)` with confidence 1-δ if:
```
P(γ(r̂_t) ≤ η_t  for all t ≥ 1)  ≥  1 - δ
```

**Equation meaning:** With high probability, the recommended arm is in the top `η_t` fraction of all arms, for every time step.

---

## 4. The core complexity measure

### 4.1 Rank-corrected inverse squared gap, equation (4)

```
G(ρ, ν)  =  ( ζ² ν ) / ( ρ (λ_ρ - λ_ν)² )   ∨   1/ρ
```

**Symbols:**
- `ρ`, `ν`: two ranks with `ρ < ν` (ρ is the "good" rank, ν is the "bad" rank)
- `λ_ρ - λ_ν`: gap between the good-arm mean and the bad-arm mean
- `ζ²`: noise variance
- `∨`: maximum of the two terms

**Three effects captured by this formula (Paper §2):**

```
G(ρ, ν) has three parts that compete:

  ζ² / (λ_ρ - λ_ν)²    ← "how hard is it to distinguish good from bad?"
                          (smaller gap = more samples needed)

  × ν / ρ              ← "needle-in-a-haystack penalty"
                          (few good arms among many = harder)

  ∨ 1 / ρ              ← "you need at least 1/ρ samples just to see
                          one top-ρ arm in the pool"
```

### 4.2 Sample complexity function, equation (5)

```
S(η)  =  inf_{ρ < η}  sup_{ν ≥ η}  G(ρ, ν)
```

**Symbols:**
- `η`: target rank (you want recommendations at least this good)
- `S(η)`: sample complexity for identifying a top-η arm

**Equation meaning:** "Find the easiest rank `ρ` (within the good ones, `ρ < η`) that is still hard to beat by any worse rank `ν ≥ η`."

### 4.3 Smallest achievable rank at time t, equation (6)

```
η*_t(ψ)  =  inf { η ∈ (0, 1)  :  S(η) ≤ t / ψ }
```

**Symbols:**
- `ψ`: a polylogarithmic-in-t-and-δ factor (paper uses `ψ ≥ 2^30 log³(5t/δ)`)
- `η*_t(ψ)`: the best rank you can guarantee for the recommendation at time `t`

**Equation meaning:** "What is the smallest rank `η` such that `t` samples are enough to identify a top-η arm?"

### 4.4 The main theorem, simplified (Theorem 2.1)

> With probability at least `1 - δ`, OSE's recommendation `r̂_t` satisfies `γ(r̂_t) ≤ η*_t(ψ)`, for every `t ≥ 1`.

**Paper citation:** Theorem 2.1 (page 5).

---

## 5. OSE algorithm (Algorithm 1 of the paper)

### 5.1 Flowchart

```
┌──────────────────────────────────────────────────┐
│  At time step t = 1, 2, 3, ...                   │
├──────────────────────────────────────────────────┤
│                                                  │
│  1. Draw U ~ Uniform(0, 1)                       │
│                                                  │
│  2. Compute exploration scope:                   │
│        Z = floor( t^U )                          │
│     (random, in [1, t])                          │
│                                                  │
│  3. Among arms {1, 2, ..., Z}:                   │
│        pull arm â_t = argmax UCB_{a, t-1}        │
│     Note: unseen arms have UCB = +∞,             │
│     so when Z > (arms seen), a new arm is tried. │
│                                                  │
│  4. Recommend:                                   │
│        r̂_t = argmax over ALL observed arms      │
│              of LCB_{a, t}                       │
│                                                  │
└──────────────────────────────────────────────────┘
```

### 5.2 Why the random scope `Z`?

- Each step, `U ~ Uniform[0, 1]` means `Z = floor(t^U)` has a "bracket" structure.
- Small `Z` (e.g. `Z = 1, 2, 3`): revisit old arms, sample them more.
- Large `Z` (e.g. `Z = t`): explore a new arm (first time it enters the scope).

This implements what Paper §3 calls "bracketing": use many scope sizes in proportion to `1/log(t)` for each bracket.

### 5.3 Why this is called "optimistic" scope exploration

- Pull rule uses UCB (optimistic estimate).
- Scope is random (sometimes wide, sometimes narrow).
- Hence "Optimistic Scope Exploration" = OSE.

### 5.4 Two drawbacks of OSE (Paper §3, pages 6-7)

1. Even clearly bad arms (like arm 1 if it is bad) keep getting pulled because `Z = 1` occurs with probability `1/log(t)` at each step. About `t/log(t)` samples are wasted on arm 1.
2. The random `Z` creates high variance in recommendations and `O(t²)` total computation.

This is why the paper introduces PROSE.

---

## 6. PROSE algorithm (Algorithm 2 of the paper)

### 6.1 What changes versus OSE

| OSE | PROSE |
|---|---|
| Arms ordered by index 1, 2, 3, ... | Arms ranked by LCB (best LCB gets rank 1) |
| Random scope `Z = floor(t^U)` | Deterministic scope cycling through `Z = floor(t^{Q(j/log t)})` with `j = 1, 2, ..., log t` |
| Sort nothing | Maintain a permutation `π` by LCB descending |

### 6.2 Flowchart

```
┌──────────────────────────────────────────────────────────┐
│  Initialize permutation π = identity, j = 1              │
├──────────────────────────────────────────────────────────┤
│  At time step t = 1, 2, 3, ...                          │
│                                                          │
│  1. If j > log(t), reset j = 1                          │
│                                                          │
│  2. Compute scope:                                      │
│        Z = floor( t ^ Q(j / log t) )                    │
│     where Q: [0, 1] → [0, 1] is a chosen quantile fn.   │
│     Default: Q(x) = x   (gives Z ≈ floor(e^j))          │
│                                                          │
│  3. Pull:                                               │
│        â_t = argmax UCB among arms with π(a) ≤ Z        │
│     (the top-Z arms by current LCB ranking)             │
│                                                          │
│  4. Update π: re-sort arms so the arm with highest      │
│     LCB has π-rank 1, next highest has π-rank 2, etc.   │
│                                                          │
│  5. Recommend: r̂_t = arm with π-rank 1                  │
│                                                          │
│  6. j = j + 1                                           │
└──────────────────────────────────────────────────────────┘
```

### 6.3 Choice of the quantile function `Q`

- `Q(x) = x` (Uniform): `Z ≈ floor(e^j)`. Recommended default for large arm counts.
- `Q(x) = x^(1/γ)` with `γ > 1`: Beta(γ, 1) quantile. More aggressive exploration (larger scopes).
- `γ → ∞`: PROSE becomes classical UCB (all arms always in scope).

Paper Figure 1 shows how `γ` concentrates `Q` toward 1.

### 6.4 Why it works better empirically (Paper §5)

- Less waste on bad arms: ranking by LCB means weak arms drop to the bottom of the ranking and are rarely in scope.
- Deterministic `Z` reduces variance across runs.
- With optimized data structures, amortized complexity is `O(log² t)` per step (Paper §3 end).

---

## 7. Runnable Python: OSE and PROSE

### 7.1 Setup and arm simulation

```python
import math
import numpy as np

def make_quantile_function(kind: str, alpha: float = 1.0):
    """
    Build the paper's quantile function λ_η.

    kind='beta':   λ_η = 1 - η^α    (bounded on [0, 1])
    kind='pareto': λ_η = η^(-α)      (unbounded on [1, ∞))

    Returns a callable mapping η ∈ (0, 1] to the mean reward.
    """
    if kind == "beta":
        return lambda eta: 1.0 - eta**alpha
    if kind == "pareto":
        return lambda eta: eta**(-alpha)
    raise ValueError(f"unknown kind: {kind}")


class ArmPool:
    """
    A reservoir of arms. Ranks γ(a) are drawn lazily when an arm is
    observed for the first time, so we can truly have 'infinite' arms.
    """
    def __init__(self, quantile_fn, noise_sigma: float, rng):
        self.quantile_fn = quantile_fn
        self.noise_sigma = noise_sigma
        self.rng = rng
        self.ranks: dict[int, float] = {}  # arm index -> γ(a)

    def _ensure_rank(self, arm_id: int) -> float:
        if arm_id not in self.ranks:
            self.ranks[arm_id] = float(self.rng.uniform(0.0, 1.0))
        return self.ranks[arm_id]

    def pull(self, arm_id: int) -> float:
        """
        Draw one noisy reward from arm `arm_id`:
           X = λ_{γ(a)} + noise
        """
        gamma = self._ensure_rank(arm_id)
        mean = self.quantile_fn(gamma)
        noise = float(self.rng.normal(0.0, self.noise_sigma))
        return mean + noise

    def true_mean(self, arm_id: int) -> float:
        """Oracle access to the true mean (for evaluation only)."""
        gamma = self._ensure_rank(arm_id)
        return self.quantile_fn(gamma)

    def true_rank(self, arm_id: int) -> float:
        """Oracle access to the true rank (for evaluation only)."""
        return self._ensure_rank(arm_id)
```

### 7.2 OSE (Algorithm 1, verbatim from the paper)

```python
class OSE:
    """
    Optimistic Scope Exploration, Algorithm 1 in Pilliat (2025).

    State per arm:
      counts[a]:  N_{a, t}  (pulls so far)
      sums[a]:    total reward observed
    """
    def __init__(self, pool: ArmPool, noise_sigma: float, beta: float, rng):
        self.pool = pool
        self.sigma = noise_sigma
        self.beta = beta                # tuning parameter (grows with log t in theory)
        self.rng = rng
        self.counts: dict[int, int] = {}
        self.sums: dict[int, float] = {}

    def _ucb(self, arm_id: int) -> float:
        n = self.counts.get(arm_id, 0)
        if n == 0:
            return float("inf")
        mean = self.sums[arm_id] / n
        bonus = math.sqrt(self.sigma * self.sigma * self.beta / n)
        return mean + bonus

    def _lcb(self, arm_id: int) -> float:
        n = self.counts.get(arm_id, 0)
        if n == 0:
            return float("-inf")
        mean = self.sums[arm_id] / n
        bonus = math.sqrt(self.sigma * self.sigma * self.beta / n)
        return mean - bonus

    def step(self, t: int) -> tuple[int, int]:
        """
        Run one step of OSE at time t (1-indexed).
        Returns (pulled_arm, recommended_arm).
        """
        # 1. Random exploration scope
        u = float(self.rng.uniform(0.0, 1.0))
        Z = max(1, int(math.floor(t ** u)))

        # 2. Pull arg max UCB among arms 1..Z (paper uses 1-indexed arms)
        pulled = max(range(1, Z + 1), key=self._ucb)
        reward = self.pool.pull(pulled)
        self.counts[pulled] = self.counts.get(pulled, 0) + 1
        self.sums[pulled] = self.sums.get(pulled, 0.0) + reward

        # 3. Recommend arg max LCB among all observed arms
        if not self.counts:
            recommended = pulled
        else:
            recommended = max(self.counts.keys(), key=self._lcb)
        return pulled, recommended
```

### 7.3 PROSE (Algorithm 2, naive version)

```python
class PROSE:
    """
    Progressive Ranking for OSE, Algorithm 2 in Pilliat (2025).

    Naive O(t log t) per step implementation: sort all observed arms
    each step. The paper describes an O(log^2 t) amortized version
    (§3, end) using two sorted structures (by LCB and by UCB within
    scope buckets). This is sufficient for tutorial use.
    """
    def __init__(self, pool: ArmPool, noise_sigma: float, beta: float,
                 gamma_Q: float = 1.0, rng=None):
        self.pool = pool
        self.sigma = noise_sigma
        self.beta = beta
        self.gamma_Q = gamma_Q      # Beta(γ, 1) quantile function exponent
        self.rng = rng
        self.counts: dict[int, int] = {}
        self.sums: dict[int, float] = {}
        self.j = 1                  # sub-round counter
        self.next_new_arm = 1       # next fresh arm ID to introduce

    def _ucb(self, arm_id: int) -> float:
        n = self.counts.get(arm_id, 0)
        if n == 0:
            return float("inf")
        mean = self.sums[arm_id] / n
        return mean + math.sqrt(self.sigma * self.sigma * self.beta / n)

    def _lcb(self, arm_id: int) -> float:
        n = self.counts.get(arm_id, 0)
        if n == 0:
            return float("-inf")
        mean = self.sums[arm_id] / n
        return mean - math.sqrt(self.sigma * self.sigma * self.beta / n)

    def _Q(self, x: float) -> float:
        """
        Q(x) = x^(1/γ): the Beta(γ, 1) quantile function (Paper §3).
        γ = 1 gives Q(x) = x (Uniform), the paper's default.
        Larger γ concentrates Q toward 1, closer to classical UCB.
        """
        return x ** (1.0 / self.gamma_Q)

    def step(self, t: int) -> tuple[int, int]:
        # 1. Reset j if needed
        log_t = max(1.0, math.log(t + 1))
        if self.j > log_t:
            self.j = 1

        # 2. Deterministic exploration scope
        scope_exp = self._Q(self.j / log_t)
        Z = max(1, int(math.floor(t ** scope_exp)))

        # 3. Rank observed arms by LCB descending
        observed = list(self.counts.keys())
        observed.sort(key=self._lcb, reverse=True)

        # 4. Candidate pool: top-Z observed arms plus one fresh arm
        #    if Z exceeds the number observed (so new-arm discovery
        #    is still possible at large scopes).
        top_observed = observed[:Z]
        candidates = list(top_observed)
        if Z > len(observed):
            candidates.append(self.next_new_arm)
            self.next_new_arm += 1

        # 5. Pull arg max UCB among candidates
        pulled = max(candidates, key=self._ucb)
        reward = self.pool.pull(pulled)
        self.counts[pulled] = self.counts.get(pulled, 0) + 1
        self.sums[pulled] = self.sums.get(pulled, 0.0) + reward

        # 6. Recommendation: top of (updated) LCB ranking
        observed_after = list(self.counts.keys())
        recommended = max(observed_after, key=self._lcb)

        self.j += 1
        return pulled, recommended
```

### 7.4 Run an experiment (Beta(1, 1/α) case, Paper §5)

```python
def run_simulation(algo_factory, num_trials: int, horizon: int,
                   alpha: float, noise_sigma: float = 1.0,
                   base_seed: int = 0) -> np.ndarray:
    """
    Simulate `num_trials` independent runs of `algo_factory()` for
    `horizon` steps on Beta(1, 1/α).

    Returns simple regret per time step, shape (num_trials, horizon).
    """
    quantile_fn = make_quantile_function("beta", alpha=alpha)
    simple_regret = np.zeros((num_trials, horizon))

    for trial in range(num_trials):
        rng = np.random.default_rng(base_seed + trial)
        pool = ArmPool(quantile_fn, noise_sigma=noise_sigma, rng=rng)
        algo = algo_factory(pool, noise_sigma, rng)
        for t in range(1, horizon + 1):
            _, recommended = algo.step(t)
            # simple regret = λ_0 - mean of recommended arm
            # for Beta(1, 1/α), λ_0 = 1
            simple_regret[trial, t - 1] = 1.0 - pool.true_mean(recommended)
    return simple_regret


# Factories: use the paper's β = 10 for both (Figure 3 setting)
def make_ose(pool, sigma, rng):
    return OSE(pool=pool, noise_sigma=sigma, beta=10.0, rng=rng)

def make_prose(pool, sigma, rng):
    return PROSE(pool=pool, noise_sigma=sigma, beta=10.0, gamma_Q=1.0, rng=rng)


# Quick run (smaller than the paper for tutorial speed)
regret_ose = run_simulation(make_ose, num_trials=20, horizon=2000, alpha=1.0)
regret_prose = run_simulation(make_prose, num_trials=20, horizon=2000, alpha=1.0)

print(f"Median simple regret at t=2000 (α=1):")
print(f"  OSE:   {np.median(regret_ose[:, -1]):.4f}")
print(f"  PROSE: {np.median(regret_prose[:, -1]):.4f}")
```

### 7.5 Expected pattern (from Paper Figure 3)

| `α` | Top arms are... | Simple regret at large t |
|---|---|---|
| 0.25 | Very rare | Hard: PROSE clearly beats OSE |
| 0.5 | Moderate | PROSE beats OSE and BSH |
| 1.0 | Common | PROSE beats BSH slightly |
| 2.0 | Very common | All methods quickly reach near-zero regret |

---

## 8. The main theoretical rates (Table 1 of the paper)

Rewritten with `t̃ = t / ψ̃`, where `ψ̃` is a large polylog factor in `t/δ`.

| Distribution `D` | `λ_η` | Upper bound on `γ(r̂_t)` | Lower bound on `λ_{γ(r̂_t)}` |
|---|---|---|---|
| Bernoulli | `u · 1{η ≤ η_0}` | `η_0 · 1{ t̃ ≥ ζ²/(η_0 u²) }` | `u · 1{ t̃ ≥ ζ²/(η_0 u²) }` |
| Beta, α < 1/2 | `1 - η^α` | `(1 ∨ ζ²/α²) / t̃` | `1 - (  (1 ∨ ζ²/α²)/t̃  )^α` |
| Beta, α ≥ 1/2 | `1 - η^α` | `1/t̃  ∨  (ζ²/t̃)^{1/(2α)}` | `1 - 1/t̃^α  ∨  sqrt(ζ²/t̃)` |
| Pareto, α < 1/2 | `η^(-α)` | `1/t̃  ∨  (ζ²/(α²t̃))^{1/(1-2α)}` | `t̃^α ∧ (α²t̃/ζ²)^{α/(1-2α)}` |
| Pareto, α ≥ 1/2 | `η^(-α)` | `1/t̃  ∨  1{t̃ ≥ ζ^{1/α}}` | `t̃^α · 1{t̃ ≥ ζ^{1/α}}` |

Phase transitions at `α = 1/2` are the paper's key theoretical contribution.

---

## 9. Empirical findings (Paper §5)

From Paper Figures 3, 4, 5 (simulations with Beta(1, 1/α), K = 5000 arms, horizon T = 50000, 2000 trials):

```
Simple regret ranking (lower is better):
  PROSE (β=10)  <  OSE (β=10)  <  BSH  <  naive UCB (until t ≫ K)

Cumulative regret ranking (at t = 10^5, lower is better):
  PROSE ≈ UCB  <  OSE  ≈  BSH

Execution time (2000 trials, T=50k, K=5k):
  PROSE: 76 ms/trial avg
  BSH:   89 ms/trial avg   (roughly 15% slower)
```

Key takeaway from the paper: PROSE matches UCB's long-term cumulative regret while also giving strong simple regret. OSE and BSH do not achieve this combination.

---

## 10. How to read the paper

### 10.1 Recommended reading order

| Step | Section | Focus |
|---|---|---|
| 1 | Abstract + §1.2 Contributions | One-page summary |
| 2 | §2 Main Result, through Theorem 2.1 | Definitions 1, 2; equations (4), (5), (6) |
| 3 | §3 Algorithms | Algorithm 1 (OSE), then Algorithm 2 (PROSE) |
| 4 | §4.1 Uniform ϵ-error bound | Connection to Zhao et al. 2023 (BSH) |
| 5 | §4.2 Beta(1, 1/α) case | The α < 1/2 vs α ≥ 1/2 transition |
| 6 | §5 Numerical Study + Figures 3-5 | Empirical evidence |
| 7 | Appendix A (proof of Theorem 2.1) | Only if you need the theory details |

### 10.2 Symbols cheat sheet

| Symbol | Meaning |
|---|---|
| `A` | Set of arms (countably infinite) |
| `γ(a)` | Rank of arm `a`, uniform in [0, 1] |
| `λ_η` | Quantile function: mean of a rank-η arm |
| `X_{a, s}` | s-th reward observed from arm a |
| `ε_{a, s}` | Noise term (sub-Gaussian, variance ζ²) |
| `ζ²` | Sub-Gaussian noise variance |
| `r̂_t` | Recommendation at time t |
| `N_{a, t}` | Number of pulls of arm a by time t |
| `X̄_{a, t}` | Empirical mean of arm a at time t |
| `β_t` | Confidence tuning parameter (paper: `6 log(5t/δ)`) |
| `δ` | Failure probability target |
| `G(ρ, ν)` | Rank-corrected inverse squared gap, equation (4) |
| `S(η)` | Sample complexity function, equation (5) |
| `η*_t(ψ)` | Best achievable rank at time t |
| `ψ_{t, δ}` | Polylog factor in theorem: `≥ 2^30 log³(5t/δ)` |
| `Z` | Exploration scope (OSE random, PROSE deterministic) |
| `Q` | Quantile function for PROSE scope, maps [0,1] to [0,1] |
| `π` | Permutation of arms by LCB rank (PROSE only) |

### 10.3 What to skip on first pass

- Appendix A (full proof of Theorem 2.1): dense concentration inequalities
- Appendix B (technical lemmas): Hoeffding, Bernstein deviations
- Pareto analysis (§4.2 Pareto part): interesting but not needed for applications where rewards are bounded
- The efficient PROSE implementation details (§3, end): important for production code, skippable for understanding the algorithm

---

## 11. Connection to your pipeline

The infinite-armed setting describes your Level 2 generator (inside each of the 6 harm categories):

| Paper concept | Your pipeline equivalent |
|---|---|
| Arm index 1, 2, 3, ... | (persona, topic, prompt) triples within a category |
| Rank `γ(a)` ~ Uniform[0, 1] | Position of this triple in the category's attack-effectiveness distribution |
| Quantile function `λ_η` | Severity achievable by the η-th quantile triple in this category |
| Budget `t` | Conversations allocated to this category |
| Noise `ε` | Variance of severity across reruns of the same triple |
| Recommendation `r̂_t` | The attack triple you consider "best so far" in this category |
| Distribution-free (unknown α) | You do NOT need to know the category's hardness shape in advance |

**Practical note for your v1 implementation:** use PROSE with the default `Q(x) = x` (gamma_Q = 1.0) and `β = 10`. These are the paper's empirical recommendations (Paper §5).

---

## 12. Next-step exercises for your RA

1. Run the code above for `alpha ∈ {0.25, 0.5, 1.0, 2.0}` and plot simple regret vs. t. Compare the shapes to Paper Figure 3.
2. Implement OSE with `β` that grows as `β_t = 6 log(5t/δ)` (Theorem 2.1 setting). Does PROSE still win?
3. Add noise `ζ = 0.5` and `ζ = 2.0`. Confirm the phase transition around `t ≈ ζ^{2α/(2α-1)}` for Beta(1, 1/α).
4. Try the Pareto quantile function (`η^(-α)`). Note simple regret is not well-defined; track `λ_{γ(r̂_t)}` directly instead.
5. Try `gamma_Q = 5, 15` in PROSE (larger γ = closer to classical UCB). Observe when each setting wins.

---

## 13. Glossary

| Word | Meaning |
|---|---|
| Anytime algorithm | Gives a valid recommendation at every step, not just at the end |
| Arm reservoir | The pool of all possible arms the agent can query |
| Bracketing trick | Split arms into groups of exponentially growing sizes (Katz-Samuels & Jamieson 2020) |
| Confidence bound | Range around the sample mean, shrinks as you collect more data |
| Distribution-free | The algorithm does not need to know the arm-mean distribution in advance |
| Doubling trick | Turn a fixed-horizon algorithm into an anytime one by running it in phases of doubling length |
| LCB | Lower Confidence Bound, pessimistic estimate of a mean |
| Mean reward | The long-run average of an arm, hidden from the agent |
| Polylog factor | A quantity of the form `log^k(t)` for some small integer `k` |
| Quantile function | `λ_η` = mean of the rank-η arm |
| Rank | `γ(a) ∈ [0, 1]`, how good an arm is. Small rank = good |
| Simple regret | `λ_0 - λ_{γ(r̂_t)}`, gap between the best possible mean and the recommended arm's mean |
| Simple reward | `λ_{γ(r̂_t)}`, the mean of the recommended arm (well-defined even for unbounded distributions) |
| Sub-Gaussian | Noise whose moment generating function is bounded like a Gaussian's |
| UCB | Upper Confidence Bound, optimistic estimate of a mean |
