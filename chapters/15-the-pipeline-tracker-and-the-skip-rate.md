# Chapter 15 — The Pipeline Tracker and the Skip Rate

<!-- voice-anchored: root style/VOICE.md. Anatomy: TIKTOC Part 10.
     Sourced from SDD Component 5, plain-summary (skip rate ≥50%, 3-3-2), recipes/tracker.md,
     "The 3-3-2 Split" essay, CHAPTER-RESEARCH-MAP. Privacy note: data/ats/ files are private (DATA_CONTRACT). Draft. Never published. -->

Imagine running the entire engine — scanning, scoring, framing, applying — and keeping no record of any of it. You'd have no response rate. No way to know whether Proven-tier applications outperform Unknown-tier ones. No signal that three weeks in, you've quietly drifted back to eight hours of clicking Submit. You'd be flying with no instruments, feeling busy, learning nothing. The decisions would still happen; you just couldn't see whether any of them worked, or correct course when they didn't.

The tracker is the instrument panel. But the most important number on it is not the one most people expect.

---

The counterintuitive heart of this chapter is this: **the target is a skip rate of at least fifty percent.** Of the roles the engine evaluates, you should be deciding not to apply to at least half.

Sit with why that's the goal and not a failure. If you apply to nearly everything the engine surfaces, your filter is too loose — you've recreated spray-and-pray with extra steps, and the targeted two hours from Chapter 2 has swollen back toward eight. A high skip rate means the filter is doing real work: separating the few roles worth a day off your ninety-day clock from the many that aren't. The skip rate is the dial that tells you whether the reallocation principle is actually operating or has quietly collapsed.

The reflex to resist is feeling good about a low skip rate because "I'm applying to a lot of things." That feeling is the volume instinct wearing a tracker. Low skip rate is not productivity. It is the filter failing. If your skip rate is twenty percent and you feel busy and productive, that pairing is the warning sign — busy and unfiltered is exactly the state the whole book exists to end.

So the tracker reads in two directions. Skip rate well under fifty percent means the filter is too permissive, or you're overriding it constantly; you're back to volume. Skip rate close to a hundred percent means either your upstream targeting is bad — no good roles reaching the scorer — or your thresholds are set impossibly high; either way, the funnel needs attention upstream. Skip rate above fifty percent with a steady flow of applies and a rising per-tier response rate: the engine is working as designed.

The skip is not an absence in the log. It is a decision, and a tracker that logs only applications is missing half the data about how well the filter works.

---

The component that makes this legible is the Pipeline Tracker, the engine's fifth component, maintained through the `tracker` recipe and analyzed with the patterns script:

```bash
# Maintain / update the decision log (tracker recipe writes data/ats/applications.md)
npm run ats:scan        # feeds new postings into the pipeline for decisions
# tracker recipe: log each decision (company, role, score, tier, timeline flag, outcome incl. skip)

# Analyze tracker/scan/pipeline data for patterns and the allocation summary
python scripts/ats/analyze-patterns.py
```

For every decision the engine produces — apply or skip — the tracker records the company and role, the composite score and tier, the timeline flag, the recommendation, and the outcome. The output of `analyze-patterns.py` is the daily allocation summary plus your skip rate and per-tier response rates. You read the skip rate first. It is the fastest signal that the method is or isn't operating.

A privacy note that belongs here before anything else: your tracker files — `data/ats/applications.md`, `pipeline.md`, scan history — contain your real targets and real activity. They are private, never committed or shared without a privacy review. This is a hard rule that becomes load-bearing in Chapter 16.

---

Consider a week of actual tracker data: thirty roles evaluated, seventeen skipped, thirteen applied, nine of the thirteen in Proven or Likely tier. Skip rate is seventeen over thirty — fifty-seven percent. Healthy: the filter is doing real work. Early response data shows Proven-tier applications drawing replies at several times the rate of the few Unknown-tier ones, which is evidence that the tiering is predictive rather than decorative. The allocation summary, though, shows apply hours creeping to three and a half per day while networking has dipped — a drift warning that the tracker caught before a full week was gone.

The right response to that picture: keep the thresholds, because the skip rate is healthy, but reclaim an hour from applying back to networking next week. That is the 3-3-2 decision rule operating on real data rather than on principle.

The limit is equally worth stating. Early in the search, per-tier response samples are tiny and noisy — a three-application "trend" is not a trend. And the skip rate is a process metric, not an outcome metric. A healthy skip rate with zero responses still means something is wrong; it just means something upstream is wrong — bad targeting, weak materials — not that the filter itself has failed. The tracker measures discipline and targeting quality. It cannot conjure offers, and it cannot tell the difference between a targeting failure and a sector-wide hiring freeze. The numbers look identical. You read them in the context of a market the tracker can't see.

---

The decision rules for reading your own skip rate are compact enough to state directly:

Below forty percent is too loose. Trust the scorer's skips; stop overriding to apply. Between forty and fifty percent is borderline; watch whether your overrides are justified or habitual. Above fifty percent with steady applies and rising per-tier response rates is healthy — hold. Above roughly eighty-five percent, nearly everything skipped, means the funnel is starved or over-tight; fix upstream targeting in Chapters 6 and 5 or loosen the thresholds slightly.

The allocation summary closes the other loop. It shows how your hours actually split across apply, network, and portfolio — so the tracker doesn't just measure targeting quality, it catches allocation drift before a week has gone sideways. The skip rate tells you the filter is working. The allocation summary tells you that you are. You need both readings to know that the search is actually running the way Chapter 2 designed it.

## LLM Exercises

1. **(Apply) Log a week.** Record a week of decisions in the tracker — every Apply and every Skip with its score, tier, and timeline flag — and generate the daily allocation summary.
2. **(Evaluate) Diagnose your skip rate.** Compute your skip rate. Is it too low, too high, or healthy? Name the one change that moves it toward a working range, using the decision rules above.
3. **(Evaluate) Tier vs. response.** Once you have enough data, compare response rates across tiers. Does Proven outperform Unknown? Say what that implies about trusting the scorer.

## Key terms

- **Pipeline Tracker:** the component that logs every decision — score, tier, timeline flag, outcome, including skips.
- **Skip rate:** the share of evaluated roles you decide not to apply to; target ≥ 50% as evidence the filter works.
- **Daily allocation summary:** the read-out of how your hours split across apply / network / portfolio (3-3-2).
- **Process vs. outcome metric:** skip rate measures discipline and targeting quality; it must be read alongside response rate and market context.

## Run-log prompt

Record the week's skip rate, the per-tier response rates with sample sizes, the allocation summary, and the one adjustment you're making next week.
