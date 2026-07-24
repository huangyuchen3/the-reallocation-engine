# Role Scorer report — 2026-07-24

*Bayesian Role Scorer (Ch.11). Weights: sponsorship 0.35, fit 0.3, role_quality 0 [role_quality weight is **[VERIFY]** — not pinned by the chapter]. Threshold 0.3. Profile requires sponsorship.*

**Summary:** 5 roles → Apply 2 · Consider 1 · Skip 2. **Skip rate 40%** (below the ~50% a healthy run skips; check the inputs).

| Role | Composite | Rec | Why | Audit (term · value · weight · source) |
|---|---|---|---|---|
| DataStax Inc — Software Engineer - Data Platform | 0.478 | **Apply** | composite 0.478 ≥ 0.3, gates healthy | sponsorship 0.9·0.35 [record]; fit 0.72·0.3 [model-judgment]; role_quality 0.787·0 [record] × liveness 1[record]×timeline 0.9[your-input] |
| Coursera Inc — Data Engineer (representative) | 0.467 | **Apply** | composite 0.467 ≥ 0.3, gates healthy | sponsorship 0.9·0.35 [record]; fit 0.68·0.3 [model-judgment]; role_quality 0.725·0 [record] × liveness 1[record]×timeline 0.9[your-input] |
| Deako Inc — Data Engineer II | 0.407 | **Consider** | above threshold (0.407) but one soft spot: sponsorship tier "Likely" | sponsorship 0.65·0.35 [record]; fit 0.75·0.3 [model-judgment]; role_quality 0.725·0 [record] × liveness 1[record]×timeline 0.9[your-input] |
| Example Corp (no H-1B record) — Data Engineer | 0.189 | **Skip** | composite 0.189 < 0.2 — time is better spent elsewhere | sponsorship 0·0.35 [record]; fit 0.7·0.3 [model-judgment]; role_quality 0.787·0 [record] × liveness 1[record]×timeline 0.9[your-input] |
| Coursera Inc — Data Engineer (expired posting) | 0.000 | **Skip** | gated: liveness ≈ 0.000 (a closed gate zeroes the composite regardless of votes) | sponsorship 0.9·0.35 [record]; fit 0.7·0.3 [model-judgment]; role_quality 0.725·0 [record] × liveness 0[record]×timeline 0.9[your-input] |

*Every term traces to its source. If you cannot explain a row term-by-term, distrust the recommendation before your confusion (Ch.11).*
