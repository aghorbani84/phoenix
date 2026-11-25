# Decision Record 0003 — cover-fallback

- **Decision:** 0003 (D03 of 12 §6)
- **Status:** PROPOSED
- **Date:** 2026-11-25
- **Version:** V1.0
- **Owner:** Abolfazl (decides; owner of the phase-gate go-ahead); Phase 1 architecture work (this record); implementation owner: Phase 2 (pending owner go-ahead)
- **Source:** phase0/03-feasibility-assessment.md §11.3 decision 3; phase0/01-threat-models.md §4 (Iran); phase0/02-competitive-analysis.md §15 Gap 5; phase1/03-iran-profile.md §3.1; phase1/01-architecture-selection.md §3–§4

## 1. Problem definition

Which cover does PHOENIX present, and in what fallback order: real TLS 1.3 (rustls) as default cover vs real QUIC (quinn) vs a non-mimicry exemption route — and the fallback order when the primary cover fails (Gap 5). The decision is needed now because D2's wire format, the session layer, and the Iran profile's transport order all consume it.

## 2. Known facts

- Only DNS (UDP/53), HTTP (80), HTTPS (443) are forwarded in Iran; all other protocols failed silently in the June 2025 in-country measurement (Aryapour 2025 §4.4). [Fact, HIGH.]
- QUIC/HTTP-3 dropped to near zero in Iran from 22 Sept 2022 while TCP/443 + TLS continued (`01:§4`). June 2025: QUIC blocked at 100% via Initial drops and still 100% blocked after the shutdown lifted, while TCP recovered (FOCI 2026, MEDIUM — secondary summary).
- B's cover (genuine TLS 1.3 to a real site) is the best-measured cover in the set — REALITY/ShadowTLS/WebTunnel are deployed and field-reported working, and pass the Wu 2023 exemption (Ex5, TLS/HTTP fingerprint) by construction. [Fact, HIGH.]
- GFW decrypts QUIC Initial packets and extracts SNI since 7 April 2024 (Zohaib et al., USENIX Security 2025). [Fact, HIGH (95%).]
- 'Looks like nothing' is a detectable class, not a hiding strategy (`01:§1.1`); a fully-encrypted first packet fails the Wu 2023 exemption rules and is blocked en masse (Nov 2021).

## 3. Assumptions

- A genuine TLS 1.3 handshake to a real site (borrowed or own-cert + real-site server, D8) can be made byte-identical to the cover server's behaviour under E2 probing. [MEDIUM — E2 at M3 is the test.]
- The fallback order B-first with A as opportunistic standby can be implemented at the session layer without leaking a linkable identifier across the switch. [MEDIUM — D21/D22, AT-E7-05.]

## 4. Unknown variables

- Whether Vision's padding defeats the Xue 2024 classifier (U4) — unmeasured; AT-E1-03 at M3.
- Whether multiplexing defeats the Xue 2024 classifier (U5) — unmeasured; AT-E1-03 at M3.
- Whether the Aparecium ShadowTLS-v3 vectors were fixed in later releases (U7) — secondary reporting only (75%).
- Whether ECH use is observable and blockable in Iran — ECH is RFC 9849 (March 2026, Standards Track, VERIFIED); the extension is visible in ClientHelloOuter and a censor can block ECH connections outright (`00:§2.5`).

## 5. Constraints

- Hard filter — security: the cover must be a real protocol, not a mimicry of one and not 'looks like nothing' (a detectable class). A non-mimicry exemption route that presents a fully-encrypted first packet fails the Wu 2023 exemption rules and is out as a default.
- Hard filter — Iran: any cover that is not TLS-over-TCP/443 is dropped silently at the Iranian border (V5) and cannot be the Iran default.
- No silent downgrade between covers/transports, ever (`04:§2.3.8`); fallback is explicit and user-visible.

## 6. Mathematical / computational model

Cover choice is a per-environment availability question first and a fingerprint question second: availability is binary (forwarded or dropped — V5), fingerprint is statistical (Adv ≤ 0.05, E1). The fallback order is a session-layer state machine (D21/D22), not a per-connection random choice: B-first everywhere; A opportunistic standby in Iran, probed in the background and admitted only on measured health, never on assumption (`05:§2.4.4`).

## 7. Alternatives (≥3 when consequential, incl. "do nothing")

| # | Alternative | Pros | Cons |
|---|-------------|------|------|
| 1 | Do nothing — no cover chosen; each transport presents whatever it presents | Zero work now. | D2 cannot be written; E1 has no cover to measure against; the Iran profile has no transport order. Fails the M0 exit criterion. |
| 2 | Real TLS 1.3 (rustls) default; real QUIC (quinn) standby; fallback B→A→queue (PICK) | Only alternative with no UDP single point of failure in Iran; best-measured cover; B is the most reversible (4/5) and lowest-effort (4/5) alternative; A retained so the close call is measured, not assumed. | B's core mitigation (Vision padding) unmeasured (U4); B degrades with TCP under loss (magnitude disputed between labs); QUIC standby must expect failure as the default outcome in Iran. |
| 3 | Real QUIC (quinn) default; TLS 1.3 fallback | Best loss resilience in shape; lowest handshake latency; most natural RTT-shaping control (QUIC owns ACK timing); most testable state machine (quinn-proto sans-io). | UDP-only default fails outright where UDP is blocked — in Iran that is intermittent-to-total failure (100% Initial drops, persistent post-shutdown); QUIC SNI decryptable at scale since April 2024; 1,216 B share in the Initial, visible before any encryption. |
| 4 | Non-mimicry exemption route as default ('looks like nothing' / fully-encrypted first packet) | No cover to maintain; no ClientHello drift to track; simplest wire format. | FAILS the hard security filter as a default: a fully-encrypted first packet fails all five Wu 2023 exemption rules and was blocked en masse (Nov 2021); 'looks like nothing' is itself a detectable class (`01:§1.1`). Listed in the table but not scored. |
| 5 | Both covers simultaneously (multipath C) as the default | Highest performance ceiling (5/5); session-preserving transport fallback (Gap 5); transport diversity raises the censor's cost. | C is deferred, not rejected (D4/D5): highest effort, largest code surface, lowest maintainability (2/5) and reversibility (2/5); scheduler itself a fingerprint surface, unproven unclassifiable. Committing to C before A and B are measured is the least reversible choice in the matrix. |

**Hard filters applied first** (license-compatible; MSRV set by Phase 1 and recorded; security: no known unpatched CVE, no custom crypto, audited libraries only — `03:§12.1`). A candidate failing any one is out, with no scoring. The non-mimicry exemption route fails the hard security filter as a default (Wu 2023 exemption rules; 'looks like nothing' is a detectable class) and is out. Multipath C is not filter-failed but is deferred by D4/D5, not adopted as the default. All other candidates use real, vetted protocols (rustls/quinn) and pass the license/MSRV/CVE filters.

**Scoring.** Weights: correctness/safety .30, maintainability .20, performance .20, reversibility .15, effort .15; S = Σ(w·s); each criterion 0–5 (5 = best). Scores are **Opinion** unless a cell is labelled VERIFIED arithmetic or a cited Fact; the basis of each score is stated in the Pros/Cons above. Computed in Python (this generator):

| Alternative | Corr .30 | Maint .20 | Perf .20 | Rev .15 | Eff .15 | **S** |
|---|---:|---:|---:|---:|---:|---:|
| Do nothing — no cover chosen; each transport presents whatever it presents | 1 | 1 | 3 | 5 | 5 | **2.60** |
| Real TLS 1.3 (rustls) default; real QUIC (quinn) standby; fallback B→A→queue (PICK) | 4 | 3 | 3 | 4 | 4 | **3.60** |
| Real QUIC (quinn) default; TLS 1.3 fallback | 3 | 4 | 4 | 3 | 3 | **3.40** |
| Non-mimicry exemption route as default ('looks like nothing' / fully-encrypted first packet) | — | — | — | — | — | **EXCLUDED by hard filter — not scored** |
| Both covers simultaneously (multipath C) as the default | 3 | 2 | 5 | 2 | 2 | **2.90** |

**Ranking:** 3.60 Real TLS 1.3 (rustls) default; real QUIC (quinn) standby; fallback B→A→queue (PICK) > 3.40 Real QUIC (quinn) default; TLS 1.3 fallback > 2.90 Both covers simultaneously (multipath C) as the default > 2.60 Do nothing — no cover chosen; each transport presents whatever it presents. Gap between top two: 0.20 points = 5.56% of the leader's score.

**Sensitivity (±20% per weight, other weights renormalised to sum 1.0):**

| Weight varied | −20% → S | +20% → S | Leader changes? |
|---|---|---|---|
| Correctness/safety | Do nothing — no cover chosen; each transport presents whatever it presents: 2.7021 / Real TLS 1.3 (rustls) default; real QUIC (quinn) standby; fallback B→A→queue (PICK): 3.5745 / Real QUIC (quinn) default; TLS 1.3 fallback: 3.4255 / Both covers simultaneously (multipath C) as the default: 2.8936 | Do nothing — no cover chosen; each transport presents whatever it presents: 2.5094 / Real TLS 1.3 (rustls) default; real QUIC (quinn) standby; fallback B→A→queue (PICK): 3.6226 / Real QUIC (quinn) default; TLS 1.3 fallback: 3.3774 / Both covers simultaneously (multipath C) as the default: 2.9057 | No |
| Maintainability | Do nothing — no cover chosen; each transport presents whatever it presents: 2.6667 / Real TLS 1.3 (rustls) default; real QUIC (quinn) standby; fallback B→A→queue (PICK): 3.6250 / Real QUIC (quinn) default; TLS 1.3 fallback: 3.3750 / Both covers simultaneously (multipath C) as the default: 2.9375 | Do nothing — no cover chosen; each transport presents whatever it presents: 2.5385 / Real TLS 1.3 (rustls) default; real QUIC (quinn) standby; fallback B→A→queue (PICK): 3.5769 / Real QUIC (quinn) default; TLS 1.3 fallback: 3.4231 / Both covers simultaneously (multipath C) as the default: 2.8654 | No |
| Performance | Do nothing — no cover chosen; each transport presents whatever it presents: 2.5833 / Real TLS 1.3 (rustls) default; real QUIC (quinn) standby; fallback B→A→queue (PICK): 3.6250 / Real QUIC (quinn) default; TLS 1.3 fallback: 3.3750 / Both covers simultaneously (multipath C) as the default: 2.8125 | Do nothing — no cover chosen; each transport presents whatever it presents: 2.6154 / Real TLS 1.3 (rustls) default; real QUIC (quinn) standby; fallback B→A→queue (PICK): 3.5769 / Real QUIC (quinn) default; TLS 1.3 fallback: 3.4231 / Both covers simultaneously (multipath C) as the default: 2.9808 | No |
| Reversibility | Do nothing — no cover chosen; each transport presents whatever it presents: 2.5258 / Real TLS 1.3 (rustls) default; real QUIC (quinn) standby; fallback B→A→queue (PICK): 3.5876 / Real QUIC (quinn) default; TLS 1.3 fallback: 3.4124 / Both covers simultaneously (multipath C) as the default: 2.9278 | Do nothing — no cover chosen; each transport presents whatever it presents: 2.6699 / Real TLS 1.3 (rustls) default; real QUIC (quinn) standby; fallback B→A→queue (PICK): 3.6117 / Real QUIC (quinn) default; TLS 1.3 fallback: 3.3883 / Both covers simultaneously (multipath C) as the default: 2.8738 | No |
| Effort | Do nothing — no cover chosen; each transport presents whatever it presents: 2.5258 / Real TLS 1.3 (rustls) default; real QUIC (quinn) standby; fallback B→A→queue (PICK): 3.5876 / Real QUIC (quinn) default; TLS 1.3 fallback: 3.4124 / Both covers simultaneously (multipath C) as the default: 2.9278 | Do nothing — no cover chosen; each transport presents whatever it presents: 2.6699 / Real TLS 1.3 (rustls) default; real QUIC (quinn) standby; fallback B→A→queue (PICK): 3.6117 / Real QUIC (quinn) default; TLS 1.3 fallback: 3.3883 / Both covers simultaneously (multipath C) as the default: 2.8738 | No |

**Tipping weight:** none — no single ±20% weight variation changes the leader. The ranking is robust to weight choice (it may still be fragile to score choice; see §12).

**Close-call handling (<10% margin):** the margin is 5.56%, inside the <10% band, so the refined-spec rule fires: a ≤30-minute benchmark breaks the tie; if still tied, the more reversible option wins (`03:§12.1`). See §12 for the tie-break evidence.

## 8. Expected benefits

TLS 1.3 default + QUIC standby + B→A→queue fallback: a default that works in the owner's environment on day one; a second cover measured in parallel so the 5.56% close call is resolved by data at M2/M3 rather than by argument; the session layer (P8) makes the fallback session-preserving instead of a fresh session per transport.

## 9. Failure modes

Vision padding fails vs Xue 2024 (AT-E1-03 at M3) → B's cover claim weakens and the pick is re-opened (D5 F2). QUIC standby admitted on assumption rather than measured health → sessions attempt a transport that is 100% blocked and burn endpoints; mitigated by the measured-health admission rule (probe must expect failure). A fixed SNI becomes a static (IP, SNI) signature (`02:§2.5`) → rotation from domains genuinely hosted on the same CDN/origin, on a profile schedule, never per-connection random (`05:§2.4.4`).

**Consequence if wrong:** If the cover choice is wrong (TLS 1.3 proves classifiable in Iran while QUIC would have worked), the MVP's default transport is detectable in its primary environment — but the session layer and the parallel A measurement bound the cost: the default order can be flipped by ADR at M4 without rebuilding the session layer. If the fallback order is wrong (A admitted too eagerly), the cost is burned endpoints and failed sessions in Iran.

**Reversibility cost of undoing it:** High: the cover choice is a default order plus a session-layer policy. Flipping B↔A costs an ADR and a configuration change; the transports themselves are reusable (they are A and B). The SNI pool and donor health are profile data, not protocol.

## 10. Cost / complexity

Implementation cost: B first (lowest effort), A in parallel (one transport, mature crates). Maintenance cost: two covers + a session layer, versus one — the price of measuring the close call instead of assuming it. Fallback-order cost is session-layer state-machine work at M2 (P8).

## 11. Experimental validation method

M2 ≤30-minute benchmark (AT-E3-01–03 + AT-E7-02): minimal A vs minimal B, clean / 1% / 5% loss + UDP-blocked run; pass/fail = E3 thresholds + UDP-blocked fallback. M3 E1/E2: AT-E1-01–03 (all three distinguishers, worst governs), AT-E1-06 (handshake-size KS per transport), AT-E2-01–05 (≥1,000 probes/type per transport). Falsifier: AT-E1-03 fails → B correctness 4→3, matrix flips to A, pick re-opened (D5 F2); AT-E7-02 fails (UDP blocked → no TCP fallback) → the fallback order is wrong and the session layer is redesigned. All 'not measured' until run — no benchmark ran in Phase 1 (`phase1/01-architecture-selection.md` §2).

## 12. Final decision + evidence

**Decision (PROPOSED): Real TLS 1.3 (rustls) default; real QUIC (quinn) standby; fallback B→A→queue**

PROPOSED: real TLS 1.3 (rustls) as the default cover, real QUIC (quinn) as the standby cover, fallback order B → A → queue (outage mode), with A admitted only on measured health in Iran and never on assumption. Fallback is automatic, explicit, and user-visible — never a silent downgrade. The non-mimicry exemption route is rejected as a default.

**Calculation that produced it:** S = 0.30·4 + 0.20·3 + 0.20·3 + 0.15·4 + 0.15·4 = 3.60, highest of the scored alternatives (QUIC default 3.40; multipath 2.90; do nothing 2.45). Margin over QUIC-default is 0.20 points = 5.56% — a close call, so the ≤30-minute benchmark trigger fires (see §7) and the pick stays provisional pending M2/M3 evidence.

**Confidence:** Opinion, HIGH (80%) that TLS 1.3 default + QUIC standby is the correct provisional cover order — Iran availability (QUIC 100% blocked, persistent) is the deciding Fact, and the cover-maturity advantage is real but narrow. MEDIUM (60%) that it survives M2/M3 measurement (the §1.5 score fragility in 01). Calibration p = 0.80 for the pick; p = 0.60 for its survival to M4.

**Status / date / deciders:** PROPOSED — 2026-11-25 — decided by Abolfazl (owner) at the Phase 1 gate; this record proposes, it does not accept. A decision without its M2/M3 evidence is PROPOSED, not ACCEPTED. Decided at: M0 (choice) → M2 (benchmark evidence) → M4 (revisit-or-stand). Evidence that would settle it: P7 benchmark at M2; AT-E1-03/06 at M3.

**Related items:** U4, U5, U8, R5, R7, C17, C18; AT-E1-01–03, AT-E1-06, AT-E2-01–05, AT-E7-02/03/05; M0, M2, M3, M4; P7; D4, D5, D6, D7, D8, D21, D22.
