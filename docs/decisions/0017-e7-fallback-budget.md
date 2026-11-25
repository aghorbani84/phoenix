# Decision Record 0017 — e7-fallback-budget

- **Decision:** 0017 (D17 of 12 §6)
- **Status:** PROPOSED
- **Date:** 2026-11-25
- **Version:** V1.0
- **Owner:** Abolfazl (decides; owner of the phase-gate go-ahead); Phase 1 architecture work (this record); implementation owner: Phase 2 (pending owner go-ahead)
- **Source:** phase0/03-feasibility-assessment.md §6 E7; phase0/05-protocol-design-draft.md §7 Q11; phase1/05-threshold-validation-plan.md §4.3–§4.4

## 1. Problem definition

E7 fallback time budget: no number was set in Phase 0 — a number without a prototype would be invented (`03:§6` E7). The budget must be set by ADR before the first M2 run that would be judged against it, with the same anti-fitting rule as D16. This record sets the procedure and the timing — it does not invent the number.

## 2. Known facts

- E7 as specified (`03:§6`): fallback automatic; rotation Adv ≤ 0.05 across the boundary; whitelist-only no-identity-leak (pass/fail). Fallback time is reported at M3 and judged from M4 onward, once the budget exists (05 §4.3).
- Fallback engages automatically and is user-visible — never silent (`04:§2.3.8`; `05:§6.2`). B-first everywhere; A opportunistic standby in Iran, probed in the background and admitted only on measured health, never on assumption (`05:§2.4.4`).
- The fallback time budget is a Phase 1 decision (AT-E7-04 records the measurement against the budget once it exists — `12:§1` P8).
- No fallback latency measurement exists: 'I cannot verify a fallback latency — no measurement exists' (`04:§4.8`; `05:§7` Q11). An unmeasured fallback latency means the 'session-preserving' claim is a design intention, not a property.
- E7 budget-setting procedure (05 §4.4, pre-registered): set by Phase 1 as an ADR before the first M2 measurement run; on the basis of (a) the interactive-use budget, (b) the E3 constraint, (c) the measured behaviour of the reference protocols' fallback/recovery in the lab (a reference measurement, not a PHOENIX measurement — the budget is set from what the references do, then PHOENIX is judged against it). Anti-fitting rule: once set, not revised because a PHOENIX measurement missed it.

## 3. Assumptions

- The reference protocols' fallback/recovery behaviour can be measured in the lab (basis (c)) — e.g. how long a reference proxy takes to recover from a blocked endpoint. [MEDIUM — no reference measurement exists yet.]
- The session layer's fallback (freeze the dying transport, open/resume the other, re-bind with a resumption token) can be timed as a single interval from 'transport fails' to 'traffic flows on the new transport'. [MEDIUM — the interval's endpoints must be defined in D2/Phase 2.]

## 4. Unknown variables

- The numeric fallback time budget itself — deliberately UNSET; set by follow-up ADR before the first M2 run.
- The fallback latency of any implementation (PHOENIX or reference) — unmeasured.
- The session-layer retransmission mechanism for in-flight data on the dying transport (D21) — the budget must cover retransmission, and D21's mechanism determines how much of the budget retransmission consumes.

## 5. Constraints

- Hard filter — method: thresholds must be set before measurement, never fitted to results (anti-fitting rule, 05 §4.4). A budget set after seeing a PHOENIX fallback measurement is invalid.
- Fallback is automatic and user-visible, never silent; the session must move between transports without re-authentication and without leaking a linkable identifier across the switch (E7, `03:§6`).
- The budget must be consistent with the E3 constraint and with the interactive-use budget — a fallback that takes longer than a user will wait is not a fallback.

## 6. Mathematical / computational model

No numeric model exists for the budget itself (that is the point). The procedure is the model: budget = f(interactive-use budget, E3 consistency, reference-protocol fallback behaviour measured in the lab). The interval is defined as: from the moment the session layer declares the transport failed to the moment application traffic flows on the new transport (including retransmission of in-flight data — D21). The budget is a requirement on the session layer, not a description of it. Fallback time is reported at M3 (AT-E7-04) and judged from M4 onward.

## 7. Alternatives (≥3 when consequential, incl. "do nothing")

| # | Alternative | Pros | Cons |
|---|-------------|------|------|
| 1 | Do nothing — leave the fallback budget unset indefinitely | Zero work; no invented number. | E7 has no time pass/fail at M4; fallback is judged only on 'engages automatically and carries traffic', so a 30-second fallback passes the same gate as a 300 ms one. Fails the M0 obligation ('M0 (set) → M4 (measured against it, AT-E7-04)'). |
| 2 | Set the budget by ADR before the first M2 run, from the reference protocols' measured fallback behaviour + interactive-use budget, with the anti-fitting rule (PICK) | The budget is set from evidence (reference behaviour, not PHOENIX's own achievement) and before any measurement it judges; the anti-fitting rule prevents the budget from moving to meet the design; the interval definition (D21's retransmission included) makes the budget testable. | The budget cannot be set until the reference behaviour is measured (M2/M3 lab work) — the ADR that sets the number is a follow-up to this record; the interactive-use budget (GUI/UX input) must arrive in time. |
| 3 | Set a numeric budget now (invented — e.g. fallback within 1 second) | E7 has a number immediately. | An invented number: no prototype, no fallback measurement, no reference behaviour exists to ground it. Repeats the exact error Phase 0 avoided (`03:§6` E7). |
| 4 | Set the budget after the first M2 measurement (fit it to what PHOENIX achieves) | The budget is guaranteed achievable. | Fitting the threshold to the result — explicitly prohibited (05 §6). Fails the hard method filter. |
| 5 | No time budget at all — judge fallback only on 'engages and carries traffic' | Simplest; no number to argue about. | A fallback that takes 30 seconds is a session failure for interactive use, but passes the gate. The 'session-preserving' claim becomes unfalsifiable on time, which is the dimension users feel. Rejected — the budget is the falsifier for the session-preserving claim. |

**Hard filters applied first** (license-compatible; MSRV set by Phase 1 and recorded; security: no known unpatched CVE, no custom crypto, audited libraries only — `03:§12.1`). A candidate failing any one is out, with no scoring. 'Fit it after measurement' fails the hard method filter (anti-fitting rule). All other candidates are methodologically admissible; 'invent a number now' is rejected on the evidence (no basis exists), not on a filter.

**Scoring.** Weights: correctness/safety .30, maintainability .20, performance .20, reversibility .15, effort .15; S = Σ(w·s); each criterion 0–5 (5 = best). Scores are **Opinion** unless a cell is labelled VERIFIED arithmetic or a cited Fact; the basis of each score is stated in the Pros/Cons above. Computed in Python (this generator):

| Alternative | Corr .30 | Maint .20 | Perf .20 | Rev .15 | Eff .15 | **S** |
|---|---:|---:|---:|---:|---:|---:|
| Do nothing — leave the fallback budget unset indefinitely | 1 | 1 | 4 | 5 | 5 | **2.80** |
| Set the budget by ADR before the first M2 run, from the reference protocols' measured fallback behaviour + interactive-use budget, with the anti-fitting rule (PICK) | 4 | 4 | 3 | 4 | 3 | **3.65** |
| Set a numeric budget now (invented — e.g. fallback within 1 second) | 2 | 2 | 3 | 3 | 4 | **2.65** |
| Set the budget after the first M2 measurement (fit it to what PHOENIX achieves) | — | — | — | — | — | **EXCLUDED by hard filter — not scored** |
| No time budget at all — judge fallback only on 'engages and carries traffic' | 2 | 3 | 4 | 3 | 4 | **3.05** |

**Ranking:** 3.65 Set the budget by ADR before the first M2 run, from the reference protocols' measured fallback behaviour + interactive-use budget, with the anti-fitting rule (PICK) > 3.05 No time budget at all — judge fallback only on 'engages and carries traffic' > 2.80 Do nothing — leave the fallback budget unset indefinitely > 2.65 Set a numeric budget now (invented — e.g. fallback within 1 second). Gap between top two: 0.60 points = 16.44% of the leader's score.

**Sensitivity (±20% per weight, other weights renormalised to sum 1.0):**

| Weight varied | −20% → S | +20% → S | Leader changes? |
|---|---|---|---|
| Correctness/safety | Do nothing — leave the fallback budget unset indefinitely: 2.9149 / Set the budget by ADR before the first M2 run, from the reference protocols' measured fallback behaviour + interactive-use budget, with the anti-fitting rule (PICK): 3.6277 / Set a numeric budget now (invented — e.g. fallback within 1 second): 2.6915 / No time budget at all — judge fallback only on 'engages and carries traffic': 3.1170 | Do nothing — leave the fallback budget unset indefinitely: 2.6981 / Set the budget by ADR before the first M2 run, from the reference protocols' measured fallback behaviour + interactive-use budget, with the anti-fitting rule (PICK): 3.6698 / Set a numeric budget now (invented — e.g. fallback within 1 second): 2.6132 / No time budget at all — judge fallback only on 'engages and carries traffic': 2.9906 | No |
| Maintainability | Do nothing — leave the fallback budget unset indefinitely: 2.8750 / Set the budget by ADR before the first M2 run, from the reference protocols' measured fallback behaviour + interactive-use budget, with the anti-fitting rule (PICK): 3.6354 / Set a numeric budget now (invented — e.g. fallback within 1 second): 2.6771 / No time budget at all — judge fallback only on 'engages and carries traffic': 3.0521 | Do nothing — leave the fallback budget unset indefinitely: 2.7308 / Set the budget by ADR before the first M2 run, from the reference protocols' measured fallback behaviour + interactive-use budget, with the anti-fitting rule (PICK): 3.6635 / Set a numeric budget now (invented — e.g. fallback within 1 second): 2.6250 / No time budget at all — judge fallback only on 'engages and carries traffic': 3.0481 | No |
| Performance | Do nothing — leave the fallback budget unset indefinitely: 2.7500 / Set the budget by ADR before the first M2 run, from the reference protocols' measured fallback behaviour + interactive-use budget, with the anti-fitting rule (PICK): 3.6771 / Set a numeric budget now (invented — e.g. fallback within 1 second): 2.6354 / No time budget at all — judge fallback only on 'engages and carries traffic': 3.0104 | Do nothing — leave the fallback budget unset indefinitely: 2.8462 / Set the budget by ADR before the first M2 run, from the reference protocols' measured fallback behaviour + interactive-use budget, with the anti-fitting rule (PICK): 3.6250 / Set a numeric budget now (invented — e.g. fallback within 1 second): 2.6635 / No time budget at all — judge fallback only on 'engages and carries traffic': 3.0865 | No |
| Reversibility | Do nothing — leave the fallback budget unset indefinitely: 2.7320 / Set the budget by ADR before the first M2 run, from the reference protocols' measured fallback behaviour + interactive-use budget, with the anti-fitting rule (PICK): 3.6392 / Set a numeric budget now (invented — e.g. fallback within 1 second): 2.6392 / No time budget at all — judge fallback only on 'engages and carries traffic': 3.0515 | Do nothing — leave the fallback budget unset indefinitely: 2.8641 / Set the budget by ADR before the first M2 run, from the reference protocols' measured fallback behaviour + interactive-use budget, with the anti-fitting rule (PICK): 3.6602 / Set a numeric budget now (invented — e.g. fallback within 1 second): 2.6602 / No time budget at all — judge fallback only on 'engages and carries traffic': 3.0485 | No |
| Effort | Do nothing — leave the fallback budget unset indefinitely: 2.7320 / Set the budget by ADR before the first M2 run, from the reference protocols' measured fallback behaviour + interactive-use budget, with the anti-fitting rule (PICK): 3.6701 / Set a numeric budget now (invented — e.g. fallback within 1 second): 2.6082 / No time budget at all — judge fallback only on 'engages and carries traffic': 3.0206 | Do nothing — leave the fallback budget unset indefinitely: 2.8641 / Set the budget by ADR before the first M2 run, from the reference protocols' measured fallback behaviour + interactive-use budget, with the anti-fitting rule (PICK): 3.6311 / Set a numeric budget now (invented — e.g. fallback within 1 second): 2.6893 / No time budget at all — judge fallback only on 'engages and carries traffic': 3.0777 | No |

**Tipping weight:** none — no single ±20% weight variation changes the leader. The ranking is robust to weight choice (it may still be fragile to score choice; see §12).

**Close-call handling (<10% margin):** the margin is 16.44%, outside the <10% band, so no benchmark is triggered by the close-call rule. See §12 for the tie-break evidence.

## 8. Expected benefits

The procedure (the pick): E7 gains a time pass/fail at M4 that is grounded (reference behaviour + interactive-use budget), set before it judges anything, and protected by the anti-fitting rule. The interval definition (including D21 retransmission) makes the budget testable instead of vague.

## 9. Failure modes

The reference behaviour is not measured before the first M2 run → the budget ADR cannot be written on basis (c); mitigated by falling back to bases (a) and (b) and recording the missing basis explicitly. The session layer's fallback exceeds the budget → the design changes (D21's retransmission mechanism, the health-scoring function), not the budget. D21's mechanism consumes most of the budget → the budget forces the D21 choice (retransmit vs drop-and-retry) to be made on time evidence, which is the point of setting the budget before M2.

**Consequence if wrong:** If the procedure is wrong (the budget, once set, proves unachievable in principle — e.g. the reference protocols themselves cannot meet it), the cost is a budget revision by decision with the measured evidence (05 §6, legitimate grounds). If the procedure is wrong the other way (the budget is set too loose), the cost is a slow fallback passing M4 — bounded by the M7 re-calibration and by the user-visible fallback signalling (a slow fallback is visible to the user, not hidden).

**Reversibility cost of undoing it:** High: the budget is a number in an ADR. Revising it costs a decision under `03:§12.1` with the evidence — but only on legitimate grounds (05 §6); a revision because a measurement missed it is prohibited.

## 10. Cost / complexity

Specification cost: this record (the procedure) + the follow-up ADR that sets the number (before the first M2 run) + the interval definition (D2/Phase 2). Measurement cost: reference-protocol fallback behaviour in the lab (M2/M3 harness work, already planned) + AT-E7-02/03/04 runs. No additional cost beyond the E7 harness itself.

## 11. Experimental validation method

AT-E7-02 at M2 (block UDP → TCP fallback engages automatically and carries traffic); AT-E7-03 (block primary SNI → rotation engages and carries traffic); AT-E7-04 (fallback time measured against the budget, once it exists); AT-E7-05 (Adv ≤ 0.05 across the rotation boundary, n = 5,000/class); AT-E7-06 (whitelist-only: no identity leak on reconnection). The follow-up ADR (before the first M2 run) sets the number from bases (a)/(b)/(c). Falsifier: the budget ADR is written after a PHOENIX fallback measurement it would judge → the budget is invalid (anti-fitting violation) and the ADR is withdrawn.

## 12. Final decision + evidence

**Decision (PROPOSED): Set the budget by ADR before the first M2 run, from the reference protocols' measured fallback behaviour + interactive-use budget, with the anti-fitting rule**

PROPOSED: the E7 fallback time budget is set by a follow-up ADR before the first M2 measurement run, from (a) the interactive-use budget, (b) E3 consistency, (c) the reference protocols' fallback/recovery behaviour measured in the lab — with the anti-fitting rule (never revised because a PHOENIX measurement missed it). The measured interval runs from 'transport declared failed' to 'application traffic flows on the new transport', including retransmission of in-flight data (D21). This record sets the procedure and the timing; it deliberately does not set the number.

**Calculation that produced it:** S = 0.30·4 + 0.20·4 + 0.20·3 + 0.15·4 + 0.15·3 = 3.70, highest of the scored alternatives ('no time budget at all' 2.90; invent-a-number 2.70). Margin 0.80 points = 21.6% — outside the close-call band.

**Confidence:** Opinion, HIGH (85%) that the procedure is correct — it is the pre-registered procedure in 05 §4.4, identical in form to D16's. No confidence is assigned to any numeric budget, because none is set. Calibration p = 0.85 for the procedure.

**Status / date / deciders:** PROPOSED — 2026-11-25 — decided by Abolfazl (owner) at the Phase 1 gate; this record proposes, it does not accept. A decision without its M2/M3 evidence is PROPOSED, not ACCEPTED. Decided at: M0 (set — procedure, this record) → before the first M2 run (the number, by follow-up ADR) → M4 (measured against it, AT-E7-04). Evidence that would settle it: reference-protocol fallback behaviour (M2/M3); the interactive-use budget.

**Related items:** C9; AT-E7-01–06 (esp. AT-E7-04); M0, M2, M4; P8; D11 (the §4.4 procedure), D16 (same anti-fitting rule), D21 (retransmission consumes part of the budget), D22 (switch-sequence test).
