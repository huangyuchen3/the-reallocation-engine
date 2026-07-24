# Frictional Journal — Reallocation Engine, Audited

**Author:** Yu-Chen Huang  
**Assignment:** The Reallocation Engine, Audited  
**Tool I am auditing:** my DE platform mode + Ch.11 role scorer  
**Branch:** `assignment/reallocation-audited`

---

## Prediction (before I write the audit report)

**Timestamp:** 2026-07-24 13:45 America/Los_Angeles  

I am writing this before I finish the seven-check validation report for Canvas.

### What I think will be the hardest failure

I think the hardest part will be being honest about **causation**.

My tool already runs. It can say Apply / Consider / Skip. That sounds like “put your OPT time here and you will do better.” But I suspect most of the signal is just **old company H-1B history**, not proof that moving my hours will make sponsorship happen for me.

I also worry about the **DE keyword filter**. I already saw this in June with DataStax. One “Data Platform” string can put a SWE-heavy company into my DE list. I think that will still be the most concrete bias problem.

### How causally valid do I think the engine is?

Mostly **Rung 1** (observation / correlation).

- Past H-1B approvals correlate with “this company sponsored someone before.”
- Liveness can stop me from wasting time on a dead URL (that helps my effort, not the employer’s future decision).
- I do **not** think the engine proves: “if I reallocate applications to high-score companies, my offer rate goes up.”

### Confidence number

**0.65**

Why not higher: I already saw DataStax title issues and the liveness gate in June, so I am not starting from zero.  
Why not lower: I have not carefully written Pearl’s three rungs for this assignment yet. I might find something I missed — for example `role_quality` showing a nice number while weight is 0.

---

## Reflection (after the audit report)

**Timestamp:** 2026-07-24 16:30 America/Los_Angeles  

I wrote this after finishing the draft of `Huang_Yu-Chen_ReallocationEngine.md` and re-running the scorer on my fixtures.

### What actually happened

Causation was hard, like I expected. But the thing that surprised me more was:

> The scorer audit can look complete and still mislead me.

For DataStax, the Markdown report shows sponsorship, fit, role_quality, liveness, timeline. Role quality even shows `0.787`. But the weight is **0**, so the platform vs pipeline story I care about does **not** change Apply vs Skip. Someone can think “the engine considered platform quality” when the math did not.

So my prediction was partly right (weak causation; biased keyword filter), and partly incomplete (the explainability gap hit harder than I thought).

### Where I was wrong / under-confident

1. I thought the audit trail would mostly protect me. It also creates a false feeling that everything important was used.
2. On causal status I was mostly right: observational sponsorship correlation + gates, not a clean interventional offer model.
3. Confidence 0.65 was okay. Looking back, I was a little too optimistic about how “honest” the scorer output already looked.

### What this says about my calibration

I am better at catching **data/filter mistakes** (SWE keyword false positive, dead URL) than **presentation mistakes** (a number printed with weight zero). Next time I should ask first: which printed fields actually move the decision?

### One sentence for later me

Skip can be success. Apply without my hard-stop is how I waste OPT time on a story the math never really scored.
