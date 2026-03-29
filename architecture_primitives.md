# Architecture Primitives

**Precise definitions of the four governance stages.**

> This is the technical companion to [`dolphin_memory_contract.md`](./dolphin_memory_contract.md).
> The contract defines the laws. This document defines the mechanisms.

---

## Overview

The adaptive cognition governance layer operates through four sequential stages. Each is independently meaningful but designed to compose.

```
Incoming Query
      │
      ▼
┌─────────────┐
│   TRIAGE    │  Should I think?
└──────┬──────┘
       │
       ▼
┌─────────────┐
│ VALIDATION  │  Is prior reasoning valid?
└──────┬──────┘
       │
       ▼
┌─────────────┐
│ CALIBRATION │  How much should I think?
└──────┬──────┘
       │
       ▼
┌──────────────┐
│CONSOLIDATION │  Should I remember this?
└──────────────┘
```

---

## Execution Hierarchy

The stages operate with strict precedence:

- **Triage** proposes reuse
- **Validation** verifies constraint consistency
- **State Reversal** can override both — a detected Reversal terminates reuse regardless of similarity or Δ
- **Calibration** determines compute allocation for non-Reversal cases
- **Consolidation** governs what enters future memory

Failure at any stage propagates forward. Correctness depends on all stages operating in sequence.

---

## Stage 1: Triage

**Function:** Determine whether full inference is required.

**Design constraint:** The triage mechanism must cost strictly less than the cheapest avoided inference. If the gate costs more than it saves, it has failed.

**Tiered decision logic (cheapest check first):**

```
Stage 1a — Lexical check
  if lexical_overlap(current, prior) > τ_lex:
      → reuse candidate (proceed to Stage 1b)
  else:
      → proceed to Stage 1b or full recompute
      (lexical mismatch does not guarantee recompute — signature check may still confirm reuse)

Stage 1b — Constraint Signature check
  if signatures_consistent(S_curr, S_prev):
      → reuse confirmed
  else:
      → proceed to Stage 1c or Delta Reasoning

Stage 1c — Embedding similarity (only if 1a and 1b ambiguous)
  if cosine_sim(embed_curr, embed_prev) > τ_embed:
      → soft influence (bias toward prior, don't override)
  else:
      → full recompute
```

**What triage does NOT determine:**
- Whether reuse is *safe* → that is Validation's job
- *How much* to recompute → that is Calibration's job

---

## Stage 2: Validation (Constraint Signatures)

**Function:** Provide a compact, comparable representation of the decision-critical state of a reasoning process.

**Core definition:**
> A Constraint Signature captures only the variables that, if changed, would alter the conclusion.
> Everything else is intentionally discarded.

**Structure:**

```python
ConstraintSignature = {
    "variables":   { "key": value },      # inputs driving the conclusion
    "types":       { "key": "type" },     # numeric | categorical | logical
    "sensitivity": { "key": float }       # 0.0–1.0, how much this variable matters
}
```

**Sensitivity scale:**

| Range | Meaning | Example |
|-------|---------|---------|
| 0.9–1.0 | Critical | Interest rate in default model |
| 0.5–0.8 | Moderate | Time horizon, secondary variables |
| 0.1–0.4 | Low | Formatting, output style |
| 0.0 | Irrelevant | Do not include in signature |

**Comparison logic:**

```python
def signatures_consistent(S_prev, S_curr, threshold=0.15):
    delta = 0.0
    all_keys = set(S_prev["variables"]) | set(S_curr["variables"])

    for key in all_keys:
        # Variable appeared or disappeared
        if key not in S_prev["variables"] or key not in S_curr["variables"]:
            # Use max of both signatures to avoid bias toward current state
            sensitivity = max(
                S_prev["sensitivity"].get(key, 0.5),
                S_curr["sensitivity"].get(key, 0.5)
            )
            delta += sensitivity
        # Variable changed value
        elif S_prev["variables"][key] != S_curr["variables"][key]:
            delta += S_curr["sensitivity"][key]

    return delta < threshold
```

**Extraction approach (minimal viable):**
- Regex for numeric values and named quantities
- Entity recognition for key actors, assets, conditions
- Domain keyword extraction for known variable types

More sophisticated extraction can be layered on without changing the contract.

**Design note:**
> Do not over-engineer signature extraction prematurely.
> An imperfect signature that catches 80% of constraint flips is more valuable than a perfect signature that costs too much to compute.

---

## Stage 3: Delta Reasoning

**Function:** Calibrate the depth of recomputation to the magnitude of change.

**The shift:**

```
Standard:      Cost = f(problem_size)
Delta Reason:  Cost = f(Δ, invariants)
```

**Δ definition:**

```
Δ = Σ (change_i × sensitivity_i)   for all i in constraint set
```

Where `change_i = 1` if the variable changed, `0` if stable.
For numeric variables, `change_i` may instead be defined as a normalized distance rather than a binary indicator — allowing partial credit for small shifts in high-sensitivity variables.

**Operational states:**

| Δ | Constraint state | Action | Relative cost |
|---|-----------------|--------|--------------|
| ~0 | All stable | Full reuse | Minimal |
| Low–moderate | Partial shift | Delta Reasoning | Reduced vs. full recompute |
| High | Major shift | Full recompute | Full |
| Any | Constraint invalidated | Hard reset | Full + cache clear |

*Specific cost estimates belong in simulation documents, where they can be grounded in observable runs rather than stated as architectural claims.*

**The Warm Scaffold concept:**

When Delta Reasoning applies, cognition separates into:

- **Warm Scaffold** — validated reasoning structure from prior turn (stable, reused)
- **Delta Layer** — constraint-sensitive components (recomputed)

The system recomputes the Delta Layer using the Scaffold as a foundation, rather than starting cold.

*This is a logical architecture model. In current transformer implementations, this would be approximated through a control layer, not literal activation reuse.*

**Governing principle:**
> The system preserves validated reasoning structure while updating constraint-sensitive variables.

---

## Stage 4: Validation (State Reversal Detection)

**Function:** Identify when prior reasoning has been invalidated and must be discarded entirely.

**Precedence rule:** Reversal takes precedence over both similarity and Δ. A detected Reversal overrides any reuse signal from Triage or Calibration.

**Three state types:**

| State | Definition | Response |
|-------|-----------|----------|
| Continuation | Extends prior logic, no core assumption altered | Reuse (low Δ) |
| Pivot | Modifies constraint(s), problem structure preserved | Delta Reasoning |
| Reversal | Prior assumption contradicted or explicitly voided | Hard reset |

**Composite Negation Score:**

```
NegationScore = w₁ × lexical_negation
              + w₂ × constraint_conflict
              + w₃ × intent_flip
```

**Component definitions:**

**Lexical negation** (w₁ ≈ 0.3)
- Trigger phrases: "ignore," "actually," "instead," "start over," "that's wrong," "forget what I said"
- Limitation: Keyword matching alone is insufficient — "not only X but also Y" is not negation
- Backstop: Constraint conflict signal corrects false positives

**Constraint conflict** (w₂ ≈ 0.5, highest weight — most objective signal)
- Triggered when: new value is logically incompatible with stored value
- For numeric: change exceeds sensitivity-weighted threshold
- For categorical: key variable shifts to a different category
- **Acts as the primary anchor signal when other signals are ambiguous** — prevents lexical noise from dominating decisions

**Intent flip** (w₃ ≈ 0.2)
- Triggered when: analytical direction reverses ("assess X" → "critique X", "assume Y" → "question Y")
- Approximation: embedding difference vector + known reframing patterns

**Decision thresholds:**

```
NegationScore > τ_reversal  →  HARD RESET (invalidate cache, full recompute)
NegationScore > τ_pivot     →  PIVOT (Delta Reasoning)
else                        →  CONTINUATION (reuse, no reset)
```

**Critical design note:**
> Reversal is not defined by high Δ.
> It is defined by constraint invalidation.
>
> A query with high semantic similarity but a contradicted constraint is still a Reversal.
> A query with very different phrasing but consistent constraints is still a Continuation.

---

## Stage 5: Surprise-Gated Consolidation

**Function:** Determine whether the outcome of this interaction should be retained in long-term memory.

*(Full specification in the [original Dolphin Memory Contract](https://github.com/leenathomas01/TITANS-MIRAS-and-Dolphin-Twin/blob/main/dolphin_memory_contract.md) and [titan_toy_extended.py](https://github.com/leenathomas01/TITANS-MIRAS-and-Dolphin-Twin/blob/main/titan_toy_extended.py). Summary here for completeness.)*

**Surprise approximation:**

```
S̃_t = α × |∇L|           (gradient magnitude — learning pressure)
     + β × pred_error     (mismatch with expectation)
     + γ × entropy        (model uncertainty / confidence gap)
```

*(Consistent with the approximation used in [`dolphin_memory_contract.md`](./dolphin_memory_contract.md), Technical Appendix A.4)*

**Gate logic:**

```
S̃_t < τ      →  SKIP — routine input, no update
S̃_t ≥ τ      →  PROPOSE — begin stability validation
passes check  →  MERGE — update long-term store
fails check   →  REJECT or DEFER — flag for review
```

**Stability check (Elastic Anchor):**

Updates are constrained by the functional importance of prior stable patterns:

```
Stiffness_i = F_i × U_i × R_i × S_i
```

Where:
- `F_i` = Fisher Information (curvature — how much this weight mattered before)
- `U_i` = Usage frequency
- `R_i` = Recency decay
- `S_i` = Consequence sensitivity

A proposed update is only permitted if it doesn't destabilize prior competence beyond threshold ε.

---

## Full Composition

The five stages form a single decision loop per incoming query:

```
1. TRIAGE
   Is this different enough to warrant attention?
   → If not: reuse immediately

2. VALIDATION (Constraint Signature)
   Is prior reasoning still valid?
   → If not: continue to reversal check

3. STATE REVERSAL CHECK
   Has a prior assumption been voided?
   → If yes: hard reset, full recompute

4. DELTA CALIBRATION
   How much has changed?
   → Compute Δ, apply Delta Reasoning accordingly

5. CONSOLIDATION
   Was there a surprising outcome worth retaining?
   → If yes: gate, validate, consider merging
```

---

## What Is Not Specified Here

This document defines logical architecture, not implementation. Left intentionally open:

- **Exact threshold values** (τ, w₁, w₂, w₃, ε) — implementation and domain-dependent
- **Extraction methods** for Constraint Signatures — intentionally flexible
- **Transformer-level implementation** — this is a control-layer model
- **Cross-session persistence** — mechanisms here are session-scoped by default

These are deliberate omissions. Fixing them prematurely would lock in assumptions that should remain tunable.

---

*Architectural reasoning artifact. Logical specification, not an implementation guide.*
*Evaluate independently. Validate rigorously. Apply domain-specific judgment.*
