# Chapter 4 — Two Customers: Writing a Recipe for the AI and the Human

*A document that tries to serve everyone serves no one — the problem is older than software, and recipes are not exempt from it.*

There is a small confusion built into the word "recipe" that I want to name before it causes trouble. A recipe lives in a markdown file. Markdown is text you read. So naturally, when someone writes a recipe, they write it for a reader — they explain what it does, they describe its purpose, they use complete sentences and transitions. The file is readable and pleasant and almost entirely useless to the agent that has to execute it tonight.

I want to be precise about what I mean by "tonight." The F-1 clock is running. You have somewhere between sixty and ninety days to line up a job offer with a company willing to sponsor, file the paperwork, and get it processed. Every recipe in this engine either runs on verified data or it doesn't. And whether it runs on verified data is decided not by what you intended when you wrote it, but by what happens when an agent reads it at eleven PM and starts executing. The document that should have said "call `npm run ats:scan` first, before anything else" instead said "the recipe is designed to use the scan results" — and the agent, unable to locate the output it needed, quietly substituted something plausible. The recipe ran. Nothing in the output announced the substitution. The data was invented.

That failure has a cause, and the cause is a design error. The recipe was written for one customer and executed by another.

## The two customers

Every recipe has two customers with completely different needs.

The first customer is the AI executing it. It needs the recipe to be **terse and imperative**. It needs to know, in the fewest words possible, what to read first, what commands to run in what order, what output files to inspect, what to log, and under what conditions to stop. It does not need context or rationale. Rationale is processing budget that could be spent on execution. The AI doesn't care that the scan recipe exists to enforce the verified-data contract — it cares that the first line says "Read `recipes/_shared.md`" and the second line names the next file. The executable recipe is optimized for an agent to follow exactly, tonight.

The second customer is the human maintaining it. This is future-you, reading in March, who has forgotten what the recipe does, has no memory of why certain design choices were made, and needs to understand the recipe well enough to fix it when it breaks or extend it when your search changes. The maintainer needs to know what the recipe does, what it depends on, how to run it, what it produces when it works, and what goes wrong when it doesn't. The maintainer does not need an imperative step-list dressed as prose — that is confusing to read and easy to skip. The human artifact is a card: purpose, dependencies, commands, outputs, failure modes. Optimized for comprehension rather than execution.

The problem is that these two needs are in tension. The imperative recipe is stripped of explanation so the agent doesn't get distracted. The human card is heavy on explanation so the maintainer understands the reasoning. A document that tries to do both splits the difference: too much prose to be clean for the agent, too imperative to make sense to a reader who has forgotten the context. A recipe written to serve both customers simultaneously serves neither.

So you write it twice.

<!-- → [DIAGRAM: Two boxes side by side, labeled "AI Artifact (recipe)" and "Human Artifact (card)". AI box lists: terse, imperative, read-first order, commands verbatim, stop conditions. Human box lists: purpose statement, dependencies, how-to-run, what-it-produces, failure modes. Both boxes share a footer labeled "Verified-Data Contract (_shared.md)" with arrows pointing into both. Caption: "Same recipe, two documents. The contract is the one thing both artifacts must honor."] -->

## What each artifact contains

The AI artifact — the recipe an agent follows — has a predictable structure. It opens with what to read first, before anything else. This is not optional and it is not decorative: loading `recipes/_shared.md` first is what makes the runtime honest by construction. The shared contract states the prime directive, identifies the sources of truth, and defines the rules for when a number is verified and when it must be labeled as judgment. An agent that loads the shared contract before running the first command operates inside a defined constraint. An agent that skips it operates from its own priors.

After the read-first order, the recipe lists the commands — actual, verbatim, runnable commands, not descriptions of what commands could be run. Then it names what to inspect in the output and what counts as evidence of a successful run: not "the output looks reasonable" but "row count is nonzero, provider hits are listed by platform, no errors in stderr." Then it states the stop conditions: what halt execution. Then it names the log entry to write. That's the recipe. It should be readable at a glance without decoding.

The human artifact is structured differently. It opens with a purpose statement that tells the maintainer what the recipe is *for* in one or two sentences. Then it lists dependencies: which files the recipe reads, which scripts it calls, which other recipes it assumes have run first. Then it gives the commands — the same commands as the recipe, but here they're annotated: what each one does, what the output is, what to notice in the result. Then it describes what the recipe produces when everything works: which files are written, what an audit looks like, what a log entry should contain. Finally, and most importantly, it lists failure modes: specific, named ways the recipe can break. Not "something might go wrong" but "if `data/ats/portals.yml` doesn't exist, the scan will silently use the example config and produce output against the example companies, not your companies."

That last section is the one that gets skipped when someone writes a recipe fast. And it is the one you need most at eleven PM on a Thursday when the output looks wrong and you don't know why.

<!-- → [TABLE: Three columns — Section, AI artifact, Human artifact. Rows: Opening / read-first list / purpose statement; Core content / imperative commands with no commentary / annotated commands with output descriptions; Evidence / stop conditions and output checks / what success looks like in full; Logging / log entry template / what the log should contain and why; Failure / not present / named failure modes with specific causes. Caption: "The structural difference between the two artifacts — the same commands appear in both, but their context is inverted."] -->

## Drift is a failure mode, not just inconvenience

There is a failure mode I have not named yet, and it is more damaging than either the missing rationale or the over-explained recipe. It is drift: the recipe changes, and the human doc doesn't.

Drift happens because updates feel minor. You change one command — you add a flag, rename an output file, change the order of two steps because the second one now depends on output from the first. You update the recipe because the recipe is what the agent runs, and the agent fails if the recipe is wrong. The human doc also needs updating, but you're in a hurry, and it's just documentation, and you'll do it later. Later doesn't come, because it never does when you're racing a clock.

Six weeks later, you read the human doc to understand why the scan recipe works the way it does. The human doc describes a workflow that no longer exists. It names a file the recipe no longer produces. It lists a command that now fails without a flag the recipe added. You can no longer trust the documentation, which means you now have to reverse-engineer the recipe to understand your own recipe, which costs more time than writing the human doc would have.

Drift is its own failure mode — not a consequence of forgetting to maintain the docs, but a structural property of having two artifacts with no enforcement binding them. The only enforcement is discipline: when you update one, you update both in the same commit, and the human doc's failure-modes section explicitly lists "human doc not updated when recipe changes" as one of the named failure modes. The system cannot automate this. The discipline is yours.

## Both artifacts honor the verified-data contract

The shared contract from Chapter 3 is the one thing both artifacts must honor, and each honors it differently.

The AI artifact honors it procedurally: `recipes/_shared.md` is the first item in the read-first list. The agent loads it, and the contract's prime directive is active for the entire run. The rules are live in context: use collected data and tested scripts first; never invent counts, rates, or coverage numbers; if a result comes from LLM judgment, label it as such.

The human artifact honors it structurally: the dependencies section lists which scripts the recipe calls and which audits it reads, making the data provenance visible to anyone reading the card. The failure modes section includes specific entries for contract violations — what happens if the script isn't called, what the output looks like when a model fills in a gap versus when a script ran. The maintainer who reads the human artifact should finish it knowing exactly which parts of the verified-data contract this recipe relies on and what breaks if that reliance is violated.

There is an additional fact here worth stating plainly. The F-1/OPT reader running this engine is both customers at different times. You author the recipe, sitting at your laptop with context about what the recipe is supposed to do, what scripts you built, what data you collected, what can go wrong. You are the human customer. Tonight, an agent runs the recipe you wrote. The agent is the AI customer. In March, you — or someone who is nearly a stranger to this codebase — reads the human doc to understand why something is behaving strangely. You are the human customer again, but with much less context than you had when you wrote it.

The reader who has forgotten everything is not a hypothetical. It is you in three months. The human artifact is a letter you are writing to that person. Write it like it is.

## The scan recipe, shown both ways

The best way to understand the two-artifact structure is to see it for one recipe. The `scan` recipe is the right choice: it is the first active recipe in the engine's chain, it calls real scripts, and it exists in the repository you are using, so the commands are not examples I've constructed for illustration.

Here is the recipe an AI would follow:

---

**scan — AI recipe**

Read first:
- `recipes/_shared.md`
- `scripts/ats/README.md`
- `data/ats/portals.example.yml`
- `data/ats/portals.yml` (if it exists)
- `data/ats/scan-history.tsv` (if it exists)
- `recipes/RUN_LOG.md`

Run:
```bash
cd scripts/ats
python3 detect-ats.py "Company Name" --output ../../data/ats/ats_detection_sample.csv
cd ../..
npm run ats:scan
npm run ats:liveness -- --file data/ats/job-urls.txt
npm run ats:verify
```

Inspect output: count rows; count provider hits by platform; record empty results separately from errors; note where output was written.

Deduplicate: prefer exact URL; then company plus normalized role title; record suspected duplicates, do not delete uncertain entries.

Stop conditions: if `data/ats/portals.yml` does not exist, copy `data/ats/portals.example.yml` and edit before proceeding. Do not run the provider scanner against the example config.

Log template:
```markdown
## YYYY-MM-DD -- ATS scan
- **Recipe:** scan
- **Input:** data/...
- **Command:** `...`
- **Output:** data/...
- **Worked:** ...
- **Did not work:** ...
- **Evidence:** row counts, provider counts, sample URLs
- **Next:** ...
```

Prompting rule: use LLM judgment only after script output has been inspected. The LLM may summarize patterns, explain likely failure causes, or propose the next test. It must not invent ATS hits, live job counts, or sponsorship claims.

---

Now here is the human card a maintainer reads:

---

**scan — human card**

**Purpose.** Detect which applicant-tracking system (ATS) a company uses and pull current job postings from configured portals. This is a data-collection step, not an analysis step. Its output feeds the pipeline recipe.

**What it can verify:** whether a company exposes Greenhouse, Lever, or Ashby jobs; whether a portal scan produced new URLs; whether a posting URL appears in scan history; whether a posting is probably live after a liveness check.

**What it cannot verify on its own:** sponsorship likelihood; role quality; whether a company is a good target. Those require SEC/H-1B and BLS/SOC data, handled by later recipes.

**Dependencies:**
- `recipes/_shared.md` — the verified-data contract; must be loaded first
- `scripts/ats/detect-ats.py` — detects ATS by company name or CSV input
- `scripts/ats/scan.mjs` (via `npm run ats:scan`) — runs configured portal scan
- `scripts/ats/` — liveness check and verify commands
- `data/ats/portals.yml` — your working portal config (not the example)
- `data/ats/scan-history.tsv` — deduplication history from prior runs

**How to run:**
```bash
# Detect ATS for specific companies:
cd scripts/ats
python3 detect-ats.py "Company A" "Company B" --output ../../data/ats/ats_detection_sample.csv

# Run full portal scan (requires portals.yml to be configured):
cd ../..
npm run ats:scan

# Check liveness of specific URLs:
npm run ats:liveness -- --file data/ats/job-urls.txt

# Verify pipeline consistency:
npm run ats:verify
```

**What it produces:** detection output CSV; scan output with provider hits by platform; liveness signals per URL; a verification audit. A log entry in `recipes/RUN_LOG.md` documenting what ran, what it found, and what failed.

**How it fails:**
1. `data/ats/portals.yml` missing — the scan will use the example config and return results for example companies, not your companies. The output will look plausible and be wrong. Check that `portals.yml` exists and is not identical to `portals.example.yml` before trusting any scan result.
2. ATS detection hits vs. live postings conflated — "ATS detected" means the provider is present. "Live posting" requires a liveness check. They are different facts. A recipe output that reports ATS hits as if they were open jobs has skipped the liveness step.
3. Recipe updated, human doc not updated — if the commands in the recipe diverge from the commands in this card, one of them is wrong. Check the recipe before troubleshooting.
4. LLM fills in missing data — if the scan produces no results and the recipe output contains confident sponsorship claims or ATS findings, the prompting rule was violated. No script output means no verified data. The output must say what it couldn't find, not invent a replacement.

---

The commands are identical between the two artifacts. What differs is the frame: the recipe assumes the reader will execute immediately; the human card assumes the reader is trying to understand. The same content, arranged for two different questions — "what do I run?" versus "what is this and how does it break?"

<!-- → [INFOGRAPHIC: Two annotated documents side by side — left labeled "AI Recipe (scan.md)" with callout arrows pointing to: read-first list, verbatim commands, stop condition, log template; right labeled "Human Card (scan — maintainer view)" with callout arrows pointing to: purpose statement, dependency list, annotated commands, failure modes numbered 1–4. Caption: "The recipe is optimized for execution. The card is optimized for comprehension. Neither does the other's job."] -->

## What writing them twice forces you to think about

There is an unexpected benefit to writing the recipe twice that I didn't anticipate when I designed this architecture. The discipline of writing the human card — especially the failure-modes section — forces you to think adversarially about your own recipe. When you sit down to write "how does this fail," you discover that you have not thought carefully about what happens when `portals.yml` is missing, or when the liveness check returns zero results, or when the output CSV is empty because none of the companies matched. Those are the moments you realize the recipe has a gap: it doesn't say what to do in those cases, which means the agent running it tonight will improvise. Improvisation in a verified-data system looks a lot like invention, which is what the contract prohibits.

Writing the failure modes is how you find the missing stop conditions. The human card is not just documentation — it is a test of the recipe.

## The bridge to Chapter 5

Writing a recipe well — two artifacts, shared contract loaded first, failure modes named, recipe and card maintained together — does not make the data the recipe reads trustworthy. The scan recipe can be structurally correct in every way this chapter describes and still return stale detection results, or miss a company because of a name-matching failure, or report a liveness signal on a posting that closed yesterday. The recipe follows the contract. The contract ensures the numbers aren't invented. Neither of those guarantees the numbers are the right numbers, measured the right way.

That question — are these the right numbers? — is the subject of Chapter 5. The verified-data contract set a floor: what you have is real. Chapter 5 asks whether real is enough.

---

## Exercises

**Warm-up**

1. *(Recall, easy)* Name the two customers a recipe must serve and describe in one sentence what each one needs from the recipe document. Why does a document that tries to serve both simultaneously tend to serve neither?
   *Tests whether you can articulate the two-customer problem before applying it.*

2. *(Recall, easy)* What is the first item in the `scan` recipe's read-first list, and why does the recipe specify it first rather than letting the agent decide when to load it?
   *Tests whether you understand the shared contract as a prior constraint on execution, not a reference document.*

3. *(Identify, easy)* List three things the `scan` recipe explicitly states it cannot verify on its own. For each one, name which other part of the engine would need to provide that information.
   *Tests whether you've understood the scan recipe's scope before extending it.*

**Application**

4. *(Apply, moderate)* Take any recipe in the `recipes/` directory other than `scan.md`. Write its human card from scratch: purpose statement, dependencies, how to run, what it produces, and at least two named failure modes. Compare your card against the recipe file — what did the original file contain that you missed, and what did you name that the original file didn't?
   *Tests the transition from understanding the two-artifact structure to producing the human artifact for a real recipe.*

5. *(Analyze, moderate)* Run `npm run ats:scan` and then `npm run ats:verify`. Read both outputs. Write the `RUN_LOG.md` entry according to the scan recipe's log template. Then write one sentence identifying whether any part of the output required LLM judgment to interpret, and if so, how that judgment is labeled.
   *Tests the run-inspect-record loop applied to the scan recipe specifically, with attention to the data/judgment boundary.*

6. *(Analyze, moderate)* The chapter describes drift as a failure mode: the recipe changes and the human doc doesn't. Describe a realistic scenario in which drift would cause an incorrect run tonight. What would the incorrect run look like, and how would you detect that it was wrong? What single change to your workflow would prevent the scenario?
   *Tests adversarial reasoning about drift as a structural problem, not just a maintenance oversight.*

**Synthesis**

7. *(Synthesize, harder)* Design the human card for a hypothetical new recipe — call it `refresh` — that re-downloads and re-verifies SEC Form D data for companies already in the pipeline. You don't have the recipe in front of you. Write the dependency list, the commands (drawing only from what `recipes/_shared.md` lists as real commands), and at least three failure modes that are specific to re-downloading data that may have already been verified. Explain your reasoning for each failure mode.
   *Tests whether you can construct the human artifact from first principles using the shared contract as your ground.*

8. *(Synthesize, harder)* The chapter argues that writing the failure-modes section of the human card is a test of the recipe. Pick one failure mode from the `scan` recipe's human card above. Trace backwards from that failure mode to a gap in the recipe — a condition the recipe doesn't handle, a step it doesn't specify, a stop condition it doesn't include. Write the missing recipe element. Then explain why the failure mode, not the recipe, was easier to see first.
   *Tests whether you can use the human artifact as a diagnostic tool for improving the AI artifact.*

**Challenge**

9. *(Evaluate, open-ended)* The two-artifact design requires discipline to maintain because there is no automatic enforcement binding the recipe and the human card. Propose a lightweight enforcement mechanism — it can be a naming convention, a commit hook, a section in the log template, or anything else — that would make drift detectable without requiring a human to compare the two artifacts manually on every update. Specify what the mechanism checks, what signal it produces when drift is detected, and what it cannot catch. Then evaluate its tradeoff: how much friction does it add per update, and how much drift does it prevent?
   *Tests whether you can design an enforcement solution that takes the two-customer constraint seriously, including its limits.*
