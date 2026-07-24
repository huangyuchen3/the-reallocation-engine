# Reallocation Engine, Audited — Yu-Chen Huang

This folder is my INFO 7375 submission for **The Reallocation Engine, Audited**.

**Branch:** `assignment/reallocation-audited`  
**Link:** https://github.com/huangyuchen3/the-reallocation-engine/tree/assignment/reallocation-audited

## What this is

I am an F-1 OPT student looking for Data Engineer / Data Architect roles (SOC 15-1243).  
This tool helps me reallocate scarce **application time**: Apply / Consider / Skip.

It uses the book’s Chapter 11 role scorer. Liveness and timeline are gates, not bonus points.

I built the mode earlier this semester. This folder is the **audit layer** for the Canvas assignment (report + journal + fixtures). Related files:

- `../domain-justification.md` — why this domain
- `../worked-run.md` — June run with real commands
- `../../../recipes/case-de-platform-cogpivot.md` — recipe

## Files here

| File | What it is |
|---|---|
| `Huang_Yu-Chen_ReallocationEngine.md` | Validation report + AI disclosure |
| `frictional-journal.md` | Prediction before + reflection after |
| `fixtures/` | `roles.json` + profile JSON I score |
| `outputs/` | Last scorer run |

(`video-script.md` is only my private speaking notes for recording. I do not need to put it on GitHub. Canvas gets the video link/file.)

## How to run

From the **repo root**:

```bash
# optional check
npm run verify

# the working tool
npm run score -- \
  assignments/submissions/yuchen-huang/reallocation-audited/fixtures/roles.json -- \
  --profile assignments/submissions/yuchen-huang/reallocation-audited/fixtures/profile-sponsor-required.json \
  --out-dir assignments/submissions/yuchen-huang/reallocation-audited/outputs
```

You should see something like: **Apply 2 · Consider 1 · Skip 2 (skip 40%)**, plus `outputs/role-scores.md`.

### Extra commands I used in the full mode

```bash
grep -iE "data engineer|data architect|database architect|big data|data platform|data engineering" \
  data/80-days-to-stay/data/SEC_DOL_H1b_data_mapped.csv | wc -l

grep "15-1243" data/bls/compact/soc_occupation_compact.csv

# needs Playwright once: npx playwright install chromium
npm run ats:liveness -- "https://job-boards.greenhouse.io/coursera/jobs/5999885004"
```

## Hard-stop

Scorer **Apply** is only a suggestion. Before I spend OPT time or submit, I still Approve / Flag / Block (see report Section 7).

## Canvas checklist

- [ ] Repo link to this branch
- [ ] Report file (or PDF with the same name)
- [ ] Video ~6 min (unlisted link or file on Canvas)
- [ ] Frictional journal in the repo
- [ ] AI Use Disclosure in the report (specific “what the AI could not do”)
