# Decision Record 0004 — architecture-proposals

- **Decision:** 0004 (D04 of 12 §6)
- **Status:** PROPOSED
- **Date:** 2026-11-25
- **Version:** V1.0
- **Owner:** Abolfazl (decides; owner of the phase-gate go-ahead); Phase 1 architecture work (this record); implementation owner: Phase 2 (pending owner go-ahead)
- **Source:** phase0/04-architecture-proposals.md §7 (matrix, §7.4 sensitivity, §7.5 recommendation); phase1/01-architecture-selection.md §1 (re-score) and §5.1 (draft D4); governing weights 03:§12.1; fix-log R1–R5 (corrected sensitivity table, R3)

## 1. Problem definition

Are the three Phase 0 architecture proposals (A QUIC-native, B TLS/TCP, C multipath) genuinely distinct alternatives, and do their decision-matrix scores survive re-scoring on current evidence? This must be decided now because the transport pick (D5) and the D2 wire specification both consume the matrix, and a design space that collapses to 'VLESS+REALITY with a new name' is a NO-GO flip (F3, `09:§4.1`).

## 2. Known facts

- Three proposals exist and genuinely differ in where the trust boundary sits, what the wire looks like, and what fails first (`04:§0.4`). [Fact, HIGH.]
- Base scores (Opinion): A 3.40, B 3.60, C 2.90; gap B–A 0.20 = 5.56% of B — a close call (<10%). Recomputed in Python in `phase1/01-architecture-selection.md` §1.2 and independently re-verified by this record's generator: A 3.40, B 3.60, C 2.90. [VERIFIED arithmetic; Opinion scores.]
- QUIC/HTTP-3 dropped to near zero in Iran from 22 Sept 2022 while TCP/443 + TLS continued (`01:§4`). [Fact, HIGH (90%).] June 2025: QUIC blocked at 100% via Initial drops, still 100% blocked after the shutdown lifted while TCP recovered (FOCI 2026 via Lantern corpus, secondary). [Fact, MEDIUM (70%).]
- GFW decrypts QUIC Initial packets at scale since 7 April 2024, extracts SNI, drops subsequent UDP on blocklisted flows (Zohaib et al., USENIX Security 2025). [Fact, HIGH (95%).]
- ML-KEM-768 ek 1,184 B / ct 1,088 B (FIPS 203 Table 3); hybrid share 1,216 B. [Fact, VERIFIED (100%).]
- Xue 2024 per-protocol TPRs (slides): VMess 77.14%, Shadowsocks 85.38%, VLESS 74.83%, Trojan 73.71%. [Fact, MEDIUM (90%, secondary — paper PDF not read, U13).]
- Xue NDSS 2025: per-visit TPR >70% at 1% FPR, ~half of detections within the first 60 packets, independent of payload obfuscation and uTLS mimicry. [Fact, HIGH (90%).]
- Sensitivity table corrected in the Phase 1 Step 0 fix round (R3): 6 cells in `04` §7.4 were wrong under the stated method (11 cells differ from the old table); the corrected table holds B > A > C in all 10 rows, gap range 0.14–0.26 points (4–7%).

## 3. Assumptions

- The Phase 0 cell scores are the best available Opinion estimates and change only on named new evidence (01 §1.3). [MEDIUM — the scores' load-bearing inputs U4/U5 are unmeasured; 'no new evidence' means 'no change' is a convention, not a measurement.]
- Weight sensitivity (±20%) is the right robustness test for the matrix; score sensitivity (§6) is reported alongside it. [MEDIUM — method per the refined spec.]
- The three proposals remain genuinely distinct after the re-score (no collapse to a single design). [HIGH — the re-score changes no cell, so the distinctions stand.]

## 4. Unknown variables

- Whether Vision's padding defeats Xue 2024 (U4) and whether multiplexing defeats it (U5) — both unmeasured; both bear on B's correctness score. Settled by AT-E1-03 at M3.
- Whether any scheduler is unclassifiable (C's highest-risk unknown, `04:§4.4`) — unsettled; C stays deferred until a measurement exists.
- Whether the Aparecium vectors against ShadowTLS v3 were fixed in later releases (U7) — unsettled; bears on B's probe surface.

## 5. Constraints

- Hard filters first: license-compatible, no known unpatched CVE, no custom crypto, audited libraries only (`03:§12.1`). All three proposals use standard vetted crypto only (rustls/quinn); none fails a hard filter at the architecture level.
- No 'undetectable' language; no superiority claim without reproducible evidence; no PHOENIX performance number asserted.
- Tier C: the transport decision's implementation (Phase 2) needs the owner's phase-gate go-ahead; this record proposes, it does not start Phase 2.
- Flip F3: if Phase 1 cannot produce three genuinely distinct proposals, the verdict is NO-GO (`09:§4.1`).

## 6. Mathematical / computational model

S = Σ(w·s), weights correctness/safety .30, maintainability .20, performance .20, reversibility .15, effort .15; scores 0–5. Sensitivity: each weight ±20% relative, renormalised. Close call (<10%) triggers a ≤30-minute benchmark. Score-level sensitivity (new in the Phase 1 re-score, 01 §1.5): a single ±1 cell change flips or ties the B–A ranking in 6 of 30 cases (A correctness +1 and B correctness −1 flip to A; A maintainability +1, A performance +1, B maintainability −1, B performance −1 tie). Dimensional check: the 0.20 gap is smaller than the weight of any single criterion, so the ranking cannot be more robust than the least-certain cell score — the model is well-formed but the input scores are Opinion, and the output inherits that uncertainty. No closed-form model of the RTT trade-off exists; the matrix does not model Gap 1.

**Verification of the matrix arithmetic (recomputed in Python by this record, 2026-11-25 — independent of `phase1/01-architecture-selection.md` §1.2/§1.4, identical results):** base S(A) = 3.40, S(B) = 3.60, S(C) = 2.90; gap B–A = 0.20 = 5.56% of B (close call). Weight sensitivity (±20% per weight, other weights renormalised to sum 1.0):

| Weight varied | −20% → S (A / B / C) | +20% → S (A / B / C) | Leader changes? |
|---|---|---|---|
| Correctness/safety (.30) | 3.4255 / 3.5745 / 2.8936 | 3.3774 / 3.6226 / 2.9057 | No — B leads both |
| Maintainability (.20) | 3.3750 / 3.6250 / 2.9375 | 3.4231 / 3.5769 / 2.8654 | No — B leads both |
| Performance (.20) | 3.3750 / 3.6250 / 2.8125 | 3.4231 / 3.5769 / 2.9808 | No — B leads both |
| Reversibility (.15) | 3.4124 / 3.5876 / 2.9278 | 3.3883 / 3.6117 / 2.8738 | No — B leads both |
| Effort (.15) | 3.4124 / 3.5876 / 2.9278 | 3.3883 / 3.6117 / 2.8738 | No — B leads both |

Stress tests (verified arithmetic): equal weights (0.20 each) → A 3.40, B 3.60, C 2.80 (same ranking); performance weighted at .30 (correctness .20, others unchanged) → A 3.50, B 3.50, C 3.10 (tie). Score-level sensitivity (verified): 6 of 30 single-cell ±1 changes flip or tie the ranking — A correctness 3→4 flips to A (3.70 > 3.60); B correctness 4→3 flips to A (3.40 > 3.30); A maintainability 4→5, A performance 4→5, B maintainability 3→2, B performance 3→2 each tie at 3.40 / 3.40. The corrected Phase 0 sensitivity table (fix-log R3) holds B > A > C in all 10 rows, gap range 0.14–0.26 points (4–7%).

## 7. Alternatives (≥3 when consequential, incl. "do nothing")

| # | Alternative | Pros | Cons |
|---|-------------|------|------|
| 1 | Do nothing — keep the Phase 0 matrix as-is without re-scoring | Zero work; Phase 0 arithmetic already verified. | The matrix would be inherited, not re-derived; new evidence (June 2025 Iran QUIC, verification-report rejections R1–R5) would go unchecked; the re-score obligation in P7 would be unmet. |
| 2 | Accept the re-score: B 3.60 > A 3.40 > C 2.90, ranking stands, close call stands (PICK) | Verified arithmetic (recomputed twice, in 01 §1.2 and by this record's generator); evidence check cell-by-cell; score fragility (§6) made explicit; benchmark trigger preserved. | The pick remains provisional; the matrix still rests on unmeasured inputs (U4/U5). |
| 3 | Revise a cell on judgment without new measurement (e.g. downgrade B correctness for U4) | Would reflect the genuine uncertainty in B's score. | Downgrading on absence of evidence is as unjustified as the score it replaces; it would flip the pick on an Opinion change with no data behind it. |
| 4 | Re-weight (e.g. performance .30) to break the close call | Would produce a decisive-looking ranking. | A tie at performance .30 (A 3.50 = B 3.50, verified arithmetic) shows re-weighting does not decide it — it exposes that the decision hinges on how much the project values loss resilience vs cover maturity, which is a judgment for D5, not a matrix fix. |

**Hard filters applied first** (license-compatible; MSRV set by Phase 1 and recorded; security: no known unpatched CVE, no custom crypto, audited libraries only — `03:§12.1`). A candidate failing any one is out, with no scoring. All three proposals pass the hard filters at the architecture level (standard vetted crypto, audited libraries, no custom crypto). No alternative is filter-excluded; the choice is between re-scoring methods, and 'revise on judgment' is rejected on method (scores change only on named evidence), not on a filter.

**Scoring.** Weights: correctness/safety .30, maintainability .20, performance .20, reversibility .15, effort .15; S = Σ(w·s); each criterion 0–5 (5 = best). Scores are **Opinion** unless a cell is labelled VERIFIED arithmetic or a cited Fact; the basis of each score is stated in the Pros/Cons above. Computed in Python (this generator):

| Alternative | Corr .30 | Maint .20 | Perf .20 | Rev .15 | Eff .15 | **S** |
|---|---:|---:|---:|---:|---:|---:|
| Do nothing — keep the Phase 0 matrix as-is without re-scoring | 2 | 3 | 3 | 5 | 5 | **3.30** |
| Accept the re-score: B 3.60 > A 3.40 > C 2.90, ranking stands, close call stands (PICK) | 4 | 4 | 3 | 4 | 3 | **3.65** |
| Revise a cell on judgment without new measurement (e.g. downgrade B correctness for U4) | 3 | 3 | 3 | 3 | 3 | **3.00** |
| Re-weight (e.g. performance .30) to break the close call | 3 | 3 | 3 | 3 | 3 | **3.00** |

**Ranking:** 3.65 Accept the re-score: B 3.60 > A 3.40 > C 2.90, ranking stands, close call stands (PICK) > 3.30 Do nothing — keep the Phase 0 matrix as-is without re-scoring > 3.00 Revise a cell on judgment without new measurement (e.g. downgrade B correctness for U4) > 3.00 Re-weight (e.g. performance .30) to break the close call. Gap between top two: 0.35 points = 9.59% of the leader's score.

**Sensitivity (±20% per weight, other weights renormalised to sum 1.0):**

| Weight varied | −20% → S | +20% → S | Leader changes? |
|---|---|---|---|
| Correctness/safety | Do nothing — keep the Phase 0 matrix as-is without re-scoring: 3.3830 / Accept the re-score: B 3.60 > A 3.40 > C 2.90, ranking stands, close call stands (PICK): 3.6277 / Revise a cell on judgment without new measurement (e.g. downgrade B correctness for U4): 3.0000 / Re-weight (e.g. performance .30) to break the close call: 3.0000 | Do nothing — keep the Phase 0 matrix as-is without re-scoring: 3.2264 / Accept the re-score: B 3.60 > A 3.40 > C 2.90, ranking stands, close call stands (PICK): 3.6698 / Revise a cell on judgment without new measurement (e.g. downgrade B correctness for U4): 3.0000 / Re-weight (e.g. performance .30) to break the close call: 3.0000 | No |
| Maintainability | Do nothing — keep the Phase 0 matrix as-is without re-scoring: 3.3125 / Accept the re-score: B 3.60 > A 3.40 > C 2.90, ranking stands, close call stands (PICK): 3.6354 / Revise a cell on judgment without new measurement (e.g. downgrade B correctness for U4): 3.0000 / Re-weight (e.g. performance .30) to break the close call: 3.0000 | Do nothing — keep the Phase 0 matrix as-is without re-scoring: 3.2885 / Accept the re-score: B 3.60 > A 3.40 > C 2.90, ranking stands, close call stands (PICK): 3.6635 / Revise a cell on judgment without new measurement (e.g. downgrade B correctness for U4): 3.0000 / Re-weight (e.g. performance .30) to break the close call: 3.0000 | No |
| Performance | Do nothing — keep the Phase 0 matrix as-is without re-scoring: 3.3125 / Accept the re-score: B 3.60 > A 3.40 > C 2.90, ranking stands, close call stands (PICK): 3.6771 / Revise a cell on judgment without new measurement (e.g. downgrade B correctness for U4): 3.0000 / Re-weight (e.g. performance .30) to break the close call: 3.0000 | Do nothing — keep the Phase 0 matrix as-is without re-scoring: 3.2885 / Accept the re-score: B 3.60 > A 3.40 > C 2.90, ranking stands, close call stands (PICK): 3.6250 / Revise a cell on judgment without new measurement (e.g. downgrade B correctness for U4): 3.0000 / Re-weight (e.g. performance .30) to break the close call: 3.0000 | No |
| Reversibility | Do nothing — keep the Phase 0 matrix as-is without re-scoring: 3.2474 / Accept the re-score: B 3.60 > A 3.40 > C 2.90, ranking stands, close call stands (PICK): 3.6392 / Revise a cell on judgment without new measurement (e.g. downgrade B correctness for U4): 3.0000 / Re-weight (e.g. performance .30) to break the close call: 3.0000 | Do nothing — keep the Phase 0 matrix as-is without re-scoring: 3.3495 / Accept the re-score: B 3.60 > A 3.40 > C 2.90, ranking stands, close call stands (PICK): 3.6602 / Revise a cell on judgment without new measurement (e.g. downgrade B correctness for U4): 3.0000 / Re-weight (e.g. performance .30) to break the close call: 3.0000 | No |
| Effort | Do nothing — keep the Phase 0 matrix as-is without re-scoring: 3.2474 / Accept the re-score: B 3.60 > A 3.40 > C 2.90, ranking stands, close call stands (PICK): 3.6701 / Revise a cell on judgment without new measurement (e.g. downgrade B correctness for U4): 3.0000 / Re-weight (e.g. performance .30) to break the close call: 3.0000 | Do nothing — keep the Phase 0 matrix as-is without re-scoring: 3.3495 / Accept the re-score: B 3.60 > A 3.40 > C 2.90, ranking stands, close call stands (PICK): 3.6311 / Revise a cell on judgment without new measurement (e.g. downgrade B correctness for U4): 3.0000 / Re-weight (e.g. performance .30) to break the close call: 3.0000 | No |

**Tipping weight:** none — no single ±20% weight variation changes the leader. The ranking is robust to weight choice (it may still be fragile to score choice; see §12).

**Close-call handling (<10% margin):** the margin is 9.59%, inside the <10% band, so the refined-spec rule fires: a ≤30-minute benchmark breaks the tie; if still tied, the more reversible option wins (`03:§12.1`). See §12 for the tie-break evidence.

## 8. Expected benefits

Accepting the re-score (the pick): a verified, re-derived matrix that the transport pick can cite; the benchmark trigger and the M2/M3 falsifiers attached to the exact cells they can move; C's deferral priced (0.70 behind B, not a close call). The corrected sensitivity table (R3) removes 6 wrong cells from the record.

## 9. Failure modes

The matrix is mistaken for a measurement: the scores are Opinion, and a future reader could cite 'B 3.60 > A 3.40' as if it were benchmark evidence. Mitigation: this record states the Opinion status and the score fragility in the same section as the scores. A single-cell score revision at M3 (notably B correctness on AT-E1-03) flips the ranking after M1/M2 work has started on B. Mitigation: B is the most reversible alternative, and the session-layer design is transport-agnostic, so the rework cost of a flip is bounded — but it is not zero, and the M4 revisit-or-stand ADR must price it. The design space collapses (flip F3): if Phase 1 finds the three proposals are not genuinely distinct, the verdict becomes NO-GO. No evidence of collapse was found in the re-score.

**Consequence if wrong:** If the re-score is wrong (a cell should have moved on evidence the check missed), the cost is a wrong default transport built first at M1 — bounded by B's reversibility (4/5) and the transport-agnostic session layer, and corrected at the M4 revisit-or-stand on M2/M3 evidence. If the matrix itself is wrong (the proposals are not distinct), F3 flips the verdict to NO-GO and the project does not start Phase 2.

**Reversibility cost of undoing it:** Full: the matrix can be re-scored again on M2/M3 evidence at zero implementation cost. The cost that matters is downstream: the matrix justifies building B first at M1, so a wrong matrix wastes a Phase 2 milestone; B's own reversibility (4/5) is the hedge against that.

## 10. Cost / complexity

Re-score cost: one analysis document (`phase1/01-architecture-selection.md`); no code, no build. Downstream cost is the M1 build order the matrix justifies.

## 11. Experimental validation method

Arithmetic: recomputed in Python (01 §1.2/§1.4; independently re-verified by this record's generator — identical results: base B 3.60 > A 3.40 > C 2.90; sensitivity table matches 01 §1.4 to 4 dp; stress tests equal-weights A 3.40/B 3.60/C 2.80 and performance-.30 tie A 3.50 = B 3.50 reproduced). Score inputs: AT-E1-03 (M3) can move B correctness; AT-E1-06 (M3) can move A correctness; the M2 ≤30-minute benchmark (AT-E3-01–03 + AT-E7-02) can move A/B performance. All 'not measured' until run. Design-space distinctness: flip F3 is checked at the M0 exit (the three proposals remain distinct in the re-score — verified in 01 §1.3).

## 12. Final decision + evidence

**Decision (PROPOSED): Accept the re-score: B 3.60 > A 3.40 > C 2.90, ranking stands, close call stands**

PROPOSED: accept the re-score. Ranking B 3.60 > A 3.40 > C 2.90 stands; close call stands; no cell changes on current evidence (01 §1.3). Evidence: Python recomputation (§1.2, §1.4 of 01) verifying Phase 0 arithmetic, plus the cell-by-cell evidence check, plus the R3-corrected sensitivity table. Revisit condition: any M2/M3 measurement named in §11 that justifies a single-cell revision re-opens this record; a revision that flips or ties the ranking re-opens D5 with it.

**Calculation that produced it:** S(A) = 0.30·3 + 0.20·4 + 0.20·4 + 0.15·3 + 0.15·3 = 0.90 + 0.80 + 0.80 + 0.45 + 0.45 = 3.40. S(B) = 0.30·4 + 0.20·3 + 0.20·3 + 0.15·4 + 0.15·4 = 1.20 + 0.60 + 0.60 + 0.60 + 0.60 = 3.60. S(C) = 0.30·3 + 0.20·2 + 0.20·5 + 0.15·2 + 0.15·2 = 0.90 + 0.40 + 1.00 + 0.30 + 0.30 = 2.90. Gap B–A = 0.20 = 5.56% of B — a close call. Weight sensitivity: B leads in all 10 variations (gap 0.14–0.26 after the R3 correction). Score sensitivity: 6 of 30 single-cell ±1 changes flip or tie the ranking — the ranking is robust to weight choice but fragile to score choice.

**Confidence:** VERIFIED (arithmetic — the recomputation, not a new measurement). Opinion, HIGH that no cell should change on current evidence (the §1.3 check is exhaustive against the Phase 0 record; 'no new evidence' is a statement about the record, not about the world). Opinion, HIGH that the ranking is fragile to score choice (§1.5). Calibration p = 0.90 for 'no cell changes on current evidence'.

**Status / date / deciders:** PROPOSED — 2026-11-25 — decided by Abolfazl (owner) at the Phase 1 gate; this record proposes, it does not accept. A decision without its M2/M3 evidence is PROPOSED, not ACCEPTED. Decided at: M0. Evidence that would settle it: M2 benchmark (performance cells); AT-E1-03/06 at M3 (correctness cells).

**Related items:** U4, U5, U7; R1–R3, R5, R7, C17, C18; AT-E1-03, AT-E1-06, AT-E3-01–03, AT-E7-02; M0 (decision), M2 (benchmark), M3 (E1 evidence), M4 (revisit-or-stand); P7; D5, D6.
