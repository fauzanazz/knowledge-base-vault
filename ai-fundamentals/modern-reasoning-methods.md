# 🧠 Modern Reasoning & Thinking Methods in Large Language Models

## A Comprehensive Guide to How AI Models Actually Think

---

## Table of Contents

1. [Introduction: The Thinking Revolution](#introduction)
2. [Chain-of-Thought (CoT) Prompting](#chain-of-thought)
3. [Self-Consistency](#self-consistency)
4. [Tree of Thought (ToT)](#tree-of-thought)
5. [Graph of Thought (GoT)](#graph-of-thought)
6. [Reasoning Tokens & The "Thinking" Paradigm](#reasoning-tokens)
7. [OpenAI o1/o3 Reasoning Models](#openai-models)
8. [DeepSeek-R1: Reasoning Through RL](#deepseek-r1)
9. [Claude's Extended Thinking](#claude-extended-thinking)
10. [Process Reward Models (PRM) vs Outcome Reward Models (ORM)](#prm-vs-orm)
11. [Monte Carlo Tree Search for Reasoning](#mcts)
12. [Search-Augmented Reasoning](#search-augmented)
13. [Test-Time Compute Scaling](#test-time-compute)
14. [The Paradigm Shift: Inference-Time vs Training-Time Compute](#paradigm-shift)
15. [Comparison Table](#comparison)
16. [Future Directions](#future)

---

## 1. Introduction: The Thinking Revolution <a name="introduction"></a>

For most of the history of large language models, intelligence was synonymous with scale. Bigger models, more parameters, more training data — these were the dominant levers AI labs pulled to improve performance. A model received a prompt, ran a single forward pass through its billions of parameters, and produced an answer. Fast. Probabilistic. Often surprisingly capable. But fundamentally limited when problems demanded sustained, multi-step logical thought.

That picture began to change in 2022 with Chain-of-Thought prompting, and shifted dramatically in 2024 when OpenAI released its o1 family. What emerged was a new paradigm: **thinking as a first-class computational resource**. Instead of just making models bigger, researchers discovered they could make models *smarter per query* by allocating more compute at inference time — letting models reason, backtrack, verify, and self-correct before committing to a final answer.

Today, every major AI lab has a "thinking" or "reasoning" model. The techniques underlying them — from CoT to MCTS to PRMs — form one of the most active and consequential areas in AI research. This article is a comprehensive tour of the state of the art.

---

## 2. Chain-of-Thought (CoT) Prompting <a name="chain-of-thought"></a>

### Origin

Chain-of-Thought prompting was introduced in January 2022 by Jason Wei, Xuezhi Wang, Dale Schuurmans, and colleagues at Google Brain in the paper *"Chain-of-Thought Prompting Elicits Reasoning in Large Language Models"* (published at NeurIPS 2022). The core idea was almost embarrassingly simple: instead of asking a model to produce an answer directly, show it examples where the answer is preceded by a step-by-step reasoning trace.

### How It Works

Standard prompting: `Q: What is 17 × 23? A: 391`

CoT prompting:
```
Q: What is 17 × 23?
A: Let me work through this step by step.
   17 × 20 = 340
   17 × 3 = 51
   340 + 51 = 391
   The answer is 391.
```

The model observes these few-shot examples and learns to generate intermediate reasoning steps for new problems before producing a final answer. The reasoning steps act as **external working memory**, allowing the model to decompose complex problems into manageable sub-steps that each fit within the model's local attention window.

### Key Results

The results were striking. Google's PaLM 540B model went from **17.9% to 58.1% accuracy** on the GSM8K math benchmark — a 224% improvement — using just eight CoT examples. Similar gains were seen across arithmetic, commonsense, and symbolic reasoning tasks.

### Variants

- **Zero-Shot CoT**: Simply appending *"Let's think step by step"* to a prompt. Surprisingly effective without any examples. Introduced by Kojima et al. (2022).
- **Manual CoT**: Human-curated reasoning examples in the prompt (original Wei et al. approach).
- **Auto-CoT**: Automatically generated CoT examples using clustering heuristics to select representative problems.
- **Multimodal CoT**: Extended to joint language + vision inputs (Meta/AWS, 2024), operating in two stages: rationale generation and answer inference.

### The Emergence Threshold

A critical finding: CoT only works reliably with models of **~100 billion parameters or more**. Below this threshold, models generate incoherent reasoning traces and perform worse than standard prompting. This suggests that CoT elicits a capability that only emerges at sufficient scale — it doesn't create reasoning ability but *unlocks* it.

### 2025 Reassessment

By 2025, research began questioning CoT's universal value. A Wharton study (*"The Decreasing Value of Chain of Thought in Prompting"*, June 2025) found CoT improvements were inconsistent across task types. Notably, a July 2025 paper found CoT can actually **harm performance** on tasks like facial recognition and simple factual recall — tasks where deliberate reasoning overcomplicates what should be a fast, pattern-matching response. For modern reasoning models like o1/o3, explicitly prompting for CoT can even hinder performance, as these models already reason internally.

---

## 3. Self-Consistency <a name="self-consistency"></a>

### Concept

Self-consistency (Wang et al., 2022) is an inference-time enhancement of CoT that treats the model as a **stochastic ensemble**. The key insight: wrong reasoning paths tend to be idiosyncratic and varied, while correct reasoning paths, though diverse in their approach, tend to converge on the same final answer.

### How It Works

1. **Generate**: Sample *N* independent reasoning paths from the same prompt at temperature > 0 (typically 0.5–1.0)
2. **Extract**: Pull the final answer from each reasoning path
3. **Vote**: Select the answer that appears most frequently (majority vote)

```
Prompt → LLM × N paths:
  Path 1: "...therefore 42" ✓
  Path 2: "...so the answer is 41" ✗  
  Path 3: "...which gives 42" ✓
  Path 4: "...the result is 42" ✓
  Path 5: "...equals 42" ✓
→ Majority vote → 42
```

### Performance Gains

Self-consistency delivers **5–25% accuracy improvements** on reasoning benchmarks. On GSM8K with 40 sampled paths: CoT alone achieves 56.5%, self-consistency achieves **74.4%**. On SVAMP: 68.9% → 82.4%.

### When It Works Best

Self-consistency shines when:
- There is a single objectively correct answer (math, logic, code)
- The model has genuine uncertainty (high-temperature sampling produces diversity)
- You can afford N × the normal inference cost

It degrades when tasks are subjective or open-ended (majority voting over essays is meaningless without a clear equivalence criterion).

### Relation to Other Techniques

Self-consistency is the simplest form of **test-time compute scaling** — you're spending more inference compute to improve accuracy. It also relates to Tree of Thought (structured search) and Process Reward Models (learned evaluation of candidate solutions). Universal Self-Consistency (USC) extends the approach to open-ended tasks by using the LLM itself to select the best answer from candidates.

---

## 4. Tree of Thought (ToT) <a name="tree-of-thought"></a>

### Motivation

CoT and self-consistency both share a fundamental limitation: they explore reasoning **linearly** or through **independent parallel paths** without any ability to backtrack, evaluate progress, or dynamically choose which branches of reasoning to pursue. Humans solving hard problems don't just follow one chain — they explore, evaluate, and prune.

Tree of Thought (ToT) was proposed by Yao et al. (Princeton/Google DeepMind, 2023) and Long (2023) to address this. The inspiration was explicit: Daniel Kahneman's distinction between **System 1** (fast, automatic) and **System 2** (slow, deliberate, exploratory) thinking. Standard LLM generation is System 1; ToT implements System 2.

### Architecture

ToT maintains an explicit **tree of partial solutions** where:
- Each **node** represents a coherent intermediate reasoning state (a "thought")
- **Branching** corresponds to generating multiple candidate next steps from any node
- The **LLM evaluates** each node's promise (as "sure/maybe/impossible")
- **Search algorithms** (BFS, DFS, beam search) navigate the tree

The four key components:

1. **Thought Decomposition**: Defining what counts as one "step" (e.g., a single equation for Game of 24, a paragraph for creative writing)
2. **Thought Generation**: Producing *k* candidate next thoughts from each state (via independent sampling or sequential proposals)
3. **State Evaluation**: Using the LLM itself as a value function to score how promising each partial solution is
4. **Search**: BFS (explore breadth before depth), DFS (commit and backtrack), or beam search

### Performance

On the Game of 24 (a mathematical reasoning task requiring using four numbers to reach 24):
- Standard prompting: 4% success
- CoT: 4% success  
- ToT (BFS, b=5): **74% success**

The gains come from ToT's ability to catch and abandon dead-end paths early, rather than committing to the first plausible-looking chain.

### Limitations

The main drawback is **inference cost**. ToT can require 10–100× more LLM calls than standard CoT. The search space grows exponentially with tree depth, requiring careful breadth/depth tradeoffs. NeurIPS 2024 work (*Chain of Preference Optimization*) demonstrated that fine-tuning LLMs on ToT search trees allows CoT to achieve similar performance without the search overhead at inference time — essentially distilling the ToT's exploration into the model's weights.

### ToT as a Prompt Pattern

A simplified "poor man's ToT" prompt pattern (Hulbert 2023):
```
Imagine three different experts are answering this question.
All experts will write down 1 step of their thinking,
then share it with the group.
If any expert realizes they're wrong at any point they leave.
The question is...
```

---

## 5. Graph of Thought (GoT) <a name="graph-of-thought"></a>

### Beyond Trees

Both CoT and ToT share a topological limitation: they only support **branching** — reasoning paths go forward and outward, but never merge. Real human problem-solving often involves synthesizing multiple partial insights into a unified solution. Graph of Thought (GoT), introduced by Besta et al. (ETH Zurich, 2023), generalizes the reasoning topology to an **arbitrary directed graph**.

```
Chain of Thought (CoT):    A → B → C → D → Answer  (Linear)

Tree of Thought (ToT):     A
                          /|\
                         B C D      (Branching, no merge)
                        /|   |
                       E F   G

Graph of Thought (GoT):    A
                          /|\
                         B C D
                        /|\ /|\
                       E F G H     (Branching AND merging)
                        \|/|/
                         I J       (Aggregation nodes)
```

### Core Operations

GoT defines four fundamental operations:

1. **Generate**: Produce new thoughts from existing ones (like CoT/ToT branching)
2. **Aggregate**: Merge multiple thoughts into a single refined thought (unique to GoT)
3. **Refine**: Iteratively improve a thought using feedback
4. **Score**: Evaluate thought quality

### The Merging Advantage

Consider sorting a large list. ToT branches into alternative sorting algorithms but never combines their outputs. GoT can:
1. Split the list into sublists
2. Sort each sublist independently (parallel branches)
3. **Merge** the sorted sublists (aggregation)
4. **Refine** the merged result

This divide-and-conquer pattern matches how humans actually solve decomposable problems, and maps naturally onto many algorithmic problems (merge sort, multi-document synthesis, multi-hypothesis research).

### Topology Summary

| Topology | Branching | Merging | Cycles | Best For |
|----------|-----------|---------|--------|----------|
| CoT | No | No | No | Linear multi-step |
| Self-Consistency | Parallel | Vote only | No | Uncertain single answers |
| ToT | Yes | No | No | Exploration, backtracking |
| GoT | Yes | Yes | Yes | Synthesis, decomposable problems |

---

## 6. Reasoning Tokens & The "Thinking" Paradigm <a name="reasoning-tokens"></a>

### What Are Reasoning Tokens?

Reasoning tokens (also called "thinking tokens," "think tokens," or simply the "reasoning trace") are the intermediate token sequences a model generates **before** its final answer. They represent the model's internal scratchpad — a stream of thought where the model plans, explores, verifies, and self-corrects.

The technical mechanism is straightforward:

**Phase 1 — Thinking**: The model generates a stream of internal tokens using the same transformer architecture as normal text generation. These tokens are processed, attend to each other, and update the model's internal state — but are either hidden from the user or presented as a distinct "thinking block."

**Phase 2 — Answering**: The final response tokens are generated, conditioned on both the original prompt and the full reasoning trace. The reasoning tokens have effectively served as "extended context" that the final answer can draw on.

Think of it this way: standard LLMs answer questions the way you might text a friend — quickly, from immediate recall. Reasoning models answer the way you'd work through a problem on paper before explaining it — the scratch work may not appear in the final answer, but it shaped every part of it.

### The System 1 / System 2 Analogy

Kahneman's dual-process theory maps neatly:
- **System 1 (fast, automatic)** → Standard LLM generation: one forward pass, immediate response
- **System 2 (slow, deliberate)** → Reasoning model with thinking tokens: sequential exploration, backtracking, verification

This isn't merely metaphorical. Reasoning tokens enable behaviors that are genuinely System 2-like:
- **Backtracking**: "Wait, that approach is wrong because... let me try instead..."
- **Hypothesis testing**: "If X is true, then Y should follow... checking... yes, that holds"
- **Self-verification**: Checking the answer against the problem constraints before finalizing

### Token Economics

Reasoning tokens aren't free. They count as output tokens for billing purposes, making reasoning model calls significantly more expensive:

| Model | Input cost | Output cost | Typical reasoning tokens |
|-------|------------|-------------|--------------------------|
| GPT-4o | ~$2.50/M | ~$10/M | 0 |
| OpenAI o3 | ~$2/M | ~$8/M | 500–30,000 |
| Claude Sonnet (extended thinking) | ~$3/M | ~$15/M | 1,000–100,000 |
| DeepSeek R1 | ~$0.55/M | ~$2.19/M | 500–20,000 |

A January 2026 analysis of 12,000 reasoning traces found that only ~21% of reasoning tokens were truly "decision-critical" — the rest was syntactic scaffolding. This has motivated research into more **token-efficient reasoning** through selective thinking, adaptive budgets, and distillation.

---

## 7. OpenAI o1/o3 Reasoning Models <a name="openai-models"></a>

### The o1 Breakthrough (September 2024)

OpenAI's o1 release in September 2024 marked a turning point. Rather than simply scaling parameters or context windows, OpenAI trained a model using **reinforcement learning to reason internally** — the model develops a private chain of thought before producing its visible response.

Key characteristics of o1:
- Trained with RL to reward reasoning processes, not just final answers
- Reasoning happens internally; users see only the final output (with token counts)
- Dramatically higher performance on math, science, and coding
- **AIME 2024** (American Invitational Mathematics Examination): o1 scored in the **89th percentile** among competitive programmers — tasks that require sustained multi-step mathematical reasoning

### OpenAI o3 (December 2024 / April 2025)

o3 (OpenAI skipped "o2" to avoid trademark conflicts with the British telecom carrier O2) substantially improved on o1:

| Feature | o1 | o3 |
|---------|----|----|
| Reasoning method | Chain-of-thought (internal) | Simulated reasoning + RL at scale |
| Tool use | Limited, prompted | Autonomous — decides when |
| Visual reasoning | Basic | Full mid-reasoning vision |
| Context window | 128K tokens | 200K tokens |
| SWE-bench score | 48.9% | 71.7% |
| Error rate vs experts | Baseline | 20% fewer major errors |

o3 introduced **configurable reasoning effort** (`low`, `medium`, `high`) — a "thinking budget" knob that trades accuracy for speed and cost. This was a crucial UX innovation: not all queries need maximum reasoning depth, and letting users/developers dial this up or down unlocks practical deployment.

### How o3 "Thinks"

OpenAI describes o3's process as **simulated reasoning (SR)**:

1. **Complexity recognition**: o3 evaluates prompt difficulty; simple queries get quick responses
2. **Private chain-of-thought**: For hard problems, o3 generates an internal reasoning chain it "argues with itself" through
3. **Autonomous tool use**: o3 can independently decide to invoke web search, code execution, or other tools mid-reasoning — the first OpenAI reasoning model to do so
4. **Self-correction**: If internal reasoning produces an inconsistent answer, o3 loops back and revises

The key architectural innovation: the RL training rewards *quality of reasoning processes*, not just correct final answers. This incentivizes the model to develop genuinely useful intermediate thinking patterns rather than plausible-sounding but hollow justifications.

### The o-Series Family

- **o1 / o1-mini**: First-generation reasoning models; established the reasoning model paradigm
- **o3 / o3-mini**: Second generation; autonomous tool use, vision mid-reasoning, 200K context
- **o3-pro**: Maximum reasoning depth; for highest-stakes tasks where reliability >> speed
- **o4-mini**: Distilled efficiency model comparable to o3 at ~half the cost

---

## 8. DeepSeek-R1: Reasoning Through Reinforcement Learning <a name="deepseek-r1"></a>

### Why R1 Matters

DeepSeek-R1, released in early 2025, sent shockwaves through the AI industry for two reasons: (1) it matched o1-level reasoning performance and (2) it did so via an **open-source release**, exposing the training methodology that OpenAI had kept proprietary. For the first time, the AI research community could study how to build state-of-the-art reasoning models.

R1 is a 671-billion parameter mixture-of-experts model. Its reasoning approach crystallized several key insights.

### The Two-Phase Training Story

#### Phase 1: R1-Zero — Pure RL, No Human Labels

The DeepSeek team first asked: *Can reasoning emerge from pure reinforcement learning, without any human-annotated reasoning examples?*

They trained **R1-Zero** using **Group Relative Policy Optimization (GRPO)** — an RL framework invented by the same team that avoids the need for a separate critic model (unlike PPO). The reward signal was purely outcome-based: did the model get the final answer right?

Remarkably, without ever being shown *how* to reason, R1-Zero **discovered that generating intermediate reasoning steps before committing to an answer improved its reward signal**. Chain-of-thought reasoning **emerged from the reward structure**, not from imitation of human examples.

R1-Zero achieved 86.7% pass@1 on AIME 2024 — comparable to o1. But it had a problem: the reasoning traces, while functionally effective, were often poorly formatted, mixed languages, and difficult for humans to follow.

#### Phase 2: R1 — Multi-Stage Training for Quality + Capability

The full R1 model uses a four-stage pipeline:

1. **Cold-start fine-tuning**: Fine-tune the base model (DeepSeek-V3-Base) on thousands of high-quality reasoning examples to establish formatting and readability
2. **Pure RL** (GRPO): Apply RL similar to R1-Zero to develop deep reasoning capability
3. **Rejection sampling + SFT**: Near RL convergence, the model generates its own synthetic training data; only high-quality examples are retained for supervised fine-tuning
4. **Final RL pass**: A final RL stage across diverse prompts for generalization

This multi-stage approach fixes R1-Zero's readability problems while preserving its reasoning power.

### The `

The 15th Fibonacci number is **610**.
```

The `` block contains:
- **Problem decomposition**: Breaking the question into sub-goals
- **Approach selection**: Considering and choosing solution strategies
- **Step execution**: Working through the solution with explicit intermediate results
- **Self-verification**: Checking answers against constraints
- **Backtracking**: "Wait, that's wrong because... let me reconsider"
- **Answer synthesis**: Committing to a final answer

Crucially, this is *not* post-hoc rationalization. The reasoning tokens are generated *before* the answer tokens, and the answer tokens attend to and are conditioned on the full reasoning trace.

### Distillation

One of R1's most impactful contributions: DeepSeek released **six distilled versions** (1.5B, 7B, 8B, 14B, 32B, 70B) created by fine-tuning smaller open-source models on R1's reasoning outputs. The 7B distilled model outperforms much larger non-reasoning models on structured tasks — demonstrating that reasoning capability can be transferred efficiently through knowledge distillation.

---

## 9. Claude's Extended Thinking <a name="claude-extended-thinking"></a>

### Anthropic's Approach

Anthropic's Claude offers **Extended Thinking** — a mechanism where Claude generates a "thinking block" before its final response. Unlike OpenAI's models (which hide reasoning entirely), Claude's extended thinking is **transparent by default**: the full reasoning trace is returned in the API response as a distinct content block.

### API Interface

```python
import anthropic

client = anthropic.Anthropic()

response = client.messages.create(
    model="claude-sonnet-4-6",
    max_tokens=16000,
    thinking={
        "type": "enabled",
        "budget_tokens": 10000  # Max tokens for thinking phase
    },
    messages=[{
        "role": "user",
        "content": "Prove that there are infinitely many prime numbers"
    }]
)

for block in response.content:
    if block.type == "thinking":
        print(f"Thinking: {block.thinking}")
    elif block.type == "text":
        print(f"Response: {block.text}")
```

The `budget_tokens` parameter gives developers direct control over the thinking budget — a key differentiator from OpenAI's effort-level system.

### Thinking Budget Guidance

| Budget | Token Range | Best For |
|--------|-------------|----------|
| Light | 1,000–5,000 | Simple calculations, short summaries |
| Moderate | 5,000–15,000 | Multi-step problems, code debugging |
| Heavy | 15,000–32,000 | Mathematical proofs, complex analysis |
| Maximum | Up to 128,000 | Research synthesis, long-form reasoning |

Note: `budget_tokens` can exceed `max_tokens` for the final response — thinking tokens are budgeted separately.

### Adaptive Thinking (Claude Opus 4.6)

Claude Opus 4.6 introduces **Adaptive Thinking**, which automatically calibrates thinking depth to problem difficulty:

```python
thinking={
    "type": "adaptive",  # Model decides depth
    "effort": "medium"   # "low", "medium", "high"
}
```

This mirrors o3's effort-level system and addresses the key practical concern: you shouldn't pay for deep reasoning on questions like "What's the capital of France?"

### Extended vs Prompted CoT

An important distinction: extended thinking is *not* the same as prompting Claude to "think step by step." Prompt-based CoT adds reasoning to the *output text*. Extended thinking uses a **genuinely separate computational phase** that runs before the response is generated — the thinking tokens are processed differently, with qualitatively different effects on the final answer quality.

Empirically, Anthropic reports accuracy on AIME 2024 increases **logarithmically** as thinking token budget increases — demonstrating a clear test-time compute scaling curve.

---

## 10. Process Reward Models (PRM) vs Outcome Reward Models (ORM) <a name="prm-vs-orm"></a>

### The Core Question: What Do We Reward?

When training or evaluating a reasoning model, a fundamental design choice arises: do you reward the **final answer** (outcome) or each **intermediate reasoning step** (process)?

### Outcome Reward Models (ORMs)

An ORM evaluates a complete solution and assigns a single scalar score based on whether the final answer is correct:

```
Problem → [Reasoning Step 1] → [Step 2] → ... → [Final Answer] → ORM → Reward
                                                                     ↑
                                                               Single score
                                                          (correct/incorrect)
```

**Advantages**: Computationally cheap (one evaluation per trajectory), scalable, doesn't require step-level annotations.

**Disadvantages**:
- **Sparse reward signal**: A 20-step derivation where step 3 contains an error still gets evaluated only at the end
- **Credit assignment problem**: If the model happens to arrive at the correct answer through flawed reasoning, ORM rewards the bad reasoning equally with good
- **Reward hacking**: Models can discover "lucky path" solutions that get correct answers through non-generalizable shortcuts

### Process Reward Models (PRMs)

A PRM assigns a score to **each individual step** in the reasoning chain:

```
Problem → [Step 1] → PRM: 0.9 → [Step 2] → PRM: 0.85 → [Step 3] → PRM: 0.1 ← ERROR!
                      ↑              ↑                      ↑
                Dense reward    Dense reward          Early error detection
```

**Advantages**:
- **Dense reward signal**: Every step gets feedback, resolving the credit assignment problem
- **Early error detection**: A bad step at position 3 is caught immediately, not obscured by subsequent "recovery"
- **Interpretability**: You can identify exactly where reasoning went wrong
- **Better search guidance**: PRMs can guide beam search, MCTS, and best-of-N selection far more effectively than ORMs

**Disadvantages**:
- **Expensive training data**: Requires step-level correctness labels, not just final answer labels
- **Miscalibration risk**: PRMs can assign high scores to locally coherent steps that lead to globally wrong answers
- **Reward hacking at step level**: Models may generate steps that score high on the PRM but aren't actually correct reasoning

### The PRM800K Benchmark

OpenAI's foundational research (*"Let's Verify Step by Step"*, Lightman et al., ICLR 2024) created PRM800K — a dataset of 800,000+ labeled reasoning steps from mathematical problem-solving. Key finding: **PRMs strongly outperform both ORMs and majority voting** at selecting correct solutions from a set of N candidates. With larger N (more candidates to choose from), the gap grows — the PRM is much better at identifying which of many candidate solutions is actually correct.

### Combined Approaches

Recent work (2025) has moved toward hybrid objectives:

$$R_{\text{combined}}(\tau) = \alpha \cdot R_{\text{outcome}}(\tau) + (1-\alpha) \cdot \frac{1}{T} \sum_{t=1}^{T} r_{\text{process}}(s_t, a_t)$$

Where α controls the balance between outcome-only and step-level supervision. The PROGRS framework (2025) demonstrated 74.9% vs 69.7% on MATH-500 by treating process rewards as *relative guidance* subordinate to outcome correctness — preventing PRM miscalibration from derailing training.

**Implicit PRMs (PRIME algorithm)**: A clever recent development shows PRMs can be trained using only outcome labels but applied as step-level evaluators at inference time, using the KL divergence between the policy and reference model as an implicit step reward:

$$r_{\text{implicit}}(s_t, a_t) = \log \pi_\theta(a_t|s_t) - \log \pi_{\text{ref}}(a_t|s_t)$$

This eliminates expensive per-step annotation while retaining the benefits of dense reward signals.

---

## 11. Monte Carlo Tree Search for Reasoning <a name="mcts"></a>

### From Games to Language

Monte Carlo Tree Search (MCTS) is the algorithm that enabled AlphaGo to defeat world champion Lee Sedol at Go in 2016. The core idea: build a search tree of possible future states, estimate which branches are promising via simulated rollouts, and allocate exploration budget to the most promising paths.

The structural similarity between game trees and reasoning trees is not coincidental — in both cases, you're navigating a vast combinatorial space where:
- Each state (board position / partial reasoning trace) leads to many possible next states
- Some paths lead to success, most don't
- A learned heuristic (position evaluation / reasoning quality assessment) can guide search far better than random exploration

### MCTS for Language Models: Four Phases

Adapting MCTS to reasoning requires mapping game concepts to language model concepts:

1. **Selection**: Traverse the tree from root to a leaf node using UCB1 (Upper Confidence Bound):
   ```
   UCB1(node) = (wins/visits) + C × √(ln(parent_visits)/visits)
   ```
   Exploitation term + Exploration bonus

2. **Expansion**: Generate *k* candidate next reasoning steps from the selected leaf, using the LLM as the generator

3. **Simulation/Evaluation**: Estimate the value of the new node — either via rollout (complete the reasoning chain and check answer) or via a trained value model (PRM)

4. **Backpropagation**: Update value estimates along the path from the newly expanded node back to the root

### Key Adaptations

**States as reasoning traces**: Instead of board positions, nodes represent partial reasoning chains — sequences of thoughts generated so far.

**LLM as policy**: The language model provides the action distribution at each node — which reasoning steps are worth exploring.

**LLM or PRM as value function**: Either use the LLM itself to evaluate partial solutions ("Is this reasoning path on track?") or train a dedicated PRM to provide value estimates.

### Applications in 2024–2025

- **ReST-MCTS***: Uses MCTS with PRM guidance to generate high-quality training data, which is then used to self-improve the base LLM — a powerful self-training loop
- **REKG-MCTS**: Applies MCTS to knowledge graph reasoning, with the LLM providing semantic guidance for path selection
- **Graph-MCTS**: Enhances RAG by using MCTS to navigate graph-structured knowledge
- **SRA-MCTS**: Self-driven reasoning augmentation for code generation, where MCTS enables self-generated intermediate reasoning steps and iterative self-evaluation

NeurIPS 2024 workshop work demonstrated MCTS-guided iterative preference learning (inspired by AlphaZero) boosting Mistral-7B accuracy from 75.9% → 81.8% on GSM8K, 28.9% → 34.7% on MATH.

### Why MCTS Works for Reasoning

The crucial ingredient is **verifiability**. MCTS works best when you can tell if you're on the right track. For mathematical problems, intermediate steps can be formally checked. For code, unit tests provide clear signals. For domains without clear intermediate verification, MCTS provides less value — it can't guide search without reliable value estimates.

---

## 12. Search-Augmented Reasoning <a name="search-augmented"></a>

### The Knowledge-Reasoning Boundary

Even the most capable reasoning model is limited by its training knowledge cutoff. Complex real-world problems often require accessing current information, specialized databases, or synthesizing information spread across multiple sources — capabilities no amount of reasoning sophistication can create from a frozen knowledge store.

Search-augmented reasoning integrates **retrieval and reasoning** as complementary processes, enabling models to actively seek information mid-reasoning rather than relying purely on parametric knowledge.

### The Evolution: RAG → Reasoning-Enhanced RAG → Synergized

**Basic RAG** (Retrieval-Augmented Generation): Retrieve relevant documents → inject into prompt → generate answer. Works for simple single-hop queries but fails on:
- Multi-step inference requiring information from multiple sources
- Questions where the right retrieval query depends on reasoning that hasn't happened yet
- Tasks requiring synthesis, not just retrieval

**Reasoning-Enhanced RAG**: Apply CoT, ToT, or other reasoning techniques *within* the RAG pipeline to improve retrieval query formulation, document selection, and answer synthesis. For example, using CoT to decompose a complex question into sub-questions, then retrieving for each sub-question independently.

**RAG-Enhanced Reasoning** (the reverse direction): Use retrieved external knowledge to ground and correct the model's reasoning chains, preventing hallucination on knowledge-intensive problems.

**Synergized RAG-Reasoning**: The frontier approach — **iterative interleaving** of retrieval and reasoning:
```
Reason → Identify knowledge gap → Retrieve → Integrate → Reason → Identify gap → Retrieve → ...
```

Models like OpenAI o3 implement this natively: during the private chain-of-thought, o3 can autonomously decide to invoke web search, code execution, or other tools — and this tool use is incorporated into the ongoing reasoning process before the final answer is produced.

### Key 2025 Systems

- **Search-o1**: Agentic search-enhanced large reasoning model that interleaves search and chain-of-thought
- **Open Deep Search**: Democratizes search-enhanced reasoning with open-source reasoning agents
- **Plan*RAG**: Efficient test-time planning for retrieval augmented generation
- **RARE** (Retrieval-Augmented Reasoning Enhancement): Specifically designed for grounding reasoning chains in retrieved facts

### Agentic Deep Research

The most ambitious instantiation: "deep research" products from OpenAI, Gemini, and Perplexity that orchestrate dozens of retrieval-reasoning cycles to produce comprehensive research reports. These systems:
1. Decompose a complex research question
2. Formulate targeted search queries
3. Retrieve and assess relevance
4. Synthesize across sources
5. Identify gaps and retrieve again
6. Produce structured outputs with citations

This represents a qualitative shift from "chatbot with web access" to "autonomous research agent."

---

## 13. Test-Time Compute Scaling <a name="test-time-compute"></a>

### The New Scaling Dimension

Traditional AI scaling laws focus on **training-time compute**: more parameters × more data × more training FLOPs → better models. This paradigm hit diminishing returns and soaring costs around 2023–2024.

Test-time compute scaling (TTC, also called inference-time scaling) introduces a **complementary axis**: for a fixed model, you can improve output quality by spending more compute at inference time. The key insight from Snell et al. (arXiv:2408.03314, 2024): there exist compute-optimal strategies for spending inference FLOPs, analogous to Chinchilla-optimal training compute allocation.

### Primary TTC Strategies

**1. Sequential reasoning (internal scaling)**
Train models to generate longer chain-of-thought internally before answering. The model allocates more tokens to reasoning, exploring more solution paths. This is what o1/o3/Claude extended thinking implement.

**2. Best-of-N sampling**
Generate N candidate responses in parallel, then select the best using a verifier (typically a PRM):
$$\tau^* = \arg\max_{\tau \in \{\tau_1,...,\tau_N\}} R_{\text{PRM}}(\tau)$$

**3. Beam search / tree search**
Maintain multiple partial solutions (beam width b), expand each, prune based on value estimates. More compute-intensive but more targeted than independent sampling.

**4. Iterative self-refinement**
Generate a response, critique it, revise, repeat. Useful when the model has a good self-evaluation capability.

### Scaling Curves

A critical empirical finding: TTC improvements follow **predictable scaling curves** analogous to training scaling laws. For reasoning tasks:
- Performance improves logarithmically with thinking token budget
- Gains are task-dependent: complex reasoning benefits most, factual recall benefits least
- Diminishing returns set in at high budgets, but the optimal frontier shifts with better verifiers

Compute-optimal TTC allocation depends on problem difficulty. The optimal number of samples N* for a given compute budget C:

$$N^*(p, C) = \arg\max_N \left[1 - (1-p)^N\right]$$

where *p* is the model's single-attempt success probability. For easy problems (*p* high), N*=1 is optimal. For very hard problems (*p* low), spending the budget on many attempts is optimal.

### The Key Tradeoff Matrix

| Strategy | Latency | Cost | Accuracy Gain | Best For |
|----------|---------|------|---------------|----------|
| Standard CoT prompt | Low | Low | +10–50% | Any complex task |
| Self-consistency (N=10) | Medium | 10× | +5–25% | Tasks with definite answers |
| Extended thinking (10K tokens) | Medium | 2–5× | +20–60% | Multi-step reasoning |
| Extended thinking (100K tokens) | High | 15–30× | +40–80% | Hardest math/science |
| ToT / MCTS | Very High | 50–100× | +50–200% | Exploration-heavy tasks |

---

## 14. The Paradigm Shift: Inference-Time vs Training-Time Compute <a name="paradigm-shift"></a>

### The Old Paradigm

From 2018 to 2023, the dominant paradigm in AI was clear: **scale training**. The scaling laws paper (Kaplan et al., 2020) demonstrated that model capabilities improved predictably with model size, dataset size, and training compute. This drove the race to build ever-larger models: GPT-3 (175B), PaLM (540B), GPT-4 (estimated ~1T+).

The implicit assumption: all the "thinking" happens at training time. The model's intelligence is encoded in its weights. At inference, you just retrieve that encoded intelligence as efficiently as possible.

### The Crack in the Paradigm

By 2023–2024, training-time scaling hit practical limits:
- Training costs became enormous ($50M–$100M+ for frontier models)
- Data quality/quantity constraints emerged
- Parameter scaling showed diminishing returns on benchmarks
- Environmental costs of massive training runs attracted scrutiny

Simultaneously, CoT experiments showed something surprising: you could get dramatically better performance from the *same model* by changing how it was *used* at inference time. The model's raw capability was being underutilized.

### The New Paradigm

Ilya Sutskever, when leaving OpenAI, described the emerging era as a shift "from scaling parameters to scaling the reasoning process." The core insight:

> **Inference-time compute is a new axis of intelligence.** For a fixed model, performance on complex tasks can be improved dramatically by spending more compute to reason — and this improvement can match or exceed what you'd get from a model trained on 10× more data or with 10× more parameters.

The economic logic is compelling. Training a model 10× larger requires 10× (or more) the capital investment, and the model is then deployed for every query at that larger size. Spending 10× more inference compute on *only the hard queries* that need it is potentially far more efficient.

### From Static to Dynamic Intelligence

The paradigm shift is from **static intelligence** (fixed capability determined at training time) to **dynamic intelligence** (capability that scales with the compute budget allocated to each query):

```
Old model:   Query → [Fixed Model] → Answer
                       (all compute at training)

New model:   Query → [Base Model] → [Thinking Phase: N tokens] → Answer
                                      (scale N with difficulty)
```

### Interplay: Training-Time AND Inference-Time

This isn't an either/or. Modern frontier models like o3 and Claude use **both axes**:

- **Training-time compute**: Builds the base model capability, teaches it *how* to reason, instills the RL-trained disposition to use thinking tokens effectively
- **Inference-time compute**: Applies that capability to specific problems with a dynamically allocated thinking budget

The training process increasingly focuses on teaching models *how to use* inference compute effectively — not just storing factual knowledge. Models trained with RL on reasoning tasks are better at allocating their thinking budget than models trained purely on next-token prediction.

### Implications

1. **Model routing**: Not every query needs o3-level reasoning. Intelligent routing between fast models (GPT-4o, Claude Haiku) and slow reasoning models (o3, Claude extended thinking) is a critical engineering concern

2. **Difficulty estimation**: Knowing in advance how hard a query is allows for optimal compute allocation. Active research area: classifying query difficulty before committing reasoning budget

3. **Verification bottleneck**: TTC only scales well when you can verify solutions. This creates a "verifiability frontier" — domains where easy verification (math, code) benefit enormously; domains where verification is hard (ethics, creative writing) benefit less

4. **The "thinking budget" as product**: Offering users/developers control over the thinking budget (o3's effort levels, Claude's budget_tokens) is itself a product innovation — you can choose accuracy vs latency vs cost per query

---

## 15. Comprehensive Comparison Table <a name="comparison"></a>

| Method | Type | Key Idea | Topology | Compute Cost | Best Use Cases | Year |
|--------|------|----------|----------|--------------|----------------|------|
| Standard prompting | Prompting | Direct Q→A | None | 1× | Simple queries | Pre-2022 |
| Chain-of-Thought (CoT) | Prompting | Step-by-step examples | Linear chain | 1.5–2× | Multi-step reasoning | 2022 |
| Zero-Shot CoT | Prompting | "Let's think step by step" | Linear chain | 1.5× | Any complex task | 2022 |
| Self-Consistency | Sampling | Majority vote over N paths | N parallel chains | N× | Tasks with definite answers | 2022 |
| Tree of Thought (ToT) | Search | BFS/DFS over reasoning tree | Tree | 10–100× | Exploration, backtracking | 2023 |
| Graph of Thought (GoT) | Search | Arbitrary graph: merge + loop | Graph | High | Decomposable synthesis | 2023 |
| ORM | Training/Eval | Score final answer only | N/A | Low | Simple pass/fail | 2022 |
| PRM | Training/Eval | Score each reasoning step | N/A | High (labeling) | Selecting among N solutions | 2023 |
| MCTS for Reasoning | Search | Explore/evaluate/backprop | Tree | Very High | Hard math, formal proofs | 2024 |
| OpenAI o1/o3 | Model (RL trained) | Internal chain-of-thought via RL | Internal tree | 5–50× | STEM, code, analysis | 2024 |
| DeepSeek-R1 | Model (RL trained) | GRPO-trained reasoning emergence | `` chain | 5–30× | Math, code, science | 2025 |
| Claude Extended Thinking | Model feature | Transparent thinking blocks | Thinking block | Configurable | Complex analysis, proofs | 2024 |
| Search-Augmented Reasoning | Architecture | Interleave retrieval + reasoning | Iterative | Variable | Knowledge-intensive tasks | 2023–2025 |
| Test-Time Compute Scaling | Framework | Dynamically allocate inference compute | Any | Configurable | Hard queries requiring accuracy | 2024–2025 |

---

## 16. Future Directions <a name="future"></a>

### 1. Parallel Thinking

Current thinking in models like Claude runs *serially* — one token at a time in a linear chain. Anthropic has stated it is experimenting with **parallel thinking**: exploring multiple reasoning branches simultaneously, much like how a human expert might pursue several solution approaches in parallel. This could dramatically improve reasoning quality without proportionally increasing latency.

### 2. Meta-Learning of Reasoning Strategies

Rather than learning *what to think*, future models may learn *how to think* — meta-strategies for allocating reasoning effort, choosing appropriate thinking styles for different problem types, and recognizing when more compute will and won't help.

### 3. Better Verification

The TTC scaling paradigm is fundamentally limited by verification quality. Advances in **formal verification integration**, **automated theorem proving**, and **symbolic math systems** could dramatically expand the domains where reasoning models can reliably self-verify and thus scale effectively.

### 4. Efficient Reasoning (Less is More)

As research identified that ~79% of reasoning tokens are non-decision-critical scaffolding, significant work is underway on **token-efficient reasoning**: training models to produce denser, more information-rich thinking traces; pruning redundant steps; and adaptive compression of reasoning chains.

### 5. Multi-Model Reasoning Architectures

Rather than one model thinking alone, future systems may involve **ensembles of specialized reasoners** — a planning model, a critic, an executor — working in concert, with explicit communication protocols between them. This mirrors organizational intelligence, where complex problems are solved by teams with specialized roles.

### 6. Training on Reasoning Traces

The reasoning traces generated by frontier models (R1, o3, Claude extended thinking) are themselves becoming valuable training data for smaller models. The **knowledge distillation of reasoning capability** — training 7B models to reason like 671B models — is an active and commercially significant research direction.

### 7. Reasoning About Uncertainty

Current reasoning models often project false confidence — they reason systematically to a wrong answer without appropriate epistemic hedging. Future work on **uncertainty-aware reasoning** would have models recognize when they're operating at the edge of their knowledge and communicate that appropriately.

---

## Conclusion

The history of AI reasoning is a story of progressively more sophisticated answers to a deceptively simple question: *how do you get a model to think before it speaks?*

Chain-of-Thought showed that the answer wasn't necessarily bigger models — it was structured intermediate steps. Self-consistency showed that diversity + aggregation could reduce error further. Tree of Thought showed that exploration and backtracking could unlock tasks that linear chains couldn't solve. Graph of Thought extended the topology to allow synthesis, not just search.

Concurrently, the reward modeling community developed the tools to *train* reasoning — PRMs providing the dense step-level signals needed to reinforce good reasoning processes, not just lucky answers. MCTS brought the search algorithms of game-playing AI to bear on language model reasoning.

OpenAI's o1 and o3, DeepSeek's R1, and Anthropic's extended thinking represent the synthesis: models that are *trained* (via RL) to use *inference-time compute* (thinking tokens) effectively on *hard problems*, guided by (implicit or explicit) process-level reward signals. The result is a class of models that can allocate "cognitive effort" dynamically — thinking longer and harder on difficult problems, returning to normal speed for easy ones.

The deeper paradigm shift is philosophical: intelligence is no longer just something encoded at training time. It's also something *applied* at inference time, through computation that can be scaled, budgeted, and directed. This opens a new design space for AI systems — where how you *use* a model is as important as how you *build* it, and where the line between the model's capability and the system's capability becomes productively blurry.

---

*Article compiled from research spanning Google Brain (CoT, 2022), Princeton/Google DeepMind (ToT, 2023), ETH Zurich (GoT, 2023), OpenAI (o1/o3, PRM800K, 2024–2025), DeepSeek (R1, 2025), Anthropic (Extended Thinking, 2024–2026), and current literature on test-time compute scaling as of April 2026.*

---

## ✅ Summary

**What I did:** Conducted 8 parallel web searches covering all requested topics — Chain-of-Thought, Tree/Graph of Thought, self-consistency, OpenAI o1/o3, DeepSeek-R1, Claude extended thinking, PRMs vs ORMs, MCTS for reasoning, search-augmented reasoning, reasoning tokens, and test-time compute scaling.

**What I found/accomplished:** Compiled a comprehensive ~6,000-word markdown article with:
- 16 major sections covering every requested topic in depth
- Technical details, key papers, benchmark numbers, and code examples
- A comprehensive comparison table across all methods
- Historical context and the core "paradigm shift" narrative tying everything together
- Forward-looking "future directions" section

**Files created/modified:** No files were written to disk — the complete article is delivered inline above as a comprehensive markdown document ready for Discord/chat rendering.

**Issues encountered:** None significant. All search queries returned high-quality, current information (including 2025–2026 research). The article synthesizes information from primary sources including OpenAI API docs, Anthropic docs, DeepSeek API docs, arXiv papers (NeurIPS 2024, ICLR 2024, ACL 2025), and technical blogs.