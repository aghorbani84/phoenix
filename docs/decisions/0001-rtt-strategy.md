# Decision Record 0001 — rtt-strategy

- **Decision:** 0001 (D01 of 12 §6)
- **Status:** ACCEPTED (F1 decided 2026-11-26 by owner delegation — see §12)
- **Date:** 2026-11-25
- **Version:** V1.0
- **Owner:** Abolfazl (decides; owner of the phase-gate go-ahead); Phase 1 architecture work (this record); implementation owner: Phase 2 (pending owner go-ahead)
- **Source:** phase1/02-rtt-strategy.md (§8 decision record); phase0/05-protocol-design-draft.md §5.5; phase0/03-feasibility-assessment.md §1.3 Condition 1, §1.4 F1; phase0/12-priority-roadmap.md §1 P1, §6 D1; Xue et al., NDSS 2025

## 1. Problem definition

Cross-layer RTT discrepancy (Xue NDSS 2025) detects proxied visits at >70% TPR at 1% FPR, independent of payload obfuscation and uTLS mimicry. It attacks session termination itself, so no wrapper fixes it. Phase 1 must produce a written RTT strategy with a falsifier, or record the gap as an accepted limitation (Condition 1 of the CONDITIONAL GO, `03:§1.3`). The decision is needed now because the E4 p99 target, the D2 timing sections, and the M4 acceptance verdict all depend on it.

## 2. Known facts

- Xue NDSS 2025: per-flow detection ~20% TPR; per-visit detection >70% TPR at 1% FPR; ~half of detections within the first 60 packets; ~80% of the 5,000 most frequently accessed websites generated a detectable fingerprint; FPR estimated ~0.6% (authors' slides, Censored Planet). [Fact, HIGH — paper verified in 00 §4.8; NDSS proceedings PDF itself not fetched in Phase 0.]
- The paper itself evaluated delayed ACK and multiplexing as mitigations and found both limited: delayed ACK 'produces atypical ACK timing that may itself be fingerprintable'; multiplexing 'only adds entropy without eliminating the RTTdiff signal'; delay-based timing obfuscation is 'fundamentally limited'. [Fact, MEDIUM — authors' slides + Lantern corpus summary of §§VI–VII, fetched 2026-11-25; two independent summaries agree.]
- Per-visit arithmetic (mathematical truth, given p = 0.20/flow): a visit of m flows is detected with probability 1 − 0.8^m; 1 − 0.8^m = 0.70 gives m ≈ 5.4 flows. A normal website visit easily opens that many flows.
- Cross-layer RTT fingerprinting is 'not verified' as deployed in every environment in the D1 matrix (`01:§2`). The lab cannot verify deployment (`03:§2.2`).
- Users who resolve DNS locally (at the client) are ~2× as susceptible as users who resolve DNS at the proxy (Lantern corpus summary of §VI-C-2). [Fact, MEDIUM.]
- The paper's own ACK-delay suggestion is up to 500 ms: on a 20 ms base RTT that is a 26× increase; 50 ms → 11×; 80 ms → 7.2×; 150 ms → 4.3×; 300 ms → 2.7× (computed 2026-11-25; mathematical truth given the 500 ms figure).

## 3. Assumptions

- The reproduced classifier in the Phase 2 harness will faithfully implement the paper's method (to be validated against the reference-proxy positive control). [MEDIUM — the method must be the paper's, not an invented one; AT-E4-07 design in 02 §7.]
- The paper's numbers transfer to the lab topology in shape if not in absolute value. [MEDIUM.]

## 4. Unknown variables

- Whether any ACK delay or multiplexing factor closes the gap within E3/E4 — to be measured by the bounded research track (R1-T, R2-T at M2).
- The owner's real destination distribution for the R3 egress co-location measurement (owner input; optional, not blocking).
- Whether any censor deploys the classifier — unverifiable from the lab.
- Whether a published definition of 'normal' timing (the authors' long-term direction, 'realistically simulating the timing patterns of normal traffic') will ever exist to simulate against. [UNKNOWN.]

## 5. Constraints

- Hard filter — security: no mitigation may be adopted on the strength of a claim the attack paper itself evaluated and found limited; a strategy must carry a falsifier.
- E3 rho ≥ 0.90 clean / ≥ 1.0 lossy; E4 p99 target (set by D16, without an RTT-shaping allowance once D1 is R5).
- No 'undetectable' language; no measurement against infrastructure PHOENIX does not own (Tier C).
- Flip F1 (`03:§1.4`; `09:§4.1`): if the owner requires RTT resistance as a release criterion, the verdict flips to NO-GO for Phase 2. That decision is the owner's, at the M4 gate.

## 6. Mathematical / computational model

RTTdiff ≈ proxy processing + propagation delay (proxy→origin leg). Visit detection 1 − 0.8^m for m flows at p = 0.20/flow (m ≈ 5.4 for 70%). ACK-delay cost multipliers 2.7×–26× at 500 ms. Pass bar for any strategy (Gap 1 testable property, `02:§15`): per-visit TPR ≤ ~10% at FPR 1% (classifier advantage near zero) **with** rho ≥ 0.90 clean **and** p99 within target **and** the shaping itself passing an E1-style distinguisher (Adv ≤ 0.05). No closed-form model of the RTT trade-off exists (`steps:§4.3` D6); only the harness can trace the (delay, TPR, rho, p99) curve.

## 7. Alternatives (≥3 when consequential, incl. "do nothing")

| # | Alternative | Pros | Cons |
|---|-------------|------|------|
| 1 | R1 — Delayed ACK / RTT shaping (up to 500 ms) | Only strategy with a concrete, paper-suggested parameter (500 ms); implementable natively on A via QUIC max_ack_delay (RFC 9000 §13.2.1, default 25 ms). | Paper evaluated it and found it limited (atypical ACK timing may itself be fingerprintable); fixed delay cannot match a per-destination RTTdiff distribution; 500 ms is additive to every handshake and of the same order as intercontinental RTTs; architecturally natural only on A, the transport that fails where UDP is blocked (Iran). |
| 2 | R2 — Multiplexing | Helps E3 under loss; helps the Xue 2024 encapsulated-TLS classifier per that paper's suggestion; already needed for other reasons. | Paper evaluated it and found it limited (adds entropy without eliminating the RTTdiff signal); per-stream misalignment persists inside the connection; shared fate — one blocked connection kills all streams. |
| 3 | R3 — Egress co-location with cached content | The suppressor effect itself is Fact (HIGH): most proxied flows terminate at CDN-cached content within 5 ms of the egress, which is why per-flow detection is only ~20%. | Deployment property, not a protocol property; helps most where detection matters least (popular cached content) and least where it matters most (non-cached long tail); no measurement of the owner's destination fraction exists — I cannot verify it; can never cover 100% of destinations. |
| 4 | R4 — Design that does not terminate the transport session at the proxy | The only candidate that would remove RTTdiff by construction rather than masking it. | No construction exists in Phase 0 or in the sources fetched for this document; any proxy that relays application data must terminate something, and whatever it terminates is measurable; restates the problem rather than solving it. Dropped as a candidate, retained as an open research direction. |
| 5 | Decoy timing / traffic splitting / constant-rate sending | Cheap to describe; familiar from website-fingerprinting defences. | Rejected by mechanism: dummy packets do not change the propagation delay of the proxy→origin leg, which is what RTTdiff measures; authors' slides list traffic splitting as 'likely creating new timing patterns that are fingerprintable'; a fixed-rate sender is itself a traffic-shape anomaly. Attacks the wrong variable. |
| 6 | Connection migration (QUIC CID-based, RFC 9000 §9) | Connectivity feature: session survives handovers. | Rejected by mechanism as an RTT mitigation: migration changes the path, but the application RTT changes by the same path change and the difference (proxy→origin leg) is untouched. Orthogonal to RTTdiff. |
| 7 | Do nothing — no written strategy, no recorded limitation | Zero work. | Violates Condition 1 of the CONDITIONAL GO; the M0 exit criterion is unmet; the MVP would carry an unrecorded detection vector, which is a security liability, not an asset. Fails the hard security filter (a known detection vector left unstated). |
| 8 | R5 — Accepted limitation, with bounded falsifiable research track (R1/R2/R3) (PICK) | Honest: the MVP's claims are scoped to what is measured; no engineering effort spent on mitigations the attack paper itself found limited; the falsifier experiments (R1-T, R2-T, R3-M) are preserved as the route back; R5 costs a document and a scope statement. | The MVP ships with a known, measured detection vector; deployment scope is limited to environments where the classifier is not verified as deployed; if the owner requires RTT resistance, F1 flips to NO-GO for Phase 2. |

**Hard filters applied first** (license-compatible; MSRV set by Phase 1 and recorded; security: no known unpatched CVE, no custom crypto, audited libraries only — `03:§12.1`). A candidate failing any one is out, with no scoring. No alternative fails a license/MSRV/CVE/custom-crypto filter at the strategy level. 'Do nothing' fails the hard security filter (Condition 1 binding; an unrecorded known detection vector) and is listed in the table but not scored. R4 is likewise not scored: no construction exists — it is not a candidate, it is a research direction.

**Scoring.** Weights: correctness/safety .30, maintainability .20, performance .20, reversibility .15, effort .15; S = Σ(w·s); each criterion 0–5 (5 = best). Scores are **Opinion** unless a cell is labelled VERIFIED arithmetic or a cited Fact; the basis of each score is stated in the Pros/Cons above. Computed in Python (this generator):

| Alternative | Corr .30 | Maint .20 | Perf .20 | Rev .15 | Eff .15 | **S** |
|---|---:|---:|---:|---:|---:|---:|
| R1 — Delayed ACK / RTT shaping (up to 500 ms) | 2 | 3 | 2 | 3 | 3 | **2.50** |
| R2 — Multiplexing | 2 | 3 | 3 | 3 | 3 | **2.70** |
| R3 — Egress co-location with cached content | 3 | 2 | 3 | 2 | 2 | **2.50** |
| R4 — Design that does not terminate the transport session at the proxy | — | — | — | — | — | **EXCLUDED by hard filter — not scored** |
| Decoy timing / traffic splitting / constant-rate sending | 1 | 2 | 2 | 2 | 2 | **1.70** |
| Connection migration (QUIC CID-based, RFC 9000 §9) | 1 | 3 | 3 | 3 | 3 | **2.40** |
| Do nothing — no written strategy, no recorded limitation | — | — | — | — | — | **EXCLUDED by hard filter — not scored** |
| R5 — Accepted limitation, with bounded falsifiable research track (R1/R2/R3) (PICK) | 4 | 4 | 4 | 4 | 4 | **4.00** |

**Ranking:** 4.00 R5 — Accepted limitation, with bounded falsifiable research track (R1/R2/R3) (PICK) > 2.70 R2 — Multiplexing > 2.50 R1 — Delayed ACK / RTT shaping (up to 500 ms) > 2.50 R3 — Egress co-location with cached content > 2.40 Connection migration (QUIC CID-based, RFC 9000 §9) > 1.70 Decoy timing / traffic splitting / constant-rate sending. Gap between top two: 1.30 points = 32.50% of the leader's score.

**Sensitivity (±20% per weight, other weights renormalised to sum 1.0):**

| Weight varied | −20% → S | +20% → S | Leader changes? |
|---|---|---|---|
| Correctness/safety | R1 — Delayed ACK / RTT shaping (up to 500 ms): 2.5319 / R2 — Multiplexing: 2.7447 / R3 — Egress co-location with cached content: 2.4681 / Decoy timing / traffic splitting / constant-rate sending: 1.7447 / Connection migration (QUIC CID-based, RFC 9000 §9): 2.4894 / R5 — Accepted limitation, with bounded falsifiable research track (R1/R2/R3) (PICK): 4.0000 | R1 — Delayed ACK / RTT shaping (up to 500 ms): 2.4717 / R2 — Multiplexing: 2.6604 / R3 — Egress co-location with cached content: 2.5283 / Decoy timing / traffic splitting / constant-rate sending: 1.6604 / Connection migration (QUIC CID-based, RFC 9000 §9): 2.3208 / R5 — Accepted limitation, with bounded falsifiable research track (R1/R2/R3) (PICK): 4.0000 | No |
| Maintainability | R1 — Delayed ACK / RTT shaping (up to 500 ms): 2.4792 / R2 — Multiplexing: 2.6875 / R3 — Egress co-location with cached content: 2.5208 / Decoy timing / traffic splitting / constant-rate sending: 1.6875 / Connection migration (QUIC CID-based, RFC 9000 §9): 2.3750 / R5 — Accepted limitation, with bounded falsifiable research track (R1/R2/R3) (PICK): 4.0000 | R1 — Delayed ACK / RTT shaping (up to 500 ms): 2.5192 / R2 — Multiplexing: 2.7115 / R3 — Egress co-location with cached content: 2.4808 / Decoy timing / traffic splitting / constant-rate sending: 1.7115 / Connection migration (QUIC CID-based, RFC 9000 §9): 2.4231 / R5 — Accepted limitation, with bounded falsifiable research track (R1/R2/R3) (PICK): 4.0000 | No |
| Performance | R1 — Delayed ACK / RTT shaping (up to 500 ms): 2.5208 / R2 — Multiplexing: 2.6875 / R3 — Egress co-location with cached content: 2.4792 / Decoy timing / traffic splitting / constant-rate sending: 1.6875 / Connection migration (QUIC CID-based, RFC 9000 §9): 2.3750 / R5 — Accepted limitation, with bounded falsifiable research track (R1/R2/R3) (PICK): 4.0000 | R1 — Delayed ACK / RTT shaping (up to 500 ms): 2.4808 / R2 — Multiplexing: 2.7115 / R3 — Egress co-location with cached content: 2.5192 / Decoy timing / traffic splitting / constant-rate sending: 1.7115 / Connection migration (QUIC CID-based, RFC 9000 §9): 2.4231 / R5 — Accepted limitation, with bounded falsifiable research track (R1/R2/R3) (PICK): 4.0000 | No |
| Reversibility | R1 — Delayed ACK / RTT shaping (up to 500 ms): 2.4845 / R2 — Multiplexing: 2.6907 / R3 — Egress co-location with cached content: 2.5155 / Decoy timing / traffic splitting / constant-rate sending: 1.6907 / Connection migration (QUIC CID-based, RFC 9000 §9): 2.3814 / R5 — Accepted limitation, with bounded falsifiable research track (R1/R2/R3) (PICK): 4.0000 | R1 — Delayed ACK / RTT shaping (up to 500 ms): 2.5146 / R2 — Multiplexing: 2.7087 / R3 — Egress co-location with cached content: 2.4854 / Decoy timing / traffic splitting / constant-rate sending: 1.7087 / Connection migration (QUIC CID-based, RFC 9000 §9): 2.4175 / R5 — Accepted limitation, with bounded falsifiable research track (R1/R2/R3) (PICK): 4.0000 | No |
| Effort | R1 — Delayed ACK / RTT shaping (up to 500 ms): 2.4845 / R2 — Multiplexing: 2.6907 / R3 — Egress co-location with cached content: 2.5155 / Decoy timing / traffic splitting / constant-rate sending: 1.6907 / Connection migration (QUIC CID-based, RFC 9000 §9): 2.3814 / R5 — Accepted limitation, with bounded falsifiable research track (R1/R2/R3) (PICK): 4.0000 | R1 — Delayed ACK / RTT shaping (up to 500 ms): 2.5146 / R2 — Multiplexing: 2.7087 / R3 — Egress co-location with cached content: 2.4854 / Decoy timing / traffic splitting / constant-rate sending: 1.7087 / Connection migration (QUIC CID-based, RFC 9000 §9): 2.4175 / R5 — Accepted limitation, with bounded falsifiable research track (R1/R2/R3) (PICK): 4.0000 | No |

**Tipping weight:** none — no single ±20% weight variation changes the leader. The ranking is robust to weight choice (it may still be fragile to score choice; see §12).

**Close-call handling (<10% margin):** the margin is 32.50%, outside the <10% band, so no benchmark is triggered by the close-call rule. See §12 for the tie-break evidence.

## 8. Expected benefits

R5 (the pick): the MVP's anti-detection claims cover first-packet entropy (E1-1), packet-length/timing ML classification (E1-2), encapsulated-TLS structure (E1-3) and active probing (E2) — and explicitly do not cover cross-layer RTT. The E4 p99 target becomes settable without an RTT-shaping allowance (D16). The research track (R1-T, R2-T, R3-M) keeps the question measurable instead of settled by argument.

## 9. Failure modes

A censor verifies deployment of the classifier → the MVP's deployment scope shrinks in that environment (monitoring duty in 02 §6.3; the environment is removed from scope by decision, D1 reopened). The owner requires RTT resistance → F1 flips to NO-GO for Phase 2 (reduced-scope re-proposal follows). R1/R2 experiments consume harness time without changing the verdict (bounded cost).

**Consequence if wrong:** If R5 is wrong (i.e. R1 or R2 could in fact close the gap within E3/E4), the cost is a delayed RTT defence: the MVP ships detectable by a cross-layer RTT classifier until the research track reopens D1. The track exists precisely to bound that cost — the (delay, TPR, rho, p99) curve is measured at M2 either way.

**Reversibility cost of undoing it:** Full: R5 is a document and a scope statement. Reopening D1 on a passing R1/R2/R3 experiment costs one ADR revision plus the shaping work that experiment justifies; no protocol work is spent on RTT shaping in the MVP, so nothing built needs undoing.

## 10. Cost / complexity

R5 costs a document and a scope statement. The research track costs three bounded harness experiments at M2 (R1-T, R2-T, R3-M). No protocol work is spent on RTT shaping in the MVP.

## 11. Experimental validation method

AT-E4-07 as specified in `phase1/02-rtt-strategy.md` §7 (lab netns, controlled RTTdiff injection D ∈ {0, 5, 20, 50, 100, 200} ms, positive control Shadowsocks-2022 reference proxy, negative control direct connection, ≥60 packets per visit, FPR fixed at 1%, ≥10 runs per setting, mean ± CI). Falsifiers (pre-registered): R1 fails if at every delay value either per-visit TPR stays >10% at 1% FPR or the delay needed to reach ≤10% pushes rho < 0.90 clean or p99 beyond the Phase 1 target; R2 fails if per-visit TPR stays >10% at every multiplexing factor (1/4/16/64); R3 fails as a strategy if <80% of owner destination bytes are within 5 ms of every candidate egress. AT-E4-07 runs at M2; the decision is recorded at M4.

## 12. Final decision + evidence

**Decision (PROPOSED): R5 — Accepted limitation, with bounded falsifiable research track (R1/R2/R3)**

D1 = R5 (accepted limitation) for the Phase 2 MVP, with the bounded research track in `phase1/02-rtt-strategy.md` §5. Evidence: Xue et al. NDSS 2025 operating point (`02` cross-cutting §3; `05:§5.5`); the paper's own evaluation finding delayed ACK and multiplexing limited (§2.3 of 02, sources fetched 2026-11-25); the absence of any construction for R4; the mechanism argument against decoy timing and migration (§3.4–§3.5 of 02). **F1 NO-GO condition (binding): if the owner requires RTT resistance as a release criterion, Phase 2 does not start** — the verdict flips to NO-GO for Phase 2 at the M4 gate (`03:§1.4`; `09:§4.1`), with a reduced-scope re-proposal. That decision is the owner's; this record gives him the evidence to make it.

**F1 ANSWERED 2026-11-26 00:06 — NO, by owner delegation.** The owner said «شروع کن خودت بهترین تصمیم بگیر» (start, make the best decision yourself). The decision taken on that delegation: **RTT resistance is NOT a release criterion for the Phase 2 MVP.** Rationale: no known solution exists (the attack paper itself found the mitigations limited; R4 has no construction); requiring it would block Phase 2 indefinitely — that is exactly what the NO-GO condition says; the MVP is scoped to environments where the cross-layer RTT classifier is not verified as deployed. The accepted limitation (R5) stands, the bounded research track (R1/R2/R3) continues and can reopen the question with evidence, and the M4 revisit stands: if evidence ever shows RTT resistance is achievable, or the owner later requires it, D1 reopens and the MVP scope is re-proposed. This is a delegated decision, recorded as such — it is the owner's word by delegation, not the owner's direct statement. Phase 2 go-ahead was given in the same delegation; see `phase2/00-kickoff.md`.

**Calculation that produced it:** R5 scores S = 0.30·4 + 0.20·4 + 0.20·4 + 0.15·4 + 0.15·4 = 4.00, the highest of the eight alternatives (next: R3 2.50). The ranking is not close (margin 37.5%), but the pick is driven by evidence, not by the score: R1/R2 are evaluated-and-limited by the attack paper itself, R4 has no construction, decoy timing/migration fail by mechanism, and 'do nothing' fails the hard filter. The score records the judgment; the evidence in §12 is what justifies it.

**Confidence:** Opinion, HIGH (85%) that R5 is the correct MVP position — the paper's own evaluation of R1/R2 is the load-bearing evidence (MEDIUM confidence in exact wording, HIGH in direction, two independent summaries agreeing). UNKNOWN whether R1/R2/R3 can ever reopen D1 — that is what the §5 research track measures. Calibration p = 0.85 for 'R5 is the correct MVP position'; p is logged in docs/calibration.md.

**Status / date / deciders:** ACCEPTED — 2026-11-26 — F1 decided by owner delegation («شروع کن خودت بهترین تصمیم بگیر», 2026-11-26 00:06): RTT resistance is NOT a release criterion for the Phase 2 MVP. The R5 position itself was PROPOSED 2026-11-25; the F1 answer (the gate) is decided. M2/M3 evidence (AT-E4-07 + R1/R2/R3 falsifiers) still measures the research track; the M4 revisit stands.

**Related items:** U2, R1, C5, C6, C15; AT-E4-07, AT-E1-02 (secondary E1 check on shaped traffic), AT-ZEN-RT-01; M0 (written), M2 (measured), M4 (decided); P1; D16 (E4 p99 target follows from D1).
