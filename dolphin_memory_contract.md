# Dolphin Memory Contract
### adaptive-cognition edition

> *Memory must not only be relevant—it must remain valid under current intent.*

---

## Purpose

This document defines the rules governing how an intelligent system manages thought, memory, and change.

It is the direct evolution of the [original Dolphin Memory Contract](https://github.com/leenathomas01/TITANS-MIRAS-and-Dolphin-Twin/blob/main/dolphin_memory_contract.md) from the Titans/MIRAS repo — which covered memory formation, retention, and forgetting. This version adds the governance layer: rules for when reasoning should be reused, revised, or discarded.

It is not an implementation guide. It is a **behavioral contract** — a set of rules that describe how cognition is *allowed* to operate in a system where compute is finite and memory is fallible.

---

## Core Principle

> **Surprise earns memory.**
> **Stability allows memory to survive.**
> **Neglect causes memory to fade.**
> **Validity determines whether memory may be used.**

The first three laws are from the original contract. The fourth is new — and it's what this repo is built around.

---

## The Two Layers

This contract governs two distinct layers of cognition:

| Layer | Question | Covered by |
|-------|----------|-----------|
| **Memory layer** | Should this experience persist? | Original Dolphin Twin contract |
| **Governance layer** | Should prior reasoning be trusted now? | This document |

Both layers are necessary. A system with memory but no governance will reuse stale conclusions. A system with governance but no memory will recompute everything from scratch.

---

## The Four Stages of Cognitive Governance

Every incoming query passes through four stages.

---

### Stage 1: Triage
**"Should I think?"**

Evaluates whether full reasoning is required, or whether prior work is reusable.

**Mechanism:** Lightweight checks before inference — lexical overlap, Constraint Signature comparison, embedding similarity.

**Tiered decision logic (cheapest first):**
1. Lexical check → obvious repetition → candidate for reuse
2. Constraint Signature check → same problem? → reuse or escalate
3. embedding similarity → nuanced match? → soft influence (bias toward prior solution) or full compute

**Design constraint:** The triage mechanism must cost strictly less than the cheapest avoided inference. If gating costs more than it saves, it has failed.

**What triage does NOT do:** It does not validate that reuse is safe. That is Stage 2's job.

---

### Stage 2: Validation
**"Is prior reasoning still valid?"**

Determines whether previous conclusions are applicable — even when the current query looks similar.

**Mechanism:** Constraint Signature matching + State Reversal detection.

**Key principle:** Relevance is insufficient. Validity is required.

> Similarity tells you the queries look alike.
> Validity tells you the answer still applies.
> These are different questions.

**State types:**

| State | Definition | Action |
|-------|-----------|--------|
| Continuation | Prior logic extends cleanly | Reuse |
| Pivot | One or more constraints shifted | Delta Reasoning |
| Reversal | Prior assumption explicitly or implicitly voided | Hard reset |

**Critical distinction:** A Reversal is not defined by how much the query changed. It is defined by whether prior constraints have been invalidated. A semantically similar query with a contradicted constraint is still a Reversal.

---

### Stage 3: Calibration
**"How much should I think?"**

Determines the depth of recomputation required.

**Mechanism:** Delta Reasoning — compute Δ (change in constraints), reuse stable structure, recompute only what changed.

**Governing principle:**
> Inference cost should scale with the magnitude of change (Δ), not the total size of the problem.

**The Warm Scaffold:**
When Delta Reasoning applies, the system separates:
- **Warm Scaffold** — validated reasoning structure from the prior turn (reused)
- **Delta Layer** — constraint-sensitive components (recomputed)

*Note: This is a logical model. In current transformer architectures, this would be approximated through a control layer, not literal activation reuse.*

---

### Stage 4: Consolidation
**"Should I remember this?"**

Decides whether the outcome of this interaction should persist in long-term memory.

**Mechanism:** Surprise-gating + stability checks.

**Key principle:** If the input is predictable, the system must silence its learning circuits.

*The full consolidation mechanism — including the Epistemic Firewall, momentum-based updates, and passive decay — is specified in the [original Dolphin Memory Contract](https://github.com/leenathomas01/TITANS-MIRAS-and-Dolphin-Twin/blob/main/dolphin_memory_contract.md) and its PyTorch simulation.*

---

## The Five Governing Laws

---

### Law 1: Selective Computation
The system must not recompute what it already knows.

If a problem is sufficiently similar to a previously solved problem, and the underlying constraints have not changed, prior reasoning may be reused.

*Corollary:* "Sufficiently similar" is defined by constraint consistency, not semantic proximity.

---

### Law 2: Validity Over Similarity
Relevance is secondary to intent.

If a user invalidates a prior assumption — explicitly or implicitly — the system must discard the associated conclusion immediately, regardless of how semantically similar the new query appears.

*Corollary:* A system that cannot distinguish "same question" from "question that looks the same" is not safe to reuse memory.

---

### Law 3: Cost Proportional to Change
Inference cost should scale with the magnitude of change (Δ), not the total size of the problem.

When a user modifies one constraint in an otherwise stable problem, the system should recompute only the portions affected by that change — reusing the validated reasoning structure for everything else.

*Corollary:* Full recomputation is not a safe default. It is a fallback for when change cannot be measured.

---

### Law 4: Explicit Invalidation
The system must detect and respect state reversal.

State reversal occurs when a user overturns a prior assumption—not merely modifies it within the same problem frame. Detection signals include lexical negation, constraint contradiction, and intent flip.

*Corollary:* A Reversal is not defined by Δ. It is defined by constraint invalidation.

---

### Law 5: Minimal Memory
Memory should retain only decision-critical invariants.

A well-formed memory is not a summary of what happened. It is a projection of what mattered. A Constraint Signature that tries to capture everything has failed its purpose.

*Corollary:* Intentional lossiness is a design feature, not a limitation.

---

## Interpretation

These laws operate hierarchically:

- Triage proposes reuse  
- Validation authorizes or rejects it  
- Calibration determines compute allocation  
- Consolidation governs future memory  

Failure at any stage propagates forward; correctness depends on all four.

---

## Core Mechanisms

---

### Constraint Signatures

A **Constraint Signature** is a compact representation of the variables that determine the outcome of a reasoning process.

It is **not** a summary of the input. It captures **only decision-critical invariants**.

**Structure:**

```
ConstraintSignature = {
    variables:    { key: value }      // inputs that drive the conclusion
    types:        { key: type }       // numeric | categorical | logical
    sensitivity:  { key: weight }     // [0.0–1.0] how much this variable matters
}
```

**Example (finance):**
```
variables:    { interest_rate: 5.5, leverage_ratio: 0.8, asset_type: "real_estate" }
types:        { interest_rate: "numeric", leverage_ratio: "numeric", asset_type: "categorical" }
sensitivity:  { interest_rate: 0.9, leverage_ratio: 0.7, asset_type: 0.6 }
```

**Design principle:** A Constraint Signature should be intentionally lossy. It discards everything that would not change the conclusion if altered.

---

### Delta Reasoning (Δ)

Δ represents the effective change in reasoning requirements between the current query and the prior state.

**Approximation:**
```
Δ = Σ (change_i × sensitivity_i)   for all i in constraint set
```
This is an approximation of effective change, not a precise scalar measure.

**Operational mapping:**

| Δ range | Constraint state | Action |
|---------|-----------------|--------|
| ~0 | All stable | Full reuse |
| Low–moderate | Partial change | Delta Reasoning (partial recompute) |
| High | Major shift | Full recompute |
| Any | Constraint invalidated | Hard reset + cache clear |

---

### State Reversal Detection

**Composite Negation Score:**

```
NegationScore = w₁ × lexical_negation
              + w₂ × constraint_conflict
              + w₃ × intent_flip
```

**Signal definitions:**

| Signal | Triggered by |
|--------|-------------|
| Lexical negation | "ignore," "actually," "instead," "start over," "that's wrong" |
| Constraint conflict | New value incompatible with stored value (weighted by sensitivity) |
| Intent flip | Shift from constructive to critical, or reversal of analytical direction |

**Decision:**
```
NegationScore > τ_reversal  →  Hard reset
NegationScore > τ_pivot     →  Pivot (Delta Reasoning)
else                        →  Continuation
```

---

### The Stale Reasoning Problem

Naive reuse introduces a failure mode symmetric to amnesia:

| Failure | Cause | Effect |
|---------|-------|--------|
| Amnesia | No memory | Redundant recomputation |
| Staleness | Blind memory | Skipped reasoning that should have happened |

Staleness is worse than amnesia—it fails silently and often with high confidence.

The Constraint Signature + State Reversal system exists specifically to prevent this.

---

## What This Contract Does Not Cover

This contract governs the **commodity layer** of cognition — routine interactions, follow-up queries, evolving conversations under stable conditions.

It does not address:

- **Genuine novel reasoning** — which remains irreducibly expensive and should be protected, not optimized away
- **Cross-session persistence** — the cache described here is session-scoped by default; long-term persistence involves safety, privacy, and governance tradeoffs outside this scope
- **Weight-level changes** — this is a control-layer model; the underlying consolidation mechanism lives in the [Dolphin Twin repo](https://github.com/leenathomas01/TITANS-MIRAS-and-Dolphin-Twin)

---

## 📑 Technical Appendix

Mathematical definitions for the mechanisms described above.

---

### A.1 Constraint Signature Comparison

For two signatures S_prev and S_curr, the constraint delta is:

```
Δ_constraint = Σ sensitivity_i × I(v_prev_i ≠ v_curr_i)
```

where I(·) is the indicator function and sensitivity_i is the weight of variable i.

A Reversal is flagged if any high-sensitivity variable is contradicted:

```
Reversal ← ∃i : sensitivity_i > τ_critical AND v_prev_i contradicts v_curr_i
```

---

### A.2 Delta Reasoning Cost Model

Standard inference: `Cost = f(|problem|)`

Delta Reasoning: `Cost = f(Δ, invariants)`

For a partially stable problem:

```
Cost_delta ≈ Cost_full × (Δ / Δ_max)
```

Where Δ_max represents total constraint replacement (full recompute).

*Note: This is a conceptual model. Actual savings depend on architecture and domain.*

---

### A.3 Negation Score

```
NegationScore = w₁ × L + w₂ × C + w₃ × F
```

Where:
- L = lexical negation signal ∈ [0,1]
- C = constraint conflict signal ∈ [0,1], weighted by sensitivity
- F = intent flip signal ∈ [0,1]
- w₁ + w₂ + w₃ = 1 (weights are domain-tunable)

Recommended starting values: w₁=0.3, w₂=0.5, w₃=0.2

*Constraint conflict carries the highest weight because it is the most objective signal.*

Thresholds (τ_reversal, τ_pivot) are domain-dependent and should be calibrated based on acceptable error tolerance.

---

### A.4 Surprise (from original contract)

Surprise is defined as the gradient norm with respect to memory parameters θ_m:

```
S_t = ||∇_{θ_m} L(x_t)||_2
```

Smoothed with momentum β:

```
S̃_t = β S̃_{t-1} + (1 - β) S_t
```

Memory update triggered when:

```
S̃_t > τ_t = μ_t + k · σ_t
```

*Full simulation in [titan_toy_extended.py](https://github.com/leenathomas01/TITANS-MIRAS-and-Dolphin-Twin/blob/main/titan_toy_extended.py)*

---

## Closing Note

This contract sits between philosophy and specification.

The original Dolphin Memory Contract closed with: *"Its purpose is to invite better implementations elsewhere."*

Same here. Take the laws, stress-test the mechanisms, find where they break.

That is still the success condition.

---

*Last updated: March 2026 · Collaborative synthesis: Claude (Anthropic), Gemini (Google), ChatGPT (OpenAI), human direction by Zee*
