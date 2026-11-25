# Decision Record 0022 — scheduler-classifiability

- **Decision:** 0022 (D22 of 12 §6)
- **Status:** PROPOSED
- **Date:** 2026-11-25
- **Version:** V1.0
- **Owner:** Abolfazl (decides; owner of the phase-gate go-ahead); Phase 1 architecture work (this record); implementation owner: Phase 2 (pending owner go-ahead)
- **Source:** phase0/05-protocol-design-draft.md §7 Q2/Q15, §2.5; phase0/04-architecture-proposals.md §4.4 (scheduler classifiability — 'I cannot verify that any scheduler is unclassifiable'); phase0/12-priority-roadmap.md §2.1 (C revival condition); phase0/03-feasibility-assessment.md §6 E7

## 1. Problem definition

Scheduler scoring / switch-sequence classifiability: is the A/B switch sequence itself classifiable, and (if C is ever revived) is any scheduler unclassifiable? No source in Phase 0 addresses scheduler fingerprinting (U9). The A/B switch is simpler than C's multipath, but its own properties (fallback latency, switch-sequence classifiability, no identity leak across the switch) still need falsifiers (Q15) — if the A/B fallback is assumed to work because it is 'simpler than C', the draft repeats C's error in miniature.

## 2. Known facts

- No source in Phase 0 addresses scheduler fingerprinting (U9); 'I cannot verify that any scheduler is unclassifiable' (`04:§4.4`); no scoring function for a per-flow/per-burst scheduler is specified in Phase 0 (`05:§2.5`).
- C is deferred, not rejected (12 §2.1). Revival condition: C is revived if and only if a Phase 1/2 measurement shows that A and B each fail a transport-loss scenario that C's session layer would survive, with the scheduler's classifiability tested under E1 (the scheduler's transport-choice sequence as an explicit classifier feature, `04:§4.4`).
- The A+B session-layer fallback already delivers the session-preserving property in its simpler sequential form (one transport at a time — 01 §3.4), which narrows what C would still have to prove: that simultaneous multipath beats sequential session-preserving fallback by a measured margin under E3 + the UDP-blocked fallback test (AT-E7-02). No such measurement exists; C stays deferred.
- The switch sequence is an E1 feature: AT-E7-05 requires Adv ≤ 0.05 / AUC ≤ 0.55 across the rotation boundary (E7, `03:§6`); the health-scoring function for transport fallback is unspecified in Phase 0 (`05:§7` Q2/Q15; `05` App. B item 5).
- The A/B switch-sequence test (AT-E7-05) is the smaller form of the scheduler question (Q2, applied to A/B — `05:§7` Q2 note).

## 3. Assumptions

- The switch sequence can be tested as an explicit classifier feature: the distinguisher is trained with the transport-choice sequence (which transport, when, in what order) as a feature, and Adv ≤ 0.05 is the bar. [MEDIUM — the test design is specified here; the distinguisher implementation is Phase 2 harness work.]
- A health-scoring function can be specified before M2 (signals, thresholds, scoring) — the switch is triggered by measured health, not by assumption. [MEDIUM — the function itself is a Phase 2 design deliverable.]

## 4. Unknown variables

- Whether any scheduler is unclassifiable — C's highest-risk unknown, unsettled; C stays deferred until a measurement exists.
- Whether the A/B switch sequence is classifiable — unsettled; AT-E7-05 at M2 is the test.
- The health-scoring function (what signals, what thresholds, what scoring) — unspecified in Phase 0; no source addresses it.
- Whether simultaneous multipath beats sequential session-preserving fallback by a measured margin — no measurement exists (C's revival condition is unmet).

## 5. Constraints

- Hard filter — security: a scheduler (or switch sequence) that is itself classifiable fails E1 even if each transport passes individually (`05:§7` Q2 falsifier). A switch design that cannot pass AT-E7-05 is out.
- C's revival requires the stated measurement (A and B each fail a transport-loss scenario C's session layer would survive, with the scheduler tested under E1). No revival without it.
- No silent downgrade; fallback is explicit and user-visible; no linkable identifier across the switch (E7/AT-E7-06).

## 6. Mathematical / computational model

Switch-sequence classifiability as a classifier problem: the feature is the sequence of transport choices (and their timing) over a session; the distinguisher's job is to separate PHOENIX's switch sequences from the cover's transport behaviour. For A/B the sequence is a fallback event (rare, triggered by failure); for C it would be a continuous scheduling decision (per-flow/per-burst). The rarer and more failure-driven the sequence, the less signal it carries — but 'rare' is not 'absent', which is why AT-E7-05 tests it. No closed-form model; the falsifier is AT-E7-05 at M2.

## 7. Alternatives (≥3 when consequential, incl. "do nothing")

| # | Alternative | Pros | Cons |
|---|-------------|------|------|
| 1 | Do nothing — no switch-sequence test; assume the A/B fallback is unclassifiable because it is 'simpler than C' | Zero work. | Repeats C's error in miniature (Q15): an unmeasured fallback is not a fallback. If the switch sequence is classifiable, E1 fails at M3 with no earlier warning. Fails the P8/P9 obligation. |
| 2 | A/B switch sequence tested as an explicit E1 feature (AT-E7-05); no scheduler scoring function adopted; C stays deferred under its revival condition (PICK) | Tests the actual risk (the A/B switch, which the MVP ships) instead of the hypothetical one (C's scheduler, which it does not); the health-scoring function is specified before M2 (measured-health admission, not assumption); C's revival condition stays intact and measurable. | The health-scoring function itself is still unspecified (a Phase 2 deliverable); the test can only run once both transports and the session layer exist (M2) — no earlier signal. |
| 3 | Adopt a scheduler scoring function now (per-flow/per-burst, with stated signals and thresholds) and test it under E1 | C's highest-risk unknown would be addressed in Phase 1, not deferred. | No source addresses scheduler fingerprinting (U9) — any scoring function written now is invented, not evidenced ('I cannot verify a correct scoring function from the Phase 0 sources', `04:§4.3.3`). Writing one now would be a design without a falsifier that can pass before the transports exist. Premature: the function can only be tested once the session layer exists (M2). |
| 4 | Revive C now (multipath with scheduler) without the revival measurement | Highest performance ceiling (5/5); session-preserving fallback by design. | Commits the least reversible choice (2/5) before A and B are measured; the scheduler is unproven unclassifiable; the revival condition (a measurement showing A and B each fail a scenario C would survive) is unmet. Fails the C-deferral decision (D4/D5) — a reversal of a Phase 1 decision without new evidence. |
| 5 | Make the switch sequence random (per-switch random transport choice) to defeat classification | Randomness is the intuitive answer to sequence classification. | Random switching is not health-driven switching — it switches away from a healthy transport for no reason, and a random sequence is itself a distribution (uniform over transports) that a classifier can learn; randomness without a health basis also breaks the 'admitted only on measured health' rule (D3/D5). A random scheduler is still a scheduler, and still unproven unclassifiable. |

**Hard filters applied first** (license-compatible; MSRV set by Phase 1 and recorded; security: no known unpatched CVE, no custom crypto, audited libraries only — `03:§12.1`). A candidate failing any one is out, with no scoring. No candidate fails a license/MSRV/CVE/custom-crypto filter. 'Revive C now' is not filter-failed but contradicts the D4/D5 deferral (which requires the revival measurement) and is rejected as a reversal without new evidence. All other candidates are admissible.

**Scoring.** Weights: correctness/safety .30, maintainability .20, performance .20, reversibility .15, effort .15; S = Σ(w·s); each criterion 0–5 (5 = best). Scores are **Opinion** unless a cell is labelled VERIFIED arithmetic or a cited Fact; the basis of each score is stated in the Pros/Cons above. Computed in Python (this generator):

| Alternative | Corr .30 | Maint .20 | Perf .20 | Rev .15 | Eff .15 | **S** |
|---|---:|---:|---:|---:|---:|---:|
| Do nothing — no switch-sequence test; assume the A/B fallback is unclassifiable because it is 'simpler than C' | 1 | 1 | 4 | 5 | 5 | **2.80** |
| A/B switch sequence tested as an explicit E1 feature (AT-E7-05); no scheduler scoring function adopted; C stays deferred under its revival condition (PICK) | 4 | 4 | 3 | 4 | 3 | **3.65** |
| Adopt a scheduler scoring function now (per-flow/per-burst, with stated signals and thresholds) and test it under E1 | 3 | 3 | 3 | 3 | 3 | **3.00** |
| Revive C now (multipath with scheduler) without the revival measurement | 3 | 2 | 5 | 2 | 2 | **2.90** |
| Make the switch sequence random (per-switch random transport choice) to defeat classification | 2 | 3 | 3 | 3 | 3 | **2.70** |

**Ranking:** 3.65 A/B switch sequence tested as an explicit E1 feature (AT-E7-05); no scheduler scoring function adopted; C stays deferred under its revival condition (PICK) > 3.00 Adopt a scheduler scoring function now (per-flow/per-burst, with stated signals and thresholds) and test it under E1 > 2.90 Revive C now (multipath with scheduler) without the revival measurement > 2.80 Do nothing — no switch-sequence test; assume the A/B fallback is unclassifiable because it is 'simpler than C' > 2.70 Make the switch sequence random (per-switch random transport choice) to defeat classification. Gap between top two: 0.65 points = 17.81% of the leader's score.

**Sensitivity (±20% per weight, other weights renormalised to sum 1.0):**

| Weight varied | −20% → S | +20% → S | Leader changes? |
|---|---|---|---|
| Correctness/safety | Do nothing — no switch-sequence test; assume the A/B fallback is unclassifiable because it is 'simpler than C': 2.9149 / A/B switch sequence tested as an explicit E1 feature (AT-E7-05); no scheduler scoring function adopted; C stays deferred under its revival condition (PICK): 3.6277 / Adopt a scheduler scoring function now (per-flow/per-burst, with stated signals and thresholds) and test it under E1: 3.0000 / Revive C now (multipath with scheduler) without the revival measurement: 2.8936 / Make the switch sequence random (per-switch random transport choice) to defeat classification: 2.7447 | Do nothing — no switch-sequence test; assume the A/B fallback is unclassifiable because it is 'simpler than C': 2.6981 / A/B switch sequence tested as an explicit E1 feature (AT-E7-05); no scheduler scoring function adopted; C stays deferred under its revival condition (PICK): 3.6698 / Adopt a scheduler scoring function now (per-flow/per-burst, with stated signals and thresholds) and test it under E1: 3.0000 / Revive C now (multipath with scheduler) without the revival measurement: 2.9057 / Make the switch sequence random (per-switch random transport choice) to defeat classification: 2.6604 | No |
| Maintainability | Do nothing — no switch-sequence test; assume the A/B fallback is unclassifiable because it is 'simpler than C': 2.8750 / A/B switch sequence tested as an explicit E1 feature (AT-E7-05); no scheduler scoring function adopted; C stays deferred under its revival condition (PICK): 3.6354 / Adopt a scheduler scoring function now (per-flow/per-burst, with stated signals and thresholds) and test it under E1: 3.0000 / Revive C now (multipath with scheduler) without the revival measurement: 2.9375 / Make the switch sequence random (per-switch random transport choice) to defeat classification: 2.6875 | Do nothing — no switch-sequence test; assume the A/B fallback is unclassifiable because it is 'simpler than C': 2.7308 / A/B switch sequence tested as an explicit E1 feature (AT-E7-05); no scheduler scoring function adopted; C stays deferred under its revival condition (PICK): 3.6635 / Adopt a scheduler scoring function now (per-flow/per-burst, with stated signals and thresholds) and test it under E1: 3.0000 / Revive C now (multipath with scheduler) without the revival measurement: 2.8654 / Make the switch sequence random (per-switch random transport choice) to defeat classification: 2.7115 | No |
| Performance | Do nothing — no switch-sequence test; assume the A/B fallback is unclassifiable because it is 'simpler than C': 2.7500 / A/B switch sequence tested as an explicit E1 feature (AT-E7-05); no scheduler scoring function adopted; C stays deferred under its revival condition (PICK): 3.6771 / Adopt a scheduler scoring function now (per-flow/per-burst, with stated signals and thresholds) and test it under E1: 3.0000 / Revive C now (multipath with scheduler) without the revival measurement: 2.8125 / Make the switch sequence random (per-switch random transport choice) to defeat classification: 2.6875 | Do nothing — no switch-sequence test; assume the A/B fallback is unclassifiable because it is 'simpler than C': 2.8462 / A/B switch sequence tested as an explicit E1 feature (AT-E7-05); no scheduler scoring function adopted; C stays deferred under its revival condition (PICK): 3.6250 / Adopt a scheduler scoring function now (per-flow/per-burst, with stated signals and thresholds) and test it under E1: 3.0000 / Revive C now (multipath with scheduler) without the revival measurement: 2.9808 / Make the switch sequence random (per-switch random transport choice) to defeat classification: 2.7115 | No |
| Reversibility | Do nothing — no switch-sequence test; assume the A/B fallback is unclassifiable because it is 'simpler than C': 2.7320 / A/B switch sequence tested as an explicit E1 feature (AT-E7-05); no scheduler scoring function adopted; C stays deferred under its revival condition (PICK): 3.6392 / Adopt a scheduler scoring function now (per-flow/per-burst, with stated signals and thresholds) and test it under E1: 3.0000 / Revive C now (multipath with scheduler) without the revival measurement: 2.9278 / Make the switch sequence random (per-switch random transport choice) to defeat classification: 2.6907 | Do nothing — no switch-sequence test; assume the A/B fallback is unclassifiable because it is 'simpler than C': 2.8641 / A/B switch sequence tested as an explicit E1 feature (AT-E7-05); no scheduler scoring function adopted; C stays deferred under its revival condition (PICK): 3.6602 / Adopt a scheduler scoring function now (per-flow/per-burst, with stated signals and thresholds) and test it under E1: 3.0000 / Revive C now (multipath with scheduler) without the revival measurement: 2.8738 / Make the switch sequence random (per-switch random transport choice) to defeat classification: 2.7087 | No |
| Effort | Do nothing — no switch-sequence test; assume the A/B fallback is unclassifiable because it is 'simpler than C': 2.7320 / A/B switch sequence tested as an explicit E1 feature (AT-E7-05); no scheduler scoring function adopted; C stays deferred under its revival condition (PICK): 3.6701 / Adopt a scheduler scoring function now (per-flow/per-burst, with stated signals and thresholds) and test it under E1: 3.0000 / Revive C now (multipath with scheduler) without the revival measurement: 2.9278 / Make the switch sequence random (per-switch random transport choice) to defeat classification: 2.6907 | Do nothing — no switch-sequence test; assume the A/B fallback is unclassifiable because it is 'simpler than C': 2.8641 / A/B switch sequence tested as an explicit E1 feature (AT-E7-05); no scheduler scoring function adopted; C stays deferred under its revival condition (PICK): 3.6311 / Adopt a scheduler scoring function now (per-flow/per-burst, with stated signals and thresholds) and test it under E1: 3.0000 / Revive C now (multipath with scheduler) without the revival measurement: 2.8738 / Make the switch sequence random (per-switch random transport choice) to defeat classification: 2.7087 | No |

**Tipping weight:** none — no single ±20% weight variation changes the leader. The ranking is robust to weight choice (it may still be fragile to score choice; see §12).

**Close-call handling (<10% margin):** the margin is 17.81%, outside the <10% band, so no benchmark is triggered by the close-call rule. See §12 for the tie-break evidence.

## 8. Expected benefits

Testing the A/B switch sequence as an explicit E1 feature: the MVP's actual switch behaviour is measured (AT-E7-05), not assumed; the health-scoring function is specified before M2 (measured-health admission); C's revival condition remains a measurable gate, not a forgotten option; the session layer's sequential fallback is credited with what it already delivers (session preservation without a scheduler).

## 9. Failure modes

AT-E7-05 fails (Adv > 0.05 across the rotation boundary) → the switch sequence is classifiable; mitigated by the failure loop (bisect: is it the switch timing, the retransmission burst (D21), or the resumption token? → change one variable → re-measure). The health-scoring function flaps (switches too often, generating switch-sequence signal) → mitigated by hysteresis in the scoring function (specified in Phase 2) and by the gradual re-admission rule. C's revival condition is met (A and B each fail a scenario C would survive) → C is revived as a Phase 2+ decision by ADR, with the scheduler tested under E1 from the start — the revival is a decision, not a surprise, because the condition was recorded here.

**Consequence if wrong:** If the pick is wrong (the A/B switch sequence is classifiable and cannot be repaired by changing one variable), the cost is a session-layer redesign at M2/M3 — bounded because AT-E7-05 runs at M2, before M4 acceptance, and the failure loop isolates the cause (switch timing vs retransmission burst vs token). If the pick is wrong the other way (a scheduler scoring function should have been adopted in Phase 1), the cost is that C's revival, if ever triggered, starts its scheduler work later — bounded because C is deferred, not rejected, and the revival condition is the trigger, not a schedule.

**Reversibility cost of undoing it:** High: the switch-sequence test is a harness test (AT-E7-05); the health-scoring function is session-layer policy. Changing either costs an ADR + a session-layer/harness change; the transports are unaffected.

## 10. Cost / complexity

Specification cost: the AT-E7-05 test design (switch sequence as an explicit classifier feature) + the health-scoring function (Phase 2 deliverable). Implementation cost: session-layer health scoring + gradual re-admission at M2. Test cost: AT-E7-05 at M2/M4 (n = 5,000/class, §3.1 sizing).

## 11. Experimental validation method

AT-E7-05 at M2 (rotation boundary Adv ≤ 0.05 / AUC ≤ 0.55, with the transport-choice sequence as an explicit classifier feature; n = 5,000/class). AT-E7-02/03 (fallback/rotation engage and carry traffic). AT-E7-06 (no identity leak). Falsifier: AT-E7-05 fails → the switch sequence is classifiable; the failure loop applies (bisect → root cause → change one variable → re-measure). C-revival falsifier: a Phase 1/2 measurement showing A and B each fail a transport-loss scenario C's session layer would survive, with the scheduler's classifiability tested under E1 — until that measurement exists, C stays deferred.

## 12. Final decision + evidence

**Decision (PROPOSED): A/B switch sequence tested as an explicit E1 feature (AT-E7-05); no scheduler scoring function adopted; C stays deferred under its revival condition**

PROPOSED: the A/B switch sequence is tested as an explicit E1 feature (AT-E7-05 at M2); no scheduler scoring function is adopted in Phase 1 (no source addresses scheduler fingerprinting — any function written now would be invented); the health-scoring function for transport fallback is specified before M2 (measured-health admission, with hysteresis and gradual re-admission); C stays deferred under its revival condition (a measurement showing A and B each fail a transport-loss scenario C's session layer would survive, with the scheduler tested under E1).

**Calculation that produced it:** S = 0.30·4 + 0.20·4 + 0.20·3 + 0.15·4 + 0.15·3 = 3.70, highest of the scored alternatives (adopt-a-scheduler-now 3.00; random switching 2.70). Margin 0.70 points = 18.9% — outside the close-call band.

**Confidence:** Opinion, HIGH (80%) that testing the A/B switch sequence (rather than adopting an unevidenced scheduler) is the correct scope — the A/B switch is the risk the MVP ships, and the scheduler question is 'I cannot verify' from every Phase 0 source. MEDIUM (65%) that AT-E7-05 passes on the first run (the retransmission burst, D21, is an untested switch-sequence feature). Calibration p = 0.80 / 0.65 respectively.

**Status / date / deciders:** PROPOSED — 2026-11-25 — decided by Abolfazl (owner) at the Phase 1 gate; this record proposes, it does not accept. A decision without its M2/M3 evidence is PROPOSED, not ACCEPTED. Decided at: M2 (A/B test) / C-revival decision (not in this roadmap — only on the revival measurement). Evidence that would settle it: AT-E7-05 at M2; the revival measurement for C.

**Related items:** U9, R4; AT-E7-02/03/05/06; M2, M4; P8, P9; D21 (the retransmission burst is a switch-sequence feature), D5 (C revival condition), D3 (measured-health admission).
