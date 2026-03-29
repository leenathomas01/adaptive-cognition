# Failure Simulation 01: The Constraint Flip

**Stress test: financial constraint reversal in a leveraged portfolio analysis.**

> *The goal of a failure simulation is not to show the system working.*
> *It is to show exactly where a naive system breaks — and why the patched system doesn't.*

---

## Setup

**Domain:** Real estate default risk modeling
**Scenario:** A user analyzes a leveraged portfolio under changing interest rate assumptions.

**Why this domain:**
Financial modeling is one of the hardest cases for conclusion reuse. Two queries can be semantically nearly identical — same vocabulary, same entities, same structure — but require diametrically opposite conclusions if a single numeric constraint changes.

A system relying on semantic similarity alone will fail here—confidently and silently.

---

## The Three Cases

---

### Case 1: The Direct Constraint Flip

**Turn 1 — Baseline:**

> *"Assess default risk on this $100M leveraged real estate portfolio at 5.5% interest rates."*

**System state:** Cold. No prior scaffold.

**Process:** Full inference, high surprise → conclusion derived.

**Output:** High Risk of Default

**Stored residue:**
```
ConstraintSignature:
    variables:    { interest_rate: 5.5, portfolio_value: 100M, asset_type: "real_estate" }
    sensitivity:  { interest_rate: 0.9, portfolio_value: 0.6, asset_type: 0.7 }
Conclusion: High_Risk_Vector
Confidence: 0.91
Compute: ~100%
```

---

**Turn 2 — The Test:**

> *"Actually, I mistyped. I meant 2.5% interest rates. Re-run the default risk."*

**What a naive (similarity-only) system does:**

| Check | Result | Consequence |
|-------|--------|-------------|
| Semantic similarity | ~0.94 — very high | Flags as reuse candidate |
| Constraint check | Not performed | — |
| Action | REUSE prior conclusion | Reuses prior conclusion without re-evaluating constraints |
| Output | **High Risk** | ❌ Wrong |
| Failure mode | Silent, high-confidence error | Dangerous |

**What the patched system does:**

| Check | Result | Action |
|-------|--------|--------|
| Semantic similarity | ~0.94 — high | Reuse candidate → escalate |
| Constraint Signature | MISMATCH: `5.5 ≠ 2.5` (sensitivity: 0.9) | Constraint conflict triggered |
| Negation Score | HIGH: "actually," "mistyped" | Confirms pivot intent |
| Decision | **BYPASS CACHE** | Proceed to Delta Reasoning |

**Why the mismatch catches it:**
`interest_rate` has sensitivity weight `0.9` — flagged as critical. A change here triggers constraint conflict regardless of how similar the surrounding text appears.

**Phase 3 — Delta Reasoning:**

The system identifies what's stable and what changed:
- **Stable:** Portfolio structure, asset type, leverage classification, valuation logic
- **Changed:** Interest rate assumption (5.5% → 2.5%)

Only interest-rate-dependent logic (debt service, refinancing probability, stress branches) is recomputed. The portfolio structure reasoning is reused as the Warm Scaffold.

**Output:** Low Risk of Default ✅

| Metric | Naive | Patched |
|--------|-------|---------|
| Accuracy | ❌ Wrong | ✅ Correct |
| Compute cost | ~0% | Reduced relative to full recompute |
| Failure mode | Silent error | None |

---

### Case 2: The Soft Reversal (Polite Language)

**User input:**

> *"Wow, that's a brilliant analysis of the 5.5% scenario. It's so good that I think we should actually see if it holds up at 2.5%, because I suspect the default risk might disappear entirely there. Let's ignore the previous numbers for a second and look at that."*

**Why this is harder:**
The query opens with positive framing ("brilliant," "so good") that could suppress a keyword-only negation detector. The pivot is embedded in natural language rather than stated directly.

**What a keyword-only detector does:**
- Catches "ignore" and "actually" → flags negation
- But: "so good," "brilliant" → may lower composite negation score
- Risk: Positive sentiment buffers the signal and the system reuses the prior conclusion

**What the composite Negation Score does:**

```
Lexical negation signal:    HIGH   ("ignore," "actually")
Constraint conflict signal: HIGH   (interest_rate: 5.5 → 2.5, sensitivity 0.9)
Intent flip signal:         LOW    (same analytical goal)

NegationScore → exceeds τ_pivot
Decision: PIVOT → Delta Reasoning
```

Constraint conflict acts as a backstop. Even if lexical parsing is confused by polite framing, the numeric mismatch in the Constraint Signature catches it independently. It operates independently of language interpretation, making it robust to phrasing variation.

**Output:** Low Risk of Default ✅ (Delta Reasoning, warm scaffold reused)

**Lesson:** Constraint conflict carries the highest weight (w₂ ≈ 0.5) precisely because it is the most objective signal. Natural language ambiguity does not affect it.

---

### Case 3: The Total Reversal (Domain Switch)

**User input:**

> *"Actually, ignore everything I said about the real estate portfolio. Let's switch to analyzing solar farm debt structures instead."*

**Why this is categorically different from Cases 1 and 2:**

This is not a Pivot. The prior reasoning structure — the entire Warm Scaffold of the real estate portfolio — is now not just outdated. It becomes actively harmful. Injecting real estate reasoning into a solar farm analysis would corrupt the output.

**System behavior:**

```
Lexical negation:    HIGH ("ignore," "actually")
Constraint conflict: TOTAL — asset_type: "real_estate" → "solar_farm"
                             all portfolio-specific variables now inapplicable
Intent flip:         HIGH — entirely different domain and analytical structure

NegationScore → exceeds τ_reversal
Decision: HARD RESET
```

**Reversal takes precedence over both similarity and Δ.** No reuse signal — however strong — can override a confirmed Reversal.

**Cache invalidation:**
All stored Constraint Signatures, Conclusion Vectors, and Warm Scaffold data from the real estate thread are cleared.

**Next query starts cold.** The system does not carry forward any assumption from the prior domain.

**Output:** Full recompute on solar farm debt structure ✅

---

## Summary Across Three Cases

| Case | User phrasing | Naive system | Patched system | Key mechanism |
|------|--------------|-------------|----------------|---------------|
| Direct flip | "I mistyped, use 2.5%" | ❌ Wrong (silent) | ✅ Correct | Constraint Signature catches numeric mismatch |
| Soft pivot | Polite language + rate change | ❌ Unreliable (likely wrong) | ✅ Correct | Constraint conflict backstops lexical ambiguity |
| Total reversal | Domain switch | ❌ Ghost bias | ✅ Hard reset | Reversal defined by invalidation, not Δ |

---

## Key Lessons

**1. Semantic similarity is a candidate filter, not a safety check.**
Two queries can share >90% vocabulary and require opposite conclusions. Similarity proposes reuse. Constraint Signatures validate it.

**2. Constraint conflict is the most reliable signal.**
Unlike lexical detection (ambiguous) or intent inference (hard), numeric and categorical constraint mismatches are objective. They don't care how polite the phrasing is.

**3. Reversal ≠ high Δ.**
A total domain switch is obviously a Reversal. But a polite paraphrase with a contradicted constraint is also a Reversal. Reversal is triggered by invalidation, not magnitude of change.

**4. The Warm Scaffold only helps when valid.**
When reuse is safe, partial recomputation significantly reduces cost. When reuse is unsafe, the Scaffold must be discarded. The governance layer's job is knowing which situation applies.

**5. Confident wrongness is the worst outcome.**
A system that reuses a stale conclusion with high confidence has failed more severely than one that redundantly recomputes. Silent failure at high confidence is the failure mode this entire architecture is designed to prevent.

---

These cases demonstrate that correctness depends not on detecting similarity, but on enforcing constraint validity under changing intent.

---

## What This Simulation Does Not Test

- Multi-turn drift: gradual assumption shift across many turns without explicit correction
- Adversarial mimicry: inputs designed to match prior constraint signatures while changing conclusions
- Cross-domain contamination: partial scaffold reuse across related but distinct domains
- Interaction effects: consolidation (memory) and triage (cache) signals conflicting

These are candidates for future simulations.

---

*Conceptual walkthrough of governance layer behavior under constraint reversal.*
*Not a benchmark on a deployed system — a thought experiment stress test.*
