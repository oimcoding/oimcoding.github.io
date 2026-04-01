# Quiz 2: Tokenization and BPE — Tutor Answers (Q1–Q25)

---

# Quiz 2, Question 1 — Origin of BPE

> **Who introduced the original Byte Pair Encoding (BPE) algorithm in 1994, and what was its original purpose?**
>
> **B. Philip Gage; for data compression to reduce file size.**

---

## Sub-questions

### What is Byte Pair Encoding?

$$\text{BPE: repeatedly replace the most frequent pair of adjacent bytes with a single new byte}$$

A greedy compression algorithm. Each iteration finds the most common adjacent pair in the data and merges it into one symbol, reducing total length.

### What is data compression?

$$\text{Compression Ratio} = \frac{\text{Original Size}}{\text{Compressed Size}}$$

Reducing the number of bytes needed to represent data. BPE achieves this by replacing repeated byte pairs with shorter symbols.

**Numeric example:** Sequence `ABABAB` → pair `AB` appears 3 times → replace with `X` → `XXX` (6 bytes → 3 bytes, ratio = 2).

---

## Main answer

**B. Philip Gage; for data compression to reduce file size.**

```
BPE timeline:

  1994: Philip Gage ──▶ Data compression (reduce file size)
                            │
                            ▼  Sennrich et al. (2016)
  2016: NLP adaptation ──▶ Subword tokenization (handle rare words)
                            │
                            ▼  GPT-2 (2019)
  2019: GPT-2 / Tiktoken ──▶ Standard LLM tokenizer
```

| Aspect | Original BPE (1994) | NLP BPE (2016+) |
|---|---|---|
| Author | Philip Gage | Sennrich et al. |
| Goal | Compress file size | Build subword vocabulary |
| Input | Raw bytes | Word-level text |
| Stopping criterion | No more frequent pairs | Target vocabulary size reached |
| Output | Compressed byte stream | Subword token sequence |

---
---

# Quiz 2, Question 2 — Softmax Bottleneck with Large Vocabulary

> **As vocabulary size (V) grows toward infinity, which component of a neural network becomes prohibitively expensive to compute?**
>
> **C. The Softmax function (calculating probabilities over all V).**

---

## Sub-questions

### What is the Softmax function?

$$\text{Softmax}(z_i) = \frac{e^{z_i}}{\sum_{j=1}^{V} e^{z_j}}$$

Converts $V$ logits into a probability distribution. The denominator requires summing over **all** $V$ classes.

### What is the computational cost of Softmax?

$$\text{Cost} = O(V) \quad \text{per prediction}$$

Each forward pass requires computing $e^{z_j}$ for every word in the vocabulary, then summing them. For $V = 1{,}000{,}000$, that's one million exponentials per token prediction.

**Numeric example:** $V = 50{,}000$ → 50,000 exponentials + 50,000 additions + 50,000 divisions per token.

### What is the linear head?

$$z = W h + b, \quad W \in \mathbb{R}^{V \times d}$$

The final layer maps hidden state $h$ (dimension $d$) to $V$ logits. Weight matrix $W$ has $V \times d$ parameters.

**Numeric example:** $V = 50{,}000$, $d = 768$ → $W$ has $38.4$ million parameters — just for the output layer.

---

## Main answer

**C. The Softmax function (calculating probabilities over all V).**

$$\text{Softmax cost} = O(V): \quad \underbrace{e^{z_1}, e^{z_2}, \dots, e^{z_V}}_{V \text{ exponentials}} \quad + \quad \underbrace{\sum_{j=1}^V e^{z_j}}_{\text{summation}} \quad + \quad \underbrace{\frac{e^{z_i}}{\text{sum}}}_{V \text{ divisions}}$$

```
Cost scaling with vocabulary size:

  V = 32K   ──▶  Manageable (standard LLM)
  V = 100K  ──▶  Expensive (larger embedding + softmax)
  V = 1M    ──▶  Prohibitive (softmax dominates compute)
  V → ∞     ──▶  Impossible
```

| Component | Cost w.r.t. V | Why |
|---|---|---|
| Embedding lookup | $O(1)$ | Index into table |
| Transformer layers | $O(1)$ | Independent of $V$ |
| Linear head ($Wh$) | $O(V \cdot d)$ | Matrix-vector multiply |
| **Softmax** | $O(V)$ | **Sum over all V classes** |
| **Total output layer** | $O(V \cdot d)$ | **Dominates as $V \to \infty$** |

---
---

# Quiz 2, Question 3 — Hashing Trick Collisions

> **What is the primary negative consequence of using the "Hashing Trick" with a fixed number of buckets?**
>
> **B. Hash collisions — different words map to the same bucket, losing distinction between them.**

---

## Sub-questions

### What is the Hashing Trick?

$$\text{index}(w) = \text{hash}(w) \mod B$$

Maps any word $w$ to a fixed-size table of $B$ buckets. No vocabulary needed — any string gets an index.

**Numeric example:** $B = 1000$, hash("cat") = 4827 → $4827 \mod 1000 = 827$. hash("dog") = 2827 → $2827 \mod 1000 = 827$. Both map to bucket 827.

### What is a hash collision?

$$\text{Collision: } w_1 \neq w_2 \text{ but } \text{hash}(w_1) \mod B = \text{hash}(w_2) \mod B$$

Two distinct words share the same bucket. Their embeddings become identical — the model cannot distinguish them.

### What is the Pigeonhole Principle?

$$|W| > B \implies \text{at least one bucket contains } \geq 2 \text{ words}$$

If the number of unique words $|W|$ exceeds the number of buckets $B$, collisions are **guaranteed**.

---

## Main answer

**B. Hash collisions — different words map to the same bucket, losing distinction between them.**

```
Hashing Trick collision example:

  "cat" ──hash──▶ 4827 ──mod 1000──▶ bucket 827 ─┐
                                                   ├──▶ Same embedding!
  "dog" ──hash──▶ 2827 ──mod 1000──▶ bucket 827 ─┘

  Model sees: embedding[827] for both → cannot distinguish "cat" from "dog"
```

| Property | Lookup Table | Hashing Trick |
|---|---|---|
| Vocabulary needed | Yes (fixed) | No |
| Memory | $O(V \times d)$ | $O(B \times d)$, $B \ll V$ |
| OOV handling | Fails (unknown token) | Always produces an index |
| Collision risk | None | Yes — **information loss** |
| Use case | Standard NLP | Memory-constrained settings |

---
---

# Quiz 2, Question 4 — Character Tokenization and Self-Attention Cost

> **Character-level tokenization increases sequence length by roughly 5x. How does this impact the compute cost of the Self-Attention mechanism?**
>
> **B. Quadratically (25x cost).**

---

## Sub-questions

### What is self-attention cost?

$$\text{Cost}(\text{Self-Attention}) = O(T^2 \cdot d)$$

where $T$ = sequence length, $d$ = embedding dimension. The $QK^T$ matrix is $T \times T$ — every token attends to every other token.

### Why is the cost quadratic in $T$?

$$QK^T \in \mathbb{R}^{T \times T}: \quad T^2 \text{ dot products}$$

Each of $T$ query vectors computes a dot product with each of $T$ key vectors.

**Numeric example:** $T = 100$ → $100^2 = 10{,}000$ dot products. $T = 500$ → $500^2 = 250{,}000$ dot products.

### What happens when $T \to 5T$?

$$(5T)^2 = 25T^2$$

Sequence length increases by factor 5 → attention cost increases by factor $5^2 = 25$.

---

## Main answer

**B. Quadratically (25x cost).**

$$\text{Subword: } T \text{ tokens} \implies O(T^2)$$

$$\text{Character: } 5T \text{ tokens} \implies O((5T)^2) = O(25T^2)$$

$$\text{Ratio} = \frac{25T^2}{T^2} = 25\times$$

```
Attention cost scaling:

  Subword (T tokens):     T² attention scores
  Character (5T tokens):  (5T)² = 25T² attention scores

  Example: T = 512 (subword)
    Subword:   512²   =    262,144 scores
    Character: 2560²  =  6,553,600 scores  (25× more)
```

| Tokenization | Sequence Length | Attention Cost | Relative Cost |
|---|---|---|---|
| Subword | $T$ | $T^2$ | $1\times$ |
| Character | $5T$ | $25T^2$ | $25\times$ |
| Byte-level | $\sim6T$ | $\sim36T^2$ | $\sim36\times$ |

---
---

# Quiz 2, Question 5 — Goldilocks Goal of Subword Tokenization

> **What is the "Goldilocks" goal of Subword Tokenization?**
>
> **C. To balance a fixed vocabulary size (efficiency) with the ability to represent rare words (coverage).**

---

## Sub-questions

### What is word-level tokenization?

$$\text{vocab} = \{\text{every unique word in corpus}\}$$

Each word gets its own token. Vocabulary grows unboundedly with corpus size. Rare words → massive vocabulary. Unknown words → `<UNK>` token (information loss).

### What is character-level tokenization?

$$\text{vocab} = \{a, b, c, \dots, z, A, \dots, Z, 0, \dots, 9, \text{punctuation}\}$$

Tiny vocabulary ($\sim$100 tokens). Zero OOV. But sequences become very long → quadratic attention cost.

### What is subword tokenization?

$$\text{vocab} = \{\text{common words} \cup \text{frequent subwords} \cup \text{rare-word fragments}\}$$

Splits rare words into known pieces. Common words stay whole. Fixed vocabulary size (e.g., 32K).

**Numeric example:** "unhappiness" → `["un", "happiness"]` or `["un", "happ", "iness"]`.

---

## Main answer

**C. To balance a fixed vocabulary size (efficiency) with the ability to represent rare words (coverage).**

```
The Goldilocks spectrum:

  ◄── Too Small (characters)      Just Right (subwords)      Too Large (words) ──►

  Vocab: ~100                     Vocab: ~32K                 Vocab: ~1M+
  Seq len: very long              Seq len: moderate           Seq len: short
  OOV: none                       OOV: none                   OOV: many
  Attention: 25x+ cost            Attention: balanced         Attention: cheap
  Semantics: weak per token       Semantics: good             Semantics: best per token
```

| Property | Character | Subword (BPE) | Word |
|---|---|---|---|
| Vocab size | $\sim$100 | $\sim$32K (fixed) | $\sim$1M+ (unbounded) |
| OOV tokens | 0 | 0 | Many |
| Sequence length | Very long ($5\text{–}6\times$) | Moderate | Short |
| Attention cost | $25\text{–}36\times$ | $1\times$ (baseline) | Cheapest |
| Rare word handling | Always works | Decomposes into pieces | `<UNK>` |

---
---

# Quiz 2, Question 6 — WordPiece vs BPE Optimization

> **Standard BPE merges based on Frequency. What does WordPiece optimize for?**
>
> **B. Likelihood of the training data (Pointwise Mutual Information).**

---

## Sub-questions

### What does BPE optimize for?

$$\text{BPE: merge the pair with highest count (frequency)}$$

$$\text{score}_\text{BPE}(a, b) = \text{count}(ab)$$

Greedy: always merge the most frequent adjacent pair, regardless of how common $a$ and $b$ are individually.

### What is Pointwise Mutual Information (PMI)?

$$\text{PMI}(a, b) = \log \frac{P(ab)}{P(a) \cdot P(b)}$$

Measures how much more likely $a$ and $b$ co-occur than expected by chance. High PMI → the pair is **surprisingly** frequent given individual frequencies.

**Numeric example:** $P(\text{"qu"}) = 0.01$, $P(\text{"q"}) = 0.01$, $P(\text{"u"}) = 0.05$.
PMI = $\log \frac{0.01}{0.01 \times 0.05} = \log 20 \approx 3.0$ (high — "q" and "u" are strongly associated).

### What does WordPiece optimize for?

$$\text{score}_\text{WP}(a, b) = \frac{\text{count}(ab)}{\text{count}(a) \times \text{count}(b)}$$

Merges the pair that maximizes the **likelihood** of the training data. Equivalent to choosing the pair with the highest PMI — not just the most frequent pair, but the most *surprisingly* co-occurring pair.

---

## Main answer

**B. Likelihood of the training data (Pointwise Mutual Information).**

```
BPE vs WordPiece merge decision:

  Candidates: ("t","h") count=1000, ("x","y") count=50

  BPE:       score("t","h") = 1000   ← wins (highest count)
             score("x","y") = 50

  WordPiece: score("t","h") = 1000 / (5000 × 4000) = 0.00005
             score("x","y") = 50 / (60 × 70)       = 0.012    ← wins (highest PMI)

  "x" and "y" are rare individually but almost always appear together → high PMI
```

| Criterion | BPE | WordPiece |
|---|---|---|
| Merge score | $\text{count}(ab)$ | $\frac{\text{count}(ab)}{\text{count}(a) \cdot \text{count}(b)}$ |
| Optimizes | Raw frequency | Likelihood / PMI |
| Bias | Favors common pairs | Favors **surprisingly** co-occurring pairs |
| Used by | GPT-2, GPT-3, GPT-4 | BERT |

---
---

# Quiz 2, Question 7 — BPE Stopping Criterion

> **In a Modern NLP context, when does the BPE training process stop?**
>
> **B. When the vocabulary reaches the target size (e.g., 32k).**

---

## Sub-questions

### How does BPE build its vocabulary?

$$|\text{vocab}|_{\text{step } n} = |\text{base vocab}| + n$$

Each merge adds exactly one new token. After $n$ merges, vocabulary has grown by $n$ tokens from the base.

**Numeric example:** Base vocab = 256 (bytes), target = 32,000. Need $32{,}000 - 256 = 31{,}744$ merges.

### What was the original stopping criterion?

In Gage's 1994 algorithm: stop when no pair occurs more than once (no compression benefit remains).

### What is the modern stopping criterion?

$$\text{Stop when } |\text{vocab}| = V_{\text{target}}$$

Fixed hyperparameter. Typical values: GPT-2 = 50,257; LLaMA = 32,000; GPT-4 = 100,256.

---

## Main answer

**B. When the vocabulary reaches the target size (e.g., 32k).**

```
BPE training loop:

  vocab = base_vocab (e.g., 256 bytes)
       │
       ▼
  while |vocab| < V_target:
       │
       ├── Count all adjacent pairs
       ├── Find most frequent pair (a, b)
       ├── Merge (a, b) → new token "ab"
       └── Add "ab" to vocab
       │
       ▼  |vocab| == V_target?
  STOP
```

| Setting | Base Vocab | Target $V$ | Merges Needed |
|---|---|---|---|
| GPT-2 | 256 | 50,257 | 50,001 |
| LLaMA 2 | 256 | 32,000 | 31,744 |
| GPT-4 | 256 | 100,256 | 100,000 |

---
---

# Quiz 2, Question 8 — BBPE Initial Base Vocabulary

> **What is the size of the initial base vocabulary for a Byte-Level BPE (BBPE) tokenizer before any merges occur?**
>
> **B. 256 (All possible byte values).**

---

## Sub-questions

### What is a byte?

$$\text{1 byte} = 8 \text{ bits} = 2^8 = 256 \text{ possible values}$$

Range: $[0, 255]$ or equivalently $[\texttt{0x00}, \texttt{0xFF}]$. Every digital file is a sequence of bytes.

### What is Byte-Level BPE?

$$\text{Base vocab} = \{0, 1, 2, \dots, 255\} \quad (|\text{base}| = 256)$$

Start with all 256 possible byte values as the initial vocabulary. Any file — any language, any encoding — can be decomposed into these 256 base tokens.

### Why 256 and not 128 (ASCII)?

ASCII covers only English letters, digits, and basic punctuation (128 characters). UTF-8 uses bytes $> 127$ for non-English characters, emojis, etc. Starting with 256 ensures **complete coverage** of all possible inputs.

---

## Main answer

**B. 256 (All possible byte values).**

$$2^8 = 256 \text{ distinct byte values} \implies \text{any byte sequence is tokenizable}$$

```
BBPE base vocabulary:

  0x00  0x01  0x02  ...  0x7F  (ASCII: 0-127)
  0x80  0x81  0x82  ...  0xFF  (Extended: 128-255)

  Total: 256 tokens — covers EVERY possible byte
```

| Base Vocab Type | Size | Coverage | OOV Risk |
|---|---|---|---|
| ASCII characters | 128 | English only | High (non-ASCII fails) |
| Unicode characters | 150,000+ | All languages | None, but huge base vocab |
| **Byte values** | **256** | **All possible inputs** | **None** |

---
---

# Quiz 2, Question 9 — Zero OOV Guarantee

> **Which method ensures zero Out-Of-Vocabulary (OOV) tokens for any possible input string?**
>
> **B. Byte-Level BPE (BBPE).**

---

## Sub-questions

### What is an OOV token?

$$w \notin \text{vocab} \implies w \to \texttt{<UNK>}$$

A word not in the tokenizer's vocabulary. Mapped to a special unknown token, losing all semantic information.

### Why does word-level tokenization produce OOV?

$$\text{vocab} = \{\text{finite set of words}\}$$

Any word not seen during training → OOV. Misspellings, neologisms, foreign words all become `<UNK>`.

### Why does BBPE guarantee zero OOV?

$$\forall \; \text{input string } s: \quad s = [b_1, b_2, \dots, b_n], \quad b_i \in \{0, \dots, 255\} \subseteq \text{vocab}$$

Every string is ultimately a sequence of bytes. Since all 256 byte values are in the base vocabulary, any input can at minimum be decomposed into individual bytes.

---

## Main answer

**B. Byte-Level BPE (BBPE).**

```
OOV handling by tokenizer type:

  Word-level:    "cryptozoology" → <UNK>         ✗ Information lost

  BPE (char):    "cryptozoology" → ["crypt","o","zoo","logy"]  ✓ if chars in vocab

  BBPE:          "🦄" → [0xF0, 0x9F, 0xA6, 0x84] → all in base vocab  ✓ ALWAYS works
```

```
Worst-case fallback:

  BBPE tries:  merged tokens first (efficient)
       │
       ├── "hello" in vocab? → YES → ["hello"]           (1 token)
       │
       └── "xyzzy" in vocab? → NO
            ├── "xy" + "zzy"? → partial match
            └── Worst case: [x, y, z, z, y]              (5 tokens — individual bytes)
                 All bytes ∈ {0..255} ⊂ vocab             ✓ Never fails
```

| Method | OOV Possible? | Why |
|---|---|---|
| Word-level | Yes | Finite word list |
| Character BPE | Rare (if all chars covered) | May miss rare Unicode |
| **BBPE** | **No — impossible** | **All 256 bytes in base vocab** |

---
---

# Quiz 2, Question 10 — SentencePiece Whitespace Handling

> **How does SentencePiece handle whitespace?**
>
> **C. It treats space as a dedicated character (often represented as ▁), making the process reversible.**

---

## Sub-questions

### Why is whitespace a problem for tokenizers?

Most tokenizers use whitespace to pre-split text into words, then tokenize each word. This **destroys** the whitespace information — you cannot reconstruct whether the original had one space, two spaces, or a tab.

### What is the ▁ (underscore) convention?

$$\text{"I love cats"} \to \text{"▁I▁love▁cats"}$$

SentencePiece replaces every space with a visible `▁` character **before** tokenization. Spaces become explicit tokens rather than silent delimiters.

### What does "reversible" mean here?

$$\text{Detokenize}(\text{Tokenize}(X)) = X$$

The original text (including all whitespace) can be perfectly reconstructed from the token sequence. No information is lost.

---

## Main answer

**C. It treats space as a dedicated character (often represented as ▁), making the process reversible.**

```
Standard tokenizer (lossy):

  "I  love  cats" ──split on space──▶ ["I", "love", "cats"]
                                         │
                                         ▼  detokenize
                                      "I love cats"     ✗ Lost the double spaces!

SentencePiece (lossless):

  "I  love  cats" ──▶ "▁I▁▁love▁▁cats" ──tokenize──▶ ["▁I", "▁▁", "love", "▁▁", "cats"]
                                                          │
                                                          ▼  detokenize
                                                       "I  love  cats"  ✓ Perfect!
```

| Property | Standard Tokenizer | SentencePiece |
|---|---|---|
| Whitespace handling | Pre-split delimiter (discarded) | Explicit `▁` character (preserved) |
| Reversible | No | Yes |
| Start-of-word marker | None or implicit | `▁` prefix on word-initial tokens |
| Language-agnostic | No (needs word boundaries) | Yes (treats raw text as input) |

---
---

# Quiz 2, Question 11 — Unigram Language Model Vocabulary Construction

> **How does the Unigram Language Model construct its vocabulary?**
>
> **B. Top-down: Starting with a huge vocabulary and pruning symbols that minimally impact likelihood.**

---

## Sub-questions

### What is the Unigram Language Model?

$$P(x_1, x_2, \dots, x_n) = \prod_{i=1}^{n} P(x_i)$$

Assumes each subword token is independent. The probability of a segmentation is the product of individual token probabilities.

### What is a top-down approach?

$$\text{Start: } |\text{vocab}| = V_\text{large} \quad \xrightarrow{\text{prune}} \quad |\text{vocab}| = V_\text{target}$$

Begin with a massive initial vocabulary (e.g., all substrings up to a maximum length), then iteratively remove tokens whose removal causes the smallest decrease in training data likelihood.

### What is the EM algorithm in this context?

1. **E-step:** Given current vocabulary + probabilities, find the best segmentation of training data (via Viterbi)
2. **M-step:** Re-estimate token probabilities from the segmentation
3. **Prune:** Remove tokens with lowest impact on likelihood
4. Repeat until target vocabulary size reached

---

## Main answer

**B. Top-down: Starting with a huge vocabulary and pruning symbols that minimally impact likelihood.**

```
Unigram vs BPE construction:

  BPE (bottom-up):          Unigram (top-down):

  Start: 256 tokens          Start: ~1,000,000 tokens
       │                          │
       ▼  add merges              ▼  prune low-impact tokens
  Grow to 32K                Shrink to 32K
```

$$\text{Pruning criterion: remove token } t \text{ that minimizes } \Delta \mathcal{L} = \mathcal{L}_{\text{without } t} - \mathcal{L}_{\text{with } t}$$

| Property | BPE | Unigram LM |
|---|---|---|
| Direction | Bottom-up (merge) | Top-down (prune) |
| Start size | Small (256) | Large (~1M) |
| End size | Target $V$ | Target $V$ |
| Criterion | Frequency of pairs | Likelihood impact |
| Segmentation | Deterministic (one path) | Probabilistic (multiple paths) |

---
---

# Quiz 2, Question 12 — Viterbi Algorithm in SentencePiece

> **What is the role of the Viterbi Algorithm in SentencePiece?**
>
> **B. To find the most probable segmentation path for a sentence during inference.**

---

## Sub-questions

### What is the segmentation problem?

$$\text{"unbreakable"} \to \begin{cases} [\text{"un"}, \text{"break"}, \text{"able"}] \\ [\text{"un"}, \text{"breakable"}] \\ [\text{"unbreak"}, \text{"able"}] \\ [\text{"u"}, \text{"n"}, \text{"b"}, \dots] \end{cases}$$

Multiple valid segmentations exist. The tokenizer must pick the **best** one.

### What is the Viterbi algorithm?

$$x^* = \arg\max_{\mathbf{x}} \prod_{i} P(x_i)$$

Dynamic programming algorithm that finds the segmentation maximizing the product of token probabilities. Runs in $O(T \cdot L)$ where $T$ = string length, $L$ = max token length.

### Why not brute-force all segmentations?

$$\text{Number of possible segmentations of length } T \approx 2^{T-1}$$

Exponential explosion. For a 20-character string: $2^{19} = 524{,}288$ possible segmentations. Viterbi solves this in linear time via dynamic programming.

---

## Main answer

**B. To find the most probable segmentation path for a sentence during inference.**

```
Viterbi dynamic programming for "cats":

  Position:  0    1    2    3    4
  String:    c    a    t    s
             │    │    │    │
  Best path: ──▶──▶──▶──▶──▶

  At each position, store:
    best_score[i] = max probability of reaching position i
    best_split[i] = where the last token started

  Traceback from end → optimal segmentation
```

$$\text{best\_score}[j] = \max_{i < j} \big(\text{best\_score}[i] + \log P(\text{substring}[i:j])\big)$$

| Approach | Time Complexity | Feasible? |
|---|---|---|
| Brute force | $O(2^T)$ | No |
| **Viterbi** | $O(T \cdot L)$ | **Yes** |

---
---

# Quiz 2, Question 13 — Re-Pair vs BPE

> **Re-Pair differs from BPE because:**
>
> **B. Re-Pair optimizes a grammar for a single document recursively, while BPE is a greedy global heuristic.**

---

## Sub-questions

### What is BPE's scope?

$$\text{BPE: learn merge rules from entire corpus} \to \text{apply same rules to any new text}$$

BPE learns a **global** merge table from the full training corpus. The same rules are applied universally during inference.

### What is Re-Pair?

$$\text{Re-Pair: compress a single document by building a grammar specific to it}$$

Finds the most frequent pair in **one document**, replaces all occurrences, and repeats. Produces a context-free grammar (set of production rules) that reconstructs the original text.

### What is a grammar-based compressor?

$$S \to A B, \quad A \to \text{"the"}, \quad B \to C D, \quad C \to \text{"cat"}, \quad D \to \text{"sat"}$$

A set of hierarchical rules that generate the original text. Each rule replaces a symbol with its expansion.

---

## Main answer

**B. Re-Pair optimizes a grammar for a single document recursively, while BPE is a greedy global heuristic.**

| Property | BPE | Re-Pair |
|---|---|---|
| Scope | Global corpus | Single document |
| Output | Merge table (reusable) | Grammar (document-specific) |
| Goal | Build tokenizer vocabulary | Compress one text optimally |
| Reusable on new text | Yes | No (grammar is text-specific) |
| Optimality | Greedy heuristic | Recursive grammar optimization |

---
---

# Quiz 2, Question 14 — Efficient BPE Data Structures

> **Which combination of data structures allows BPE to run efficiently (better than O(N^2))?**
>
> **B. Doubly Linked List + Max-Heap/Hash Map.**

---

## Sub-questions

### Why is naive BPE slow?

$$\text{Naive BPE: } O(N \cdot M) \text{ per merge, where } N = \text{sequence length, } M = \text{number of merges}$$

Each merge requires scanning the entire sequence to find and replace all instances of the most frequent pair. Repeated scanning is expensive.

### What does the Doubly Linked List provide?

$$\text{Delete/Insert at known position: } O(1)$$

Sequence stored as linked list of tokens. When merging pair $(a, b)$, splice out the two nodes and insert one merged node — constant time per occurrence.

### What does the Max-Heap + Hash Map provide?

$$\text{Find most frequent pair: } O(\log N) \text{ via heap}$$
$$\text{Update pair counts: } O(1) \text{ via hash map}$$

Heap keeps pairs sorted by frequency (extract-max in $O(\log N)$). Hash map provides $O(1)$ lookup to update counts when merges change local pair statistics.

---

## Main answer

**B. Doubly Linked List + Max-Heap/Hash Map.**

```
Efficient BPE architecture:

  ┌─────────────────────────────────────────┐
  │  Doubly Linked List (token sequence)    │
  │  [t] ⇄ [h] ⇄ [e] ⇄ [c] ⇄ [a] ⇄ [t]  │
  │                                         │
  │  Merge "t"+"h" → "th":                  │
  │  [th] ⇄ [e] ⇄ [c] ⇄ [a] ⇄ [t]        │  O(1) per occurrence
  └─────────────────────────────────────────┘

  ┌─────────────────────────────────────────┐
  │  Max-Heap (pair frequencies)            │
  │  top: ("t","h") = 1000                  │  O(log N) extract-max
  │       ("h","e") = 800                   │
  │       ("c","a") = 600                   │
  │                                         │
  │  Hash Map (pair → count)                │
  │  {"th": 1000, "he": 800, "ca": 600}    │  O(1) lookup/update
  └─────────────────────────────────────────┘
```

| Operation | Naive | Optimized (DLL + Heap) |
|---|---|---|
| Find top pair | $O(N)$ scan | $O(\log N)$ heap extract |
| Apply one merge | $O(N)$ scan | $O(k)$ where $k$ = occurrences |
| Update pair counts | Recount all | $O(1)$ per affected neighbor |
| Total for $M$ merges | $O(N \cdot M)$ | $O(N \log N + M \log N)$ |

---
---

# Quiz 2, Question 15 — WordPiece ## Prefix

> **What does the "##" prefix indicate in BERT's WordPiece tokenizer?**
>
> **C. It indicates the token is a continuation (suffix) of the previous token, not a word start.**

---

## Sub-questions

### What is WordPiece tokenization?

BERT's tokenizer. Splits words into subword tokens. First token of each word has no prefix. Continuation tokens get `##` prefix.

### How does ## work in practice?

$$\text{"unhappiness"} \to [\text{"un"}, \text{"##happi"}, \text{"##ness"}]$$

- `"un"` — no prefix → word start
- `"##happi"` — `##` prefix → continuation
- `"##ness"` — `##` prefix → continuation

**Detokenization:** Remove `##` and concatenate: `"un" + "happi" + "ness" = "unhappiness"`.

---

## Main answer

**C. It indicates the token is a continuation (suffix) of the previous token, not a word start.**

```
WordPiece tokenization example:

  Input: "playing basketball"

  Tokens: ["play", "##ing", "basket", "##ball"]
            ↑        ↑        ↑          ↑
          word     suffix    word      suffix
          start    (##)      start     (##)

  Detokenize: "play"+"ing" = "playing", "basket"+"ball" = "basketball"
```

| Prefix | Meaning | Example |
|---|---|---|
| None | Word-initial token | `"play"`, `"basket"` |
| `##` | Continuation / suffix | `"##ing"`, `"##ball"` |

---
---

# Quiz 2, Question 16 — BPE and Number Comparison

> **Why might a BPE-based model struggle to tell you that "5.11" is larger than "5.9"?**
>
> **B. BPE tokenizes numbers into arbitrary subword chunks (e.g., "5.11" → "5", ".11"), so the model never sees the full number as a coherent value.**

---

## Sub-questions

### How does BPE tokenize numbers?

$$\text{"5.11"} \to [\text{"5"}, \text{".11"}] \quad \text{or} \quad [\text{"5"}, \text{"."}, \text{"11"}]$$

BPE has no concept of numerical value. It merges based on frequency in the training corpus. The resulting tokens split numbers at arbitrary boundaries.

### Why does this break numerical reasoning?

$$\text{Token "5" + Token ".11"} \neq \text{the number } 5.11$$

The model processes each token independently through embeddings. There is no built-in mechanism to reconstruct the mathematical value from fragmented tokens.

**Numeric example:** "5.11" → `["5", ".11"]` and "5.9" → `["5", ".9"]`. The model must infer that `.11 < .9` is false, but it sees these as opaque string tokens, not decimal fractions.

---

## Main answer

**B. BPE tokenizes numbers into arbitrary subword chunks, so the model never sees the full number as a coherent value.**

```
BPE tokenization of numbers:

  "5.11"  →  ["5", ".11"]     Model sees two unrelated tokens
  "5.9"   →  ["5", ".9"]      Model sees two unrelated tokens

  Human: 5.11 > 5.9 (obvious: compare decimal values)
  Model: token ".11" vs token ".9" → no numerical comparison available
         ".11" has more characters → might guess ".11" > ".9"?
         Or ".9" has higher single digit → might guess ".9" > ".11"?
```

```
Root cause (logic chain):

  BPE is frequency-based ──▶ splits numbers arbitrarily ──▶ tokens ≠ numbers
       ──▶ no numerical semantics ──▶ comparison fails
```

| Input | BPE Tokens | Mathematical Value | Model's View |
|---|---|---|---|
| "5.11" | `["5", ".11"]` | 5.11 | Two opaque symbols |
| "5.9" | `["5", ".9"]` | 5.9 | Two opaque symbols |
| Comparison | — | 5.11 > 5.9 | Uncertain / error-prone |

---
---

# Quiz 2, Question 17 — Regex Pre-tokenization

> **Why do we use Regex pre-tokenization (the "Guardrails") before running BPE?**
>
> **B. To prevent merges across word/category boundaries (e.g., stopping a space from merging with a letter, or numbers merging with words).**

---

## Sub-questions

### What is pre-tokenization?

$$\text{raw text} \xrightarrow{\text{regex split}} \text{chunks} \xrightarrow{\text{BPE}} \text{tokens}$$

Splitting text into coarse chunks **before** BPE runs. BPE merges can only happen **within** a chunk, never across chunk boundaries.

### What would happen without guardrails?

$$\text{"dog cat"} \xrightarrow{\text{BPE}} [\text{"dog cat"}]$$

BPE might merge the space with adjacent letters, creating tokens like `"g c"` or `" cat"`. These cross-boundary tokens are semantically meaningless and waste vocabulary slots.

### What does the GPT-2 regex look like?

```
'?[a-zA-Z]+|'?[0-9]+| ?[^\s\w]+|\s+
```

Separates: words (with optional leading apostrophe), numbers, punctuation, whitespace. BPE operates within each match — never across categories.

---

## Main answer

**B. To prevent merges across word/category boundaries.**

```
Without regex guardrails:

  "hello world" → BPE might learn: "o w" as a token  ✗ (cross-word)
  "price: $50"  → BPE might learn: ": $" as a token  ✗ (cross-category)

With regex guardrails:

  "hello world" → ["hello", " world"] → BPE within each chunk
  "price: $50"  → ["price", ":", " $", "50"] → BPE within each chunk

  Merges stay within boundaries:
    "hello" → "hel" + "lo"    ✓ (within word)
    "50"    → stays "50"      ✓ (within number)
```

| Category | Regex Match | BPE Scope |
|---|---|---|
| Words | `'?[a-zA-Z]+` | Merge letters within a word |
| Numbers | `[0-9]+` | Merge digits within a number |
| Punctuation | `[^\s\w]+` | Merge punctuation symbols |
| Whitespace | `\s+` | Keep spaces separate |

---
---

# Quiz 2, Question 18 — Tiktoken Speed

> **Why is Tiktoken (Llama 3) faster than standard BPE implementations?**
>
> **B. It prioritizes looking up the full token in the map before attempting to merge parts.**

---

## Sub-questions

### How does standard BPE tokenize at inference?

$$\text{Split into characters} \to \text{apply merge rules one by one in priority order}$$

Iteratively scan the sequence, find the highest-priority merge, apply it, repeat. Each pass requires scanning the full sequence.

### How does Tiktoken tokenize?

$$\text{Try longest match first} \to \text{fall back to shorter tokens}$$

Looks up each word in the merge table **as a whole** first. If found, done in one step. Only falls back to iterative merging for unknown or rare words.

### Why is lookup faster than iterative merging?

$$\text{Hash table lookup: } O(1) \quad \text{vs} \quad \text{Iterative merging: } O(k) \text{ merges per word}$$

For common words (already in the vocabulary as complete tokens), a single hash lookup replaces potentially dozens of merge operations.

---

## Main answer

**B. It prioritizes looking up the full token in the map before attempting to merge parts.**

```
Standard BPE (slow path — always used):

  "playing" → ['p','l','a','y','i','n','g']
           → ['pl','a','y','i','n','g']     merge #1
           → ['pla','y','i','n','g']        merge #2
           → ['play','i','n','g']           merge #3
           → ['play','ing']                 merge #4
           → ['playing']                    merge #5     (5 steps)

Tiktoken (fast path — try whole token first):

  "playing" → lookup("playing") → found! → ['playing']  (1 step)
```

| Approach | Common Words | Rare Words | Speed |
|---|---|---|---|
| Standard BPE | Iterative merges | Iterative merges | Slow |
| **Tiktoken** | **Direct lookup ($O(1)$)** | Falls back to merges | **Fast** |

---
---

# Quiz 2, Question 19 — SentencePiece Losslessness

> **What property makes SentencePiece "Lossless"?**
>
> **B. Detokenize(Tokenize(X)) == X (The original text is perfectly reconstructable, including spaces).**

---

## Sub-questions

### What is lossless tokenization?

$$\text{Detokenize}(\text{Tokenize}(X)) = X \quad \forall \; X$$

The original text — including whitespace, capitalization, and special characters — can be perfectly recovered from the token sequence. No information is destroyed.

### What makes most tokenizers lossy?

Standard tokenizers split on whitespace first, discarding the space characters. Multiple spaces, tabs, or leading/trailing whitespace are lost.

$$\text{"Hello   World"} \xrightarrow{\text{split}} [\text{"Hello"}, \text{"World"}] \xrightarrow{\text{detok}} \text{"Hello World"} \neq \text{"Hello   World"}$$

### How does SentencePiece achieve losslessness?

$$\text{Space} \to \text{▁ character (explicit token)}$$

By encoding whitespace as a visible character (`▁`) within the token sequence, no information is discarded. The `▁` survives tokenization and is converted back to a space during detokenization.

---

## Main answer

**B. Detokenize(Tokenize(X)) == X**

```
Lossless round-trip:

  Original:     "I  love  NLP"
       │
       ▼  Tokenize
  Tokens:       ["▁I", "▁▁", "love", "▁▁", "NLP"]
       │
       ▼  Detokenize (replace ▁ with space)
  Recovered:    "I  love  NLP"   ✓ Identical to original!
```

| Property | Standard Tokenizer | SentencePiece |
|---|---|---|
| Whitespace | Discarded during split | Encoded as `▁` |
| Round-trip | $\text{Detok}(\text{Tok}(X)) \neq X$ | $\text{Detok}(\text{Tok}(X)) = X$ |
| Lossless | No | **Yes** |

---
---

# Quiz 2, Question 20 — Token Blindness to Characters

> **Why can't ChatGPT reverse the word "Lollipop" easily?**
>
> **B. The model sees tokens, not individual characters (e.g., "Loll" + "ipop"), so it lacks direct access to the character-level sequence.**

---

## Sub-questions

### How does BPE tokenize "Lollipop"?

$$\text{"Lollipop"} \to [\text{"Loll"}, \text{"ipop"}]$$

The model receives two token embeddings. It does not see the individual characters `L, o, l, l, i, p, o, p`.

### Why does character-level access matter for reversal?

$$\text{Reverse("Lollipop")} = \text{"popilloL"}$$

Reversal requires iterating over individual characters in reverse order. But the model operates on tokens, not characters — it must **infer** character composition from token semantics.

### What is "token blindness"?

$$\text{Token} \to \text{embedding vector} \quad (\text{characters are not explicitly represented})$$

The mapping from token to embedding is a lookup table. The internal structure of the token (its individual characters) is not directly accessible to the model's computation.

---

## Main answer

**B. The model sees tokens, not individual characters.**

```
What the model sees:

  Input: "Reverse Lollipop"

  Human sees:  L - o - l - l - i - p - o - p   (8 characters)
  Model sees:  [Loll] [ipop]                    (2 tokens)

  To reverse, model must:
    1. Know "Loll" = L,o,l,l    (not directly available)
    2. Know "ipop" = i,p,o,p    (not directly available)
    3. Concatenate: L,o,l,l,i,p,o,p
    4. Reverse: p,o,p,i,l,l,o,L
    5. Re-tokenize result

  Model cannot easily do steps 1-2 → struggles with reversal
```

```
Token blindness (logic chain):

  BPE ──▶ tokens ≠ characters ──▶ no character-level access
       ──▶ character manipulation fails
```

| Task Type | Difficulty for LLM | Why |
|---|---|---|
| Word meaning | Easy | Semantic embeddings |
| Spelling | Hard | Requires character access |
| **Reversal** | **Hard** | **Characters hidden inside tokens** |
| Counting letters | Hard | Same token blindness |

---
---

# Quiz 2, Question 21 — Manual BPE Execution (Essay)

> **Manual BPE Execution:** Consider a corpus with the following word counts: ("hug", 5), ("pug", 3), ("pun", 2), ("bun", 4). Starting with character tokens, perform the first TWO iterations of the BPE algorithm.
>
> a) Which pair is merged first? What is its frequency?
> b) Which pair is merged second? What is its frequency?

---

## Sub-questions

### What are the character tokens?

$$\text{vocab} = \{h, u, g, p, n, b\}$$

Each word is split into characters:

| Word | Characters | Count |
|---|---|---|
| hug | h u g | 5 |
| pug | p u g | 3 |
| pun | p u n | 2 |
| bun | b u n | 4 |

### How do we count adjacent pairs?

Sum the word count for every occurrence of each pair across all words:

| Pair | Appears in | Total Count |
|---|---|---|
| h u | "hug" × 5 | 5 |
| u g | "hug" × 5 + "pug" × 3 | **8** |
| p u | "pug" × 3 + "pun" × 2 | 5 |
| u n | "pun" × 2 + "bun" × 4 | 6 |
| b u | "bun" × 4 | 4 |

---

## Main answer

### Iteration 1: Merge the most frequent pair

$$\text{Most frequent: } (u, g) \text{ with count } 8$$

Apply merge: `u g → ug`

| Word | After Merge | Count |
|---|---|---|
| hug | h **ug** | 5 |
| pug | p **ug** | 3 |
| pun | p u n | 2 |
| bun | b u n | 4 |

**Answer (a): Pair (u, g) is merged first with frequency 8.**

### Iteration 2: Recount pairs after merge

| Pair | Appears in | Total Count |
|---|---|---|
| h ug | "hug" × 5 | 5 |
| p ug | "pug" × 3 | 3 |
| p u | "pun" × 2 | 2 |
| u n | "pun" × 2 + "bun" × 4 | **6** |
| b u | "bun" × 4 | 4 |

$$\text{Most frequent: } (u, n) \text{ with count } 6$$

Apply merge: `u n → un`

| Word | After Merge | Count |
|---|---|---|
| hug | h ug | 5 |
| pug | p ug | 3 |
| pun | p **un** | 2 |
| bun | b **un** | 4 |

**Answer (b): Pair (u, n) is merged second with frequency 6.**

```
BPE merge history:

  Iteration 1:  (u, g) → "ug"   frequency = 8
  Iteration 2:  (u, n) → "un"   frequency = 6

  Vocabulary after 2 merges: {h, u, g, p, n, b, ug, un}
```

---
---

# Quiz 2, Question 22 — Unigram Probability Calculation (Essay)

> **Unigram Probability Calculation (Semantic Cohesion):** You are tokenizing the word "damage". Your Unigram model has probabilities: P("dam") = 0.03, P("age") = 0.02, P("damage") = 0.0005. Compare the probability score of the split segmentation ["dam", "age"] versus the whole word ["damage"]. Which segmentation will the Viterbi algorithm select?

---

## Sub-questions

### How does the Unigram model score a segmentation?

$$\text{Score}(\mathbf{x}) = \prod_i P(x_i) \quad \text{or equivalently} \quad \log\text{Score} = \sum_i \log P(x_i)$$

Multiply (or sum log-probabilities of) all tokens in the segmentation.

### What is the Viterbi selection criterion?

$$x^* = \arg\max_{\mathbf{x}} \prod_i P(x_i)$$

Choose the segmentation with the **highest** probability score.

---

## Main answer

### Segmentation A: ["dam", "age"]

$$P(\text{"dam", "age"}) = P(\text{"dam"}) \times P(\text{"age"}) = 0.03 \times 0.02 = 0.0006$$

### Segmentation B: ["damage"]

$$P(\text{"damage"}) = 0.0005$$

### Comparison

$$0.0006 > 0.0005$$

```
Score comparison:

  ["dam", "age"]:  0.03 × 0.02 = 0.0006  ← WINNER
  ["damage"]:      0.0005

  Viterbi selects: ["dam", "age"]
```

**The Viterbi algorithm selects ["dam", "age"]** because $0.0006 > 0.0005$.

This illustrates a tradeoff: the split segmentation has higher probability even though "damage" is semantically one word. The Unigram model is purely statistical — it favors whichever decomposition has higher joint probability, regardless of semantic cohesion.

| Segmentation | Tokens | Score | Selected? |
|---|---|---|---|
| ["dam", "age"] | 2 | $0.03 \times 0.02 = 0.0006$ | **Yes** |
| ["damage"] | 1 | $0.0005$ | No |

---
---

# Quiz 2, Question 23 — Byte-Level Analysis of Dog Face Emoji (Essay)

> **Byte-Level Analysis:** The Dog Face emoji is represented by 4 bytes in UTF-8: [F0, 9F, 90, B6]. If you feed this emoji into a completely untrained BBPE tokenizer (base vocab only), what will the output tokens be? Why?

---

## Sub-questions

### What is the BBPE base vocabulary?

$$\text{Base vocab} = \{0\text{x}00, 0\text{x}01, \dots, 0\text{x}FF\} \quad (256 \text{ tokens})$$

All 256 possible byte values. No merges have been learned yet in an untrained tokenizer.

### What is UTF-8 encoding?

$$\text{Unicode codepoint} \to \text{1–4 bytes}$$

Multi-byte encoding. Characters outside ASCII use 2-4 bytes. Emojis typically use 4 bytes.

**The Dog Face emoji (🐶):** U+1F436 → UTF-8 bytes: `F0 9F 90 B6`.

### What happens with no merges?

$$\text{No merges} \implies \text{each byte stays as its own token}$$

An untrained BBPE tokenizer has learned zero merge rules. It can only decompose input into individual bytes from the base vocabulary.

---

## Main answer

**Output tokens: [0xF0, 0x9F, 0x90, 0xB6] — four individual byte tokens.**

$$🐶 \xrightarrow{\text{UTF-8}} [\texttt{F0}, \texttt{9F}, \texttt{90}, \texttt{B6}] \xrightarrow{\text{untrained BBPE}} [\text{token}_\texttt{F0}, \text{token}_\texttt{9F}, \text{token}_\texttt{90}, \text{token}_\texttt{B6}]$$

```
Processing pipeline:

  🐶 (Dog Face Emoji)
    │
    ▼  UTF-8 encode
  [F0, 9F, 90, B6]     (4 raw bytes)
    │
    ▼  BBPE (untrained — no merges available)
  [token_F0, token_9F, token_90, token_B6]   (4 tokens)

  Why 4 tokens?
    • Base vocab has all 256 bytes: F0 ✓, 9F ✓, 90 ✓, B6 ✓
    • No merge rules exist yet → no pairs can be combined
    • Each byte becomes its own token
```

**With a trained tokenizer**, these 4 bytes would likely be merged:

| Tokenizer State | Output | Token Count |
|---|---|---|
| **Untrained (base only)** | **[F0, 9F, 90, B6]** | **4** |
| Partially trained | [F0 9F, 90 B6] | 2 |
| Well-trained | [F0 9F 90 B6] (= 🐶) | 1 |

---
---

# Quiz 2, Question 24 — GPT-2 Regex and Contractions (Essay)

> **Regex Interpretation:** The GPT-2 regex pattern `'s|'t|'re|'ve|'m|...` is designed to handle contractions. If the input is "I'm going", how does this regex split the contraction "I'm"? Why is this linguistically preferable to splitting it as "I" + " ' " + "m"?

---

## Sub-questions

### What does the GPT-2 regex do with contractions?

$$\text{Pattern: } \texttt{'s|'t|'re|'ve|'m} \quad \text{(match contraction suffixes first)}$$

The regex tries to match these contraction patterns **before** other rules. `'m` matches as a single unit.

### How is "I'm" split?

$$\text{"I'm"} \to [\text{"I"}, \text{"'m"}]$$

The regex first matches `I` as a word, then `'m` as a contraction suffix. Result: two meaningful chunks.

---

## Main answer

**The regex splits "I'm" into ["I", "'m"].**

```
GPT-2 regex processing of "I'm going":

  "I'm going"
    │
    ▼  regex matches (in priority order)
  Match 1: "I"    (word pattern: [a-zA-Z]+)
  Match 2: "'m"   (contraction pattern: 'm)
  Match 3: " going" (word pattern with leading space)

  Result: ["I", "'m", " going"]
```

**Why is ["I", "'m"] better than ["I", "'", "m"]?**

$$\text{"'m"} = \text{contraction of "am"} \quad (\text{one morpheme, one meaning})$$

Keeping `'m` as a single token preserves its linguistic meaning as a contraction of "am." Splitting into `"'"` + `"m"` destroys this — the apostrophe and the letter `m` have no independent meaning.

```
Linguistic analysis:

  "I'm" = "I" + "am"

  Good split:  ["I", "'m"]       ← "'m" = morpheme meaning "am"
  Bad split:   ["I", "'", "m"]   ← "'" = punctuation? "m" = letter?
                                    Meaning destroyed across 2 tokens
```

| Split | Tokens | Linguistic Validity |
|---|---|---|
| ["I", "'m"] | 2 | `'m` = "am" (meaningful morpheme) |
| ["I", "'", "m"] | 3 | `'` and `m` are meaningless fragments |
| ["I'm"] | 1 | Valid but wastes vocab on every contraction form |

---
---

# Quiz 2, Question 25 — BPE vs WordPiece Optimization Goals (Essay)

> **Conceptual Contrast:** Explain the fundamental difference in the Optimization Goal between BPE and WordPiece. (Hint: BPE asks "What is most frequent?"; what does WordPiece ask?)

---

## Sub-questions

### What is BPE's optimization goal?

$$\text{BPE merge criterion: } \arg\max_{(a,b)} \text{count}(a, b)$$

BPE asks: **"Which pair appears most often?"** Pure frequency — the most common adjacent pair gets merged regardless of the individual token frequencies.

### What is WordPiece's optimization goal?

$$\text{WordPiece merge criterion: } \arg\max_{(a,b)} \frac{P(ab)}{P(a) \cdot P(b)}$$

WordPiece asks: **"Which pair, when merged, most increases the likelihood of the training data?"** Equivalent to maximizing Pointwise Mutual Information — favoring pairs that co-occur more than expected by chance.

---

## Main answer

**BPE asks: "What is most frequent?"**
**WordPiece asks: "What is most surprisingly co-occurring?"**

$$\text{BPE: } \text{score}(a,b) = \text{count}(ab)$$

$$\text{WordPiece: } \text{score}(a,b) = \frac{\text{count}(ab)}{\text{count}(a) \times \text{count}(b)} \propto \text{PMI}(a,b)$$

```
Concrete example:

  Pair ("t","h"):  count("th") = 10,000
                   count("t") = 50,000    count("h") = 40,000

  Pair ("x","q"):  count("xq") = 100
                   count("x") = 120       count("q") = 110

  BPE score:
    ("t","h") = 10,000        ← BPE picks this (higher count)
    ("x","q") = 100

  WordPiece score:
    ("t","h") = 10,000 / (50,000 × 40,000) = 0.000005
    ("x","q") = 100 / (120 × 110)          = 0.0076    ← WordPiece picks this

  "x" and "q" almost always appear together → high mutual information
  "t" and "h" are common individually → high count but low surprise
```

```
Optimization philosophy:

  BPE:       frequency ──▶ "What pattern is most COMMON?"
                            Merges popular pairs first
                            Bias toward high-frequency language

  WordPiece: likelihood ──▶ "What merge BEST EXPLAINS the data?"
                            Merges surprisingly co-occurring pairs
                            Bias toward statistically meaningful units
```

| Aspect | BPE | WordPiece |
|---|---|---|
| Question asked | "Most frequent pair?" | "Most informative merge?" |
| Score formula | $\text{count}(ab)$ | $\frac{\text{count}(ab)}{\text{count}(a) \cdot \text{count}(b)}$ |
| Equivalent to | Frequency ranking | PMI / likelihood maximization |
| Bias | Common substrings | Statistically surprising co-occurrences |
| Used by | GPT-2, GPT-3, GPT-4, LLaMA | BERT |
| Result | Good compression | Better linguistic units |

---
---
