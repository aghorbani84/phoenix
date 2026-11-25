# Decision Record 0006 — hybrid-kem-placement

- **Decision:** 0006 (D06 of 12 §6)
- **Status:** PROPOSED
- **Date:** 2026-11-25
- **Version:** V1.0
- **Owner:** Abolfazl (decides; owner of the phase-gate go-ahead); Phase 1 architecture work (this record); implementation owner: Phase 2 (pending owner go-ahead)
- **Source:** phase0/05-protocol-design-draft.md §7 Q4; phase1/01-architecture-selection.md §4.2; phase0/03-feasibility-assessment.md §4.2–§4.4; phase0/steps-1-4-assessment.md §4.3 D5

## 1. Problem definition

Hybrid-KEM placement (Gap 3): fragment/pad the 1,216 B hybrid share to the cover's size distribution, or defer the KEM inside the encrypted channel — decided per transport, per profile. The pick does not change the KEM problem — it splits it per transport, which is the hard part. The 1,216 B hybrid X25519MLKEM768 share is a size fingerprint on both transports, but in different places with different mitigations.

## 2. Known facts

- Hybrid share 1,216 B = 1,184 B ML-KEM-768 ek + 32 B X25519 (FIPS 203 Table 3, VERIFIED). ~2.3 KB of PQC material per handshake (1,184 B ek one way + 1,088 B ct the other).
- On B the share sits in the ClientHello key_share, where fragment/pad must match a real browser's ClientHello-size distribution. On A it sits in the QUIC Initial's CRYPTO frames, where fragment/pad must match a real HTTP/3 client's Initial-size distribution — and the Initial is decryptable by any observer (Initial keys derive from a public salt), so the size signal is visible before any encryption. [Fact, HIGH — 01 §4.2.]
- The hybrid ClientHello is multi-segment by construction in a way that constrains packetisation (`steps:§4.3` D5).
- ML-KEM-768 handshake latency in ms is unmeasured (U10); Theory (75%) that the round trip dominates.

## 3. Assumptions

- Fragmentation/padding rules that match a real browser's ClientHello-size distribution (B) and a real HTTP/3 client's Initial-size distribution (A) can be specified byte-exactly in D2. [MEDIUM — the distributions themselves must be measured from the cover corpus; no such distribution exists yet.]
- A deferred KEM leaves a classically-only first session; the harvest-now window for that session is bounded and stated (one session's worth of recorded traffic, protected by the classical handshake until the KEM completes inside the channel). [MEDIUM — the bound is a design statement, not a measurement.]

## 4. Unknown variables

- Whether the 1,216 B share is fragmentable/paddable to match a real browser's ClientHello-size distribution (Q4 / `03:§11.4` item 3) — AT-E1-06 at M3 is the test.
- The real-browser ClientHello-size and HTTP/3 Initial-size distributions themselves — no corpus exists yet (D19).
- Whether the hybrid ClientHello's multi-segment construction constrains packetisation in a way fragment/pad cannot repair (D5).

## 5. Constraints

- Hard filter — security: no pure-ML-KEM handshake path; the KEM must be pluggable (D2); standard algorithms only.
- The placement is decided per transport, not once globally (D6, `05:§7` Q4). A global placement that ignores the Initial-decryptability asymmetry is out.
- D2 must carry the fragmentation/padding rule byte-exactly for each transport; a class missing from D2 is an automatic E2 failure (`03:§7.3`).

## 6. Mathematical / computational model

Per-transport size arithmetic: B — the 1,216 B share must fit a ClientHello whose size distribution is that of real browsers (multi-segment ClientHello constrains packetisation, D5). A — the 1,216 B share must fit a QUIC Initial whose CRYPTO frames are decryptable by any observer, so the size signal is visible before encryption; padding an Initial is constrained by the Initial packet-size limits and by the fact that the observer can read the CRYPTO length fields. Deferred KEM removes the fingerprint at the cost of a classically-only first session (harvest-now window = that session). No closed-form model; the falsifier is AT-E1-06 (≥1,000 handshake captures, KS p > 0.05 vs cover, with PQC present) per transport.

## 7. Alternatives (≥3 when consequential, incl. "do nothing")

| # | Alternative | Pros | Cons |
|---|-------------|------|------|
| 1 | Do nothing — no placement decided; each transport does what its library defaults to | Zero work now. | The 1,216 B share ships unmitigated on both transports — a size fingerprint on every new connection (Gap 3 failure by default). Fails the P6 obligation. |
| 2 | In-handshake fragment/pad on both transports (non-Iran profiles; B first) | PQC protection from the first handshake; no harvest-now window; one placement rule for both transports. | Unproven: no evidence fragment/pad matches either cover distribution; on A the Initial is decryptable, so the size signal is visible before encryption and padding must survive a reader who can see the CRYPTO lengths; multi-segment ClientHello constrains packetisation (D5). |
| 3 | Deferred KEM on both transports (classical outer handshake; ML-KEM-768 inside the encrypted channel) | Removes the fingerprint entirely (no PQC material in the handshake); simplest to specify in D2; same rule both transports. | Classically-only first session: a harvest-now window for that session, bounded but real; if the inner KEM never completes (connection dies early), the session stays classical — the fallback behaviour must be specified (fail closed or stay classical?). |
| 4 | Per-transport split: deferred KEM on A (Initial decryptable), fragment/pad on B (ClientHello) — Iran profile deferred on both (PICK) | Matches the mitigation to the transport's actual exposure: A cannot hide the share (decryptable Initial), so it defers; B can plausibly pad a ClientHello, so it tries fragment/pad with AT-E1-06 as the judge; Iran defers on both until the fingerprint cost is measured (D2). | Two placement rules to specify, implement, and test; the B fragment/pad rule is still unproven until AT-E1-06 runs; the deferred-KEM harvest-now window remains on A and on the Iran profile. |
| 5 | No hybrid KEM at all (classical only, everywhere, permanently) | No fingerprint cost anywhere; simplest D2. | Contradicts D2 (pluggable KEM, hybrid opt-in) and forfeits the longevity justification for China/Russia; the KEM is not pluggable, so adding PQC later is a redesign. Rejected as a placement, not as a profile choice — classical-only is a valid profile setting, not a protocol decision. |

**Hard filters applied first** (license-compatible; MSRV set by Phase 1 and recorded; security: no known unpatched CVE, no custom crypto, audited libraries only — `03:§12.1`). A candidate failing any one is out, with no scoring. No candidate fails a license/MSRV/CVE/custom-crypto filter (all use X25519MLKEM768, a standard hybrid). 'No hybrid KEM at all' is not filter-failed but contradicts the D2 posture decision (pluggable KEM) and is scored as a placement, not adopted.

**Scoring.** Weights: correctness/safety .30, maintainability .20, performance .20, reversibility .15, effort .15; S = Σ(w·s); each criterion 0–5 (5 = best). Scores are **Opinion** unless a cell is labelled VERIFIED arithmetic or a cited Fact; the basis of each score is stated in the Pros/Cons above. Computed in Python (this generator):

| Alternative | Corr .30 | Maint .20 | Perf .20 | Rev .15 | Eff .15 | **S** |
|---|---:|---:|---:|---:|---:|---:|
| Do nothing — no placement decided; each transport does what its library defaults to | 1 | 1 | 3 | 5 | 5 | **2.60** |
| In-handshake fragment/pad on both transports (non-Iran profiles; B first) | 3 | 3 | 3 | 3 | 3 | **3.00** |
| Deferred KEM on both transports (classical outer handshake; ML-KEM-768 inside the encrypted channel) | 3 | 4 | 3 | 4 | 3 | **3.35** |
| Per-transport split: deferred KEM on A (Initial decryptable), fragment/pad on B (ClientHello) — Iran profile deferred on both (PICK) | 4 | 3 | 3 | 4 | 3 | **3.45** |
| No hybrid KEM at all (classical only, everywhere, permanently) | 2 | 4 | 4 | 2 | 4 | **3.10** |

**Ranking:** 3.45 Per-transport split: deferred KEM on A (Initial decryptable), fragment/pad on B (ClientHello) — Iran profile deferred on both (PICK) > 3.35 Deferred KEM on both transports (classical outer handshake; ML-KEM-768 inside the encrypted channel) > 3.10 No hybrid KEM at all (classical only, everywhere, permanently) > 3.00 In-handshake fragment/pad on both transports (non-Iran profiles; B first) > 2.60 Do nothing — no placement decided; each transport does what its library defaults to. Gap between top two: 0.10 points = 2.90% of the leader's score.

**Sensitivity (±20% per weight, other weights renormalised to sum 1.0):**

| Weight varied | −20% → S | +20% → S | Leader changes? |
|---|---|---|---|
| Correctness/safety | Do nothing — no placement decided; each transport does what its library defaults to: 2.7021 / In-handshake fragment/pad on both transports (non-Iran profiles; B first): 3.0000 / Deferred KEM on both transports (classical outer handshake; ML-KEM-768 inside the encrypted channel): 3.3723 / Per-transport split: deferred KEM on A (Initial decryptable), fragment/pad on B (ClientHello) — Iran profile deferred on both (PICK): 3.4149 / No hybrid KEM at all (classical only, everywhere, permanently): 3.1702 | Do nothing — no placement decided; each transport does what its library defaults to: 2.5094 / In-handshake fragment/pad on both transports (non-Iran profiles; B first): 3.0000 / Deferred KEM on both transports (classical outer handshake; ML-KEM-768 inside the encrypted channel): 3.3302 / Per-transport split: deferred KEM on A (Initial decryptable), fragment/pad on B (ClientHello) — Iran profile deferred on both (PICK): 3.4811 / No hybrid KEM at all (classical only, everywhere, permanently): 3.0377 | No |
| Maintainability | Do nothing — no placement decided; each transport does what its library defaults to: 2.6667 / In-handshake fragment/pad on both transports (non-Iran profiles; B first): 3.0000 / Deferred KEM on both transports (classical outer handshake; ML-KEM-768 inside the encrypted channel): 3.3229 / Per-transport split: deferred KEM on A (Initial decryptable), fragment/pad on B (ClientHello) — Iran profile deferred on both (PICK): 3.4688 / No hybrid KEM at all (classical only, everywhere, permanently): 3.0625 | Do nothing — no placement decided; each transport does what its library defaults to: 2.5385 / In-handshake fragment/pad on both transports (non-Iran profiles; B first): 3.0000 / Deferred KEM on both transports (classical outer handshake; ML-KEM-768 inside the encrypted channel): 3.3750 / Per-transport split: deferred KEM on A (Initial decryptable), fragment/pad on B (ClientHello) — Iran profile deferred on both (PICK): 3.4327 / No hybrid KEM at all (classical only, everywhere, permanently): 3.1346 | No |
| Performance | Do nothing — no placement decided; each transport does what its library defaults to: 2.5833 / In-handshake fragment/pad on both transports (non-Iran profiles; B first): 3.0000 / Deferred KEM on both transports (classical outer handshake; ML-KEM-768 inside the encrypted channel): 3.3646 / Per-transport split: deferred KEM on A (Initial decryptable), fragment/pad on B (ClientHello) — Iran profile deferred on both (PICK): 3.4688 / No hybrid KEM at all (classical only, everywhere, permanently): 3.0625 | Do nothing — no placement decided; each transport does what its library defaults to: 2.6154 / In-handshake fragment/pad on both transports (non-Iran profiles; B first): 3.0000 / Deferred KEM on both transports (classical outer handshake; ML-KEM-768 inside the encrypted channel): 3.3365 / Per-transport split: deferred KEM on A (Initial decryptable), fragment/pad on B (ClientHello) — Iran profile deferred on both (PICK): 3.4327 / No hybrid KEM at all (classical only, everywhere, permanently): 3.1346 | No |
| Reversibility | Do nothing — no placement decided; each transport does what its library defaults to: 2.5258 / In-handshake fragment/pad on both transports (non-Iran profiles; B first): 3.0000 / Deferred KEM on both transports (classical outer handshake; ML-KEM-768 inside the encrypted channel): 3.3299 / Per-transport split: deferred KEM on A (Initial decryptable), fragment/pad on B (ClientHello) — Iran profile deferred on both (PICK): 3.4330 / No hybrid KEM at all (classical only, everywhere, permanently): 3.1340 | Do nothing — no placement decided; each transport does what its library defaults to: 2.6699 / In-handshake fragment/pad on both transports (non-Iran profiles; B first): 3.0000 / Deferred KEM on both transports (classical outer handshake; ML-KEM-768 inside the encrypted channel): 3.3689 / Per-transport split: deferred KEM on A (Initial decryptable), fragment/pad on B (ClientHello) — Iran profile deferred on both (PICK): 3.4660 / No hybrid KEM at all (classical only, everywhere, permanently): 3.0680 | No |
| Effort | Do nothing — no placement decided; each transport does what its library defaults to: 2.5258 / In-handshake fragment/pad on both transports (non-Iran profiles; B first): 3.0000 / Deferred KEM on both transports (classical outer handshake; ML-KEM-768 inside the encrypted channel): 3.3608 / Per-transport split: deferred KEM on A (Initial decryptable), fragment/pad on B (ClientHello) — Iran profile deferred on both (PICK): 3.4639 / No hybrid KEM at all (classical only, everywhere, permanently): 3.0722 | Do nothing — no placement decided; each transport does what its library defaults to: 2.6699 / In-handshake fragment/pad on both transports (non-Iran profiles; B first): 3.0000 / Deferred KEM on both transports (classical outer handshake; ML-KEM-768 inside the encrypted channel): 3.3398 / Per-transport split: deferred KEM on A (Initial decryptable), fragment/pad on B (ClientHello) — Iran profile deferred on both (PICK): 3.4369 / No hybrid KEM at all (classical only, everywhere, permanently): 3.1262 | No |

**Tipping weight:** none — no single ±20% weight variation changes the leader. The ranking is robust to weight choice (it may still be fragile to score choice; see §12).

**Close-call handling (<10% margin):** the margin is 2.90%, inside the <10% band, so the refined-spec rule fires: a ≤30-minute benchmark breaks the tie; if still tied, the more reversible option wins (`03:§12.1`). See §12 for the tie-break evidence.

## 8. Expected benefits

Per-transport split + Iran deferral: the fingerprint is removed where it cannot be hidden (A), attempted where it plausibly can be (B, judged by AT-E1-06), and not paid at all where the benefit is unverified (Iran). The harvest-now window is bounded, stated, and confined to the deferred paths.

## 9. Failure modes

Fragment/pad on B fails AT-E1-06 (KS p ≤ 0.05 with PQC present, and neither fragment/pad nor deferred KEM repairs it) → the B placement falls back to deferred KEM by ADR, and B correctness is revised (D5 F3). Deferred KEM's inner handshake is interrupted → the session stays classical; the specified behaviour (fail closed vs stay classical) must be in D2 before M1, or the failure mode is unspecified. A middlebox drops the 1,216 B ClientHello outright → fragment/pad cannot help; deferred KEM is the fallback.

**Consequence if wrong:** If fragment/pad on B is wrong (it cannot match the real-browser ClientHello distribution), the cost is a detectable 1,216 B size fingerprint on every B handshake in every hybrid profile until AT-E1-06 catches it at M3 — bounded by the M3 falsifier and the deferred-KEM fallback. If deferred KEM on A is wrong (the harvest-now window matters more than the fingerprint), the cost is recorded classical-only sessions in the A profile — a long-horizon loss, stated in advance.

**Reversibility cost of undoing it:** High: placement is a per-transport, per-profile rule in D2. Switching B from fragment/pad to deferred KEM costs an ADR and a D2 edit; the KEM itself is pluggable, so no protocol redesign is needed.

## 10. Cost / complexity

Specification cost: two placement rules in D2 (fragment/pad rule for B with its padding function; deferred-KEM rule for A and Iran). Implementation cost: fragment/pad on B (padding logic + distribution matching); deferred KEM (inner handshake after the outer channel is established). Test cost: AT-E1-06 per transport, per placement.

## 11. Experimental validation method

AT-E1-06 at M3 (≥1,000 handshake captures per transport, KS p > 0.05 on handshake-size distribution vs cover, with PQC present; recommended ≥10,000 handshakes, D_crit ≈ 0.019 — `phase1/05-threshold-validation-plan.md` §3.3). Falsifier: AT-E1-06 fails for a transport → that transport's placement is wrong (fragment/pad does not repair it, or deferred KEM was not actually deferred); the placement is revised by ADR. AT-E4-06 at M4 (ML-KEM-768 handshake latency delta with/without the hybrid share, reported).

## 12. Final decision + evidence

**Decision (PROPOSED): Per-transport split: deferred KEM on A (Initial decryptable), fragment/pad on B (ClientHello) — Iran profile deferred on both**

PROPOSED: per-transport placement — deferred KEM on A (the QUIC Initial is decryptable, so the share cannot be hidden there); fragment/pad on B (ClientHello key_share, matched to a real browser's ClientHello-size distribution, judged by AT-E1-06); deferred KEM on both transports for the Iran profile until the fingerprint cost is measured. The KEM remains pluggable (D2). If AT-E1-06 fails for B, B falls back to deferred KEM by ADR.

**Calculation that produced it:** S(split) = 0.30·4 + 0.20·3 + 0.20·3 + 0.15·4 + 0.15·3 = 3.45; S(fragment/pad both) = 3.00; S(deferred both) = 3.35; S(no hybrid) = 3.05. Margin over deferred-both is 0.10 points = 2.9% — a close call, so the close-call rule fires and the tie-break is the evidence: the split matches the mitigation to each transport's measured exposure (Initial decryptability on A), which deferred-both does not, and it preserves the B fragment/pad experiment that AT-E1-06 can judge. If still tied on evidence, the more reversible option wins — both are equally reversible, so the split stands on the exposure argument, recorded here.

**Confidence:** Opinion, MEDIUM (65%) — the split is the best available judgment on an unmeasured question (Q4): the A-side reasoning (decryptable Initial) is a Fact about QUIC, but whether fragment/pad works on B is UNKNOWN until AT-E1-06 runs at M3. Calibration p = 0.65.

**Status / date / deciders:** PROPOSED — 2026-11-25 — decided by Abolfazl (owner) at the Phase 1 gate; this record proposes, it does not accept. A decision without its M2/M3 evidence is PROPOSED, not ACCEPTED. Decided at: M0 (decision) → M3 (falsifier AT-E1-06). Evidence that would settle it: AT-E1-06 per transport at M3; the real-browser ClientHello/Initial size distributions from the corpus (D19).

**Related items:** U6, U10, R3, C8, C10, C11, C12, C13; AT-E1-06, AT-E4-06; M0, M3, M4; P6; D2, D5, D19.
