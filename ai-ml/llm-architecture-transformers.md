# Assignment 3 Tutorial & Reference Guide
## From Llama to Mixtral: Implementing Sparse Mixture of Experts (MoE)
### CPSC5910/4910 — Large Language Models

> **Audience**: CS students comfortable with Python but new to machine learning, neural networks, and LLMs.
> **Goal**: Give you every concept and tool you need to earn full marks on Assignment 3.

---

## Table of Contents

1. [Conceptual Background: What You Need to Know Before Coding](#1-conceptual-background)
   - 1.1 What is an LLM?
   - 1.2 The Transformer Architecture (30-second version)
   - 1.3 What is a Feed-Forward Network (FFN)?
   - 1.4 The Problem: Dense FFNs Are Expensive
   - 1.5 The Solution: Mixture of Experts (MoE)
   - 1.6 Sparse vs. Dense MoE
2. [PyTorch Crash Course: Only What You Need](#2-pytorch-crash-course)
   - 2.1 Tensors (Multi-dimensional arrays)
   - 2.2 Tensor Shapes & Dimensions
   - 2.3 `nn.Module` — Building Blocks of Neural Networks
   - 2.4 `nn.Linear` — The Fundamental Operation
   - 2.5 Activation Functions (`F.silu`, `F.softmax`)
   - 2.6 `torch.topk` — Selecting the Best Values
   - 2.7 Boolean Masking
   - 2.8 Broadcasting & `unsqueeze`
   - 2.9 `torch.where` — Conditional Selection
   - 2.10 The Training Loop (forward, loss, backward, step)
   - 2.11 Computational Graphs & Gradients
3. [Part-by-Part Walkthrough](#3-part-by-part-walkthrough)
   - 3.1 Setup Code Explained
   - 3.2 Part A: The Router (30 pts)
   - 3.3 Part B: The Sparse MoE Layer (40 pts)
   - 3.4 Part C: The Mixtral Block (10 pts)
   - 3.5 Part D: Training Loop & Gradient Verification (20 pts)
4. [Common Pitfalls & Debugging Tips](#4-common-pitfalls--debugging-tips)
5. [Quick Reference Card](#5-quick-reference-card)
6. [Further Reading & Resources](#6-further-reading--resources)

---

## 1. Conceptual Background

### 1.1 What is an LLM?

A **Large Language Model** (LLM) is a program that has been trained on massive amounts of text to predict "what word comes next." Models like ChatGPT, Llama, and Mixtral are all LLMs. Under the hood, they are giant mathematical functions with billions of tunable numbers (called **parameters** or **weights**). During training, these numbers are adjusted so the model gets better at prediction.

You don't need to understand training deeply for this assignment — you just need to know that **an LLM is made of stacked building blocks**, and your job is to build one of those blocks.

### 1.2 The Transformer Architecture (30-second version)

Almost every modern LLM uses the **Transformer** architecture. A Transformer is a stack of identical **blocks** (layers). Each block has two main sub-components:

```
Input
  │
  ▼
┌─────────────────────┐
│   Attention Layer    │  ← "Which other words should I pay attention to?"
└─────────────────────┘
  │
  ▼
┌─────────────────────┐
│  Feed-Forward Net    │  ← "Now process this word's information."
│      (FFN)           │
└─────────────────────┘
  │
  ▼
Output (fed into next block)
```

- **Attention**: Lets each word look at other words in the sentence to gather context. (You do NOT implement this — PyTorch provides it.)
- **FFN**: A small neural network that processes each word independently. (You WILL modify this part.)

### 1.3 What is a Feed-Forward Network (FFN)?

An FFN is the simplest type of neural network. It takes an input vector, multiplies it by a matrix of weights, applies a non-linear function, and produces an output vector.

```
input (size 128) → [multiply by weights] → [activation function] → [multiply by weights] → output (size 128)
```

In this assignment, the FFN uses a specific design called **SwiGLU** (used in Llama 3). You don't need to invent it — it's provided for you as the `SwiGLUExpert` class. Just think of it as: **a box that takes a vector in and gives a vector out, both the same size**.

### 1.4 The Problem: Dense FFNs Are Expensive

In a standard Transformer, **every word** passes through the **same, single, large FFN**. If you want the model to "know more," you have to make that FFN bigger — which means more computation for *every single word*, even simple ones like "the" or "a."

Imagine a hospital where every patient, no matter their symptoms, must see every single specialist. That's wasteful.

### 1.5 The Solution: Mixture of Experts (MoE)

Instead of one giant FFN, use **many smaller FFNs** (called **"experts"**). Add a **router** (also called a "gating network") that looks at each word and decides: *"Which experts should handle this word?"*

```
                    ┌── Expert 0 (small FFN)
                    ├── Expert 1 (small FFN)
Input → Router →    ├── Expert 2 (small FFN)   → Combine selected outputs → Output
   "which experts?" ├── ...
                    └── Expert 7 (small FFN)
```

In this assignment: **8 experts** total, and the router picks the **top 2** for each word.

Going back to our hospital analogy: now a triage nurse (the router) sends each patient only to the 2 most relevant specialists.

### 1.6 Sparse vs. Dense MoE

| Type | What happens | Cost |
|------|-------------|------|
| **Dense** MoE | Every word goes through ALL experts, results are averaged | Very expensive (defeats the purpose) |
| **Sparse** MoE | Every word goes through only the **Top-K** experts (K=2) | Efficient! Only 2 of 8 experts run per word |

**This assignment implements Sparse MoE (SMoE).**

The math is:

```
For each word x:
  1. Router scores all 8 experts:  scores = W_gate * x        → 8 numbers
  2. Pick top 2 scores:           top_values, top_indices = topk(scores, k=2)
  3. Normalize into probabilities: weights = softmax(top_values) → 2 numbers that sum to 1.0
  4. Run x through only those 2 experts, weight their outputs:
     output = weight_1 * Expert_A(x) + weight_2 * Expert_B(x)
```

---

## 2. PyTorch Crash Course

PyTorch is a Python library for building and training neural networks. Here's exactly what you need for this assignment.

### 2.1 Tensors (Multi-dimensional arrays)

A **tensor** is PyTorch's version of a NumPy array, but with two superpowers:
1. It can run on GPUs for speed.
2. It tracks every math operation done to it, so it can compute **gradients** (needed for training).

```python
import torch

a = torch.tensor([1.0, 2.0, 3.0])       # 1D tensor (vector)
b = torch.zeros(3, 4)                     # 2D tensor of zeros, shape (3, 4)
c = torch.randn(2, 16, 128)              # 3D tensor of random numbers, shape (2, 16, 128)
```

### 2.2 Tensor Shapes & Dimensions

This assignment uses 3D tensors extensively. The standard shape is **(B, T, D)**:

| Symbol | Meaning | Assignment Value | What it represents |
|--------|---------|------------------|--------------------|
| **B** | Batch size | 2 | Number of sentences processed in parallel |
| **T** | Sequence length | 16 | Number of words (tokens) per sentence |
| **D** | Hidden dimension | 128 | Size of the vector representing each word |

So a tensor of shape `(2, 16, 128)` means: **2 sentences, each with 16 words, each word represented as a 128-number vector.**

**Dimension indexing** (`dim=`):
- `dim=0` → the batch dimension (B)
- `dim=1` → the sequence/time dimension (T)
- `dim=2` or `dim=-1` → the last dimension (D, or whatever is last)

### 2.3 `nn.Module` — Building Blocks of Neural Networks

Every neural network component in PyTorch inherits from `nn.Module`. Think of it as a class with:
- An `__init__` method where you define the learnable weights.
- A `forward` method where you define the computation.

```python
import torch.nn as nn

class MyLayer(nn.Module):
    def __init__(self, input_size, output_size):
        super().__init__()                              # Always call this first
        self.weights = nn.Linear(input_size, output_size)  # Learnable weights

    def forward(self, x):
        return self.weights(x)                          # What happens when you call the layer
```

Usage:
```python
layer = MyLayer(128, 64)
output = layer(some_input)   # Calls forward() automatically
```

**`nn.ModuleList`**: A special list that tells PyTorch "these are all sub-modules with learnable weights." Used in the assignment to hold the 8 experts:
```python
self.experts = nn.ModuleList([SwiGLUExpert(128, 256) for _ in range(8)])
# Access: self.experts[0], self.experts[1], ..., self.experts[7]
```

### 2.4 `nn.Linear` — The Fundamental Operation

`nn.Linear(in_features, out_features)` is matrix multiplication with learnable weights.

```python
linear = nn.Linear(128, 8, bias=False)
# Internally holds a weight matrix of shape (8, 128)
# Given input of shape (B, T, 128), output is shape (B, T, 8)

x = torch.randn(2, 16, 128)
out = linear(x)   # shape: (2, 16, 8)
```

**In the router**: `nn.Linear(d_model=128, num_experts=8)` takes each word's 128-dim vector and produces 8 scores (one per expert).

### 2.5 Activation Functions

**`F.silu`** (Swish): A smooth, non-linear function used inside SwiGLU. You don't need to use it directly — it's already in the provided `SwiGLUExpert` class.

**`F.softmax`**: Converts a list of raw numbers ("logits") into probabilities that sum to 1.0.

```python
import torch.nn.functional as F

logits = torch.tensor([2.0, 1.0, 0.1])
probs = F.softmax(logits, dim=-1)
# Result: tensor([0.6590, 0.2424, 0.0986])  — they sum to 1.0
```

**Critical for Part A**: You apply softmax to **only the top-K values**, not all 8 expert scores.

### 2.6 `torch.topk` — Selecting the Best Values (KEY for Part A)

`torch.topk(input, k, dim=-1)` returns the `k` largest values and their indices along a dimension.

```python
scores = torch.tensor([0.1, 0.9, 0.3, 0.7, 0.2, 0.5, 0.8, 0.4])
#                       0    1    2    3    4    5    6    7   ← indices

values, indices = torch.topk(scores, k=2)
# values:  tensor([0.9, 0.8])   ← the two highest scores
# indices: tensor([1, 6])       ← which experts they came from
```

For a 3D tensor of shape `(B, T, num_experts)`:
```python
gate_logits = torch.randn(2, 16, 8)  # (B, T, 8) — scores for 8 experts
top_k_values, selected_indices = torch.topk(gate_logits, k=2, dim=-1)
# top_k_values shape:    (2, 16, 2)  — the 2 highest scores per token
# selected_indices shape: (2, 16, 2)  — which 2 experts were chosen
```

### 2.7 Boolean Masking (KEY for Part B)

A boolean mask is a tensor of `True`/`False` values used to select or filter elements.

```python
indices = torch.tensor([[1, 3],    # Token 0 chose experts 1, 3
                         [0, 6]])   # Token 1 chose experts 0, 6

# "Did any token choose expert 1?"
mask = (indices == 1).any(dim=-1)
# mask: tensor([True, False])
# Token 0 chose expert 1: True.  Token 1 didn't: False.
```

**For the 3D case in this assignment:**
```python
selected_indices  # shape: (B, T, top_k) = (2, 16, 2)

# For expert i=3: which tokens selected expert 3?
mask = (selected_indices == 3).any(dim=-1)
# mask shape: (B, T) = (2, 16) — True where a token chose expert 3
```

### 2.8 Broadcasting & `unsqueeze` (KEY for Part B)

**Broadcasting** is PyTorch's ability to automatically expand tensor dimensions to make operations compatible.

**`unsqueeze(dim)`** adds a dimension of size 1 at position `dim`:

```python
mask = torch.tensor([True, False, True])  # shape: (3,)
mask.unsqueeze(-1)                         # shape: (3, 1)
# tensor([[True],
#         [False],
#         [True]])
```

**Why you need this**: When you have a mask of shape `(B, T)` and want to multiply it with `x` of shape `(B, T, D)`, the dimensions don't align. You unsqueeze the mask:

```python
mask        # shape: (B, T)
x           # shape: (B, T, D)

masked_x = x * mask.unsqueeze(-1)
# mask.unsqueeze(-1) shape: (B, T, 1) — broadcasts across D dimension
# masked_x shape: (B, T, D) — zeros out entire rows where mask is False
```

**Similarly for routing weights:**
```python
expert_weight  # shape: (B, T)   — the weight for one expert per token
expert_output  # shape: (B, T, D)

weighted_output = expert_output * expert_weight.unsqueeze(-1)
# shape: (B, T, D) — each token's output scaled by its routing weight
```

### 2.9 `torch.where` — Conditional Selection (KEY for Part B)

`torch.where(condition, value_if_true, value_if_false)` — works element-wise.

```python
mask = torch.tensor([True, False, True])
a = torch.tensor([10.0, 20.0, 30.0])

result = torch.where(mask, a, torch.tensor(0.0))
# result: tensor([10.0,  0.0, 30.0])
# Where mask is True → take from a; where False → use 0.0
```

**In Part B**, you'll use this to extract the routing weight for expert `i`:

```python
# routing_weights: (B, T, top_k=2) — the two weights per token
# selected_indices: (B, T, top_k=2) — which experts were chosen

# For expert i, find its weight (or 0.0 if not selected)
# Step 1: Find WHICH slot (0 or 1) has expert i
match_mask = (selected_indices == i)   # (B, T, 2) — True at the matching slot

# Step 2: Zero out non-matching weights, then sum across top_k
expert_weights = (routing_weights * match_mask).sum(dim=-1)  # (B, T)
# This gives the weight for expert i per token, or 0.0 if not selected
```

### 2.10 The Training Loop (KEY for Part D)

Training a neural network follows 5 steps, repeated many times:

```python
model = MyModel()
optimizer = torch.optim.AdamW(model.parameters(), lr=3e-4)  # Controls HOW weights update
loss_fn = nn.MSELoss()  # Measures "how wrong is the prediction?"

for step in range(num_steps):
    # Step 1: ZERO GRADIENTS — clear leftover gradients from last step
    optimizer.zero_grad()

    # Step 2: FORWARD PASS — feed input through the model
    output = model(input_data)

    # Step 3: COMPUTE LOSS — measure how far output is from target
    loss = loss_fn(output, target_data)

    # Step 4: BACKWARD PASS — compute gradients of loss w.r.t. all weights
    loss.backward()

    # Step 5: OPTIMIZER STEP — update weights using gradients
    optimizer.step()
```

**Analogy**: Imagine learning to throw darts.
1. **Zero grad**: Forget your previous adjustment.
2. **Forward**: Throw the dart.
3. **Loss**: Measure how far from the bullseye.
4. **Backward**: Figure out "should I throw more left, more right, harder, softer?"
5. **Step**: Adjust your aim accordingly.

### 2.11 Computational Graphs & Gradients

When you do math with PyTorch tensors, PyTorch secretly builds a **computational graph** — a record of every operation. When you call `loss.backward()`, PyTorch walks this graph backwards to compute **gradients** (how much each weight contributed to the error).

**Why this matters for Part B**: If you accidentally use an operation that **breaks the graph** (like converting to numpy, using `.item()` in the middle of computation, or using non-differentiable operations), gradients won't flow back to the router, and the router can never learn.

**Rule of thumb**: Keep everything as PyTorch tensor operations. Don't convert things to plain Python numbers mid-computation.

```python
# GOOD (differentiable):
masked_x = x * mask.unsqueeze(-1).float()

# BAD (breaks the graph):
for b in range(B):
    for t in range(T):
        if mask[b, t].item():        # .item() exits PyTorch world
            result[b, t] = expert(x[b, t])
```

---

## 3. Part-by-Part Walkthrough

### 3.1 Setup Code Explained

The setup code gives you:

**Configuration dictionary** — all the numbers that define the model:
```python
Config = {
    'd_model': 128,              # Each word = a vector of 128 numbers
    'n_heads': 4,                # (For attention, not your concern)
    'seq_len': 16,               # Each sentence = 16 words
    'batch_size': 2,             # Process 2 sentences at a time
    'num_experts': 8,            # 8 expert FFNs
    'num_experts_per_token': 2,  # Each word uses 2 of the 8 experts
    'expert_hidden_dim': 256     # Internal size of each expert
}
```

**`SwiGLUExpert` class** — a single expert FFN. It's a black box for your purposes:
- **Input**: tensor of shape `(*, 128)` (any batch dimensions, last dim = d_model)
- **Output**: tensor of same shape `(*, 128)`
- **Internally**: projects 128 → 256, applies SwiGLU activation, projects 256 → 128

You do not modify this class. You just **use** it.

---

### 3.2 Part A: The Router — `TopKRouter` [30 Points]

**What the router does**: Takes word vectors and decides which 2 experts (out of 8) each word should use.

**Input**: `x` of shape `(B, T, D)` = `(2, 16, 128)`
**Outputs**:
- `routing_weights`: shape `(B, T, top_k)` = `(2, 16, 2)` — probabilities summing to 1.0
- `selected_indices`: shape `(B, T, top_k)` = `(2, 16, 2)` — expert IDs (0-7)

**What's already done for you:**
- Step 1: `gate_logits = self.gate(x)` — projects each word's 128-dim vector to 8 scores.

**What you need to implement:**

**Step 2** — Use `torch.topk` to select the 2 highest scores:
```python
# gate_logits shape: (B, T, 8)
# We want the top 2 values and their indices along the last dimension
top_k_values, selected_indices = torch.topk(gate_logits, self.top_k, dim=-1)
# top_k_values shape: (B, T, 2) — the 2 highest scores
# selected_indices shape: (B, T, 2) — which experts they correspond to
```

**Step 3** — Apply softmax to **only** the top-K values:
```python
routing_weights = F.softmax(top_k_values, dim=-1)
# shape: (B, T, 2) — two probabilities that sum to 1.0
```

**Why softmax on only top-K?** If we softmax all 8 scores, the 6 rejected experts would still get some probability. By softmax-ing only the top 2, we ensure all probability mass goes to the selected experts.

**Test to pass**: `routing_weights.sum(dim=-1)` should be all 1.0s.

---

### 3.3 Part B: The Sparse MoE Layer — `SparseMoELayer` [40 Points]

This is the hardest part. Here's the step-by-step logic.

**Big picture**: For each of the 8 experts, figure out which tokens chose it, run those tokens through it, weight the output, and add to the final result.

**What's already done for you:**
- The router and experts are initialized.
- `routing_weights, selected_indices = self.router(x)` is called.
- `final_output = torch.zeros_like(x)` is initialized.
- The `for i in range(self.num_experts)` loop structure.

**What you implement inside the loop** (for expert `i`):

```
Loop iteration for expert i:

(a) Which tokens chose expert i?
    mask = (selected_indices == i).any(dim=-1)
    # selected_indices: (B, T, 2), checking if i appears in either slot
    # .any(dim=-1) collapses the top_k dim → mask shape: (B, T)
    # mask[b][t] = True if token (b,t) selected expert i

(b) What weight did those tokens assign to expert i?
    # Where expert i matches, grab the weight; elsewhere use 0.0
    weight_mask = (selected_indices == i).float()          # (B, T, 2)
    expert_weights = (routing_weights * weight_mask).sum(dim=-1)  # (B, T)
    # For tokens that didn't choose expert i, this is 0.0
    # For tokens that did, this is the softmax probability

(c) Mask the input — zero out tokens NOT going to this expert:
    masked_input = x * mask.unsqueeze(-1).float()
    # mask.unsqueeze(-1): (B, T, 1) — broadcasts over D=128
    # masked_input: (B, T, D) — non-selected tokens are all-zeros vectors

(d) Run through the expert:
    expert_output = expert_layer(masked_input)
    # expert_output: (B, T, D)
    # Non-selected tokens produce ~zero output (input was zeros)

(e) Weight the output and accumulate:
    final_output += expert_output * expert_weights.unsqueeze(-1)
    # expert_weights.unsqueeze(-1): (B, T, 1) — broadcasts over D
    # This scales each token's output by its routing probability
```

**Visual example** for one token that selected experts 2 (weight 0.6) and 5 (weight 0.4):

```
Expert 0: mask=False, weight=0.0 → contributes nothing
Expert 1: mask=False, weight=0.0 → contributes nothing
Expert 2: mask=True,  weight=0.6 → contributes 0.6 * Expert2(x)
Expert 3: mask=False, weight=0.0 → contributes nothing
Expert 4: mask=False, weight=0.0 → contributes nothing
Expert 5: mask=True,  weight=0.4 → contributes 0.4 * Expert5(x)
Expert 6: mask=False, weight=0.0 → contributes nothing
Expert 7: mask=False, weight=0.0 → contributes nothing

final_output = 0.6 * Expert2(x) + 0.4 * Expert5(x)
```

**Tests to pass**:
- Output shape must equal input shape: `(B, T, D)`
- Output must NOT be identical to input (the layer actually did something)

---

### 3.4 Part C: The Mixtral Block — `MixtralBlock` [10 Points]

This part is straightforward — you're assembling pre-built components.

A standard Transformer block is:
```
x → LayerNorm → Attention → add residual → LayerNorm → FFN → add residual → output
```

The **only change** from a standard block to a "Mixtral" block: replace the single FFN with your `SparseMoELayer`.

**What you implement:**
```python
self.ffn = SparseMoELayer(config)
```

That's it. The `forward` method is already provided — it applies attention with a residual connection, then your MoE FFN with a residual connection.

**What is a residual connection?** `x = x + layer(x)`. It means "add the layer's output back to the input." This helps deep networks train by ensuring information can flow straight through. You don't need to implement this — it's already in the provided `forward` method.

---

### 3.5 Part D: Training Loop & Gradient Verification [20 Points]

**Purpose**: Prove that your implementation is mathematically correct by showing that:
1. The loss decreases over 5 training steps.
2. The router's weights receive non-zero gradients (meaning the computational graph is intact).

**What you implement** (inside the `for step in range(1, 6)` loop):

```python
# a. Zero gradients from previous step
optimizer.zero_grad()

# b. Forward pass — run input through the model
output = block(dummy_input)

# c. Compute loss — how different is output from target?
loss = loss_fn(output, dummy_target)

# d. Backward pass — compute gradients
loss.backward()

# e. Update weights
optimizer.step()
```

**Replace the placeholder** `loss = torch.tensor(0.0)` with the actual loss from step (c).

**What "gradient norm" means**: After `loss.backward()`, every weight in the model has a `.grad` attribute — a tensor showing how much that weight should change. The **norm** (magnitude) of this gradient tells you "how strongly is the training signal reaching this weight." If it's 0.0, the router isn't learning.

**Expected output**: Loss numbers that generally decrease, and router gradient norms that are non-zero (e.g., 0.01 or higher).

---

## 4. Common Pitfalls & Debugging Tips

### Shape Mismatches
Always print shapes when debugging:
```python
print(f"x shape: {x.shape}")
print(f"mask shape: {mask.shape}")
print(f"expert_weights shape: {expert_weights.shape}")
```

### Forgetting `.float()` on Boolean Masks
Boolean tensors can't be multiplied with float tensors directly in some contexts:
```python
# If you get a type error, cast the mask:
masked_input = x * mask.unsqueeze(-1).float()  # .float() converts True→1.0, False→0.0
```

### Breaking the Computational Graph
These operations **break gradients** — avoid them in forward computation:
- `.item()` — converts tensor to plain Python number
- `.detach()` — explicitly detaches from graph
- `.numpy()` — converts to NumPy (no gradients)
- Python `if` on tensor values — use `torch.where` instead

### Routing Weights Not Summing to 1.0
Make sure your softmax is on `dim=-1` (the top_k dimension), not some other dimension.

### The "Identity Function" Error
If `moe_output` equals `dummy_inputs`, your expert loop isn't doing anything. Check that:
- Your mask is actually `True` for some tokens.
- You're calling `expert_layer(masked_input)`, not `expert_layer(x)`.
- You're accumulating into `final_output` with `+=`.

### Loss Not Decreasing
- Make sure `block.train()` was called.
- Make sure you're calling `optimizer.zero_grad()` **before** the forward pass.
- Make sure `loss.backward()` is called **before** `optimizer.step()`.

---

## 5. Quick Reference Card

### Key Tensor Shapes in This Assignment

| Variable | Shape | Meaning |
|----------|-------|---------|
| `x` (input) | `(2, 16, 128)` | 2 batches, 16 tokens, 128-dim vectors |
| `gate_logits` | `(2, 16, 8)` | Raw score for each of 8 experts |
| `top_k_values` | `(2, 16, 2)` | Scores of top 2 experts per token |
| `selected_indices` | `(2, 16, 2)` | Which 2 experts were chosen (values 0-7) |
| `routing_weights` | `(2, 16, 2)` | Softmax probabilities of top 2 (sum to 1.0) |
| `mask` (per expert) | `(2, 16)` | True/False: did this token pick this expert? |
| `expert_weights` | `(2, 16)` | The routing probability for this expert (or 0.0) |
| `final_output` | `(2, 16, 128)` | Weighted sum of expert outputs |

### Key PyTorch Functions

| Function | What It Does | Example |
|----------|-------------|---------|
| `nn.Linear(in, out)` | Learnable matrix multiply | `nn.Linear(128, 8)` |
| `torch.topk(x, k, dim)` | Get k largest values + indices | `torch.topk(logits, 2, dim=-1)` |
| `F.softmax(x, dim)` | Convert to probabilities (sum to 1) | `F.softmax(values, dim=-1)` |
| `tensor.any(dim)` | True if any element is True along dim | `(indices == i).any(dim=-1)` |
| `tensor.unsqueeze(dim)` | Add a size-1 dimension | `mask.unsqueeze(-1)` → adds trailing dim |
| `torch.where(cond, a, b)` | Element-wise if/else | `torch.where(mask, weights, 0.0)` |
| `torch.zeros_like(x)` | Zeros tensor, same shape/device as x | `torch.zeros_like(x)` |
| `optimizer.zero_grad()` | Clear old gradients | Called at start of each training step |
| `loss.backward()` | Compute gradients via backprop | Called after computing loss |
| `optimizer.step()` | Update weights using gradients | Called after backward |

### Complete Code Skeleton (All Parts)

```python
# ========== PART A: TopKRouter ==========
class TopKRouter(nn.Module):
    def __init__(self, d_model, num_experts, top_k):
        super().__init__()
        self.gate = nn.Linear(d_model, num_experts, bias=False)
        self.top_k = top_k

    def forward(self, x):
        gate_logits = self.gate(x)                              # (B, T, num_experts)
        top_k_values, selected_indices = torch.topk(            # YOUR CODE: use torch.topk
            gate_logits, self.top_k, dim=-1)
        routing_weights = F.softmax(top_k_values, dim=-1)       # YOUR CODE: softmax on top-k
        return routing_weights, selected_indices


# ========== PART B: SparseMoELayer ==========
class SparseMoELayer(nn.Module):
    def __init__(self, config):
        super().__init__()
        self.num_experts = config['num_experts']
        self.router = TopKRouter(config['d_model'], self.num_experts,
                                 config['num_experts_per_token'])
        self.experts = nn.ModuleList([
            SwiGLUExpert(config['d_model'], config['expert_hidden_dim'])
            for _ in range(self.num_experts)
        ])

    def forward(self, x):
        B, T, D = x.shape
        routing_weights, selected_indices = self.router(x)
        final_output = torch.zeros_like(x)

        for i in range(self.num_experts):
            expert_layer = self.experts[i]
            # a. Boolean mask: which tokens chose expert i?
            mask = (selected_indices == i).any(dim=-1)             # (B, T)
            # b. Get the routing weight for expert i per token
            weight_mask = (selected_indices == i).float()          # (B, T, top_k)
            expert_weights = (routing_weights * weight_mask).sum(dim=-1)  # (B, T)
            # c. Mask the input
            masked_input = x * mask.unsqueeze(-1).float()          # (B, T, D)
            # d. Run through expert
            expert_output = expert_layer(masked_input)             # (B, T, D)
            # e. Weight and accumulate
            final_output += expert_output * expert_weights.unsqueeze(-1)

        return final_output


# ========== PART C: MixtralBlock ==========
class MixtralBlock(nn.Module):
    def __init__(self, config):
        super().__init__()
        self.ln1 = nn.LayerNorm(config['d_model'])
        self.attn = nn.MultiheadAttention(config['d_model'], config['n_heads'],
                                           batch_first=True)
        self.ln2 = nn.LayerNorm(config['d_model'])
        self.ffn = SparseMoELayer(config)                          # YOUR CODE

    def forward(self, x):
        attn_out, _ = self.attn(self.ln1(x), self.ln1(x), self.ln1(x))
        x = x + attn_out
        x = x + self.ffn(self.ln2(x))
        return x


# ========== PART D: Training Loop ==========
block = MixtralBlock(Config).to(device)
optimizer = optim.AdamW(block.parameters(), lr=3e-4)
loss_fn = nn.MSELoss()

dummy_input = torch.randn(Config['batch_size'], Config['seq_len'],
                           Config['d_model']).to(device)
dummy_target = torch.roll(dummy_input, shifts=1, dims=1)
block.train()

for step in range(1, 6):
    optimizer.zero_grad()              # a. Zero gradients
    output = block(dummy_input)        # b. Forward pass
    loss = loss_fn(output, dummy_target)  # c. Compute loss
    loss.backward()                    # d. Backward pass
    optimizer.step()                   # e. Update weights
```

---

## 6. Further Reading & Resources

### Assignment References (from the PDF)
- **Mixture of Experts Explained** (Hugging Face Blog) — Best starting point, visual and accessible: `https://huggingface.co/blog/moe`
- **Mixtral of Experts** (Jiang et al., 2024) — The paper this assignment is based on: `https://arxiv.org/abs/2401.04088`
- **Outrageously Large Neural Networks** (Shazeer et al., 2017) — Foundational MoE paper: `https://arxiv.org/abs/1701.06538`
- **Switch Transformers** (Fedus et al., 2021) — Advanced MoE with Top-1 routing: `https://arxiv.org/abs/2101.03961`

### PyTorch Documentation (Bookmark These)
- **`torch.topk`**: `https://pytorch.org/docs/stable/generated/torch.topk.html`
- **`torch.nn.Linear`**: `https://pytorch.org/docs/stable/generated/torch.nn.Linear.html`
- **`torch.nn.functional.softmax`**: `https://pytorch.org/docs/stable/generated/torch.nn.functional.softmax.html`
- **`torch.where`**: `https://pytorch.org/docs/stable/generated/torch.where.html`
- **Tensor Broadcasting Semantics**: `https://pytorch.org/docs/stable/notes/broadcasting.html`
- **Autograd (how gradients work)**: `https://pytorch.org/tutorials/beginner/blitz/autograd_tutorial.html`

### Introductory Resources (If You're New to Everything)
- **3Blue1Brown: Neural Networks** (YouTube) — The best visual introduction to what neural networks do: `https://www.youtube.com/playlist?list=PLZHQObOWTQDNU6R1_67000Dx_ZCJB-3pi`
- **PyTorch in 60 Minutes** — Official beginner tutorial: `https://pytorch.org/tutorials/beginner/deep_learning_60min_blitz.html`
- **The Illustrated Transformer** (Jay Alammar) — Visual explanation of Transformers: `https://jalammar.github.io/illustrated-transformer/`

---

*Tutorial prepared for CPSC5910/4910 Assignment 3. Good luck!*
