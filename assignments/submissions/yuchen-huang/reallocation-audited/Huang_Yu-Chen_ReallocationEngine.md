# Huang_Yu-Chen_ReallocationEngine

**Course:** INFO 7375 — Computational Skepticism for AI  
**Assignment:** The Reallocation Engine, Audited — Build a Useful Tool That Doubts Itself  
**Author:** Yu-Chen Huang  
**Date:** 2026-07-24  
**Repo branch:** `assignment/reallocation-audited`  
**Book chapter I anchor to:** Chapter 11, The Bayesian Role Scorer  
(composite = weighted votes × liveness × timeline; liveness and timeline are gates, not extra points)

I already built this mode earlier in the semester. Links:

- Recipe: `recipes/case-de-platform-cogpivot.md`
- Why this domain: `assignments/submissions/yuchen-huang/domain-justification.md`
- My June run with real commands: `assignments/submissions/yuchen-huang/worked-run.md`

How to run everything in this folder: see `README.md`.

---

## 0. What this tool is, in normal words

I am on F-1 OPT. I need sponsorship. I cannot apply to every company.

So the scarce resource here is my **application time** — reading JDs, fixing my resume, clicking submit.

The tool looks at a few companies/roles and tells me **Apply**, **Consider**, or **Skip**. That is the reallocation: move hours to A, do not waste hours on B.

**One-sentence objective:**  
Spend my OPT effort first on companies that have stronger historical H-1B evidence, and only if the posting still looks alive.

**What this objective does NOT optimize:**  
Best salary. Best learning. Best “platform” job for my career. Chance that *this* company will sponsor *me* for *this* posting. Also, cognitive pivot / role quality is in my input file, but the default weight is **0**, so platform vs pipeline does not change the final number yet.

**Uncertainty (honest):**  
A score like 0.478 looks exact, but some inputs are my judgment (`fit`) or my profile (`timeline`). H-1B history is only past company data. Apply is a **suggestion**, not “go submit now.”

**Where I would not trust it:**  
I will not trust it to decide a real application unless I read the titles myself, check liveness, and clear my hard-stop (Section 7).

---

## 1. Working reallocation tool

### 1.1 What actually runs

This is a real command, not a messy notebook. I reuse the repo Ch.11 scorer with my own fixture files from the DE mode:

```bash
npm run score -- \
  assignments/submissions/yuchen-huang/reallocation-audited/fixtures/roles.json -- \
  --profile assignments/submissions/yuchen-huang/reallocation-audited/fixtures/profile-sponsor-required.json \
  --out-dir assignments/submissions/yuchen-huang/reallocation-audited/outputs
```

I also use (from my recipe / June run):

- grep filter on the local H-1B CSV for DE-ish titles
- BLS lookup for SOC **15-1243** cognitive pivot
- `npm run ats:liveness -- <job-url>` before I treat a URL as live

### 1.2 What it recommended (I re-ran this on 2026-07-24)

Same 5 roles I used in June. Output is in `outputs/role-scores.md`:

| Where my effort goes | Company / role | Score | My caveat |
|---|---|---|---|
| **Toward (Apply)** | DataStax — Software Engineer - Data Platform | 0.478 | Strong H-1B record, but titles look SWE-heavy |
| **Toward (Apply)** | Coursera — Data Engineer (representative) | 0.467 | Good sponsor signal; title is representative, not always a live DE URL |
| **Maybe (Consider)** | Deako — Data Engineer II | 0.407 | Only 8 approvals, so I keep it Consider |
| **Away (Skip)** | Example Corp — no H-1B | 0.189 | No sponsorship record |
| **Away (Skip)** | Coursera expired posting | **0.000** | Liveness gate closed |

Skip rate = 2/5 = **40%**. The book often talks about skipping at least half in a healthy run. My sample is a bit low, so I should not feel too confident about the shortlist.

**Uncertainty beyond one number:**

- Apply only means: score ≥ 0.3 and gates open — still needs my hard-stop
- Consider means: ok-ish score, but sponsorship tier is soft (Likely)
- Skip means: low score **or** a closed gate
- For DataStax, even with a high score, I am **not** confident it is a DE-first sponsor (only 1 of 3 titles looks DE-related)
- `fit` is my judgment. I do not pretend it is a measured probability.

### 1.3 One example of the math

DataStax:

`(0.9·0.35 + 0.72·0.3 + 0.787·0) × 1.0 × 0.9 = 0.478 → Apply`

I want people to notice: `role_quality` shows 0.787, but weight is **0**, so it adds nothing.

---

## 2. Data validation & GIGO gate

### 2.1 Assumptions in the data that are not always true

When I first used the H-1B file, it was easy to trust every number. Then I found these problems:

| The data kind of assumes… | But actually… |
|---|---|
| Company H-1B history = DE sponsorship for my title | Titles can be mostly SWE; one “data” word still passes grep |
| SOC 15-1243 pivot = what *this* job does | Same SOC can be platform design or warehouse/pipeline work |
| A URL I saved is still open | Jobs expire; Greenhouse redirects |
| “Data Engineer” in a filter = my platform goal | Job titles are marketing |
| Non-null approvals = they will sponsor me | Past company filings ≠ my future case |

Files I really used:

- `data/80-days-to-stay/data/SEC_DOL_H1b_data_mapped.csv`
- `data/bls/compact/soc_occupation_compact.csv`
- Public Greenhouse URLs for liveness tests

Also important: a lot of companies in the mapped file have empty H-1B fields (my recipe notes about 94.9%). Missing is common. It is not a rare bug.

### 2.2 My quality gate (a human can check this)

A row can go into scoring only if:

1. The H-1B CSV and BLS compact CSV exist and open.
2. If I claim sponsorship > 0, that company row has non-null `Total Approvals`.
3. I write down `top_job_titles_sponsored`, not only “grep matched.”
4. I can say which command / when I got the row.
5. If the decision is about a specific URL, liveness must be `active`, or I set factor = 0 and accept Skip.
6. I do not invent approval counts. Record or label as judgment.

### 2.3 What fails, and what I do

| Failure | What I do |
|---|---|
| Company not in CSV | Stop. No guessing from a website or ChatGPT. |
| Approvals null | Treat like non-sponsor / Skip path in this mode. |
| Grep match but SWE-majority titles (DataStax) | Mark **investigate**, not auto Apply. |
| Expired / uncertain URL | Close liveness gate; score can go to 0; Skip that posting. |
| Pivot used as posting fact without reading JD | Label **inferred**. |

Real rejection from my June run: fake/invalid Coursera job id → `expired` → liveness 0 → Skip at 0.000 even though sponsorship was Proven.

---

## 3. Bias audit (data → output)

### 3.1 How bias shows up

1. **Who is in the data:** Companies that already filed H-1B look “real.” Companies with no filings look like “None,” even if some might sponsor later.
2. **My filter:** Keyword grep likes companies that say “data engineer” or “data platform,” including SWE-heavy lists.
3. **My objective:** Chasing historical sponsorship helps big filers and hurts no-record companies.
4. **Feedback loop:** If I only apply to high scores, I may never learn about smaller true-DE sponsors. Next shortlist gets even more narrow.

Who wins: high-volume H-1B filers with DE-ish title words.  
Who loses: no-record companies, small DE sponsors (like Deako), and real DE roles hidden under SWE title lists if I get lazy.

### 3.2 A simple number on my 5-row run

I define selection rate = share of rows the machine marks Apply (before I override).

| Group | Rows | Apply | Selection rate |
|---|---|---|---|
| High-volume Proven (≥50 approvals) | 2 (DataStax, Coursera live) | 2 | **1.00** |
| Low-volume Likely | 1 (Deako) | 0 | **0.00** (Consider) |
| No H-1B record | 1 | 0 | **0.00** |
| Gate closed (expired URL) | 1 | 0 | **0.00** |

Low-volume Apply rate / high-volume Apply rate = 0 / 1 = **0**.

This sample is tiny, so I am not claiming a big population result. I am saying: on *my* run, Apply sticks to high-volume Proven sponsors. That is a warning about the objective.

### 3.3 Two fairness ideas that fight each other

**Definition A — “any DE title is enough”:**  
If there is at least one DE-related sponsored title, the company can reach Apply after gates.

**Definition B — “most titles should be DE”:**  
Only if sponsored titles are majority DE-related should it reach Apply. SWE-majority should stay Consider or blocked by me.

DataStax:

- Under A, Apply looks kind of fair (there is a Data Platform title).
- Under B, Apply looks unfair (only 1 of 3 titles is DE-ish).

**What I choose for my own time:** Definition **B** as my human gate, even if the scorer still prints Apply under something closer to A.

Cost: I move slower; I read titles more.  
Benefit: I waste less OPT time on SWE-primary companies that only look DE because of one keyword.

**Best place to fix this:** Improve the DE title filter *before* scoring (my recipe TODO), and keep a human stop for SWE-majority. Changing only the Apply threshold does not fix the upstream label problem.

---

## 4. Explainability & why it can still mislead

### 4.1 What explanation I use

I did not add a separate SHAP notebook. The scorer already prints an audit trail, which is the explanation I use:

- each vote: value, weight, contribution, source
- each gate: multiplier, source
- full arithmetic line

DataStax Apply example:

- sponsorship 0.9 · 0.35 → 0.315 [record]
- fit 0.72 · 0.30 → 0.216 [model-judgment]
- role_quality 0.787 · **0** → **0** [record]
- × liveness 1 × timeline 0.9

The math is correct. That is not the whole story.

### 4.2 Named case where the explanation lies by omission

**Case: DataStax Apply, role_quality shown but weight = 0.**

What the audit *shows:* a full list, including role_quality 0.787.  
What I actually need to know: is this platform DE work, or mostly SWE sponsorship history?

They do not match:

- The table makes it feel like platform/quality evidence was part of the decision.
- Contribution is zero. Apply is mostly sponsorship + my fit + timeline.
- The title “Software Engineer - Data Platform” also makes me *feel* the platform story is confirmed, even though BLS pivot never moved the score.

So the explanation is accurate on arithmetic, and misleading on importance.

**Second case from June:** I used a Coursera Greenhouse URL that was `active`, but the live job was **Chief of Staff, Data**, not a Data Engineer posting. I used it as a live-board proxy. Saying `liveness = 1 [record]` is true for that URL, and still easy to misread as “DE posting is live.”

---

## 5. Pearl’s three rungs (causal check)

The dangerous sentence is: “If I move my hours to company B, I get a better outcome.” That is a causal claim. I need to check if my engine really supports it.

### Rung 1 — Observation

From my runs, Apply / higher scores go with:

- more historical H-1B approvals / Proven tier
- open liveness
- higher `fit` numbers I typed

Pattern I observe: companies with strong past filing records get more of my recommended effort.

### Rung 2 — Intervention

If I really reallocate (apply to DataStax/Coursera, skip no-sponsor and ghosts), what changes?

- I can change: where *I* spend hours, and whether I avoid dead URLs.
- I cannot change with this tool: whether the employer will sponsor me, headcount, or if the work is platform vs pipeline.

Is the engine optimizing an interventional quantity? **Mostly no.**  
It ranks observational sponsorship evidence, then multiplies by gates. It does not estimate “what happens if I do(apply to B).”

Confounders that can break the nice correlation:

| Confounder | Why I care |
|---|---|
| Company size / age | Big filers look safe, but maybe slow or SWE-only for new grads |
| Role mix | Past filings may be SWE while I want DE |
| Location | Bay Area filings ≠ the Seattle team hiring now |
| Policy / year | Old approval rates may not match current lottery / budget |
| My level / stack | Past sponsors may not hire someone like me |
| Ghost jobs | Nice brand + dead posting can look good until liveness |

### Rung 3 — One counterfactual

**Case:** Coursera expired / invalid URL from my June run.  
**What happened:** liveness = 0 → score 0 → Skip.  
**Question:** If there was no liveness gate (or if I ignored Skip) and I spent a week on that URL, what would happen?

**My judgment:** I would probably waste that week on a posting that is not open. The gate does not prove I get an interview on a live URL. It only supports: putting effort on a dead URL does not help.

Assumptions:

- the `expired` label was correct for that URL
- “one week of tailoring” is the alternative I care about
- no recruiter email would save that exact dead link

### Honest verdict

A lot of this engine reallocates on **correlation that feels like causation**.

More carefully:

- For “don’t waste time on dead postings”: partly supported (liveness gate).
- For “Apply here → higher chance I get sponsored”: **not established.** That is past sponsorship correlation plus my judgments.
- I keep this tool as a prioritization helper with hard stops, not as an outcome predictor.

---

## 6. Adversarial robustness / where it breaks

### Perturbation 1 — Dead URL (I ran this)

Keep Proven sponsorship and high fit. Set liveness = 0.  
Result: score → **0.000 → Skip**.  
Good: gate works. Bad: if someone skips the liveness command and only looks at sponsorship, they still *feel* Apply.

### Perturbation 2 — Keyword noise (I ran this in June)

DataStax titles are mostly SWE; one “Data Platform” string.  
Result: still in my DE grep list (70 companies); scorer can still say Apply.  
Failure: if I trust the rank and never read titles, I reallocate effort to a SWE-heavy sponsor.

### Perturbation 3 — Changing my fit number

DataStax uses fit 0.72. If I drop fit to about 0.2 (more honest if platform evidence is weak):

- votes ≈ 0.9·0.35 + 0.2·0.3 = 0.375  
- × 0.9 timeline ≈ **0.338** → still near Apply

If fit = 0.1: about **0.311** — still borderline Apply.

So sponsorship weight is strong. Fragility is not only “the score flips too easily.” Sometimes it **fails to flip** when my domain doubt says it should.

### Limits I accept

- Only 5 roles in this sample
- `role_quality` weight 0 hides my platform goal in the math
- Liveness depends on Playwright and the careers site; sites change

---

## 7. Delegation map + hard-stop gate

### 7.1 Who decides what

| Part | Tool does | I do | Override |
|---|---|---|---|
| H-1B CSV numbers | Reads records | Confirm it is the right company | Stop if match is unclear |
| DE keyword filter | Makes a shortlist | Decide if SWE-majority is ok | Downgrade to investigate / Skip |
| BLS pivot | Looks up SOC score | Choose sub-SOC prior if I did not read JD | Label inferred; I can ignore |
| `fit` | Uses the number I typed | I choose the number | Do not let AI invent fit quietly |
| Apply / Consider / Skip | Machine recommendation | Decide if I spend hours | **Hard-stop before apply** |
| Liveness script | active / expired / uncertain | Trust or not | Expired → I do not apply to that URL |
| Timeline | Multiplies | OPT dates in my profile | I attest dates; no guessing |

### 7.2 Hard-stop (I will not skip this)

The tool can recommend. It must not move the resource alone.

In my case, “moving the resource” means: spending application hours, submitting an application, or deciding “I will pursue this.”

Before I tailor a resume or click submit, I choose:

| Response | When | Who |
|---|---|---|
| **Approve** | Titles look DE-relevant, liveness active if there is a URL, timeline still ok, I accept leftover uncertainty | Me |
| **Flag** | SWE-majority titles, proxy URL, Consider tier, or role_quality “looks used” but weight is 0 | Me — read JD / fix inputs / re-score |
| **Block** | No H-1B when I need sponsorship; expired/uncertain liveness; unclear company; invented counts | Me — Skip |

Why this is non-negotiable: if I treat Apply as auto-approved, a correlation list can burn weeks of OPT time. That is how a helpful tool becomes an unattended bad decision.

In practice: this repo does not auto-apply in a browser (good). My recipe gates + this report define the stop. In June I did not apply just because the score said Apply.

---

## Uncertainty in one place

- Table: scores, skip rate, source tags in `outputs/role-scores.md`
- Plain sentence: **“This tool helps me guess where not to waste applications. It does not know if a company will sponsor me.”**
- Distrust line: **I do not trust Apply when titles are SWE-majority, or when role_quality is printed with weight 0 like it mattered.**

---

## AI USE DISCLOSURE

**Tool(s) used:**  
Cursor (Composer) for this audited write-up pass; earlier Claude/Cursor help when I built the June mode (`case-de-platform-cogpivot`), profile/resume attestation, and worked-run notes.

**Portions assisted:**  
Helping me organize the seven assignment sections into one report file; checking that my fixtures still score; reminding me which rubric items I still needed (Pearl rungs, fairness tradeoff language, hard-stop table); cleaning grammar in places.

**How used:**  
I did the core work first over weeks: chose the DE/OPT domain, wrote the recipe, picked DataStax / Coursera / Deako from the H-1B file, set my own `fit` and timeline values, ran `npm run score` and `npm run ats:liveness`, and wrote the June worked-run with real terminal output. For this Canvas assignment I used AI more like a writing/structure assistant on top of that evidence — not as the person who discovered DataStax’s title problem or the liveness break test.

**What I changed / own:**  
Company shortlist and control rows; all `fit` numbers; Deako as Likely (only 8 approvals); the DataStax “investigate, don’t auto-Apply” rule; choosing fairness definition B for my human gate; the causal verdict that sponsorship history ≠ “I will get sponsored”; Approve / Flag / Block rules; and every place I say I still would not trust the tool.

**What the AI could not do:**  
After the scorer printed **Apply (0.478)** for DataStax, the model can easily explain the arithmetic and even sound calm about “gates make it safe.” It could not take responsibility for my OPT clock. I had to look at `top_job_titles_sponsored` myself (Software Engineer, Escalations Engineer, and only one Data Platform string) and decide: **I will not spend application weeks on this Apply until I verify the role is really DE/platform.** That is my visa risk and my career goal, not a sentence the model can be accountable for if those weeks are wasted.

---

## Appendix A — Files in this folder

| File | What it is |
|---|---|
| `README.md` | How to run |
| `frictional-journal.md` | Prediction before + reflection after |
| `fixtures/roles.json` | Input I score |
| `fixtures/profile-sponsor-required.json` | JSON profile for the scorer |
| `outputs/role-scores.md` | Human audit table |
| `outputs/role-scores.json` | Machine output |

## Appendix B

Video goes to Canvas (link or file). Speaking notes stay on my laptop if I want; they are not required in the repo.
