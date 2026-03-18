# Quiz 1: LLM Fundamentals and Architecture — Tutor Answers (Q1–Q25)

---

# Quiz 1, Question 1 — Three Principal Stages of Building an LLM

> **In this course, what are the three principal stages involved in the process of coding a large language model (LLM) from scratch?**
>
> **B. Implementing the LLM architecture and data preparation; pretraining the LLM to create a foundation model; and fine-tuning the foundation model**

---

## Sub-questions

### What is LLM architecture and data preparation?

Defining the neural network structure (layers, attention heads, embedding dimensions) and building the data pipeline (tokenization, batching, sequence formatting).

$$\text{Architecture: } \theta = \{W_Q, W_K, W_V, W_O, W_{FFN}, \dots\}$$

### What is pretraining?

$$\min_\theta \; -\sum_{t=1}^{T} \log P_\theta(x_t \mid x_{<t})$$

Training on a massive unlabeled text corpus using next-token prediction. Produces a **foundation model** — a general-purpose language model with no task-specific behavior.

### What is fine-tuning?

$$\theta_\text{fine-tuned} = \theta_\text{pretrained} - \alpha \nabla \mathcal{L}_\text{task}$$

Adapting the pretrained model to a specific task or behavior (e.g., instruction following, Q&A) using a smaller, labeled dataset. Changes the model from "predicts text" to "follows instructions."

---

## Main answer

**B. Implementing the LLM architecture and data preparation; pretraining the LLM to create a foundation model; and fine-tuning the foundation model**

```
The three stages:

  Stage 1                    Stage 2                    Stage 3
  ┌──────────────────┐      ┌──────────────────┐      ┌──────────────────┐
  │ Architecture +   │ ───▶ │ Pretraining      │ ───▶ │ Fine-tuning      │
  │ Data Preparation │      │                  │      │                  │
  │                  │      │ Massive corpus   │      │ Task-specific    │
  │ • Define layers  │      │ Next-token pred  │      │ labeled data     │
  │ • Tokenizer      │      │ Self-supervised  │      │ Supervised       │
  │ • Data pipeline  │      │                  │      │                  │
  │                  │      │ Output:          │      │ Output:          │
  │                  │      │ Foundation Model │      │ Task Model       │
  └──────────────────┘      └──────────────────┘      └──────────────────┘
```

| Stage | Input | Method | Output |
|---|---|---|---|
| 1. Architecture + Data | Design choices | Engineering | Model skeleton + data pipeline |
| 2. Pretraining | Massive unlabeled text | Self-supervised (next-token) | Foundation model |
| 3. Fine-tuning | Small labeled dataset | Supervised | Task-specific model |

---
---

# Quiz 1, Question 2 — GPT vs Original Transformer Architecture

> **What is the primary architectural difference between the original Transformer model proposed in 'Attention is All You Need' and the subsequent GPT series of models?**
>
> **A. The GPT series is exclusively a decoder-only architecture, whereas the original Transformer utilized both an encoder and a decoder.**

---

## Sub-questions

### What is the original Transformer?

Proposed in Vaswani et al. (2017). **Encoder-decoder** architecture designed for sequence-to-sequence tasks (e.g., machine translation).

$$\text{Input} \xrightarrow{\text{Encoder}} \text{Context Vectors} \xrightarrow{\text{Decoder}} \text{Output}$$

The encoder reads the full input; the decoder generates the output token-by-token using both the encoder's context and its own previous outputs.

### What is a decoder-only architecture?

$$P(x_t \mid x_1, \dots, x_{t-1})$$

Only the decoder half is used. No separate encoder. The model processes a single sequence autoregressively — each token attends only to preceding tokens via causal masking.

### What is an encoder-only architecture?

$$\text{BERT}: P(x_\text{mask} \mid x_{\text{all other}})$$

Only the encoder half. Bidirectional attention — each token sees the full sequence. Used for classification and understanding tasks, not generation.

---

## Main answer

**A. The GPT series is exclusively a decoder-only architecture, whereas the original Transformer utilized both an encoder and a decoder.**

```
Original Transformer (2017):           GPT Series:

  ┌──────────┐   ┌──────────┐         ┌──────────┐
  │ ENCODER  │──▶│ DECODER  │         │ DECODER  │
  │          │   │          │         │  ONLY    │
  │ Bi-dir   │   │ Causal   │         │          │
  │ attention│   │ + cross  │         │ Causal   │
  │          │   │ attention │         │ attention│
  └──────────┘   └──────────┘         └──────────┘
  Input seq       Output seq          Single sequence
  (full view)     (left-to-right)     (left-to-right)
```

| Property | Original Transformer | GPT |
|---|---|---|
| Architecture | Encoder + Decoder | Decoder only |
| Attention in encoder | Bidirectional | N/A |
| Attention in decoder | Causal + Cross-attention | Causal only |
| Primary task | Seq-to-seq (translation) | Autoregressive generation |
| Input/Output | Separate sequences | Single sequence |

---
---

# Quiz 1, Question 3 — BERT vs GPT for Sentiment Analysis

> **Which of the following best explains why the BERT model is particularly well-suited for tasks like sentiment analysis compared to a GPT model?**
>
> **B. BERT's training objective of predicting masked words in a sequence allows it to understand context from both left and right directions**

---

## Sub-questions

### What is BERT's training objective?

$$\text{Masked Language Modeling (MLM)}: P(x_\text{mask} \mid x_1, \dots, x_{i-1}, x_{i+1}, \dots, x_T)$$

Randomly mask ~15% of tokens; predict them using the full surrounding context (both left and right). This produces **bidirectional** representations.

### What is GPT's training objective?

$$\text{Causal LM}: P(x_t \mid x_1, \dots, x_{t-1})$$

Predict the next token using only **preceding** tokens. The representation at position $t$ has no information about tokens after position $t$.

### What is sentiment analysis?

Classification task: given a full sentence, predict a label (positive / negative / neutral). Requires understanding the **entire** sentence — both the words before and after each position matter.

**Example:** "The movie was not good, but the ending was _amazing_." — the word "not" modifies "good" (left context), and "amazing" at the end shifts overall sentiment (right context).

---

## Main answer

**B. BERT's training objective of predicting masked words in a sequence allows it to understand context from both left and right directions**

```
BERT (Bidirectional):

  "The movie was [MASK] good"

  [MASK] sees:  ◀── "The movie was"  AND  "good" ──▶
                    LEFT context          RIGHT context

  → Rich, full-sentence understanding


GPT (Unidirectional):

  "The movie was ___"

  Position 4 sees:  ◀── "The movie was"  ONLY
                        LEFT context

  → Cannot see what comes after
```

$$\text{BERT: } h_i = f(x_1, \dots, x_{i-1}, x_{i+1}, \dots, x_T) \quad \text{(full context)}$$

$$\text{GPT: } h_i = f(x_1, \dots, x_{i-1}) \quad \text{(left context only)}$$

| Property | BERT | GPT |
|---|---|---|
| Context direction | Bidirectional (left + right) | Unidirectional (left only) |
| Training objective | Masked Language Model | Next-token prediction |
| Representation quality for classification | Richer (sees full sentence) | Partial (misses right context) |
| Best for | Understanding tasks (classification, NER) | Generation tasks |

---
---

# Quiz 1, Question 4 — Emergent Behavior in LLMs

> **The term 'emergent behavior' in the context of LLMs describes which phenomenon?**
>
> **D. The model's capacity to perform tasks it was not explicitly trained for, such as translation or arithmetic, as a byproduct of its primary training.**

---

## Sub-questions

### What is the primary training objective of LLMs?

$$\min_\theta \; -\sum_t \log P_\theta(x_t \mid x_{<t})$$

Next-token prediction on a large text corpus. The model is never explicitly given translation pairs, math problems, or coding exercises as labeled tasks.

### What does "emergent" mean?

A capability that appears at sufficient scale (parameters, data, compute) without being explicitly optimized for. Not present in smaller models; suddenly appears as the model grows.

### What are examples of emergent behaviors?

- Translation (trained on multilingual text, never on parallel corpora)
- Arithmetic (learned from text containing numbers)
- Chain-of-thought reasoning (appears at ~100B+ parameters)
- Code generation (learned from code in the training corpus)

---

## Main answer

**D. The model's capacity to perform tasks it was not explicitly trained for, such as translation or arithmetic, as a byproduct of its primary training.**

```
Emergence logic chain:

  Train on next-token prediction
       │
       ▼  (massive scale: data + parameters)
  Model learns internal representations
       │
       ▼
  Representations encode translation, math, reasoning
       │
       ▼
  Abilities "emerge" without explicit supervision
```

```
Scale vs. capability:

  Task
  Performance
   ↑
   │              ╱── emergent capability
   │             ╱    (sudden jump)
   │          __╱
   │  ───────╱
   │  flat (no ability)
   └──────────────────────────▶ Model Scale (params)
             ↑
       threshold where
       capability emerges
```

| Property | Explicitly Trained | Emergent |
|---|---|---|
| Objective | Directly optimized | Byproduct of primary training |
| Appears at small scale | Yes | No |
| Examples | Next-token prediction | Translation, arithmetic, reasoning |
| Requires | Task-specific data | Sufficient scale + diverse data |

---
---

# Quiz 1, Question 5 — Autoregressive Pre-training

> **The pre-training process for a GPT model is described as 'auto-regressive.' What does this term signify?**
>
> **A. The model's previous outputs are fed back as inputs for generating subsequent outputs.**

---

## Sub-questions

### What does "autoregressive" mean mathematically?

$$P(x_1, x_2, \dots, x_T) = \prod_{t=1}^{T} P(x_t \mid x_1, \dots, x_{t-1})$$

The joint probability of a sequence is decomposed into a product of conditional probabilities. Each token depends on all previous tokens.

### How does this work during generation?

$$x_t = \text{sample}\!\left(P_\theta(\cdot \mid x_1, \dots, x_{t-1})\right)$$

At each step, the model produces a probability distribution over the vocabulary, samples one token, appends it to the sequence, and uses the extended sequence as input for the next step.

---

## Main answer

**A. The model's previous outputs are fed back as inputs for generating subsequent outputs.**

```
Autoregressive generation:

  Step 1: Input: [The]        → Predict: "cat"
  Step 2: Input: [The, cat]   → Predict: "sat"
  Step 3: Input: [The, cat, sat] → Predict: "on"
  ...
  Each output becomes part of the next input
```

```
Feedback loop:

  ┌─────────┐    ┌─────────┐    ┌─────────┐
  │ Input   │───▶│  Model  │───▶│ Output  │──┐
  │ x₁..xₜ₋₁│    │  P(xₜ)  │    │  xₜ     │  │
  └─────────┘    └─────────┘    └─────────┘  │
       ▲                                      │
       └──────────── fed back as input ───────┘
```

$$P(\text{"The cat sat on"}) = P(\text{The}) \cdot P(\text{cat} \mid \text{The}) \cdot P(\text{sat} \mid \text{The cat}) \cdot P(\text{on} \mid \text{The cat sat})$$

| Property | Autoregressive | Non-autoregressive |
|---|---|---|
| Generation order | Left-to-right, one token at a time | All tokens simultaneously |
| Each step uses | All previous outputs | Fixed input only |
| Example model | GPT | BERT (not generative) |
| Feedback loop | Yes | No |

---
---

# Quiz 1, Question 6 — GPT-3 as a Few-Shot Learner

> **The GPT-3 paper, 'Language Models are Few-Shot Learners,' primarily argued that the model excelled in which capacity?**
>
> **A. Learning a new task effectively after being provided with a small number of examples in the prompt.**

---

## Sub-questions

### What is few-shot learning?

$$\text{Prompt} = [\text{Example}_1, \text{Example}_2, \dots, \text{Example}_k, \text{Query}]$$

Providing $k$ input-output examples in the prompt (typically $k = 1\text{–}32$) so the model can infer the task pattern and apply it to a new query. No gradient updates — pure inference.

### What is in-context learning?

The mechanism by which a model performs few-shot learning: the examples are in the **context window**, and the model uses attention to identify the pattern and generalize.

### How does few-shot differ from fine-tuning?

| Aspect | Few-shot | Fine-tuning |
|---|---|---|
| Gradient updates | None | Yes |
| Training data needed | 1–32 examples in prompt | Thousands+ labeled examples |
| Model weights change | No | Yes |
| Speed | Instant | Hours/days |

---

## Main answer

**A. Learning a new task effectively after being provided with a small number of examples in the prompt.**

```
Few-shot prompting:

  Prompt:
  ┌───────────────────────────────────────────┐
  │  "Translate English to French:"           │
  │  "sea otter" → "loutre de mer"            │  ← Example 1
  │  "cheese" → "fromage"                     │  ← Example 2
  │  "plaid" →                                │  ← Query
  └───────────────────────────────────────────┘

  Model output: "plaid" (or "tissu écossais")

  No fine-tuning. No gradient updates. Pure pattern recognition.
```

```
GPT-3 learning spectrum:

  Zero-shot        Few-shot         Fine-tuning
  (0 examples)     (1-32 examples)  (thousands+)
       │                │                │
  Task described    Examples in      Train on labeled
  in plain text     the prompt       dataset
       │                │                │
  No updates        No updates       Gradient updates
```

The key insight of the GPT-3 paper: at sufficient scale (175B parameters), few-shot prompting can **match or approach** the performance of smaller fine-tuned models, without modifying any weights.

---
---

# Quiz 1, Question 7 — Encoder Block Function

> **What is the principal function of the encoder block within the original Transformer architecture?**
>
> **A. To convert input tokens into high-dimensional vector embeddings that capture their semantic meaning and context.**

---

## Sub-questions

### What is an input token?

$$x_i \in \{1, 2, \dots, V\} \quad \text{(integer ID from vocabulary)}$$

A discrete symbol from the vocabulary. Carries no semantic information by itself — just an index.

### What is a vector embedding?

$$e_i = \text{Embedding}(x_i) \in \mathbb{R}^{d_\text{model}}$$

A dense, continuous vector of dimension $d_\text{model}$ (e.g., 512 or 768). Encodes semantic meaning: similar words have similar vectors.

### What does "capture context" mean?

Through self-attention, the encoder transforms each token's embedding based on all other tokens in the input:

$$h_i = f(e_1, e_2, \dots, e_T) \quad \text{(context-dependent representation)}$$

The output vector for "bank" differs depending on whether the input contains "river" or "money."

---

## Main answer

**A. To convert input tokens into high-dimensional vector embeddings that capture their semantic meaning and context.**

```
Encoder pipeline:

  Token IDs          Embeddings           Contextualized Vectors
  ┌────────┐        ┌────────────┐       ┌─────────────────────┐
  │ [42]   │  ───▶  │ [0.2, 0.8, │ ───▶  │ [0.5, -0.1, 0.9,   │
  │ [17]   │  Embed │  -0.3, ...]│ Self- │  0.2, ...]          │
  │ [89]   │  Layer │ [0.1, -0.5,│ Attn  │ [-0.3, 0.7, 0.4,   │
  │ [3]    │        │  0.7, ...] │ + FFN │  -0.1, ...]         │
  └────────┘        │ ...        │       │ ...                 │
                    └────────────┘       └─────────────────────┘
  No meaning         Static meaning       Context-aware meaning
```

$$\text{Token} \xrightarrow{\text{Embedding}} \mathbb{R}^{d} \xrightarrow{\text{Self-Attention + FFN}} \mathbb{R}^{d} \text{ (contextualized)}$$

| Stage | Representation | Context-aware? |
|---|---|---|
| Input token | Integer ID | No |
| After embedding layer | Static vector | No (same vector regardless of context) |
| After encoder block | Contextualized vector | Yes (depends on all other tokens) |

---
---

# Quiz 1, Question 8 — LLM vs Transformer Terminology

> **Why is it inaccurate to use the terms 'LLM' and 'Transformer' interchangeably?**
>
> **B. LLMs can be built on older architectures like RNNs, and Transformers can be used for non-language tasks like computer vision.**

---

## Sub-questions

### What is an LLM?

A **Large Language Model** — any large-scale neural network trained on text for language tasks. Defined by its application domain (language) and scale, not its architecture.

### What is a Transformer?

An **architecture** (attention-based neural network design) introduced in 2017. A building block that can be used for many domains.

### Can LLMs use non-Transformer architectures?

Yes. Early large language models used RNNs (e.g., ELMo) and LSTMs. Recent work explores state-space models (Mamba) as alternatives to Transformers.

### Can Transformers be used outside language?

Yes. Vision Transformers (ViT) for images, AlphaFold for protein structure, Decision Transformers for reinforcement learning, Audio Spectrogram Transformers for sound.

---

## Main answer

**B. LLMs can be built on older architectures like RNNs, and Transformers can be used for non-language tasks like computer vision.**

```
The two concepts are independent axes:

                     Architecture
                  RNN    Transformer    SSM (Mamba)
           ┌──────────┬────────────┬──────────────┐
  Language │  ELMo    │  GPT, BERT │  Mamba-LM    │  ← LLMs
  Domain   ├──────────┼────────────┼──────────────┤
  Vision   │  —       │  ViT       │  —            │  ← Not LLMs
           ├──────────┼────────────┼──────────────┤
  Protein  │  —       │  AlphaFold │  —            │  ← Not LLMs
           └──────────┴────────────┴──────────────┘
```

| Term | Defined by | Scope |
|---|---|---|
| LLM | Domain (language) + scale | Any architecture |
| Transformer | Architecture (attention mechanism) | Any domain |
| Overlap | GPT, LLaMA, BERT | Transformer-based LLMs |

---
---

# Quiz 1, Question 9 — Fine-tuning for Private Data

> **A financial firm wants to create an AI assistant that answers employee questions based on its private, internal market analysis reports. Starting with a powerful pre-trained model like GPT-4, what is the essential next stage?**
>
> **B. Fine-tuning the model on a labeled dataset consisting of the firm's internal reports and corresponding Q&A pairs.**

---

## Sub-questions

### What does the pre-trained model know?

General language knowledge from public internet text. It has no knowledge of the firm's private reports, internal terminology, or proprietary analysis.

### What is fine-tuning?

$$\theta_\text{new} = \theta_\text{pretrained} - \alpha \sum_{\text{internal data}} \nabla \mathcal{L}_\text{task}$$

Training the pretrained model further on domain-specific data so it learns to answer questions about the firm's private content.

### Why not just prompt the model?

The model cannot answer questions about documents it has never seen. The private reports are not in its training data. Fine-tuning encodes this knowledge into the model's weights.

---

## Main answer

**B. Fine-tuning the model on a labeled dataset consisting of the firm's internal reports and corresponding Q&A pairs.**

```
Pipeline for private AI assistant:

  ┌──────────────┐     ┌────────────────────┐     ┌──────────────────┐
  │ GPT-4        │────▶│ Fine-tune on        │────▶│ Private Q&A      │
  │ (pretrained) │     │ internal reports    │     │ Assistant        │
  │              │     │ + Q&A pairs         │     │                  │
  │ General      │     │ Supervised training │     │ Domain-specific  │
  │ knowledge    │     │                    │     │ knowledge        │
  └──────────────┘     └────────────────────┘     └──────────────────┘
```

| Approach | Access to Private Data | Quality |
|---|---|---|
| Pre-trained model only | No | Cannot answer private questions |
| **Fine-tuning on internal data** | **Yes (in weights)** | **High — learns domain patterns** |
| RAG (retrieval) | Yes (at inference) | Good — alternative approach |

---
---

# Quiz 1, Question 10 — Self-Attention as the Secret Sauce

> **What is the 'secret sauce' within the Transformer architecture that allows it to effectively capture long-range dependencies in text?**
>
> **C. The self-attention mechanism, which weighs the importance of all words in the input relative to each other.**

---

## Sub-questions

### What is a long-range dependency?

A semantic relationship between words far apart in a sequence.

**Example:** "The **cat**, which sat on the mat and played with yarn all afternoon, **was** tired." — "was" must agree with "cat" (20+ tokens apart).

### What is self-attention?

$$\text{Attention}(Q, K, V) = \text{Softmax}\!\left(\frac{QK^T}{\sqrt{d_k}}\right) V$$

Each token computes a weighted sum of all other tokens' representations. The weights are determined by query-key dot products — how "relevant" each pair of tokens is.

### Why can't RNNs capture long-range dependencies well?

$$h_t = f(h_{t-1}, x_t)$$

RNNs process tokens sequentially. Information from early tokens must survive through every intermediate hidden state. In practice, gradients vanish over long sequences — early tokens are "forgotten."

### Why does self-attention solve this?

Every token directly attends to every other token in a single operation. Distance between tokens is irrelevant — "cat" and "was" have a direct connection regardless of sequence length.

$$\text{Path length: } O(1) \text{ (self-attention)} \quad \text{vs.} \quad O(T) \text{ (RNN)}$$

---

## Main answer

**C. The self-attention mechanism, which weighs the importance of all words in the input relative to each other.**

```
RNN: Information must travel step-by-step

  cat ──▶ which ──▶ sat ──▶ on ──▶ ... ──▶ was
  │                                          │
  └── signal degrades over 20 steps ─────────┘


Self-Attention: Direct connection between all pairs

  cat ──────────────────────────────────── was
   ↕      ↕       ↕      ↕      ↕         ↕
  which  sat     on     the    mat    afternoon

  Every word "sees" every other word in one step
```

| Property | RNN | Self-Attention |
|---|---|---|
| Path between distant tokens | $O(T)$ steps | $O(1)$ direct |
| Long-range dependencies | Weak (vanishing gradient) | Strong (direct attention) |
| Parallelizable | No (sequential) | Yes (all pairs at once) |
| Compute cost | $O(T)$ | $O(T^2)$ per layer |

---
---

# Quiz 1, Question 11 — Unsupervised Learning in GPT Pre-training

> **How do GPT models achieve 'unsupervised learning' during the pre-training phase?**
>
> **B. By using the inherent structure of the text itself, where the next word in a sentence serves as the 'label' for the preceding words.**

---

## Sub-questions

### What is unsupervised learning?

Learning from data without human-provided labels. No one manually annotates the training examples.

### What is self-supervised learning?

A specific form of unsupervised learning where the labels are derived automatically from the data itself. GPT's pre-training is technically **self-supervised**.

### How does text provide its own labels?

$$\text{Input: } [x_1, x_2, \dots, x_{t-1}] \quad \text{Label: } x_t$$

Every sequence of text contains its own supervision signal: the next word. No human annotation needed — the structure of language is the label.

**Example:** "The cat sat on the" → label = "mat". The text itself provides this.

---

## Main answer

**B. By using the inherent structure of the text itself, where the next word in a sentence serves as the 'label' for the preceding words.**

```
Self-supervised labeling (automatic):

  Raw text: "The cat sat on the mat"

  Training pair 1:  Input: [The]              → Label: cat
  Training pair 2:  Input: [The, cat]         → Label: sat
  Training pair 3:  Input: [The, cat, sat]    → Label: on
  Training pair 4:  Input: [The, cat, sat, on] → Label: the
  Training pair 5:  Input: [The, cat, sat, on, the] → Label: mat

  No human annotator needed. Text IS the label.
```

$$\mathcal{L} = -\sum_{t=1}^{T} \log P_\theta(x_t \mid x_1, \dots, x_{t-1})$$

| Aspect | Traditional Supervised | GPT Self-supervised |
|---|---|---|
| Labels | Human-annotated | Derived from text itself |
| Label for position $t$ | External annotation | Next word $x_t$ |
| Scalability | Limited by annotation cost | Unlimited (any text works) |
| Data source | Labeled datasets | Raw internet text |

---
---

# Quiz 1, Question 12 — GPT-3 Training Dataset Composition

> **What constituted the largest portion (around 60%) of the 300 billion token dataset used to pre-train GPT-3?**
>
> **D. A massive, filtered crawl of the public internet known as Common Crawl.**

---

## Sub-questions

### What is Common Crawl?

A nonprofit that continuously crawls the web and provides petabytes of raw HTML data. Publicly available. Contains billions of web pages — news, blogs, forums, etc.

### Why filter Common Crawl?

Raw Common Crawl contains duplicates, spam, low-quality text, and harmful content. GPT-3 used a **filtered** version: deduplicated, quality-scored, and cleaned.

### What were the other data sources?

- **WebText2** (~22%): curated web pages linked from Reddit (high quality)
- **Books1 + Books2** (~16%): book corpora
- **Wikipedia** (~3%): high-quality encyclopedic text

---

## Main answer

**D. A massive, filtered crawl of the public internet known as Common Crawl.**

```
GPT-3 training data composition (300B tokens):

  ┌────────────────────────────────────────────┐
  │ Common Crawl (filtered)    ~60%            │
  │ ████████████████████████████████████████    │
  ├────────────────────────────────────────────┤
  │ WebText2                   ~22%            │
  │ ██████████████                             │
  ├────────────────────────────────────────────┤
  │ Books1 + Books2            ~16%            │
  │ ██████████                                 │
  ├────────────────────────────────────────────┤
  │ Wikipedia                  ~3%             │
  │ ██                                         │
  └────────────────────────────────────────────┘
```

| Source | Approx. % of Tokens | Quality | Role |
|---|---|---|---|
| **Common Crawl** | **~60%** | Medium (filtered) | Volume — broad web knowledge |
| WebText2 | ~22% | High (Reddit-curated) | Quality web content |
| Books | ~16% | High | Long-form, structured text |
| Wikipedia | ~3% | Very high | Factual, encyclopedic |

---
---

# Quiz 1, Question 13 — Decoder Inputs in Encoder-Decoder Transformer

> **In the simplified Transformer architecture for translation, what two key inputs does the decoder receive to predict the next word in the output sequence?**
>
> **C. The vector embeddings from the encoder and the partial output text that has been generated so far.**

---

## Sub-questions

### What does the encoder output?

$$H_\text{enc} = \text{Encoder}(x_1, \dots, x_S) \in \mathbb{R}^{S \times d}$$

Contextualized vector representations of the entire input (source) sequence. Each input token has a rich vector encoding its meaning in context.

### What is the partial output (target prefix)?

$$y_{<t} = [y_1, y_2, \dots, y_{t-1}]$$

The tokens generated so far in the output language. During training, this is the ground-truth prefix; during inference, it is the model's own previous outputs.

### How does the decoder use both inputs?

The decoder has two attention mechanisms:
1. **Causal self-attention** on the partial output $y_{<t}$
2. **Cross-attention** on the encoder outputs $H_\text{enc}$

$$\text{Cross-Attention}(Q_\text{dec}, K_\text{enc}, V_\text{enc})$$

---

## Main answer

**C. The vector embeddings from the encoder and the partial output text that has been generated so far.**

```
Decoder's two inputs:

  Input 1: Encoder output          Input 2: Partial output
  ┌─────────────────────┐         ┌─────────────────────┐
  │ "Le chat est assis"  │         │ "The cat is"        │
  │ → [h₁, h₂, h₃, h₄] │         │ (generated so far)  │
  │ (context vectors)    │         │                     │
  └─────────┬───────────┘         └─────────┬───────────┘
            │                               │
            ▼                               ▼
      ┌──────────────────────────────────────────┐
      │              DECODER                      │
      │  Causal Self-Attn (on partial output)    │
      │  Cross-Attention  (on encoder vectors)   │
      │  Feed-Forward                            │
      └────────────────┬─────────────────────────┘
                       │
                       ▼
                  Predict: "sitting"
```

| Decoder Input | Source | Attention Type | Purpose |
|---|---|---|---|
| Encoder vectors | Source sentence encoding | Cross-attention | "What to translate" |
| Partial output | Previously generated tokens | Causal self-attention | "What I've said so far" |

---
---

# Quiz 1, Question 14 — Old NLP Model vs Modern LLM for Translation

> **Which statement best describes the relationship between an old NLP model for a specific task (e.g., translation) and a modern LLM?**
>
> **A. The old NLP model was trained specifically for translation, while the LLM was trained on next-word prediction and gained translation as an emergent property.**

---

## Sub-questions

### How were old NLP translation models trained?

$$\theta^* = \arg\min_\theta \sum_{(x,y) \in \mathcal{D}_\text{parallel}} -\log P_\theta(y \mid x)$$

Trained on **parallel corpora** — aligned sentence pairs in source and target languages. The entire training objective was translation. Could only translate.

### How do LLMs learn translation?

$$\theta^* = \arg\min_\theta -\sum_t \log P_\theta(x_t \mid x_{<t})$$

Trained on next-token prediction over multilingual text. The training data happens to contain translations, bilingual text, and cross-lingual patterns. Translation ability emerges as a byproduct.

---

## Main answer

**A. The old NLP model was trained specifically for translation, while the LLM was trained on next-word prediction and gained translation as an emergent property.**

```
Old NLP model:                    Modern LLM:

  Parallel corpus                  Massive diverse text
  (en↔fr pairs)                    (includes multilingual)
       │                                │
       ▼                                ▼
  Train for translation            Train for next-token prediction
       │                                │
       ▼                                ▼
  Can ONLY translate               Can translate + summarize +
                                   code + reason + ... (emergent)
```

| Property | Old NLP Translation Model | Modern LLM |
|---|---|---|
| Training data | Parallel sentence pairs | General text corpus |
| Training objective | Translation loss | Next-token prediction |
| Translation ability | Explicitly trained | Emergent property |
| Other capabilities | None | Many (generalist) |

---
---

# Quiz 1, Question 15 — Decoder Block in GPT

> **What is the primary role of the 'decoder' block in a GPT-style, decoder-only architecture?**
>
> **A. To iteratively predict the next token in a sequence based on all the preceding tokens.**

---

## Sub-questions

### What is a decoder block?

A single Transformer layer consisting of:

$$\text{Decoder Block} = \text{Causal Self-Attention} \rightarrow \text{Layer Norm} \rightarrow \text{Feed-Forward} \rightarrow \text{Layer Norm}$$

GPT stacks multiple decoder blocks (e.g., 96 in GPT-3).

### What does "causal" mean here?

$$\text{Token } t \text{ attends only to } \{x_1, x_2, \dots, x_t\}$$

Future tokens are masked. This enforces the autoregressive property — predictions can only use past information.

### What does "iteratively" mean?

$$x_1 \rightarrow x_2 \rightarrow x_3 \rightarrow \dots \rightarrow x_T$$

One token is generated per forward pass. The new token is appended and the process repeats.

---

## Main answer

**A. To iteratively predict the next token in a sequence based on all the preceding tokens.**

```
GPT decoder operation:

  Input: [x₁, x₂, ..., xₜ₋₁]
              │
              ▼
  ┌─────────────────────┐
  │ Causal Self-Attention│ ← each token sees only past tokens
  ├─────────────────────┤
  │ Feed-Forward Network │
  ├─────────────────────┤
  │  × N layers          │ ← stacked decoder blocks
  └──────────┬──────────┘
              │
              ▼
  ┌─────────────────────┐
  │ Linear Head + Softmax│
  └──────────┬──────────┘
              │
              ▼
  P(xₜ | x₁, ..., xₜ₋₁) → sample xₜ → append → repeat
```

| Component | Role |
|---|---|
| Causal self-attention | Relate each token to all preceding tokens |
| Feed-forward network | Transform representations nonlinearly |
| Stacking ($N$ blocks) | Build increasingly abstract representations |
| Linear head | Map final representation to vocabulary logits |

---
---

# Quiz 1, Question 16 — torch.tensor vs torch.from_numpy

> **When creating tensors from a NumPy array ary, what is the key difference between torch.tensor(ary) and torch.from_numpy(ary)?**
>
> **C. torch.tensor creates a copy, while torch.from_numpy shares memory.**

---

## Sub-questions

### What does "creates a copy" mean?

$$\text{torch.tensor(ary)} \rightarrow \text{new memory allocation}$$

The tensor gets its own independent memory. Modifying the original NumPy array does not affect the tensor, and vice versa.

### What does "shares memory" mean?

$$\text{torch.from\_numpy(ary)} \rightarrow \text{same memory buffer}$$

The tensor and the NumPy array point to the same underlying data. Modifying one changes the other.

---

## Main answer

**C. torch.tensor creates a copy, while torch.from_numpy shares memory.**

```python
import numpy as np, torch

ary = np.array([1.0, 2.0, 3.0])

# Copy — independent memory
t1 = torch.tensor(ary)
ary[0] = 99.0
# t1[0] is still 1.0  ← unaffected

# Shared — same memory
t2 = torch.from_numpy(ary)
ary[1] = 88.0
# t2[1] is now 88.0   ← changed!
```

```
Memory layout:

  torch.tensor(ary):
  ┌──────────┐     ┌──────────┐
  │ NumPy    │     │ Tensor   │
  │ [1, 2, 3]│     │ [1, 2, 3]│    ← separate memory
  └──────────┘     └──────────┘

  torch.from_numpy(ary):
  ┌──────────┐
  │ [1, 2, 3]│  ← shared memory
  └──────────┘
    ↑       ↑
  NumPy   Tensor
```

| Property | `torch.tensor()` | `torch.from_numpy()` |
|---|---|---|
| Memory | New copy | Shared buffer |
| Modify NumPy → affects tensor? | No | Yes |
| Modify tensor → affects NumPy? | No | Yes |
| Use case | Independence needed | Zero-copy efficiency |

---
---

# Quiz 1, Question 17 — view() RuntimeError After Transpose

> **You have a tensor t of shape (2, 3). You call t = t.T (transpose). Why does t.view(6) raise a RuntimeError, but t.reshape(6) works?**
>
> **C. The transpose operation makes the tensor non-contiguous in memory.**

---

## Sub-questions

### What is contiguous memory?

Elements stored in a single, unbroken block in row-major (C) order. A $(2,3)$ tensor stores elements as: $[a_{00}, a_{01}, a_{02}, a_{10}, a_{11}, a_{12}]$.

### What does transpose do to memory layout?

$$t = \begin{bmatrix} a & b & c \\ d & e & f \end{bmatrix} \xrightarrow{.T} \begin{bmatrix} a & d \\ b & e \\ c & f \end{bmatrix}$$

`.T` does **not** rearrange memory. It only changes the stride metadata. The data in memory is still $[a, b, c, d, e, f]$, but the new shape $(3,2)$ expects $[a, d, b, e, c, f]$. This mismatch = non-contiguous.

### What is the difference between view() and reshape()?

- `view()`: Requires contiguous memory. Fails if not contiguous.
- `reshape()`: Calls `.contiguous().view()` internally if needed. Always works (may copy data).

---

## Main answer

**C. The transpose operation makes the tensor non-contiguous in memory.**

```
Memory layout after transpose:

  Original (2,3) — contiguous:
  Memory: [a, b, c, d, e, f]
  Access: [0][0]=a, [0][1]=b, [0][2]=c, [1][0]=d, [1][1]=e, [1][2]=f  ✓

  After .T → (3,2) — non-contiguous:
  Memory: [a, b, c, d, e, f]   ← same memory, not rearranged!
  Access: [0][0]=a, [0][1]=d, [1][0]=b, [1][1]=e, [2][0]=c, [2][1]=f
                         ↑ jump in memory!

  view(6) requires sequential memory → RuntimeError!
  reshape(6) calls .contiguous() first → copies data → works!
```

| Method | Requires Contiguous | Handles Non-contiguous | May Copy Data |
|---|---|---|---|
| `view()` | Yes (crashes if not) | No | Never |
| `reshape()` | No | Yes (auto `.contiguous()`) | Only if needed |

---
---

# Quiz 1, Question 18 — Matrix Multiplication (@ vs *)

> **Given tensor A with shape (2, 3) and B with shape (3, 2). Which statement is true regarding A @ B and A * B?**
>
> **D. A @ B works, but A * B fails.**

---

## Sub-questions

### What is `@` (matrix multiplication)?

$$A @ B: \quad (2, 3) \times (3, 2) \rightarrow (2, 2)$$

Standard matrix multiplication. Inner dimensions must match: $A$'s columns (3) = $B$'s rows (3). Result shape = outer dimensions.

### What is `*` (element-wise multiplication)?

$$A * B: \quad (2, 3) \odot (3, 2) \rightarrow \text{?}$$

Hadamard product — multiplies corresponding elements. Requires shapes to be **broadcastable** (compatible in each dimension). $(2,3)$ and $(3,2)$ are not broadcastable: dimension 0 is $2$ vs $3$ (neither is 1), dimension 1 is $3$ vs $2$ (neither is 1).

### What is broadcasting?

Tensors with different shapes can be element-wise operated if, for each dimension, the sizes are equal or one of them is 1. $(2,3)$ and $(3,2)$ fail this check in both dimensions.

---

## Main answer

**D. A @ B works, but A * B fails.**

```
A @ B (matrix multiplication):

  A (2,3)  @  B (3,2)  →  C (2,2)

  [a b c]     [x u]       [ax+by+cz  au+bv+cw]
  [d e f]  @  [y v]   =   [dx+ey+fz  du+ev+fw]
              [z w]

  Inner dims match (3 == 3) → ✓


A * B (element-wise):

  A (2,3)  *  B (3,2)  →  ERROR

  Dim 0: 2 vs 3 → not equal, neither is 1 → ✗
  Dim 1: 3 vs 2 → not equal, neither is 1 → ✗

  Not broadcastable → RuntimeError!
```

| Operation | Symbol | Shape Rule | $(2,3)$ and $(3,2)$ | Result |
|---|---|---|---|---|
| Matrix multiply | `@` | Inner dims match | $3 == 3$ ✓ | $(2,2)$ |
| Element-wise | `*` | Broadcastable | $2 \neq 3$, $3 \neq 2$ ✗ | Error |

---
---

# Quiz 1, Question 19 — Forgetting optimizer.zero_grad()

> **What happens if you forget to call optimizer.zero_grad() inside your training loop?**
>
> **C. Gradients from previous batches accumulate (add up) to the current gradients.**

---

## Sub-questions

### What does `.backward()` do?

$$\texttt{.grad} \mathrel{+}= \nabla_\theta \mathcal{L}$$

Computes gradients and **adds** them to the existing `.grad` attribute of each parameter. It does not replace — it accumulates.

### What does `optimizer.zero_grad()` do?

$$\texttt{.grad} \leftarrow 0 \quad \forall \; \theta$$

Resets all gradient accumulators to zero. Without this call, the next `backward()` adds new gradients on top of old ones.

### What is the consequence of accumulation?

$$\texttt{.grad}_\text{step 3} = \nabla_1 + \nabla_2 + \nabla_3 \quad \text{(should be just } \nabla_3 \text{)}$$

The gradient grows with each batch. Weight updates become increasingly large and incorrect, causing training instability or divergence.

---

## Main answer

**C. Gradients from previous batches accumulate (add up) to the current gradients.**

```
Without zero_grad():

  Batch 1: .grad = ∇₁
  Batch 2: .grad = ∇₁ + ∇₂         ← wrong! should be ∇₂
  Batch 3: .grad = ∇₁ + ∇₂ + ∇₃   ← wrong! should be ∇₃
  ...
  Gradients grow unboundedly → training diverges


With zero_grad():

  Batch 1: zero_grad → .grad = 0 → backward → .grad = ∇₁     ✓
  Batch 2: zero_grad → .grad = 0 → backward → .grad = ∇₂     ✓
  Batch 3: zero_grad → .grad = 0 → backward → .grad = ∇₃     ✓
```

```
Correct training loop order:

  optimizer.zero_grad()   ← reset gradients
       │
       ▼
  output = model(input)   ← forward pass
       │
       ▼
  loss = criterion(output, target)
       │
       ▼
  loss.backward()         ← compute gradients
       │
       ▼
  optimizer.step()        ← update weights
```

| Scenario | `.grad` after batch $k$ | Correct? |
|---|---|---|
| With `zero_grad()` | $\nabla_k$ | Yes |
| Without `zero_grad()` | $\sum_{i=1}^{k} \nabla_i$ | No (accumulates) |

---
---

# Quiz 1, Question 20 — Why Use cross_entropy on Raw Logits

> **In pyTorch, why use cross_entropy on the raw logits directly instead of after applying softmax activation function?**
>
> **B. It has built-in softmax for better numerical stability**

---

## Sub-questions

### What are raw logits?

$$z_i \in (-\infty, +\infty)$$

Unbounded output scores from the final linear layer. Not yet probabilities.

### What goes wrong with manual softmax + log?

$$\text{Softmax}(z_i) = \frac{e^{z_i}}{\sum_j e^{z_j}} \approx 0 \quad \text{(when } z_i \ll \max(z) \text{)}$$

$$\log(0) = -\infty \rightarrow \text{NaN}$$

Two-step approach: softmax produces near-zero values, then log produces $-\infty$ or NaN.

### How does F.cross_entropy avoid this?

$$\mathcal{L} = -z_c + \log\!\left(\sum_j e^{z_j}\right)$$

Internally uses the **LogSumExp trick**: stays in log-space, never computes a near-zero probability, never takes log of zero.

---

## Main answer

**B. It has built-in softmax for better numerical stability**

```
Dangerous (manual):

  logits ──▶ softmax() ──▶ tiny p ≈ 0 ──▶ log(≈0) ──▶ -inf / NaN ──▶ crash


Safe (F.cross_entropy):

  logits ──▶ F.cross_entropy() ──▶ stable loss
              (log-domain math, never computes p ≈ 0)
```

| Approach | Computation | Risk |
|---|---|---|
| `softmax()` then `log()` | Explicit $e^z$ then $\log$ | $\log(0) = -\infty$ → NaN |
| `F.cross_entropy(logits)` | $-z_c + \log\sum e^{z_j}$ (log-space) | None — numerically stable |

---
---

# Quiz 1, Question 21 — model.eval() Behavior

> **What does calling model.eval() actually do?**
>
> **D. It changes the behavior of specific layers like Dropout and BatchNorm to inference mode.**

---

## Sub-questions

### What is Dropout during training?

$$h_i = \begin{cases} 0 & \text{with probability } p \\ \frac{h_i}{1-p} & \text{with probability } 1-p \end{cases}$$

Randomly zeroes neurons to prevent co-adaptation. After `model.eval()`, dropout is **disabled** — all neurons active.

### What is BatchNorm during training?

Uses **batch statistics** (mean and variance of current mini-batch). After `model.eval()`, uses **running statistics** (exponential moving average accumulated during training) for deterministic output.

### What does model.eval() NOT do?

It does **not** disable gradient computation. It does **not** save VRAM. That is the job of `torch.no_grad()`.

---

## Main answer

**D. It changes the behavior of specific layers like Dropout and BatchNorm to inference mode.**

```
model.eval() effects:

  ┌────────────────────────────────────────┐
  │ model.training = False                 │
  ├────────────────────────────────────────┤
  │ Dropout:   random zeroing → OFF        │
  │ BatchNorm: batch stats → running stats │
  ├────────────────────────────────────────┤
  │ Does NOT stop gradient computation     │
  │ Does NOT save VRAM                     │
  └────────────────────────────────────────┘
```

| Layer | `model.train()` | `model.eval()` |
|---|---|---|
| Dropout | Randomly drops neurons | All neurons active |
| BatchNorm | Batch statistics | Running statistics |
| Other layers | No change | No change |
| Gradients | Computed | Still computed (need `no_grad()` to stop) |

---
---

# Quiz 1, Question 22 — torch.no_grad() Benefits

> **What is the primary benefit of wrapping inference code in with torch.no_grad():?**
>
> **A. It prevents the construction of the computational graph, saving memory and speed.**

---

## Sub-questions

### What is the computational graph?

A directed acyclic graph (DAG) of tensor operations built during the forward pass. Each node stores intermediate values needed for `backward()`. Consumes significant VRAM.

### Why is it unnecessary during inference?

During inference, we only need the forward pass output. No `backward()` call will follow, so storing intermediates for gradient computation is pure waste.

### What does `torch.no_grad()` do?

$$\text{requires\_grad} \leftarrow \text{False} \quad \text{(within context)}$$

Disables autograd engine. No graph is built, no intermediate activations are stored, forward pass runs faster.

---

## Main answer

**A. It prevents the construction of the computational graph, saving memory and speed.**

```
With graph (training):

  x ──▶ linear₁ ──▶ relu ──▶ linear₂ ──▶ output
          │            │          │
          ▼            ▼          ▼
     [save input] [save mask] [save input]   ← VRAM consumed
                                              for backward()

With torch.no_grad() (inference):

  x ──▶ linear₁ ──▶ relu ──▶ linear₂ ──▶ output
                                              ← nothing stored
                                              faster + less VRAM
```

| Aspect | Without `no_grad()` | With `no_grad()` |
|---|---|---|
| Computational graph | Built | Not built |
| Intermediate activations | Stored | Not stored |
| VRAM usage | High | Low |
| Forward pass speed | Slower | Faster |
| Can call `backward()` | Yes | No |

---
---

# Quiz 1, Question 23 — How optimizer.step() Knows the Direction

> **How does optimizer.step() know in which direction to update the parameters?**
>
> **C. It uses the .grad attribute stored in each parameter tensor (populated by backward).**

---

## Sub-questions

### What does `backward()` produce?

$$\texttt{param.grad} = \frac{\partial \mathcal{L}}{\partial \texttt{param}} \quad \forall \; \texttt{param} \in \text{model.parameters()}$$

Each parameter tensor gets a `.grad` attribute containing the gradient of the loss with respect to that parameter.

### What does `optimizer.step()` do?

$$\theta \leftarrow \theta - \alpha \cdot \texttt{param.grad}$$

Reads `.grad` from each parameter and applies the update rule (SGD, Adam, etc.). The gradient tells both the **direction** and **magnitude** of steepest ascent; the optimizer moves **opposite** to it.

---

## Main answer

**C. It uses the .grad attribute stored in each parameter tensor (populated by backward).**

```
Data flow:

  loss.backward()
       │
       ▼
  Populates param.grad for every parameter
       │
       ▼
  optimizer.step()
       │
       ▼
  Reads param.grad → applies update rule → modifies param.data
```

$$\text{backward: } \texttt{.grad} = \nabla_\theta \mathcal{L} \qquad \text{step: } \theta \leftarrow \theta - \alpha \cdot \texttt{.grad}$$

| Component | Responsibility |
|---|---|
| `loss.backward()` | Compute gradients, store in `.grad` |
| `optimizer.step()` | Read `.grad`, update weights |
| `.grad` attribute | Communication channel between the two |

---
---

# Quiz 1, Question 24 — shuffle=True in DataLoader

> **Why do we typically set shuffle=True in the DataLoader for the training set?**
>
> **B. To ensure data is Independent and Identically Distributed (I.I.D.) across batches.**

---

## Sub-questions

### What is I.I.D.?

$$\text{Independent: } P(x_i, x_j) = P(x_i) \cdot P(x_j)$$

$$\text{Identically Distributed: each } x_i \sim \text{same distribution}$$

Each mini-batch should be a representative random sample of the full dataset, not a biased subset.

### What happens without shuffling?

If data is ordered (e.g., all positive examples first, then all negative), consecutive batches contain only one class. The model oscillates — learning one class, then unlearning it for the other. Gradients are biased.

### What does shuffling achieve?

Each batch contains a random mix of examples → gradient estimates are unbiased → training converges smoothly.

---

## Main answer

**B. To ensure data is Independent and Identically Distributed (I.I.D.) across batches.**

```
Without shuffle (ordered data):

  Batch 1: [pos, pos, pos, pos]  → gradient biased toward positive
  Batch 2: [pos, pos, pos, pos]  → still biased
  ...
  Batch N: [neg, neg, neg, neg]  → suddenly biased toward negative
  → Oscillating, unstable training


With shuffle (randomized):

  Batch 1: [pos, neg, pos, neg]  → balanced gradient
  Batch 2: [neg, pos, neg, pos]  → balanced gradient
  → Smooth, stable convergence
```

| Property | `shuffle=False` | `shuffle=True` |
|---|---|---|
| Batch composition | Order-dependent (possibly biased) | Random (representative) |
| Gradient estimates | Biased | Unbiased |
| Convergence | Slow / oscillating | Smooth / stable |
| I.I.D. assumption | Violated | Satisfied |

---
---

# Quiz 1, Question 25 — model.state_dict()

> **When you save model.state_dict(), what are you saving?**
>
> **D. A dictionary mapping parameter names to their weight/bias tensors only.**

---

## Sub-questions

### What is a state_dict?

$$\texttt{state\_dict()} = \{\texttt{"layer1.weight"}: W_1, \; \texttt{"layer1.bias"}: b_1, \; \dots\}$$

An `OrderedDict` where keys are parameter names (strings) and values are the corresponding tensors. Contains **only learned parameters** (weights and biases).

### What does it NOT contain?

- Model architecture/code (not stored — you must define the model class separately)
- Optimizer state (momentum, adaptive learning rates — saved separately via `optimizer.state_dict()`)
- Training hyperparameters

### How is it used?

```python
# Save
torch.save(model.state_dict(), "model.pt")

# Load (must define model architecture first)
model = MyModel()                        # recreate architecture
model.load_state_dict(torch.load("model.pt"))  # load weights
```

---

## Main answer

**D. A dictionary mapping parameter names to their weight/bias tensors only.**

```
model.state_dict() contents:

  {
    "embedding.weight":    tensor(V, d),
    "layer1.attention.Wq": tensor(d, d),
    "layer1.attention.Wk": tensor(d, d),
    "layer1.attention.Wv": tensor(d, d),
    "layer1.ffn.weight":   tensor(4d, d),
    "layer1.ffn.bias":     tensor(4d),
    ...
    "output.weight":       tensor(V, d)
  }

  Keys: parameter names (strings)
  Values: weight/bias tensors
  Nothing else.
```

| Included | Not Included |
|---|---|
| Weight tensors | Model architecture/code |
| Bias tensors | Optimizer state (momentum, etc.) |
| Parameter names (keys) | Hyperparameters |
| — | Training history |

```
Save / Load workflow:

  Save:   model.state_dict() ──▶ torch.save() ──▶ file.pt

  Load:   Define model class (architecture)
              │
              ▼
          model = MyModel()
              │
              ▼
          torch.load("file.pt") ──▶ model.load_state_dict()
```
