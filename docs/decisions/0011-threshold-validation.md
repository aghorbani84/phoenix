# Decision Record 0011 — threshold-validation

- **Decision:** 0011 (D11 of 12 §6)
- **Status:** PROPOSED
- **Date:** 2026-11-25
- **Version:** V1.0
- **Owner:** Abolfazl (decides; owner of the phase-gate go-ahead); Phase 1 architecture work (this record); implementation owner: Phase 2 (pending owner go-ahead)
- **Source:** phase1/05-threshold-validation-plan.md (§8 decision record); phase0/03-feasibility-assessment.md §1.3 Condition 3, §6, §12.1, §12.4; phase0/12-priority-roadmap.md §1 P4, §6 D11

## 1. Problem definition

Threshold-validation plan: how do the E1–E7 thresholds (all Opinion, initial targets, no measured basis — U11) get calibrated against reference protocols before PHOENIX is judged against them? A pass against an uncalibrated threshold is not evidence (`09:§4.2` R-D). The plan must exist in Phase 1 (Condition 3) and execute at M3, with AT-E1-04 running first.

## 2. Known facts

- All E1–E7 thresholds are Opinion with no measured basis (U11): Adv ≤ 0.05 / AUC ≤ 0.55; ≥1,000 probes/type; rho ≥ 0.90/1.0; KS p > 0.05; ±1% derived-vs-measured overhead.
- Power analysis (mathematical truth given the stated premises, computed 2026-11-25 in 05 §3): at n = 1,000/class the Adv ≤ 0.05 threshold is only ~2.2σ above zero (SE ≈ 0.022 at worst-case p = 0.5) — marginally resolvable; a measured Adv = 0.04 at n = 1,000 has a 95% CI of roughly [−0.004, 0.084], consistent with both passing and failing. Primary target n = 5,000/class (5σ; CI ±0.020).
- KS critical values (mathematical truth): at n = 1,000/type D_crit ≈ 0.061; at n = 10,000/type D_crit ≈ 0.019. The E2 KS test at n = 1,000 can only detect distribution differences larger than ~6%.
- AUC is better powered than Adv at the same n (Hanley–McNeil: AUC 0.55 vs chance 0.50 is ~3.9σ at n = 1,000); the two legs must be reported separately, never collapsed into one number.
- E5 denominator ambiguity (05 §3.5): Reading A (±1% of the overhead, ±352 B on the fixed profile) is likely unachievable as stated; Reading B (±1% of total wire bytes, ≈ ±14,856 B) is achievable and meaningful. The fixed profile is 1,000 × 1,400 B + 100 × 64 B = 1,406,400 application bytes; derived overhead ≈ 35,200 B.

## 3. Assumptions

- The published detection rates (Wu 2023, Xue 2024) are extractable from the papers (pre-calibration steps C-1/C-2 in 05 §4.1 — if they are not, the separation fallback applies: RB-1 flagged at ≥10× the cover's flag rate). [MEDIUM.]
- Probe cost in the lab permits n = 10,000/type for the timing KS (to be confirmed at M3). [MEDIUM.]

## 4. Unknown variables

- Reference-implementation behaviour in the lab; run-to-run variance on the lab machine (E3 CI width); pcap noise floor (E5 denominator choice).
- Xue 2024 per-protocol TPR/FPR beyond the slides (U13 — paper PDF not read); PTPerf per-protocol numbers (U12 — not extracted); Aparecium primary source (U7). All three are pre-calibration steps C-2/C-3/C-4.
- Whether the harness can reproduce the known break — that is itself the AT-E1-04 result, not assumed here.

## 5. Constraints

- Hard filter — method: AT-E1-04 runs FIRST, before any E1 result is reportable (`09:§3.2` S2; `12:§1` P4). No exception. Any E1 result produced before AT-E1-04 passes is discarded, not 'kept as preliminary'.
- A threshold revision is a decision under `03:§12.1`, not a silent edit (`09:§4.3`; `12:§1` P4). Illegitimate grounds (never a revision): a PHOENIX measurement missed the threshold; a deadline is near; the threshold is 'too strict' without evidence about the instrument or the use case.
- No result without corpus version + harness SHA + CI (`03:§6` common requirements).

## 6. Mathematical / computational model

Difference-of-proportions SE = √(0.5/n) worst case; KS D_crit = 1.36·√(2/n); bootstrap CI (10,000 resamples) for rho; binary KATs/audit. Pass rule (proposed, Opinion HIGH, to be confirmed at M3): PASS iff the upper bound of the two-sided 95% CI on Adv is ≤ 0.05 (at n = 5,000 this requires measured Adv ≤ 0.030; at n = 1,000 it would require ≤ 0.006 — the quantitative proof that n = 1,000 cannot serve as the gate). E3 proposed rule: PASS iff point estimate rho ≥ threshold AND lower 95% CI bound ≥ threshold − 0.05. E5: denominator pinned at M3 (recommended Reading B).

## 7. Alternatives (≥3 when consequential, incl. "do nothing")

| # | Alternative | Pros | Cons |
|---|-------------|------|------|
| 1 | Do nothing — judge PHOENIX directly against the Opinion thresholds, no calibration | Zero work; fastest to a verdict. | A pass against an uncalibrated threshold is not evidence; a threshold too loose passes a design a real censor would detect (false confidence — the dangerous direction); a threshold too tight wastes work and pressures silent revision. Fails the hard method filter (Condition 3). |
| 2 | This plan: AT-E1-04 first, CI-based pass rules, n = 5,000/class, per-threshold calibration (PICK) | The harness proves it can detect a known-bad design and not flag a known-good control before it judges PHOENIX; every pass/fail is resolvable at 5σ (Adv) with known CI; threshold revisions are decisions with evidence. | Costs a calibration run at M3 (reference builds + AT-E1-04 + controls + per-threshold procedures) and a re-calibration at M7; if AT-E1-04 fails, no E1 result is reportable until the harness is fixed. |
| 3 | Keep n = 1,000/class and report point estimates | Cheapest; matches the Phase 0 design numbers literally. | §3.1 proves it cannot resolve the threshold (2.2σ; a measured Adv = 0.04 is consistent with both passing and failing). Reporting a point-estimate pass at n = 1,000 is dishonest. Rejected on the arithmetic. |
| 4 | Calibrate against a single reference only (no negative controls) | Cheaper than the full plan; still tests the harness against a known break. | A harness that flags one bad design may still be miscalibrated on others; without negative controls (direct connection, genuine cover server) a distinguisher that flags everything passes calibration. Rejected — both checks are required. |

**Hard filters applied first** (license-compatible; MSRV set by Phase 1 and recorded; security: no known unpatched CVE, no custom crypto, audited libraries only — `03:§12.1`). A candidate failing any one is out, with no scoring. 'Do nothing' and 'n = 1,000 point estimates' fail the hard method filter (Condition 3; the §3.1 arithmetic proves n = 1,000 cannot gate Adv). All other candidates are methodologically admissible; the single-reference option is rejected on completeness, not on a filter.

**Scoring.** Weights: correctness/safety .30, maintainability .20, performance .20, reversibility .15, effort .15; S = Σ(w·s); each criterion 0–5 (5 = best). Scores are **Opinion** unless a cell is labelled VERIFIED arithmetic or a cited Fact; the basis of each score is stated in the Pros/Cons above. Computed in Python (this generator):

| Alternative | Corr .30 | Maint .20 | Perf .20 | Rev .15 | Eff .15 | **S** |
|---|---:|---:|---:|---:|---:|---:|
| Do nothing — judge PHOENIX directly against the Opinion thresholds, no calibration | — | — | — | — | — | **EXCLUDED by hard filter — not scored** |
| This plan: AT-E1-04 first, CI-based pass rules, n = 5,000/class, per-threshold calibration (PICK) | 4 | 4 | 3 | 4 | 4 | **3.80** |
| Keep n = 1,000/class and report point estimates | — | — | — | — | — | **EXCLUDED by hard filter — not scored** |
| Calibrate against a single reference only (no negative controls) | 3 | 3 | 3 | 3 | 3 | **3.00** |

**Ranking:** 3.80 This plan: AT-E1-04 first, CI-based pass rules, n = 5,000/class, per-threshold calibration (PICK) > 3.00 Calibrate against a single reference only (no negative controls). Gap between top two: 0.80 points = 21.05% of the leader's score.

**Sensitivity (±20% per weight, other weights renormalised to sum 1.0):**

| Weight varied | −20% → S | +20% → S | Leader changes? |
|---|---|---|---|
| Correctness/safety | This plan: AT-E1-04 first, CI-based pass rules, n = 5,000/class, per-threshold calibration (PICK): 3.7872 / Calibrate against a single reference only (no negative controls): 3.0000 | This plan: AT-E1-04 first, CI-based pass rules, n = 5,000/class, per-threshold calibration (PICK): 3.8113 / Calibrate against a single reference only (no negative controls): 3.0000 | No |
| Maintainability | This plan: AT-E1-04 first, CI-based pass rules, n = 5,000/class, per-threshold calibration (PICK): 3.7917 / Calibrate against a single reference only (no negative controls): 3.0000 | This plan: AT-E1-04 first, CI-based pass rules, n = 5,000/class, per-threshold calibration (PICK): 3.8077 / Calibrate against a single reference only (no negative controls): 3.0000 | No |
| Performance | This plan: AT-E1-04 first, CI-based pass rules, n = 5,000/class, per-threshold calibration (PICK): 3.8333 / Calibrate against a single reference only (no negative controls): 3.0000 | This plan: AT-E1-04 first, CI-based pass rules, n = 5,000/class, per-threshold calibration (PICK): 3.7692 / Calibrate against a single reference only (no negative controls): 3.0000 | No |
| Reversibility | This plan: AT-E1-04 first, CI-based pass rules, n = 5,000/class, per-threshold calibration (PICK): 3.7938 / Calibrate against a single reference only (no negative controls): 3.0000 | This plan: AT-E1-04 first, CI-based pass rules, n = 5,000/class, per-threshold calibration (PICK): 3.8058 / Calibrate against a single reference only (no negative controls): 3.0000 | No |
| Effort | This plan: AT-E1-04 first, CI-based pass rules, n = 5,000/class, per-threshold calibration (PICK): 3.7938 / Calibrate against a single reference only (no negative controls): 3.0000 | This plan: AT-E1-04 first, CI-based pass rules, n = 5,000/class, per-threshold calibration (PICK): 3.8058 / Calibrate against a single reference only (no negative controls): 3.0000 | No |

**Tipping weight:** none — no single ±20% weight variation changes the leader. The ranking is robust to weight choice (it may still be fragile to score choice; see §12).

**Close-call handling (<10% margin):** the margin is 21.05%, outside the <10% band, so no benchmark is triggered by the close-call rule. See §12 for the tie-break evidence.

## 8. Expected benefits

Every future pass/fail is resolvable at 5σ (Adv) with known CI; the harness's validity is demonstrated, not assumed; threshold revisions are decisions with evidence; the E5 denominator and E3 pass rule are pinned before the first run, not after it (anti-fitting).

## 9. Failure modes

AT-E1-04 fails (harness invalid — 05 §5): all E1 results produced up to that point are discarded; the failure loop applies to the harness (bisect → root cause → change one variable → re-measure); after 3 independent repair attempts the calibration is blocked and escalated to the owner with the exact unblocking step. Corpus falls short of 10,000/class → downgraded resolution (companion document §5.4), results downgraded, not silently accepted. E3 variance makes the pass rule unachievable at 10 runs → more runs first (§3.4), revise the rule only by decision.

**Consequence if wrong:** If the plan is wrong (the CI-based pass rules are too strict, or Reading B for E5 is the wrong denominator), the cost is a threshold revision by decision at M3 with the calibration evidence attached — which the plan itself provides for (05 §6). If the plan is wrong in the other direction (the harness passes calibration but is still miscalibrated on a distinguisher class the references do not cover), the cost is false confidence in E1 — bounded by the re-calibration triggers (corpus version change, harness change, new published operating point) and the M7 re-calibration.

**Reversibility cost of undoing it:** High: the plan is a procedure + a calibration log. Revising a pass rule costs an ADR with the calibration evidence; re-running calibration costs one M3/M7 run. Nothing built depends on a threshold value except the pass/fail labels, which are re-labelled (old threshold, old value) and not re-scored silently.

## 10. Cost / complexity

One calibration run at M3 (reference builds RB-1/RB-2/RB-3 + AT-E1-04 + controls + per-threshold procedures); re-calibration at M7; the calibration log as standing bookkeeping. No PHOENIX code is needed for calibration itself. Pre-calibration paper work in Phase 1 (C-1…C-6).

## 11. Experimental validation method

AT-E1-04 first at M3 (05 §4.2): Wu 2023 exemption rules vs RB-1 (Shadowsocks-2022 without countermeasures), measured flag rate vs the pre-registered expectation (C-1); negative controls (all three E1 distinguishers vs CG-1 direct connection, FPR reported — AT-E1-05); Xue 2024 reproducer check vs RB-2 (TPR within the pre-registered interval, C-2). Falsifier (harness-side, not design-side): if the harness cannot reproduce the known break, the harness is invalid, not the design — no E1 result is reportable until AT-E1-04 passes (`09:§3.1`). Per-threshold procedures in 05 §4.3; re-calibration at M7 and on the §7 triggers.

## 12. Final decision + evidence

**Decision (PROPOSED): This plan: AT-E1-04 first, CI-based pass rules, n = 5,000/class, per-threshold calibration**

PROPOSED: adopt the threshold-validation plan in `phase1/05-threshold-validation-plan.md`; AT-E1-04 runs first at M3, before any E1 result is reportable; pass means the upper 95% CI bound is within threshold at n = 5,000/class; E5 denominator pinned at M3 (recommended Reading B, ±1% of total wire bytes); E3 pass rule proposed (point estimate ≥ threshold AND lower 95% CI bound ≥ threshold − 0.05), confirmed or revised at M3 with the measured variance.

**Calculation that produced it:** S = 0.30·4 + 0.20·4 + 0.20·3 + 0.15·4 + 0.15·3 = 3.70, highest of the scored alternatives (single-reference 3.00; n = 1,000 point estimates 2.85). Margin 0.70 points = 18.9% — outside the close-call band. The calculation records the judgment; the load-bearing evidence is the §3 power analysis (mathematical truth given the premises), which proves n = 1,000 cannot gate Adv.

**Confidence:** Opinion, HIGH (85%) that the power analysis is correct as arithmetic; MEDIUM (70%) that the proposed pass rules (CI-based Adv, threshold−0.05 rho allowance, Reading B for E5) are the right ones — they are confirmed or revised at M3 with the measured evidence, per §4.3. Calibration p = 0.85 for the arithmetic; p = 0.70 for the proposed rules.

**Status / date / deciders:** PROPOSED — 2026-11-25 — decided by Abolfazl (owner) at the Phase 1 gate; this record proposes, it does not accept. A decision without its M2/M3 evidence is PROPOSED, not ACCEPTED. Decided at: M0 (plan) → M3 (calibration runs) → M7 (re-calibration). Evidence that would settle it: AT-E1-04 at M3; per-threshold calibration results.

**Related items:** U11, U12, U13, R2, C3, C14; AT-E1-04 (first), AT-E1-01–03/05/06, AT-E2-01–05, AT-E3-01–03, AT-E4-06/07, AT-E5-01/02, AT-E6-01–06, AT-E7-02–06; M0, M3, M7; P4; D19 (corpus the plan calibrates against), D16, D17 (the two UNSET targets the plan's §4.4 procedure sets).
