Below is the **fully developed 13-slide content** plus the **explanation notes after each slide**.

Use slide citations as **[1], [2], [3], [4]** and put full IEEE references on Slide 13.

---

# Slide 1 — Title Page

## Sparse Mixture-of-Experts Transformers for Generative Language Models

**Topic ID:** 31
**Category:** Scaling Generative Transformers
**Focus:** Token routing, sparse activation, expert capacity, and scaling efficiency

**Presented by:** Your Name / Group Name
**Module:** Advanced ML
**MSc Data Science & AI**

### Clarity / Presenter Notes

Start with the main idea:

> “Today we explain how modern language models can increase total parameter capacity without activating every parameter for every token.”

The key message is:

> MoE Transformers scale by using **conditional computation**: each token is routed to only a small subset of expert feed-forward networks.

---

# Slide 2 — Table of Contents

## Presentation Roadmap

1. Motivation: why dense scaling becomes expensive
2. Dense Transformer vs Sparse MoE Transformer
3. Where MoE fits inside the Transformer
4. Token routing mechanism
5. Sparse activation and scaling efficiency
6. Expert capacity and load balancing
7. Switch Transformer case study
8. GLaM case study
9. Switch vs GLaM comparison
10. Challenges, limitations, and key takeaway

### Clarity / Presenter Notes

Say:

> “We first build the intuition, then explain the routing mechanism, then connect it to the two main papers: Switch Transformer and GLaM.”

Do not spend much time here. This slide is just orientation.

---

# Slide 3 — Why Dense Scaling Becomes Expensive

## Motivation: Scaling Language Models

Large dense Transformers improve capacity by increasing:

* number of layers,
* hidden dimension,
* FFN size,
* total parameters.

But in a **dense model**, most major parameters are used for every token.

[
\text{More parameters}
\Rightarrow
\text{more computation per token}
]

## Core Problem

How can we increase model capacity without increasing per-token computation proportionally?

## MoE Answer

Use **conditional computation**:

[
\text{Activate only selected experts for each token}
]

**Citation:** [1], [2]

### Clarity / Presenter Notes

Explain dense scaling simply:

> “In a normal dense Transformer, if we make the model larger, each token usually has to pass through more or larger computations.”

Then introduce the MoE motivation:

> “MoE tries to separate total model capacity from active computation.”

Important distinction:

* **Total parameters** = all parameters stored in the model.
* **Active parameters** = parameters used for a specific token.

Shazeer et al. introduced sparse MoE as a practical way to increase model capacity through conditional computation, and Switch Transformer later adapted this idea to Transformer scaling. ([arXiv][1])

---

# Slide 4 — Dense Transformer vs Sparse MoE Transformer

## Dense Transformer FFN

Every token uses the same feed-forward network:

[
y = FFN(x)
]

```text
Token 1 ─┐
Token 2 ─┼──→ Shared FFN
Token 3 ─┘
```

## Sparse MoE Transformer

Each token is routed to selected expert FFNs:

[
y = \sum_{i \in TopK} p_i(x)E_i(x)
]

```text
Token → Router → Expert 1 / Expert 2 / ... / Expert N
```

## Key Difference

Dense Transformer:

[
\text{same FFN for all tokens}
]

MoE Transformer:

[
\text{different tokens may use different experts}
]

**Citation:** [1], [2]

### Clarity / Presenter Notes

Say:

> “The difference is not that MoE removes the Transformer. It modifies selected FFN layers inside the Transformer.”

Explain:

* Dense FFN: one shared transformation.
* MoE FFN: many possible transformations.
* Router decides which expert is used.

Important correction:

> “Sparse does not mean the weight matrices are full of zeros. Sparse means only a few experts are activated for each token.”

---

# Slide 5 — Where MoE Fits Inside a Transformer

## Standard Transformer Block

[
\text{Self-Attention} \rightarrow \text{Feed-Forward Network}
]

## MoE Transformer Block

[
\text{Self-Attention} \rightarrow \text{Router + Expert FFNs}
]

## Usually Replaced Component

[
\boxed{\text{MoE usually replaces the FFN sublayer, not attention}}
]

## Why FFN?

* FFNs contain many parameters.
* FFNs operate independently on each token.
* Expert FFNs can share the same input/output dimension.
* Attention can remain dense and shared.

**Citation:** [2], [3]

### Clarity / Presenter Notes

This is one of the most important slides for the quiz.

Say clearly:

> “The experts usually replace the Transformer’s feed-forward network, not the self-attention mechanism.”

Why?

> “Attention mixes information across tokens. The FFN then transforms each token separately, so it is natural to route each token to a selected FFN expert.”

Likely question:

**Q: Can MoE replace attention?**
A: Research can apply expert ideas elsewhere, but in Switch Transformer and GLaM, the central MoE replacement is the FFN sublayer.

Switch Transformer simplifies MoE routing in Transformer FFN layers, while GLaM replaces the feed-forward component of every other Transformer layer with MoE layers. ([arXiv][2])

---

# Slide 6 — What Is an Expert?

## Expert = Independent Feed-Forward Network

A MoE layer contains multiple expert FFNs:

[
E_1(x), E_2(x), \ldots, E_N(x)
]

Each expert has its own parameters:

[
E_i(x)=W_{2,i}\phi(W_{1,i}x+b_{1,i})+b_{2,i}
]

## Important Point

Experts are not separate full language models.

They are usually **alternative FFN subnetworks** inside Transformer layers.

## Intuition

Experts can learn different transformations, but their specializations are learned, not manually assigned.

**Citation:** [1], [2]

### Clarity / Presenter Notes

Say:

> “An expert is a neural subnetwork, usually an FFN. All experts have similar structure, but each has different learned weights.”

Avoid saying:

> “Expert 1 is grammar, expert 2 is mathematics, expert 3 is code.”

Better:

> “Experts may specialize in different patterns, but that specialization is emergent and not always human-interpretable.”

Likely question:

**Q: Are experts complete Transformers?**
A: Usually no. In these models, experts are FFN subnetworks within selected Transformer layers.

---

# Slide 7 — Router: How Tokens Choose Experts

## Router / Gating Network

For each token representation (x), the router computes expert scores:

[
z = W_r x
]

Then converts scores into probabilities:

[
p_i(x)=\frac{e^{z_i}}{\sum_j e^{z_j}}
]

The router selects the highest-scoring expert or experts:

[
TopK(p(x), k)
]

## Example

[
p(x)=[0.05,\ 0.60,\ 0.10,\ 0.25]
]

Top-1 selects:

[
E_2
]

Top-2 selects:

[
E_2,\ E_4
]

**Citation:** [1], [2], [3]

### Clarity / Presenter Notes

Explain step by step:

1. A token already has a contextual vector from the Transformer.
2. The router scores all experts.
3. It selects the top one or two experts.
4. Only those selected experts process the token.

Important:

> “The router does not usually look at the raw word only. It uses the contextual hidden representation.”

Example:

The word “bank” in “river bank” and “money bank” may be routed differently because its hidden representation is different.

Likely question:

**Q: Does the router compute all experts first?**
A: No. It computes small routing scores for all experts, but only selected experts perform the expensive FFN computation.

---

# Slide 8 — Sparse Activation and Scaling Efficiency

## Many Experts Stored, Few Experts Activated

If a layer has (N) experts and each token uses (k):

[
\text{Active expert fraction}=\frac{k}{N}
]

## Example: 64 Experts

Top-1 routing:

[
\frac{1}{64}=1.56%
]

Top-2 routing:

[
\frac{2}{64}=3.125%
]

## Key Insight

[
\boxed{
\text{Total parameters} \neq \text{Active parameters per token}
}
]

MoE increases total capacity while keeping active expert computation much smaller.

**Citation:** [1], [2], [3]

### Clarity / Presenter Notes

This is the central technical idea.

Say:

> “A dense model with 64 FFNs would compute all 64. A sparse MoE with top-1 computes only one expert for that token.”

Use a simple example:

If each expert has 100 million parameters:

* 64 experts = 6.4 billion total expert parameters.
* Top-1 active expert parameters = 100 million.
* Top-2 active expert parameters = 200 million.

Important caution:

> “This does not mean MoE is free. The model still stores all experts and must route tokens efficiently.”

GLaM is a strong example: its largest model has 1.2 trillion total parameters, but activates only a smaller subnetwork per token. ([arXiv][3])

---

# Slide 9 — Expert Capacity and Load Balancing

## Expert Capacity

Each expert can process only a limited number of token assignments:

[
C \approx \frac{T \times k}{N} \times \text{capacity factor}
]

Where:

* (T) = tokens in batch,
* (k) = experts selected per token,
* (N) = number of experts.

## Load Imbalance Problem

Bad routing:

```text
Expert 1: 700 tokens
Expert 2: 200 tokens
Expert 3: 80 tokens
Expert 4: 20 tokens
```

## Why It Matters

* overloaded experts,
* dropped or skipped expert computation,
* undertrained experts,
* poor hardware utilization.

**Citation:** [1], [2], [4]

### Clarity / Presenter Notes

Explain:

> “If too many tokens choose the same expert, that expert becomes overloaded.”

Capacity example:

If there are 1,024 tokens, 8 experts, top-1 routing, and capacity factor 1.25:

[
C=\frac{1024}{8}\times1.25=160
]

Each expert can process about 160 tokens.

If an expert receives 180 tokens, 20 exceed capacity.

In Switch Transformer, overflowed tokens may skip that expert FFN computation at that layer but still continue through the residual path.

Likely question:

**Q: Are dropped tokens deleted from the sentence?**
A: No. They skip the expert computation at that MoE layer; they are not removed from the whole model.

Switch Transformer directly discusses expert capacity, load balancing, communication cost, and training instability as practical barriers in sparse MoE scaling. ([arXiv][2])

---

# Slide 10 — Case Study 1: Switch Transformer

## Switch Transformer

**Fedus, Zoph & Shazeer**

## Main Idea

Simplify MoE routing by sending each token to **one expert**.

[
k=1
]

## Key Contributions

* Top-1 routing.
* Reduced routing complexity.
* Expert capacity control.
* Auxiliary load-balancing loss.
* Router stability techniques.
* Scaling to trillion-parameter models.

## Reported Result

Up to **7× pretraining speed improvement** in selected comparisons with the same computational resources.

**Citation:** [2]

### Clarity / Presenter Notes

Say:

> “Switch Transformer asks: can we make MoE simpler?”

Answer:

> “Yes, by routing each token to only one expert.”

Explain top-1 benefit:

* less expert computation,
* less communication,
* easier dispatch,
* simpler capacity management.

But also mention limitation:

> “Top-1 depends heavily on the selected expert. If routing is poor, there is no second expert to compensate.”

The Switch paper reports up to 7× pretraining speedups in selected T5-based comparisons and scaling up to trillion-parameter models. ([arXiv][2])

Likely question:

**Q: Is Switch always 7× faster?**
A: No. That is the authors’ reported result under their experimental settings. Speed depends on hardware, implementation, model size, and communication cost.

---

# Slide 11 — Case Study 2: GLaM

## GLaM: Generalist Language Model

**Du et al.**

## Main Idea

Use a sparsely activated MoE decoder-only language model with **top-2 routing**.

[
k=2
]

## Architecture

* Decoder-only Transformer.
* MoE layer in every other Transformer layer.
* 64 experts per MoE layer in the largest model.
* Two experts activated per token.

## Reported Result

Largest GLaM:

[
1.2T \text{ total parameters}
]

with much smaller active computation per token.

Reported about:

* half GPT-3 inference FLOPs,
* one-third GPT-3 training energy,
* stronger average zero-shot and one-shot performance across 29 tasks.

**Citation:** [3]

### Clarity / Presenter Notes

Say:

> “GLaM shows the total-versus-active parameter idea very clearly.”

Explain:

* The model stores 1.2 trillion parameters.
* It does not activate all 1.2 trillion for every token.
* It activates a sparse subnetwork through top-2 routing.

Important comparison:

> “Switch uses top-1 for simplicity. GLaM uses top-2 for more flexible expert combination.”

Caution:

> “The GPT-3 comparison should be stated as reported by the authors, because training data, architecture, and implementation details differ.”

GLaM reports a 1.2-trillion-parameter sparse model, about one-third GPT-3 training energy, half inference FLOPs, and stronger average zero-shot and one-shot results across 29 NLP tasks in the authors’ comparison. ([arXiv][3])

---

# Slide 12 — Challenges, Comparison, and Key Takeaway

## Switch vs GLaM

| Feature              | Switch Transformer             | GLaM                           |
| -------------------- | ------------------------------ | ------------------------------ |
| Routing              | Top-1                          | Top-2                          |
| Active experts/token | 1                              | 2                              |
| Main strength        | Simplicity                     | Flexible expert combination    |
| Main cost            | Depends on one expert          | More compute and communication |
| Shared idea          | Sparse conditional computation | Sparse conditional computation |

## Remaining Challenges

* load balancing,
* expert capacity overflow,
* token dropping,
* communication overhead,
* routing instability,
* large memory requirement.

## Final Takeaway

[
\boxed{
\text{MoE scales capacity by activating only selected experts per token}
}
]

But:

[
\boxed{
\text{Sparse compute introduces routing and systems complexity}
}
]

**Citation:** [1], [2], [3], [4]

### Clarity / Presenter Notes

This is the conclusion slide.

Say:

> “MoE is not simply bigger equals cheaper. It is a trade-off.”

The trade-off:

[
\text{More total parameters}
\rightarrow
\text{more model capacity}
]

but:

[
\text{sparse routing}
\rightarrow
\text{capacity, balancing, and communication challenges}
]

Mention Expert Choice briefly:

> “Later work such as Expert Choice Routing shows that routing and load balancing are still active research problems. Instead of tokens choosing experts, experts choose tokens to guarantee balanced expert loads.”

Expert Choice Routing reverses conventional token-choice routing by letting experts select tokens, giving each expert a fixed bucket size and improving load balancing. ([NeurIPS Papers][4])

End with:

> “Sparse MoE Transformers increase capacity through selective computation, not free computation.”

---

# Slide 13 — References

## References

[1] N. Shazeer, A. Mirhoseini, K. Maziarz, A. Davis, Q. V. Le, G. E. Hinton, and J. Dean, “Outrageously Large Neural Networks: The Sparsely-Gated Mixture-of-Experts Layer,” *International Conference on Learning Representations*, 2017.

[2] W. Fedus, B. Zoph, and N. Shazeer, “Switch Transformers: Scaling to Trillion Parameter Models with Simple and Efficient Sparsity,” *Journal of Machine Learning Research*, vol. 23, no. 120, pp. 1–39, 2022.

[3] N. Du *et al*., “GLaM: Efficient Scaling of Language Models with Mixture-of-Experts,” *International Conference on Machine Learning*, pp. 5547–5569, 2022.

[4] Y. Zhou *et al*., “Mixture-of-Experts with Expert Choice Routing,” *Advances in Neural Information Processing Systems*, vol. 35, pp. 7103–7114, 2022.

### Clarity / Presenter Notes

No long explanation needed. Just say:

> “These are the primary papers used for the presentation, with Switch Transformer and GLaM as the main case studies.”

[1]: https://arxiv.org/abs/1701.06538?utm_source=chatgpt.com "Outrageously Large Neural Networks: The Sparsely-Gated Mixture-of-Experts Layer"
[2]: https://arxiv.org/abs/2101.03961?utm_source=chatgpt.com "Switch Transformers: Scaling to Trillion Parameter Models with Simple and Efficient Sparsity"
[3]: https://arxiv.org/abs/2112.06905?utm_source=chatgpt.com "GLaM: Efficient Scaling of Language Models with Mixture-of-Experts"
[4]: https://papers.nips.cc/paper_files/paper/2022/hash/2f00ecd787b432c1d36f3de9800728eb-Abstract-Conference.html?utm_source=chatgpt.com "Mixture-of-Experts with Expert Choice Routing"
