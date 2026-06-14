# Chapter 14 — Recipes: Operating the Engine
*The fluent surface from Chapter 1 has come back, this time wearing the engine's own uniform.*

Here is a failure mode worth understanding before you run anything. You invoke a recipe called `oferta` to evaluate a role. It returns a confident assessment: sponsorship looks strong, fit is solid, the composite is above threshold, you should apply. The output reads exactly like the verified findings you've trusted all book — sourced factors, labeled judgments, a traceable recommendation. But under the hood, something has changed. This run never called the sponsorship pipeline. It never read an audit. It asked a language model what it thought and dressed the guess in the format of a finding. You can't tell from the output. That's the whole danger.

The difference between a recipe that *runs a script and reads an audit* and a recipe that *quietly becomes a chat prompt* is the difference between the entire method working and the entire method failing silently. This chapter is about telling them apart — not once, but every time you run the engine.

## What a recipe is

The engine's runtime lives in a directory called `recipes/`. A recipe is a named operation you invoke, and each one declares what scripts it calls, what data it reads, and what it logs. The declaration is not decorative. It is the thing you verify. A recipe that calls no scripts and reads no audits is producing model judgment, not findings — and the output format will not tell you which one you're looking at.

Every recipe opens by loading `recipes/_shared.md`, the contract from Chapter 3. The contract is supposed to make the runtime honest by construction: every output labeled, every source named, every judgment flagged. Your job is to confirm, run by run, that it actually is.

The recipes split into two groups, and understanding the split is the chapter's load-bearing insight.

**Active recipes** are the ones that do verified work. `scan` detects the ATS and pulls the company's current postings. `pipeline` runs the scoring pass — sponsorship, fit, liveness, timeline — against the pulled data. `oferta` assembles the four factors into the composite from Chapter 11 and returns a sourced Apply/Consider/Skip recommendation. `tracker` logs decisions across the search (Chapter 15). `pdf` renders the ATS-safe résumé (Chapter 13). These recipes call real scripts. They read real audits. Their outputs are findings.

**Draft and helper recipes** are scaffolds — `apply`, `contacto`, `deep`, `followup`, `interview-prep`, `ofertas`, `project`, `training`, and others. They can be useful. But until you have confirmed, on a specific run, that a given draft recipe calls scripts and writes a log, you treat its output as model judgment. Not worthless. Not a finding.

<!-- → [TABLE: Two-column table listing active recipes vs. draft/helper recipes. Active column: scan, pipeline, oferta, tracker, pdf — with one-line description of what each calls. Draft column: apply, contacto, deep, followup, interview-prep, ofertas, project, training — with the label "verify before trusting" for each. Caption: "The split is not permanent — a draft recipe that you verify runs scripts becomes trustworthy. The taxonomy is about evidence, not hierarchy."] -->

The taxonomy is not a permanent ranking. A draft recipe you have verified is trustworthy for that run. An active recipe that has been edited without your knowledge might not be. The classification is always about what the recipe did on this run, not what it is named.

## The loop

Operating the engine is one loop, repeated:

1. **Run** an active recipe against a real target.
2. **Inspect** the output *and its provenance* — did it call the script? Is there an audit? Which numbers trace to records and which are labeled judgments?
3. **Record** the run in `RUN_LOG.md` — what you ran, what it returned, what you decided.

The loop is deliberately boring, and the boredom is the safety. Each pass leaves a trace. A decision made three weeks ago can be reconstructed, questioned, and updated when new information arrives. The moment you skip the inspect step — accepting a recipe's output because it looks right, because the format is familiar, because the recommendation matches what you hoped — you've reopened the fluency trap. The surface was the danger in Chapter 1. It is still the danger in Chapter 14.

## A full sequence, end to end

Take one role from URL to evaluation:

```bash
# 1. scan — detect the ATS and pull the company's current postings
npm run ats:scan

# 2. pipeline — score the pulled roles
#    (run via the pipeline recipe / auto-pipeline)

# 3. oferta — evaluate one role into a composite + Apply/Consider/Skip
#    (oferta recipe; returns the sourced composite from Chapter 11)

# 4. verify — confirm the pipeline's data is internally consistent
npm run ats:verify
```

Four commands. Four links in a chain. What you read is the chain — not just the final recommendation, but each link's provenance. The scan result tells you which ATS was detected and which postings were pulled. The pipeline pass shows the four scored factors with their source labels. The `oferta` composite traces each term to its origin. The verification audit confirms the underlying data is consistent. A recommendation without its provenance is just an opinion in a formatted box.

## One target, fully operated

Let me make the loop concrete. A single role URL at a Likely-tier startup.

`scan` detects Lever and pulls the current posting. The posting is nine days old, description is specific, company hiring count has changed since last week — the liveness signal from Chapter 8 is clean.

The `pipeline` pass scores the role. Sponsorship probability from the LCA and H-1B join: 0.65 — Likely tier, not Proven. Fit from CV-vs-JD comparison: 0.72 — a model judgment, labeled as such. Liveness: live. Timeline factor: 0.8, from the reader's OPT expiration and estimated processing window.

`oferta` assembles the composite. Sponsorship at Likely (0.65) carries less weight than Proven (0.9) would. Fit is reasonable. The multipliers hold. The composite lands above threshold but not comfortably — the recommendation is **Consider**, not Apply.

`ats:verify` runs. The data is internally consistent. No flag.

`RUN_LOG.md` gets one entry: the command sequence, the composite with each factor labeled by source, the recommendation, and a one-line decision: *Compare against other Considers before spending an application slot.*

That's the whole loop. What makes it different from just reading the recommendation is the inspect step — knowing that the sponsorship term is 0.65 not 0.9, knowing why the composite landed on Consider rather than Apply, knowing which factor would have to improve to push it across the threshold. The recommendation is the headline. The provenance is the argument.

<!-- → [DIAGRAM: Flow diagram showing scan → pipeline → oferta → verify as four sequential boxes, each with a "provenance checkpoint" annotation (e.g., "ATS detected, postings list," "four factors with source labels," "composite with traced terms," "consistency audit"). Arrow at the end pointing to RUN_LOG.md. Caption: "The chain is only as trustworthy as its weakest provenance link — the verify step confirms consistency, not correctness."] -->

## The failure that looks like success

The error that ends the method quietly: trusting a recipe by its name rather than its behavior.

"It's called `oferta`, so it must be evaluating with real data." Names are labels. Behavior is evidence. A recipe can be edited — by you, by a collaborator, by a future version of the system — so that it stops calling its script and starts generating plausible-sounding output from the model's priors. The output format stays the same. The sourced-factors layout stays the same. The Apply/Consider/Skip recommendation stays the same. Nothing in the presentation tells you that the sponsorship probability now comes from the model's sense of what a good biotech sponsorship probability should be rather than from an LCA record.

The inspect step is the only protection. On every run of every recipe you plan to trust, you confirm: did it call the script? Is there an audit? Can I trace the sponsorship number to a record?

If you can't answer yes to those questions, the output is a model judgment. It may be useful. It may even be accurate. But it is not a finding, and you should not make a Skip or Apply decision on its basis.

<!-- → [TABLE: Decision matrix — three rows: "Active recipe, provenance visible (script called, audit present)," "Draft/helper recipe, or provenance absent," "Any recipe whose output you can't trace." For each: treatment (finding / model judgment / distrust the output), action (record and act / useful for drafting, not for decisions / re-run with inspection). Caption: "The recipe's name is not the evidence. The provenance is the evidence."] -->

## What the loop cannot do

The run-inspect-record loop guarantees you *can* see provenance. It cannot guarantee you will draw the right conclusion from what you see.

Today's scan might have hit a stale cache and returned a posting that closed yesterday — the data is real, but the world has moved. The fit score of 0.72 might be confidently wrong for a role where your unusual project background is exactly what the team needs and no keyword in the job description captured it. The timeline factor rests on an estimate of processing time that the system cannot verify against the actual adjudication queue.

The engine runs the components and surfaces the evidence. The decision about whether a given run is trustworthy enough to act on is yours, every time. Automation makes the loop fast. It does not make it self-policing. A method that runs itself without a human in the inspect step is not the method.

## The shape of what the book has built

This is the chapter where all five components run together for the first time. Sponsorship detection (Chapter 7), liveness classification (Chapter 8), role quality (Chapter 9), timeline (Chapter 10), the composite scorer (Chapter 11) — each one producing a number with a labeled source, each number flowing into the oferta composite, each composite going into the run log as a traceable decision.

The engine is not complicated. It is thorough. The sophistication is not in the architecture; it is in the habit of inspection. You can run the whole sequence in under ten minutes for a single role. What takes discipline is doing the inspect step every time instead of shortcutting to the recommendation because the format looks familiar.

Running the engine produces decisions. A decision you can't reconstruct is one you can't learn from — and learning from the search is the subject of what comes next.

---

## Exercises

**Warm-up**

1. *(Recall, easy)* Name the five active recipes and describe in one sentence what each one does. Then name three draft/helper recipes and explain what "verify before trusting" means in practice for each.
   *Tests whether you can articulate the active/draft distinction before applying it.*

2. *(Recall, easy)* List the three steps of the run-inspect-record loop. For each step, write one sentence describing what you are confirming and why skipping that step reopens the fluency trap.
   *Tests whether you understand the loop as a safety mechanism, not just an operating procedure.*

3. *(Identify, easy)* What is provenance, and why does a composite recommendation without it reduce to an opinion in a formatted box? Give one example from the `oferta` output of what provenance looks like and what its absence would look like.
   *Tests whether you can distinguish a finding from a well-formatted guess.*

**Application**

4. *(Apply, moderate)* Run a full `scan → pipeline → oferta → verify` sequence on one real target role. Log each step in `RUN_LOG.md` with the command, key output, and provenance. Write the final recommendation with each of its four factors labeled by source type.
   *Tests the transition from understanding the sequence to executing it on live data.*

5. *(Analyze, moderate)* Pick one draft or helper recipe you haven't verified. Run it and inspect the output for provenance signals: did it call a script, is there an audit, can you trace any number to a record? Write your verdict — finding-grade or judgment-grade — and explain the specific evidence that drove the classification.
   *Tests the verify-before-trusting discipline on a recipe that doesn't announce its status.*

6. *(Analyze, moderate)* Describe how you would detect that an active recipe has drifted — stopped calling its script without announcing it. What would be present in a genuine active-recipe run that would be absent in a drifted run? What is the earliest point in the inspect step where you would catch the failure?
   *Tests adversarial reasoning about recipe drift as a realistic failure mode.*

**Synthesis**

7. *(Synthesize, harder)* The chapter argues that the run-inspect-record loop's safety comes from the inspect step, not from the automation. Construct a scenario in which the loop runs correctly — scripts called, audit present, log written — but produces a confidently wrong recommendation. Identify which component's input was bad, trace how the error propagated through the composite, and describe what the run log would and would not tell you about the failure.
   *Tests whether you understand the loop as necessary but not sufficient for correct decisions.*

8. *(Synthesize, harder)* The `recipes/_shared.md` contract is supposed to make the runtime honest by construction. Describe two ways the contract could fail — not because it is written incorrectly, but because the runtime doesn't enforce it. For each failure, write the specific `RUN_LOG.md` entry that would expose the problem and the one you would see if you weren't inspecting carefully.
   *Tests whether you can reason about contract enforcement as a behavioral property, not just a textual property.*

**Challenge**

9. *(Evaluate, open-ended)* Design a lightweight audit protocol — taking no more than two minutes per run — that would reliably catch the three most likely failure modes of the run-inspect-record loop: recipe drift, stale cache data, and a mislabeled model judgment treated as a record. For each failure mode, specify the exact check, the observable signal that indicates failure, and the corrective action. Then identify which of the three is hardest to catch and explain why.
   *Tests whether you can reason about the loop's failure surface as an engineering problem with practical constraints.*
