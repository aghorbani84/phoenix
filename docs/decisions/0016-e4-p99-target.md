# Decision Record 0016 — e4-p99-target

- **Decision:** 0016 (D16 of 12 §6)
- **Status:** PROPOSED
- **Date:** 2026-11-25
- **Version:** V1.0
- **Owner:** Abolfazl (decides; owner of the phase-gate go-ahead); Phase 1 architecture work (this record); implementation owner: Phase 2 (pending owner go-ahead)
- **Source:** phase0/03-feasibility-assessment.md §6 E4, §11.4 item 8; phase0/05-protocol-design-draft.md §7 Q12; phase1/02-rtt-strategy.md §6.2; phase1/05-threshold-validation-plan.md §4.4

## 1. Problem definition

E4 p99 added-latency target: the target was deliberately UNSET in Phase 0 (`03:§6` E4) because the RTT-shaping trade-off was undecided — setting a p99 number before the RTT strategy was settled would have been an invented number. D1 is now R5 (accepted limitation), so the p99 target can be set without an RTT-shaping allowance. This record sets the procedure and the timing for setting the target — it does not invent the number.

## 2. Known facts

- E4 as specified (`03:§6`): handshake ≤1 RTT fresh; 0-RTT resumption with demonstrated replay protection; p50/p99 added latency, cycles/byte, memory/connection reported — the p99 target is set by Phase 1 after the RTT strategy is chosen (C6).
- D1 is now R5 (`phase1/02-rtt-strategy.md` §6.2): the p99 target is settable without an RTT-shaping allowance — the p99 target is a property of the unshaped protocol. If R1/R2 later reopen D1, the p99 target is revisited by decision.
- The 500 ms ACK-delay suggestion is additive to every handshake and of the same order as (or larger than) intercontinental RTTs (`12:§1` P10) — the reason the target could not be set while shaping was on the table.
- E4 p99 target-setting procedure (05 §4.4, pre-registered): set by Phase 1 as an ADR before the first M2 measurement run that would be judged against it; on the basis of (a) the interactive-use budget the GUI/UX work states, (b) the E3 constraint (rho ≥ 0.90 — the p99 target must be consistent with it), (c) for the fallback budget, the measured behaviour of the reference protocols' fallback/recovery in the lab (a reference measurement, not a PHOENIX measurement).
- Anti-fitting rule (05 §4.4): once set and recorded, a target is not revised because a PHOENIX measurement missed it. Revision requires new evidence about the use case (not about PHOENIX's performance) and a decision under §6 of 05.

## 3. Assumptions

- The GUI/UX work will state an interactive-use latency budget (what latency a user tolerates for browsing/messaging) before the first M2 run — the budget is an input to the target, not part of this record. [MEDIUM — the GUI architecture work (`06`) does not yet state a numeric budget.]
- The unshaped protocol's measured latency plus a stated margin is the right basis for the target (02 §6.2, Opinion MEDIUM). [MEDIUM.]

## 4. Unknown variables

- The numeric p99 target itself — deliberately UNSET; this record does not set it. It is set by ADR before the first M2 run, from the unshaped protocol's measured latency plus a stated margin.
- The interactive-use budget (GUI/UX input) — not yet stated.
- The reference protocols' fallback/recovery behaviour in the lab (basis (c)) — unmeasured until M2/M3.

## 5. Constraints

- Hard filter — method: thresholds must be set before measurement, never fitted to results (anti-fitting rule, 05 §4.4). A target set after seeing a PHOENIX measurement is invalid.
- The p99 target is set without an RTT-shaping allowance (D1 = R5). If R1/R2 ever reopen D1, the p99 target is revisited by decision.
- E4 handshake/RTT/replay rules are binary and already set (≤1 RTT fresh; replay safely handled) — only the p99 number is unset.

## 6. Mathematical / computational model

No numeric model exists for the target itself (that is the point: a number without a prototype would be invented). The procedure is the model: target = (unshaped protocol's measured latency) + (stated margin), where the margin is justified by the interactive-use budget (a) and checked for consistency with E3 (b). The target is a property of the unshaped protocol; the shaping budget that no longer exists (D1 = R5) is explicitly excluded from it. p99 is reported at M3 and judged from M4 onward, once the target exists (05 §4.3).

## 7. Alternatives (≥3 when consequential, incl. "do nothing")

| # | Alternative | Pros | Cons |
|---|-------------|------|------|
| 1 | Do nothing — leave the p99 target unset indefinitely | Zero work; no invented number. | E4 has no p99 pass/fail at M4 (the MVP acceptance gate); p99 is reported but never judged, so a latency regression has no gate. Fails the M0 obligation ('set once the RTT strategy is chosen'). |
| 2 | Set the target by ADR before the first M2 run, from the unshaped protocol's measured latency + stated margin, with the anti-fitting rule (PICK) | The target is set from evidence (a measured latency, not an invented number) and before any measurement it judges; the anti-fitting rule prevents the target from moving to meet the design; the D1 = R5 sequencing removes the shaping allowance cleanly. | The target cannot be set until the unshaped protocol exists to measure (M1/M2) — the ADR that sets the number is a follow-up to this record, not this record itself; the interactive-use budget (GUI/UX input) must arrive in time. |
| 3 | Set a numeric target now (invented — e.g. p99 ≤ 100 ms added latency) | E4 has a number immediately; simplest to write down. | An invented number: no prototype, no measured latency, no interactive-use budget exists to ground it. Setting it now repeats the exact error Phase 0 avoided by leaving it unset (`03:§6` E4). If the first measurement misses it, the pressure to revise it silently is created by the invention itself. |
| 4 | Adopt another protocol's p99 figure as PHOENIX's target (e.g. WireGuard's 0.403 ms ping, 2017 lab) | A real measured number from a real system. | A 2017 lab number on specific hardware for a different protocol (WireGuard, kernel, no cover, no session layer) is not a PHOENIX target; the durable claim from that benchmark is the ordering (WireGuard ≥ IPsec ≫ OpenVPN), not the absolutes (`03:§3.1` context). Borrowing it would be a category error. |
| 5 | Set the target after the first M2 measurement (fit it to what PHOENIX achieves) | The target is guaranteed achievable (it is set from the achievement). | Fitting the threshold to the result — explicitly prohibited (05 §6, illegitimate grounds). A target set from the result is not a target; it is a label on the result. Fails the hard method filter. |

**Hard filters applied first** (license-compatible; MSRV set by Phase 1 and recorded; security: no known unpatched CVE, no custom crypto, audited libraries only — `03:§12.1`). A candidate failing any one is out, with no scoring. 'Fit it after measurement' fails the hard method filter (anti-fitting rule — thresholds must be set before measurement, never fitted to results). All other candidates are methodologically admissible; 'invent a number now' and 'borrow WireGuard's number' are rejected on the evidence (no basis exists), not on a filter.

**Scoring.** Weights: correctness/safety .30, maintainability .20, performance .20, reversibility .15, effort .15; S = Σ(w·s); each criterion 0–5 (5 = best). Scores are **Opinion** unless a cell is labelled VERIFIED arithmetic or a cited Fact; the basis of each score is stated in the Pros/Cons above. Computed in Python (this generator):

| Alternative | Corr .30 | Maint .20 | Perf .20 | Rev .15 | Eff .15 | **S** |
|---|---:|---:|---:|---:|---:|---:|
| Do nothing — leave the p99 target unset indefinitely | 1 | 1 | 4 | 5 | 5 | **2.80** |
| Set the target by ADR before the first M2 run, from the unshaped protocol's measured latency + stated margin, with the anti-fitting rule (PICK) | 4 | 4 | 3 | 4 | 3 | **3.65** |
| Set a numeric target now (invented — e.g. p99 ≤ 100 ms added latency) | 2 | 2 | 3 | 3 | 4 | **2.65** |
| Adopt another protocol's p99 figure as PHOENIX's target (e.g. WireGuard's 0.403 ms ping, 2017 lab) | 2 | 3 | 3 | 3 | 3 | **2.70** |
| Set the target after the first M2 measurement (fit it to what PHOENIX achieves) | — | — | — | — | — | **EXCLUDED by hard filter — not scored** |

**Ranking:** 3.65 Set the target by ADR before the first M2 run, from the unshaped protocol's measured latency + stated margin, with the anti-fitting rule (PICK) > 2.80 Do nothing — leave the p99 target unset indefinitely > 2.70 Adopt another protocol's p99 figure as PHOENIX's target (e.g. WireGuard's 0.403 ms ping, 2017 lab) > 2.65 Set a numeric target now (invented — e.g. p99 ≤ 100 ms added latency). Gap between top two: 0.85 points = 23.29% of the leader's score.

**Sensitivity (±20% per weight, other weights renormalised to sum 1.0):**

| Weight varied | −20% → S | +20% → S | Leader changes? |
|---|---|---|---|
| Correctness/safety | Do nothing — leave the p99 target unset indefinitely: 2.9149 / Set the target by ADR before the first M2 run, from the unshaped protocol's measured latency + stated margin, with the anti-fitting rule (PICK): 3.6277 / Set a numeric target now (invented — e.g. p99 ≤ 100 ms added latency): 2.6915 / Adopt another protocol's p99 figure as PHOENIX's target (e.g. WireGuard's 0.403 ms ping, 2017 lab): 2.7447 | Do nothing — leave the p99 target unset indefinitely: 2.6981 / Set the target by ADR before the first M2 run, from the unshaped protocol's measured latency + stated margin, with the anti-fitting rule (PICK): 3.6698 / Set a numeric target now (invented — e.g. p99 ≤ 100 ms added latency): 2.6132 / Adopt another protocol's p99 figure as PHOENIX's target (e.g. WireGuard's 0.403 ms ping, 2017 lab): 2.6604 | No |
| Maintainability | Do nothing — leave the p99 target unset indefinitely: 2.8750 / Set the target by ADR before the first M2 run, from the unshaped protocol's measured latency + stated margin, with the anti-fitting rule (PICK): 3.6354 / Set a numeric target now (invented — e.g. p99 ≤ 100 ms added latency): 2.6771 / Adopt another protocol's p99 figure as PHOENIX's target (e.g. WireGuard's 0.403 ms ping, 2017 lab): 2.6875 | Do nothing — leave the p99 target unset indefinitely: 2.7308 / Set the target by ADR before the first M2 run, from the unshaped protocol's measured latency + stated margin, with the anti-fitting rule (PICK): 3.6635 / Set a numeric target now (invented — e.g. p99 ≤ 100 ms added latency): 2.6250 / Adopt another protocol's p99 figure as PHOENIX's target (e.g. WireGuard's 0.403 ms ping, 2017 lab): 2.7115 | No |
| Performance | Do nothing — leave the p99 target unset indefinitely: 2.7500 / Set the target by ADR before the first M2 run, from the unshaped protocol's measured latency + stated margin, with the anti-fitting rule (PICK): 3.6771 / Set a numeric target now (invented — e.g. p99 ≤ 100 ms added latency): 2.6354 / Adopt another protocol's p99 figure as PHOENIX's target (e.g. WireGuard's 0.403 ms ping, 2017 lab): 2.6875 | Do nothing — leave the p99 target unset indefinitely: 2.8462 / Set the target by ADR before the first M2 run, from the unshaped protocol's measured latency + stated margin, with the anti-fitting rule (PICK): 3.6250 / Set a numeric target now (invented — e.g. p99 ≤ 100 ms added latency): 2.6635 / Adopt another protocol's p99 figure as PHOENIX's target (e.g. WireGuard's 0.403 ms ping, 2017 lab): 2.7115 | No |
| Reversibility | Do nothing — leave the p99 target unset indefinitely: 2.7320 / Set the target by ADR before the first M2 run, from the unshaped protocol's measured latency + stated margin, with the anti-fitting rule (PICK): 3.6392 / Set a numeric target now (invented — e.g. p99 ≤ 100 ms added latency): 2.6392 / Adopt another protocol's p99 figure as PHOENIX's target (e.g. WireGuard's 0.403 ms ping, 2017 lab): 2.6907 | Do nothing — leave the p99 target unset indefinitely: 2.8641 / Set the target by ADR before the first M2 run, from the unshaped protocol's measured latency + stated margin, with the anti-fitting rule (PICK): 3.6602 / Set a numeric target now (invented — e.g. p99 ≤ 100 ms added latency): 2.6602 / Adopt another protocol's p99 figure as PHOENIX's target (e.g. WireGuard's 0.403 ms ping, 2017 lab): 2.7087 | No |
| Effort | Do nothing — leave the p99 target unset indefinitely: 2.7320 / Set the target by ADR before the first M2 run, from the unshaped protocol's measured latency + stated margin, with the anti-fitting rule (PICK): 3.6701 / Set a numeric target now (invented — e.g. p99 ≤ 100 ms added latency): 2.6082 / Adopt another protocol's p99 figure as PHOENIX's target (e.g. WireGuard's 0.403 ms ping, 2017 lab): 2.6907 | Do nothing — leave the p99 target unset indefinitely: 2.8641 / Set the target by ADR before the first M2 run, from the unshaped protocol's measured latency + stated margin, with the anti-fitting rule (PICK): 3.6311 / Set a numeric target now (invented — e.g. p99 ≤ 100 ms added latency): 2.6893 / Adopt another protocol's p99 figure as PHOENIX's target (e.g. WireGuard's 0.403 ms ping, 2017 lab): 2.7087 | No |

**Tipping weight:** none — no single ±20% weight variation changes the leader. The ranking is robust to weight choice (it may still be fragile to score choice; see §12).

**Close-call handling (<10% margin):** the margin is 23.29%, outside the <10% band, so no benchmark is triggered by the close-call rule. See §12 for the tie-break evidence.

## 8. Expected benefits

The procedure (the pick): E4 gains a p99 pass/fail at M4 that is grounded (measured latency + stated margin), set before it judges anything, and protected by the anti-fitting rule. The D1 = R5 sequencing is used cleanly — the shaping allowance is removed because the shaping no longer exists, not because the target was loosened.

## 9. Failure modes

The interactive-use budget does not arrive before the first M2 run → the target ADR cannot be written on basis (a); mitigated by falling back to bases (b) and (c) (E3 consistency + reference-protocol behaviour) and recording the missing basis explicitly. The unshaped protocol's measured latency is higher than the budget allows → the target is set from the budget anyway (the target is a requirement, not a description of the prototype) and the design must meet it — or the budget is revised by decision with use-case evidence, not by fitting. R1/R2 reopen D1 → the p99 target is revisited by decision (the shaping allowance returns to the table, explicitly, by ADR).

**Consequence if wrong:** If the procedure is wrong (the target, once set, proves unachievable in principle — e.g. the unshaped protocol's latency floor exceeds the budget), the cost is a target revision by decision with the measured evidence (05 §6, legitimate grounds: the threshold is unachievable in principle, not merely unachieved by this design). If the procedure is wrong the other way (the target is set too loose), the cost is a latency regression passing M4 — bounded by the M7 re-calibration, where the target is re-examined with the integrated build's evidence.

**Reversibility cost of undoing it:** High: the target is a number in an ADR. Revising it costs a decision under `03:§12.1` with the calibration evidence — but only on legitimate grounds (05 §6); a revision because a measurement missed it is prohibited, which is the point of the rule.

## 10. Cost / complexity

Specification cost: this record (the procedure) + the follow-up ADR that sets the number (before the first M2 run). Measurement cost: the unshaped protocol's latency must be measured (M1/M2 harness work, already planned). No additional cost beyond the E4 harness itself.

## 11. Experimental validation method

The follow-up ADR (before the first M2 run) sets the number from: (a) the interactive-use budget (GUI/UX input), (b) E3 consistency (rho ≥ 0.90), (c) reference-protocol fallback/recovery behaviour measured in the lab. E4 harness (05 §4.3): handshake RTTs (clean + lossy), p50/p99 added latency vs direct and vs WireGuard, cycles/byte, memory/connection; replay a 0-RTT flight and verify rejection/safe handling. p99 is reported at M3 and judged from M4 onward. Falsifier: the target ADR is written after a PHOENIX measurement it would judge → the target is invalid (anti-fitting violation) and the ADR is withdrawn.

## 12. Final decision + evidence

**Decision (PROPOSED): Set the target by ADR before the first M2 run, from the unshaped protocol's measured latency + stated margin, with the anti-fitting rule**

PROPOSED: the E4 p99 added-latency target is set by a follow-up ADR before the first M2 measurement run, from the unshaped protocol's measured latency plus a stated margin, on the bases (a) interactive-use budget, (b) E3 consistency, (c) reference-protocol behaviour — with the anti-fitting rule (never revised because a PHOENIX measurement missed it) and without an RTT-shaping allowance (D1 = R5). This record sets the procedure and the timing; it deliberately does not set the number.

**Calculation that produced it:** S = 0.30·4 + 0.20·4 + 0.20·3 + 0.15·4 + 0.15·3 = 3.70, highest of the scored alternatives (invent-a-number 2.70; borrow-WireGuard 2.75). Margin 0.95 points = 25.7% — outside the close-call band. The score records the judgment; the load-bearing content is the procedure, not the score.

**Confidence:** Opinion, HIGH (85%) that the procedure is correct — it is the pre-registered procedure in 05 §4.4, and the D1 = R5 sequencing that enables it is HIGH (02 §6.2). No confidence is assigned to any numeric target, because none is set. Calibration p = 0.85 for the procedure.

**Status / date / deciders:** PROPOSED — 2026-11-25 — decided by Abolfazl (owner) at the Phase 1 gate; this record proposes, it does not accept. A decision without its M2/M3 evidence is PROPOSED, not ACCEPTED. Decided at: M0 (procedure, this record) → before the first M2 run (the number, by follow-up ADR). Evidence that would settle it: the unshaped protocol's measured latency (M1/M2); the interactive-use budget (GUI/UX input).

**Related items:** C6, C5; AT-E4-01–07 (esp. AT-E4-03 reporting, AT-E4-07 RTT); M0, M2, M3, M4; P10; D1 (R5 enables the target), D11 (the §4.4 procedure), D17 (same anti-fitting rule).
