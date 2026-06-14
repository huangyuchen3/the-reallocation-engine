# Chapter 3 — The Verified-Data Contract
*The one rule that separates a filter you can trust from a guess wearing the costume of one.*

There is a particular kind of wrong answer that is worse than no answer at all, and it is this: an answer that sounds right. Not obviously wrong. Not hedged or uncertain. Fluent, confident, plausible — and untrue.

I want to be precise about what I mean, because the distinction matters more here than almost anywhere else I can think of. A language model asked whether a particular Cambridge biotech sponsors work visas will not say "I don't know." It will say something like: "as a mid-size biotech competing for specialized talent, this company likely sponsors visas for research-focused roles." Every word of that sentence is defensible. The reasoning is plausible. The conclusion is probably true for a lot of companies in that description. And it is not a finding. It is the most coherent-sounding sentence the model could assemble from patterns in its training data — which is a completely different operation from looking at a record.

The Department of Labor publishes every Labor Condition Application a U.S. employer files before hiring a foreign worker. The USCIS publishes H-1B approval and denial counts by employer. If you run a script against those records and ask the same question, you get a different kind of answer: this company filed fifteen LCAs in three years and had an 85% H-1B approval rate. Or you get zero filings, ever. The number is not the most plausible thing you could say — it is what actually happened, as far as the public record goes.

The whole architecture of this book is built on that distinction. One answer is fluent. The other is *true*. The discipline required to build on the second, and only the second, is what I mean by the verified-data contract.

<!-- → [INFOGRAPHIC: Two-column side-by-side: left column labeled "Model's answer" shows the fluent prose sentence with no source; right column labeled "Verified answer" shows the LCA count, approval rate, and year, traced to DOL and USCIS records. Caption: "Same question. One answer is plausible. One is checkable."] -->

## Why the contract is a single rule, not a set of guidelines

I've seen job-search systems — and empirical systems of all kinds — that try to manage the fluency problem with caveats: add a disclaimer, note that the model might be wrong, invite the user to verify. This doesn't work, and it doesn't work for a specific reason. A caveat lives outside the output. It is a warning label on a product, not a change to what the product contains. If the number inside the output was produced by a language model filling in a plausible-sounding value, the caveat is decoration on a fabrication.

The contract is different because it is a *prior constraint on what gets to enter the system at all*, not an advisory attached to what came out. The rule is stated once, in the shared configuration that every recipe in this system reads before doing anything else: **Run the script and read the audit before you prompt. Never invent a count, a rate, or a coverage number.**

That is it. One rule. The simplicity is intentional. Complex rules are negotiated; a one-rule contract is harder to quietly break. The model's role in this system is to help you *read* data — to frame a finding, identify what's interesting about a number, suggest what to look at next. It is not permitted to be the source of a number. Sources of truth in this system are specific things: script outputs, audit reports, logged runs. Not the model's best guess at what the number probably is.

## What the system actually measures, and where it lives

The verified-data contract is not a philosophy statement. It is enforced by the architecture. There are three subsystems that produce the numbers this book is built on, and each of them writes to auditable records you can read and question.

<!-- → [DIAGRAM: Three boxes labeled scripts/sec/ (funding), scripts/ats/ (postings and liveness), scripts/bls/ (role quality), each with an arrow pointing to a corresponding *-audit.md output, all feeding into RUN_LOG.md. Caption: "The three pipelines and where their output lives — every number traces back through this graph."] -->

`scripts/sec/` pulls from SEC filings to measure company funding. When a company raises a round or files material financials, that event is in the public record. The script finds it; the audit reports what was found and what was dropped. `scripts/ats/` queries job-board data to measure whether a company is actively posting and whether particular openings are live. `scripts/bls/` draws on Bureau of Labor Statistics data to assess role quality — compensation, demand trajectory, geographic concentration. Each subsystem writes an audit: a record of what the pipeline did on a given run, how many rows it processed, what coverage it achieved, what it couldn't match.

`DATA_CONTRACT.md` defines what data exists and where it lives — including which files are private (your own application records, credentials, anything that identifies you personally) and how those are handled. `RUN_LOG.md` is the system's memory: every run leaves a trace, so a decision you make today on the strength of a number can be reconstructed and questioned tomorrow. This is the same practice good empirical work has always demanded — state your method, show your coverage, let a claim be checked.

## The tool prefers what it already trusts

There is a mechanism underneath the contract I haven't named yet, and it is the part people miss when they picture a verification tool. They imagine it reaching out to the network every time — hitting the Department of Labor server, re-downloading the disclosure file, re-counting from scratch. That is not what happens, and the difference is the whole reason the contract is cheap to keep.

Each of these tools checks for **local verified data first**. Once the DOL disclosure data has been downloaded and checked, it lives in a local store. When a script runs, it asks a prior question before it asks the network anything: *do I already have verified data for this, and is it still fresh?* If the answer is yes, the script reads the local copy and never touches the network. The audit says so plainly — *served from cache* — so you can see that this run rested on data already verified, not on a fresh pull nobody checked.

Only when the local answer is no — the data is missing, or older than the freshness window the contract defines — does the tool go and fetch. And fetching is not the same as trusting. The moment new data arrives it is **verified on arrival**: the schema is what was expected, the row counts are sane, the source is the one it claims to be. Only then is it written to the local store, and only then does a line go into the run log recording that a fetch happened and what it pulled. The network is the fallback, not the default.

This is not mainly an optimization for speed, though it is faster. It is what makes a run **reproducible**. A number you cited last Tuesday can be regenerated today from the same local verified data, because the tool did not silently re-pull a changed file underneath you. When the data genuinely changes — a new quarter of filings drops — the fetch is logged, visible, and dated, so the change is a recorded event instead of a mystery. Verified-once-then-cached is how "never invent a number" stays affordable: you are not re-earning trust on every run, you are reusing trust you already audited.

## The smallest honest thing you can do right now

Before you understand every component, before the sponsorship pipeline and the ATS scraper and the funding detector are fully built, there is one thing you can do that establishes the contract in practice rather than just in principle. Run a verification:

```bash
npm run ats:verify
```

This calls `scripts/ats/verify-pipeline.mjs`, which checks the tracker and scan data for internal consistency and prints what it found. The output is the audit. I want you to run this not because you need the result yet, but because there is a specific feeling I want you to have: the difference between being told the data is fine and *seeing the check pass*. One of those is someone's word. The other is evidence. The contract is the decision to require the second.

<!-- → [TABLE: Three columns — Pipeline, What it verifies, What the audit reports. Rows: ats:verify / tracker-scan consistency / row counts, coverage, drop reasons; sec:verify / filing-record completeness / companies matched, date range, gaps; bls:verify / role-quality data freshness / series IDs, last update, missing occupations. Caption: "The three verification commands and what each audit tells you."] -->

## The seam where fluency sneaks back in

Here is the error I want to warn you about specifically, because it is not obvious and it catches almost everyone, including me when I'm moving fast. You run the script. The script returns a real number — fifteen filings, 85% approval rate, sourced to DOL records. Now you hand that number to a language model and ask it to interpret the finding. The model says: "fifteen filings over three years is strong for a company of this size in this sector."

Did the model count anything? No. It estimated. It applied a pattern from its training data about what "strong" looks like for biotechs of similar size — a pattern it cannot actually show you, because it does not know what the pattern is, only that the output sounds reasonable. The moment an unsourced judgment re-enters through interpretation, the contract has been quietly broken. You ran the script, which was right. You then let the model paper over a gap in the reading, which was wrong.

The discipline required is to watch the seam between the data and the reading of the data. Data claim: fifteen filings, 85% approval rate, sourced to DOL/USCIS. Model judgment: this is strong for a company of this type. Both are present in that paragraph. Only the first can be defended against scrutiny. The second is permitted — it is genuinely useful — but it has to be *labeled as judgment*, not dressed as a fact derived from counting.

For any sentence in a system output, one question settles it: could this sentence have been produced by counting records? If yes, it must trace to a script output or an audit report. If it cannot be traced, it is not allowed to stand. If no — if it is a reading, a framing, a suggestion — it is model judgment, and it is allowed, but it must be visible as such.

<!-- → [INFOGRAPHIC: Decision tree — "Could this sentence have been produced by counting records?" Yes branch leads to "Data claim: must trace to script output or audit." No branch leads to "Model judgment: allowed, but must be labeled." Both branches end at "Is the label visible in the output?" Caption: "The one-question test for every sentence in a mixed output."] -->

## What the data can't tell you

The contract guarantees that counts and rates are real. It does not guarantee that the right things were measured. This is a distinction the contract cannot collapse for you, and I want to be honest about where it leaves you.

The data knows a company filed fifteen LCAs. It does not know that all fifteen were for one senior scientist role that was filled and will not open again. It knows an approval rate; it does not know that the company was acquired last month and the sponsorship policy changed last week. A name-matching failure — "Google LLC," "Google Inc," "Alphabet" — can make a prolific sponsor look like a non-filer. A company can sponsor and stay below the detection threshold if they filed in a window your data doesn't cover.

The contract stops you from building on fiction. It does not give you omniscience. What it gives you is a floor: you know the numbers you have are real, and you know where the gaps are, because the audit reported them instead of hiding them. A 94%-accurate system can still harm someone systematically if no one asks what it failed to measure. The contract makes you the person who asks that question — not the model, not the system, you. The gaps it reports are your honest starting debt: claims you currently cannot source, labeled as such, waiting for better data.

<!-- → [TABLE: Two columns — What the contract guarantees / What the contract cannot guarantee. Rows include: counts and rates are real / that the right things were measured; gaps are labeled not hidden / that coverage is complete; decisions trace to auditable records / that the records captured the full picture; model judgments are labeled / that those judgments are correct. Caption: "The floor the contract provides and the ceiling it doesn't claim to reach."] -->

## The shape of everything that follows

Before any of those tools, two chapters finish the method this one began. Chapter 4 shows that every tool here is a *recipe* with two customers — the AI that runs it and the human who maintains it — and that you therefore write each one twice. Chapter 5 takes the floor this chapter laid down (the numbers are real) and asks the harder question the contract cannot answer on its own: are they the *right* numbers, measured the right way?

Then the building starts. Every chapter from there forward builds one of the three sources of truth or shows you how to read what they produce. Chapter 6 builds the funding detector — SEC filings, round size, recency, the money that forces companies to hire. Chapter 7 builds the sponsorship pipeline. Chapter 8 builds the posting liveness check. Each of them opens by reading the shared contract, each of them writes to an audit, each of them contributes a line to the run log.

The prime directive is already set: run the script and read the audit before you prompt. Never invent a count, a rate, or a coverage number. From here, the work is building the scripts worth running.

The one question I haven't fully answered yet: of all the companies in the world, which ones just got the money that forces them to hire? That's where we go next — and the answer, it turns out, is sitting in a federal filing database, waiting to be counted.

---

## Exercises

**Warm-up**

1. *(Recall, easy)* State the prime directive in your own words — not the exact phrasing, but the principle it enforces. What two-step sequence does it require before any model prompt, and what does each step accomplish?
   *Tests whether you can articulate the contract without reciting it verbatim.*

2. *(Identify, easy)* Name the three script subsystems in this chapter and the source of truth each one draws from. What does each pipeline write as its output?
   *Tests whether you've mapped the architecture to its actual data sources.*

3. *(Recall, easy)* What is `RUN_LOG.md` for, and why does the system require it rather than just trusting that you remember what you ran?
   *Tests the reasoning behind the audit trail, not just its existence.*

**Application**

4. *(Apply, moderate)* Run `npm run ats:verify`. Read the output. Write a `RUN_LOG.md` entry covering: what you ran, what the audit reported, and one thing the output told you that you would otherwise have assumed without evidence.
   *Tests the transition from knowing the contract to executing it.*

5. *(Analyze, moderate)* Take this sentence: "This well-funded Series B startup probably sponsors visas and would be a strong culture fit for someone with your background." Break it into its data claims and its model judgments. Label each. For each data claim, name the source it would need to trace to. For each model judgment, note whether it is labeled as judgment in the original sentence.
   *Tests the data-claim vs. model-judgment distinction on a realistic example.*

6. *(Apply, moderate)* Name one company you believe sponsors H-1B visas that might not appear in the public LCA/USCIS data — and explain why (recent acquisition, name-matching failure, filing timing, role type). Write one sentence describing how you would label that gap in an output rather than guessing past it.
   *Tests coverage-gap reasoning: what to do when the contract can't give you the answer.*

**Synthesis**

7. *(Synthesize, harder)* A colleague proposes adding a fourth data source to the system: LinkedIn profile counts per company as a proxy for growth. Walk through the contract requirements this would need to satisfy before the system could treat those counts as verified data rather than model-estimated data. What would the script, the audit, and the run-log entry each need to contain?
   *Tests whether you can apply the contract's architecture to a novel source.*

8. *(Synthesize, harder)* Consider the seam problem described in the chapter — verified numbers re-entering interpretation as unsourced estimates. Design a simple output-formatting convention (a label, a section header, a field name — your choice) that would make the data-claim / model-judgment boundary visible to someone reading a system output who hasn't read this chapter. Justify your design choice.
   *Tests whether you can operationalize the distinction, not just state it.*

**Challenge**

9. *(Evaluate, open-ended)* The chapter states that a 94%-accurate system can still harm someone systematically if no one asks what it failed to measure. Construct one realistic failure scenario in the job-search context — a specific type of company, candidate, or role — where the verified-data contract would pass all three of its checks (real counts, labeled gaps, audited runs) and still produce a systematically misleading result. What would a fifth check look like that catches this failure?
   *Tests whether you can find the limits of the contract itself, not just apply it correctly.*
