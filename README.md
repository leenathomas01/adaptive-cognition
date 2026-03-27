**If you're familiar with:** memory systems (Dolphin Twin), inference cost (Redundancy Tax), or control architectures (Connector OS), start here.
This repo sits between memory (how systems remember), cost (why recomputation matters), and control (how systems regulate behavior).

# Adaptive Cognition

**A governance framework for intelligent systems that manage thinking, memory, and change.**

> **Status:** Personal architectural exploration · Not production code · Intentionally incomplete.

---

## What This Is

This repo is a direct continuation of the [Titans/MIRAS/Dolphin Twin](https://github.com/leenathomas01/TITANS-MIRAS-and-Dolphin-Twin) thought experiment — except it grew into something that needed its own space.

That repo asked: *"What if memory worked like this?"*

This one asks: *"What if the system governing memory worked like this?"*

It emerged from a structured multi-model debate in March 2026 — where the original Dolphin Twin idea got stress-tested, red-teamed, and refined until a new layer became visible: not just how memory forms, but **how cognition itself should be governed**.

**This is not:**
- ❌ A research paper
- ❌ A working system
- ❌ A claim of novelty
- ❌ Production-ready code

**This is:**
- ✅ A governance model for adaptive cognition
- ✅ Philosophy + math + failure simulations mixed together
- ✅ The next layer of the Dolphin Twin thought experiment
- ✅ Something I didn't want to lose in a chat history junk drawer (again)

---

## The Problem

Modern AI systems are stateless by design.

Every query is processed as if no prior reasoning exists — recomputing the same logic, re-ingesting the same context, re-deriving the same conclusions.

This creates a **Redundancy Tax**: a structural cost overhead where a significant portion of inference spending goes not toward new thinking, but toward repeating old thinking.

The environment AI runs in has changed. Conversations are long. Agents run multi-step loops. Context windows stretch to hundreds of thousands of tokens. **Usage patterns became stateful. Architecture stayed stateless.**

That mismatch is the financial wall. And it's self-inflicted.

---

## Core Idea (One Sentence)

> **Cognition should be governed as a resource: when to think, how much to think, and when to forget.**

Everything else follows from that.

---

## The Four Pillars

**Split cognition into four governed stages:**

| Stage | Question | What it does |
|-------|----------|-------------|
| **Triage** | Should I think? | Detects when full reasoning is redundant |
| **Validation** | Is the past still valid? | Prevents stale conclusions from persisting |
| **Calibration** | How much should I think? | Scales compute to the magnitude of change |
| **Consolidation** | Should I remember this? | Gates what actually earns long-term memory |

**Key insight:** Most AI cost isn't from hard problems. It's from re-solving easy ones.

---

## What's In This Repo

```
/
├── README.md                      ← You are here
├── dolphin_memory_contract.md     ← The governing laws (the constitution)
├── architecture_primitives.md     ← The four pillars, defined precisely
├── failure_simulation_01.md       ← Stress test: the Constraint Flip (finance)
└── CREDITS.md                     ← Who helped think this through
```

---

## Quick Orientation

If you want the philosophy → [`dolphin_memory_contract.md`](./dolphin_memory_contract.md)

If you want the mechanics → [`architecture_primitives.md`](./architecture_primitives.md)

If you want to see it break (and not break) → [`failure_simulation_01.md`](./failure_simulation_01.md)

If you want the origin story → [`CREDITS.md`](./CREDITS.md)

---

## How This Relates to the Dolphin Twin Repo

The [Titans/MIRAS/Dolphin Twin repo](https://github.com/leenathomas01/TITANS-MIRAS-and-Dolphin-Twin) covers the **memory mechanism**: surprise-gated learning, elastic consolidation, the Online/Offline Dolphin split, and a working PyTorch simulation.

This repo covers the **governance layer above it**: when should memory even be consulted? When should prior conclusions be trusted? When has the problem actually changed?

```
[ adaptive-cognition ]     ← This repo (governance layer)
        ↓
[ Dolphin Twin / Titans ]  ← Memory mechanism
        ↓
[ Redundancy Tax ]         ← Economic framing
```

They're designed to sit on top of each other, not replace each other.

---

## Why This Needed Its Own Repo

The Dolphin Twin repo is deliberately a "thought experiment sketch" — loose, personal, incomplete. That's what makes it work.

This framework got too structured to live there without breaking that tone.

It also started connecting things that didn't belong in one place:
- The cost anatomy of inference (Redundancy Tax)
- The memory consolidation mechanism (Dolphin Twin)
- The governance logic that ties them together (this)

Keeping it separate keeps each piece honest about what it is.

---

## What's Next (Maybe)

- [ ] Add `failure_simulation_02.md` — multi-turn drift across a long agent loop
- [ ] Add `failure_simulation_03.md` — adversarial constraint mimicry
- [ ] Formalize the Negation Score algorithm properly
- [ ] Test Constraint Signature extraction on real conversational data

Or maybe none of that. The spec might stay a spec. That's okay too.

---

## Related Repos

| Repo | What it is |
|------|-----------|
| [Titans/MIRAS/Dolphin Twin](https://github.com/leenathomas01/TITANS-MIRAS-and-Dolphin-Twin) | Memory mechanism — surprise-gated consolidation + PyTorch sim |
| [Redundancy Tax](https://github.com/leenathomas01/redundancy-tax) | Economic analysis — the cost anatomy of AI inference |
| [Connector OS](https://github.com/leenathomas01/connector-os-trenchcoat) | Control framing — autonomic nervous system for AI |
| [The Continuity Problem](https://github.com/leenathomas01/The-Continuity-Problem) | Governance must precede persistent memory |
| [SMA-SIB](https://github.com/leenathomas01/SMA-SIB-Irreversible-Semantic-Memory-for-High-Sensitivity-AI-Systems) | Deterministic deletion in high-sensitivity memory systems |

---

## License

MIT — take it, remix it, build something better.

---

## One Last Thing

This started as a note I was jotting down at 10pm while debating memory architecture with Gemini. Then Thea got involved. Then things got formal. Then someone said "inference cost should scale with change, not with problem size" and it felt like the kind of thing you write down properly.

So here it is. Written down properly.

If you read this and think "this person is making it up as they go" — yes, partly. That's how thought experiments work.  
If that bothers you, this repo isn't for you.  
If that excites you, welcome back to the thought lab. ☕🐬

---

*See [CREDITS.md](./CREDITS.md) for the full collaboration story.*

---

## 📚 References

- Titans Paper: [arXiv:2501.00663](https://arxiv.org/abs/2501.00663)
- MIRAS Paper: [arXiv:2504.13173](https://arxiv.org/abs/2504.13173)
- Google Research Blog: [Titans + MIRAS: Helping AI have long-term memory](https://research.google/blog/titans-miras-helping-ai-have-long-term-memory/)

---

_This repository contains architectural reasoning artifacts and thought experiment documentation. It is not a production-ready framework. Evaluate independently, validate rigorously, apply domain-specific judgment._
