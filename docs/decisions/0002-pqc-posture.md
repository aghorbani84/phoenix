# Decision Record 0002 — pqc-posture

- **Decision:** 0002 (D02 of 12 §6)
- **Status:** PROPOSED
- **Date:** 2026-11-25
- **Version:** V1.0
- **Owner:** Abolfazl (decides; owner of the phase-gate go-ahead); Phase 1 architecture work (this record); implementation owner: Phase 2 (pending owner go-ahead)
- **Source:** phase0/03-feasibility-assessment.md §4.2–§4.4, §11.3 decision 2; phase1/03-iran-profile.md §3.7; phase1/01-architecture-selection.md §4.2; FIPS 203 Table 3 (ML-KEM-768 ek 1,184 / dk 2,400 / ct 1,088 / ss 32 — VERIFIED)

## 1. Problem definition

Post-quantum key exchange must be positioned per environment: hybrid (X25519MLKEM768) as default vs classical default with hybrid opt-in per profile, and whether the Iran profile defers PQC. The decision is needed now because D2's handshake fields, the KEM's pluggability, and the 1,216 B hybrid-share fingerprint all depend on it, and the fingerprint cost is paid on every handshake from day one.

## 2. Known facts

- ML-KEM-768 sizes VERIFIED against FIPS 203 Table 3: ek 1,184 B, dk 2,400 B, ct 1,088 B, ss 32 B. Hybrid share X25519MLKEM768 = 1,184 + 32 = 1,216 B — ~38× the classical key-share field (mathematical truth). ~2.3 KB of PQC material per handshake (1,184 B ek one way + 1,088 B ct the other).
- Quantum gives no obfuscation advantage; hybrid — not pure-PQC — is the default; no SLH-DSA on the handshake path; ML-DSA only for long-term server authentication, if at all; standard algorithms only, no custom crypto (`03:§4.4`, binding).
- RFC 9954 (Hybrid Key Exchange in TLS 1.3, July 2026, Informational) and RFC 10024 (PQ/T Hybrid Key Agreement Mechanisms for TLS 1.3, August 2026, Standards Track) are the final published numbers — verified against the RFC Editor (fix-log R1–R5, C2/C3). RFC 9846 (July 2026) obsoletes RFC 8446 as the TLS 1.3 specification.
- Per-environment justification (`03:§4.3`): China — PQC justified as longevity, not stealth (Opinion 85%); Iran — weakly, deferred (Opinion 70%); Russia — as longevity (Opinion 75%); Turkmenistan/Kazakhstan/Belarus — no, not on current evidence (Opinion 75%).
- Iran profile (`phase1/03-iran-profile.md` §3.7): the verified Iran capabilities are classical; harvest-now-decrypt-later by Iran is not verified; the immediate costs (1,216 B hybrid share fingerprint, middlebox drops) are paid now for a speculative long-horizon benefit.

## 3. Assumptions

- No quantum adversary exists today; PQC protects recorded traffic against a future quantum adversary, not a single packet against today's DPI. [HIGH — `03:§4.3`.]
- The KEM can be designed as pluggable (classical default, hybrid opt-in per profile) without hard-coding it per transport. [MEDIUM — design claim, unimplemented.]

## 4. Unknown variables

- ML-KEM-768 handshake latency in ms — no measured figure found (U10); retired by AT-E4-06 at M4. Theory (75%) that the round trip dominates.
- Whether the 1,216 B share is fragmentable/paddable to match a real browser's ClientHello-size distribution (D6, AT-E1-06 at M3).
- Whether any censor harvests traffic for future quantum decryption — not verified for any environment in the threat model.

## 5. Constraints

- Hard filter — security: no pure-ML-KEM handshake path; no SLH-DSA on the handshake path; no custom crypto; standard algorithms only (ML-KEM-768 per FIPS 203). A candidate using pure PQC or custom crypto is out.
- Hard filter — license/MSRV: the KEM implementation must come from an audited library (RustCrypto / pqcrypto family — library selection is a Phase 1 decision, not a Phase 0 claim) with no known unpatched CVE.
- The hash research track never overrides the no-custom-crypto rule; no experimental construction from that track is used in PHOENIX Core.

## 6. Mathematical / computational model

Fingerprint arithmetic: hybrid share 1,216 B vs classical 32 B key share = 38×. Per-handshake PQC material ≈ 2,272 B (1,184 ek + 1,088 ct). The trade-off is a size/timing signature paid on every handshake (present cost, certain) against harvest-now-decrypt-later protection (future benefit, speculative, long-horizon). No closed-form model of the fingerprint-vs-longevity trade-off exists; the falsifier is AT-E1-06 (handshake-size KS) at M3 and AT-E4-06 (latency delta) at M4.

## 7. Alternatives (≥3 when consequential, incl. "do nothing")

| # | Alternative | Pros | Cons |
|---|-------------|------|------|
| 1 | Do nothing — no PQC posture decided; defer to implementation | Zero work now. | D2 cannot be written (handshake fields carry the KEM material); the 1,216 B share ships by accident or not at all; the Iran profile has no deferral to rely on. Fails the P6 obligation. |
| 2 | Hybrid default everywhere (X25519MLKEM768 on every handshake, every profile) | Maximum harvest-now protection from day one; single code path; simplest to reason about cryptographically. | Pays the 1,216 B fingerprint cost on every handshake in every environment, including Iran where the benefit is unverified and the cost is paid now; the fingerprint is a size/timing signature by construction; least reversible once deployed (clients must all support it). |
| 3 | Classical default, hybrid opt-in per profile; KEM pluggable (PICK) | Pays the fingerprint cost only where the longevity justification holds (China/Russia); Iran defers until AT-E1-06 measures the cost; pluggable KEM means the posture can change per profile without a protocol redesign; most reversible. | Two code paths to maintain and test; a profile misconfiguration could ship hybrid where it was not intended (mitigated: hybrid is opt-in, never default-on by accident); the first classical-only session has a harvest-now window (bounded and stated, see D6). |
| 4 | Pure ML-KEM handshake (no classical component) | Cleanest post-quantum story; no classical component to harvest. | FAILS the hard security filter (`03:§4.4`: hybrid, not pure-PQC, is the default). Listed in the table but not scored; not a candidate. |
| 5 | No PQC at all (classical only, KEM not pluggable) | Smallest handshake, no fingerprint cost, simplest implementation. | No crypto-agility: adding PQC later is a protocol redesign, not a configuration change; forfeits the longevity justification for China/Russia where harvest-now is plausible; the KEM is hard-coded per transport, which D6 forbids. |

**Hard filters applied first** (license-compatible; MSRV set by Phase 1 and recorded; security: no known unpatched CVE, no custom crypto, audited libraries only — `03:§12.1`). A candidate failing any one is out, with no scoring. Pure ML-KEM fails the hard security filter (`03:§4.4` — hybrid, not pure-PQC) and is out, no scoring counted. All other candidates use standard vetted algorithms only and pass the license/MSRV/CVE filters at the posture level (library selection is a separate Phase 1 decision).

**Scoring.** Weights: correctness/safety .30, maintainability .20, performance .20, reversibility .15, effort .15; S = Σ(w·s); each criterion 0–5 (5 = best). Scores are **Opinion** unless a cell is labelled VERIFIED arithmetic or a cited Fact; the basis of each score is stated in the Pros/Cons above. Computed in Python (this generator):

| Alternative | Corr .30 | Maint .20 | Perf .20 | Rev .15 | Eff .15 | **S** |
|---|---:|---:|---:|---:|---:|---:|
| Do nothing — no PQC posture decided; defer to implementation | 1 | 1 | 3 | 5 | 5 | **2.60** |
| Hybrid default everywhere (X25519MLKEM768 on every handshake, every profile) | 3 | 3 | 3 | 2 | 3 | **2.85** |
| Classical default, hybrid opt-in per profile; KEM pluggable (PICK) | 4 | 4 | 3 | 4 | 3 | **3.65** |
| Pure ML-KEM handshake (no classical component) | — | — | — | — | — | **EXCLUDED by hard filter — not scored** |
| No PQC at all (classical only, KEM not pluggable) | 2 | 4 | 4 | 2 | 4 | **3.10** |

**Ranking:** 3.65 Classical default, hybrid opt-in per profile; KEM pluggable (PICK) > 3.10 No PQC at all (classical only, KEM not pluggable) > 2.85 Hybrid default everywhere (X25519MLKEM768 on every handshake, every profile) > 2.60 Do nothing — no PQC posture decided; defer to implementation. Gap between top two: 0.55 points = 15.07% of the leader's score.

**Sensitivity (±20% per weight, other weights renormalised to sum 1.0):**

| Weight varied | −20% → S | +20% → S | Leader changes? |
|---|---|---|---|
| Correctness/safety | Do nothing — no PQC posture decided; defer to implementation: 2.7021 / Hybrid default everywhere (X25519MLKEM768 on every handshake, every profile): 2.8404 / Classical default, hybrid opt-in per profile; KEM pluggable (PICK): 3.6277 / No PQC at all (classical only, KEM not pluggable): 3.1702 | Do nothing — no PQC posture decided; defer to implementation: 2.5094 / Hybrid default everywhere (X25519MLKEM768 on every handshake, every profile): 2.8585 / Classical default, hybrid opt-in per profile; KEM pluggable (PICK): 3.6698 / No PQC at all (classical only, KEM not pluggable): 3.0377 | No |
| Maintainability | Do nothing — no PQC posture decided; defer to implementation: 2.6667 / Hybrid default everywhere (X25519MLKEM768 on every handshake, every profile): 2.8438 / Classical default, hybrid opt-in per profile; KEM pluggable (PICK): 3.6354 / No PQC at all (classical only, KEM not pluggable): 3.0625 | Do nothing — no PQC posture decided; defer to implementation: 2.5385 / Hybrid default everywhere (X25519MLKEM768 on every handshake, every profile): 2.8558 / Classical default, hybrid opt-in per profile; KEM pluggable (PICK): 3.6635 / No PQC at all (classical only, KEM not pluggable): 3.1346 | No |
| Performance | Do nothing — no PQC posture decided; defer to implementation: 2.5833 / Hybrid default everywhere (X25519MLKEM768 on every handshake, every profile): 2.8438 / Classical default, hybrid opt-in per profile; KEM pluggable (PICK): 3.6771 / No PQC at all (classical only, KEM not pluggable): 3.0625 | Do nothing — no PQC posture decided; defer to implementation: 2.6154 / Hybrid default everywhere (X25519MLKEM768 on every handshake, every profile): 2.8558 / Classical default, hybrid opt-in per profile; KEM pluggable (PICK): 3.6250 / No PQC at all (classical only, KEM not pluggable): 3.1346 | No |
| Reversibility | Do nothing — no PQC posture decided; defer to implementation: 2.5258 / Hybrid default everywhere (X25519MLKEM768 on every handshake, every profile): 2.8763 / Classical default, hybrid opt-in per profile; KEM pluggable (PICK): 3.6392 / No PQC at all (classical only, KEM not pluggable): 3.1340 | Do nothing — no PQC posture decided; defer to implementation: 2.6699 / Hybrid default everywhere (X25519MLKEM768 on every handshake, every profile): 2.8252 / Classical default, hybrid opt-in per profile; KEM pluggable (PICK): 3.6602 / No PQC at all (classical only, KEM not pluggable): 3.0680 | No |
| Effort | Do nothing — no PQC posture decided; defer to implementation: 2.5258 / Hybrid default everywhere (X25519MLKEM768 on every handshake, every profile): 2.8454 / Classical default, hybrid opt-in per profile; KEM pluggable (PICK): 3.6701 / No PQC at all (classical only, KEM not pluggable): 3.0722 | Do nothing — no PQC posture decided; defer to implementation: 2.6699 / Hybrid default everywhere (X25519MLKEM768 on every handshake, every profile): 2.8544 / Classical default, hybrid opt-in per profile; KEM pluggable (PICK): 3.6311 / No PQC at all (classical only, KEM not pluggable): 3.1262 | No |

**Tipping weight:** none — no single ±20% weight variation changes the leader. The ranking is robust to weight choice (it may still be fragile to score choice; see §12).

**Close-call handling (<10% margin):** the margin is 15.07%, outside the <10% band, so no benchmark is triggered by the close-call rule. See §12 for the tie-break evidence.

## 8. Expected benefits

Classical default + hybrid opt-in + pluggable KEM: the Iran profile can defer PQC (its verified capabilities are classical); China/Russia profiles can opt in for longevity; the posture is a configuration choice per profile, not a protocol fork; AT-E1-06 at M3 measures the fingerprint cost per transport before any profile opts in.

## 9. Failure modes

A profile opts into hybrid before AT-E1-06 measures the fingerprint → the 1,216 B share is detectable by size on every handshake in that profile (Gap 3). Mitigation: opt-in is gated on the M3 measurement, not on preference. A harvest-now adversary records classical-only sessions → the stated, bounded window for the first session under deferred KEM (D6). A library CVE in the chosen KEM implementation → the pluggable design lets the KEM be swapped without a protocol redesign.

**Consequence if wrong:** If the posture is wrong in the 'too little PQC' direction, recorded traffic from classical-only profiles is harvestable for future quantum decryption — a long-horizon, unmeasurable-today loss. If wrong in the 'too much PQC' direction, every handshake in Iran carries a 1,216 B size fingerprint that a classifier can use today — a present, measurable loss. The asymmetry (present-certain cost vs future-speculative benefit) is why the pick defers in Iran and opts in only where longevity is justified.

**Reversibility cost of undoing it:** High: the KEM is pluggable, so changing a profile's posture is a configuration change plus a client update, not a protocol redesign. Reversing a hybrid-default deployment would cost a handshake-format change; reversing an opt-in costs a profile edit.

## 10. Cost / complexity

Design cost: the pluggable-KEM abstraction in D2/M1 (one abstraction, used by both transports). Runtime cost: classical by default (no PQC cost in Iran); hybrid opt-in pays ~2.3 KB/handshake plus the ML-KEM-768 encapsulation cost (latency unmeasured — AT-E4-06 at M4).

## 11. Experimental validation method

AT-E1-06 at M3 (≥1,000 handshake captures, KS p > 0.05 on handshake-size distribution vs cover, with PQC present — per transport, per profile). AT-E4-06 at M4 (ML-KEM-768 handshake latency delta with/without the hybrid share, reported). Falsifier: either test fails → the PQC integration (fragment/pad to match cover, or deferred KEM) is wrong (`steps:§3.2` U6) and the profile's opt-in is withdrawn. E6: KATs for every primitive (ML-KEM-768 vs NIST ACVP vectors), cargo audit + deny clean on the release SHA (AT-E6-01/02/04/05/06).

## 12. Final decision + evidence

**Decision (PROPOSED): Classical default, hybrid opt-in per profile; KEM pluggable**

PROPOSED: classical default, hybrid (X25519MLKEM768) opt-in per profile, KEM designed as pluggable — not hard-coded per transport. The Iran profile defers PQC until the fingerprint cost is measured (AT-E1-06, M3). China/Russia profiles may opt in on the longevity justification. No pure-ML-KEM handshake path; no SLH-DSA on the handshake path.

**Calculation that produced it:** S = 0.30·4 + 0.20·4 + 0.20·3 + 0.15·4 + 0.15·3 = 3.65, highest of the scored alternatives (hybrid default 2.85; no-PQC 3.05; do nothing 2.45). Margin over the runner-up (no-PQC, 3.05) is 0.60 points = 16.4% — outside the close-call band. Pure ML-KEM is excluded by the hard filter before scoring.

**Confidence:** Opinion, HIGH (80%) that classical-default + per-profile hybrid opt-in is the correct posture — the per-environment justification table (`03:§4.3`) is Opinion 70–85% per row and the fingerprint arithmetic is VERIFIED (FIPS 203 sizes). MEDIUM that the Iran deferral survives M3 measurement (AT-E1-06 is the test). Calibration p = 0.80.

**Status / date / deciders:** PROPOSED — 2026-11-25 — decided by Abolfazl (owner) at the Phase 1 gate; this record proposes, it does not accept. A decision without its M2/M3 evidence is PROPOSED, not ACCEPTED. Decided at: M0. Evidence that would settle it: AT-E1-06 at M3 (fingerprint cost per transport/profile); AT-E4-06 at M4 (latency delta).

**Related items:** U6, U10, R3, C8, C10, C11, C12, C13; AT-E1-06, AT-E4-06, AT-E6-01/02/04/05/06; M0 (decision), M1 (implementation), M3/M4 (falsifiers); P6; D6 (placement per transport), D3 (cover choice).
