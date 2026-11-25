# Decision Record 0012 — threat-model-master-prompt

- **Decision:** 0012 (D12 of 12 §6)
- **Status:** PROPOSED
- **Date:** 2026-11-25
- **Version:** V1.0
- **Owner:** Abolfazl (decides; owner of the phase-gate go-ahead); Phase 1 architecture work (this record); implementation owner: Phase 2 (pending owner go-ahead)
- **Source:** phase0/01-threat-models.md (§2 D1 matrix, §4 Iran, §10.3 collateral, §11); phase0/03-feasibility-assessment.md §11.3 decision 8; Addendum #2 (Master Prompt §8); phase1/03-iran-profile.md (2025–26 updates V1–V12, U1-a…U1-j)

## 1. Problem definition

Threat model §8 (Master Prompt): per-threat likelihood / impact / mitigation / test / residual, as a Phase 1 deliverable. The D1 adversary capability matrix must be carried into Phase 1 with the 2025–26 Iran updates, and every threat must name the test that would detect it and the residual risk that remains after mitigation.

## 2. Known facts

- D1 matrix (`01:§2`): per-environment adversary capabilities, with 'not verified' ≠ 'not deployed' — an UNKNOWN capability may exist and simply be unpublished (`01:§11`).
- Iran 2025–26 updates (phase1/03-iran-profile.md §1, V1–V12): centralised DPI at a national gateway (V1); SNI RST immediately after ClientHello (V2, mechanism now directly observed); DNS poisoning >90% to 10.10.34.34 (V3); HTTP Host/keyword filtering + block-page injection (V4, case-sensitive matching nuance); protocol whitelist — only DNS/HTTP/HTTPS forwarded (V5); QUIC 100% blocked, persistent post-shutdown (V6); scheduled total outages as a normal mode, incl. the June 2025 stealth blackout with BGP staying up (V7); DoH blocked (V8); throttling (V9); adaptive blocking, preparatory changes ~5 days before shutdowns (V10); circumvention-tool and VPN-site blocking (V11); two distinguishable censorship injectors (V12).
- Iran collateral tolerance is very high: ~400,000 Instagram businesses lost Sept–Dec 2022, ~8 trillion tomans in one month, shutdowns continuing (Filterwatch, `01:§4`); 'hide in popular traffic' raises the censor's cost far less in Iran than in China (`phase1/03-iran-profile.md` §3.5).
- China: GFW decrypts QUIC Initial packets and extracts SNI since 7 April 2024 (Zohaib 2025); a 1-byte packet can trigger a 120–180 s IP:port block (Wu 2023). Russia: escalation ladder, corporate tunnels exempt. Turkmenistan: >122k domains, 6k over-blocking rules → >5.4M domains. Kazakhstan: only PKI-adversarial (state root CA MITM 2019/2020). Belarus: per-ISP RST fingerprints, evidence 2020, currency unknown.
- Xue NDSS 2025 cross-layer RTT: per-visit >70% TPR at 1% FPR — 'not verified' as deployed in every environment (`01:§2`).

## 3. Assumptions

- The 2025–26 papers' findings hold for the owner's ISP and network type until a per-ISP measurement says otherwise (U1-i). [MEDIUM.]
- OONI-derived aggregations (Oct–Nov 2026) reflect the underlying OONI data, to be confirmed by the Tier A pull. [MEDIUM.]
- Per-environment collateral assumptions in this record are policy inferences from past behaviour and can change without notice (`03:§8.2` context). [MEDIUM.]

## 4. Unknown variables

- Iran DPI classifiers at Wu-2023 depth (U1-a…U1-j): entropy/popcount first-packet tests, packet-size/timing classifiers, TLS-in-TLS deployment, cross-layer RTT deployment, active probing/replay, JA3/JA4, ML classification, flow correlation/reputation, per-ISP heterogeneity, post-January-2026 state. All UNKNOWN.
- Whether any censor deploys the Xue 2025 cross-layer RTT classifier — unverifiable from the lab.
- Belarus evidence currency (2020-era, stale); Turkmenistan/Kazakhstan current enforcement details.

## 5. Constraints

- Hard filter — honesty: 'not verified' is never written as 'not deployed'; vendor marketing claims are rejected as evidence (`01:§1.2`, `01:§11`); no user-safety guarantee is made (user safety depends on enforcement practice, not packets — `03:§2.2`).
- No 'undetectable' language; no superiority claim without reproducible evidence.
- Tier C: any test against infrastructure PHOENIX does not own stops for one explicit owner ask; the Tier C in-country vantage study (Iran profile §4.3) is specified, not executed, and needs a separate explicit decision + ethics/safety review.

## 6. Mathematical / computational model

Per-threat table (below, in §12): likelihood × impact → mitigation → test → residual. Likelihood/impact are Opinion (calibrated, with % where they drive a design decision); the test column names the acceptance test that would detect the threat; the residual column states what remains after mitigation. The model is a register, not a computation — no single score is computed across threats, because the threats are not commensurable (a shutdown and a classifier are different kinds of failure).

## 7. Alternatives (≥3 when consequential, incl. "do nothing")

| # | Alternative | Pros | Cons |
|---|-------------|------|------|
| 1 | Do nothing — carry the Phase 0 threat model forward without a Phase 1 §8 record | Zero work. | The 2025–26 Iran updates (QUIC sticky blocking, stealth blackout, SNI RST observed) never enter the design record; later milestones proceed on a GFW-only assumption, which `09:§4.2` R-C forbids once the profile exists. Fails the M0 obligation. |
| 2 | Phase 1 §8 threat model: per-threat likelihood/impact/mitigation/test/residual, with the Iran profile as first-class input (PICK) | Every threat names its test and its residual; the 2025–26 Iran evidence is folded in; the GFW model does not transfer by default; the falsifier (a measurement showing Iran deploys a classifier the design does not survive) is specified per threat class. | The classifier rows are all UNKNOWN — the table is honest about what it does not know, which limits how much design it can justify; the per-ISP and post-Jan-2026 gaps remain open until the Tier A pulls. |
| 3 | GFW-model threat model, ported to Iran with a note | Cheapest to write; reuses the best-characterised adversary (Wu 2023 depth). | The transfer is ungrounded (Condition 2, `03:§1.3`): Iran's verified behaviours (protocol whitelist, sticky QUIC blocking, stealth blackout, high collateral tolerance) differ from the GFW model in ways that change design decisions. Rejected on the evidence. |
| 4 | Vendor-claims threat model (ML classifiers, JA3, 'WireGuard in 100 packets' as verified) | Covers more threats on paper. | FAILS the hard honesty filter: vendor marketing claims are rejected as evidence (`01:§1.2`, `01:§11`) and remain rejected. Designing against them would spend engineering on unverified threats while the verified ones (V1–V12) go unaddressed. Out. |

**Hard filters applied first** (license-compatible; MSRV set by Phase 1 and recorded; security: no known unpatched CVE, no custom crypto, audited libraries only — `03:§12.1`). A candidate failing any one is out, with no scoring. The vendor-claims option fails the hard honesty filter (vendor marketing is not evidence) and is out. The GFW-port option is not filter-failed but violates Condition 2 (Iran as a first-class design input) and is rejected on the evidence. All other candidates are admissible.

**Scoring.** Weights: correctness/safety .30, maintainability .20, performance .20, reversibility .15, effort .15; S = Σ(w·s); each criterion 0–5 (5 = best). Scores are **Opinion** unless a cell is labelled VERIFIED arithmetic or a cited Fact; the basis of each score is stated in the Pros/Cons above. Computed in Python (this generator):

| Alternative | Corr .30 | Maint .20 | Perf .20 | Rev .15 | Eff .15 | **S** |
|---|---:|---:|---:|---:|---:|---:|
| Do nothing — carry the Phase 0 threat model forward without a Phase 1 §8 record | 1 | 1 | 3 | 5 | 5 | **2.60** |
| Phase 1 §8 threat model: per-threat likelihood/impact/mitigation/test/residual, with the Iran profile as first-class input (PICK) | 4 | 4 | 3 | 4 | 3 | **3.65** |
| GFW-model threat model, ported to Iran with a note | 2 | 3 | 3 | 3 | 4 | **2.85** |
| Vendor-claims threat model (ML classifiers, JA3, 'WireGuard in 100 packets' as verified) | — | — | — | — | — | **EXCLUDED by hard filter — not scored** |

**Ranking:** 3.65 Phase 1 §8 threat model: per-threat likelihood/impact/mitigation/test/residual, with the Iran profile as first-class input (PICK) > 2.85 GFW-model threat model, ported to Iran with a note > 2.60 Do nothing — carry the Phase 0 threat model forward without a Phase 1 §8 record. Gap between top two: 0.80 points = 21.92% of the leader's score.

**Sensitivity (±20% per weight, other weights renormalised to sum 1.0):**

| Weight varied | −20% → S | +20% → S | Leader changes? |
|---|---|---|---|
| Correctness/safety | Do nothing — carry the Phase 0 threat model forward without a Phase 1 §8 record: 2.7021 / Phase 1 §8 threat model: per-threat likelihood/impact/mitigation/test/residual, with the Iran profile as first-class input (PICK): 3.6277 / GFW-model threat model, ported to Iran with a note: 2.9043 | Do nothing — carry the Phase 0 threat model forward without a Phase 1 §8 record: 2.5094 / Phase 1 §8 threat model: per-threat likelihood/impact/mitigation/test/residual, with the Iran profile as first-class input (PICK): 3.6698 / GFW-model threat model, ported to Iran with a note: 2.8019 | No |
| Maintainability | Do nothing — carry the Phase 0 threat model forward without a Phase 1 §8 record: 2.6667 / Phase 1 §8 threat model: per-threat likelihood/impact/mitigation/test/residual, with the Iran profile as first-class input (PICK): 3.6354 / GFW-model threat model, ported to Iran with a note: 2.8438 | Do nothing — carry the Phase 0 threat model forward without a Phase 1 §8 record: 2.5385 / Phase 1 §8 threat model: per-threat likelihood/impact/mitigation/test/residual, with the Iran profile as first-class input (PICK): 3.6635 / GFW-model threat model, ported to Iran with a note: 2.8558 | No |
| Performance | Do nothing — carry the Phase 0 threat model forward without a Phase 1 §8 record: 2.5833 / Phase 1 §8 threat model: per-threat likelihood/impact/mitigation/test/residual, with the Iran profile as first-class input (PICK): 3.6771 / GFW-model threat model, ported to Iran with a note: 2.8438 | Do nothing — carry the Phase 0 threat model forward without a Phase 1 §8 record: 2.6154 / Phase 1 §8 threat model: per-threat likelihood/impact/mitigation/test/residual, with the Iran profile as first-class input (PICK): 3.6250 / GFW-model threat model, ported to Iran with a note: 2.8558 | No |
| Reversibility | Do nothing — carry the Phase 0 threat model forward without a Phase 1 §8 record: 2.5258 / Phase 1 §8 threat model: per-threat likelihood/impact/mitigation/test/residual, with the Iran profile as first-class input (PICK): 3.6392 / GFW-model threat model, ported to Iran with a note: 2.8454 | Do nothing — carry the Phase 0 threat model forward without a Phase 1 §8 record: 2.6699 / Phase 1 §8 threat model: per-threat likelihood/impact/mitigation/test/residual, with the Iran profile as first-class input (PICK): 3.6602 / GFW-model threat model, ported to Iran with a note: 2.8544 | No |
| Effort | Do nothing — carry the Phase 0 threat model forward without a Phase 1 §8 record: 2.5258 / Phase 1 §8 threat model: per-threat likelihood/impact/mitigation/test/residual, with the Iran profile as first-class input (PICK): 3.6701 / GFW-model threat model, ported to Iran with a note: 2.8144 | Do nothing — carry the Phase 0 threat model forward without a Phase 1 §8 record: 2.6699 / Phase 1 §8 threat model: per-threat likelihood/impact/mitigation/test/residual, with the Iran profile as first-class input (PICK): 3.6311 / GFW-model threat model, ported to Iran with a note: 2.8835 | No |

**Tipping weight:** none — no single ±20% weight variation changes the leader. The ranking is robust to weight choice (it may still be fragile to score choice; see §12).

**Close-call handling (<10% margin):** the margin is 21.92%, outside the <10% band, so no benchmark is triggered by the close-call rule. See §12 for the tie-break evidence.

## 8. Expected benefits

A threat model in which every row is testable: each threat names the acceptance test that detects it (E1/E2/E7/AT-VPS/AT-GUI as applicable) and the residual that remains after mitigation. The Iran profile's verified blocking behaviours ground the transport order, SNI policy, fallback order, outage behaviour, and PQC deferral — instead of GFW analogy.

## 9. Failure modes

Per-ISP heterogeneity (U1-i) makes the profile wrong for the owner's ISP → the design's Iran assumptions fail on his network; mitigated by stating the assumption and by the Tier A pulls (OONI per-ASN breakdown). Post-Jan-2026 drift (U1-j) → the June 2025 findings no longer hold; mitigated by the falsifier (§11) and the Tier A/B pulls. A falsifier fires (Iran deploys a classifier the design does not survive) → the profile is revised in place (dated revision note, old text retained, never silently overwritten) and the affected design requirements are flagged; the falsifier is a revision trigger, not a project-killer — except where it triggers D1's F1 (cross-layer RTT deployment + owner requires RTT resistance).

**Consequence if wrong:** If the threat model is wrong (a threat's likelihood is overstated), the cost is over-engineering against a phantom — bounded by the falsifier tests, which measure rather than assume. If it is wrong the other way (a deployed capability marked UNKNOWN), the cost is a design that does not survive a real classifier — bounded by the same falsifier: a measurement showing the deployment revises the profile and re-opens the affected ADRs (D1, D2, D7/D8) by decision.

**Reversibility cost of undoing it:** High: the threat model is a document. Revising a row costs a dated revision note; the design requirements that rest on a revised row are flagged and re-decided by ADR. Nothing built depends on a likelihood estimate except through the ADRs that cite it.

## 10. Cost / complexity

One Phase 1 deliverable (this record + the Iran profile); Tier A pulls (OONI/Censored Planet/Cloudflare Radar/Filterwatch/Lantern corpus) and Tier B paper reads (FOCI 2026 PDF, January 2026 shutdown paper, WWW '26 phased-shutdown paper, Bock 2020 re-read) as specified in Iran profile §4.1–§4.2. Tier C (in-country vantage study) is specified, not executed, and costs nothing unless the owner authorises it separately.

## 11. Experimental validation method

Falsifier (Iran profile §5): a measurement showing Iran deploys a classifier the current design does not survive — any one of: (1) Wu-class entropy/popcount first-packet test; (2) packet-size/timing classifier flagging the PHOENIX shape; (3) Xue 2024 encapsulated-TLS classifier; (4) Xue 2025 cross-layer RTT classifier (→ D1's R5 limitation becomes active for Iran; deployment scope revisited at the phase gate); (5) GFW-class active probing/replay; (6) JA3/JA4 or ML protocol classification; (7) a post-Jan-2026 measurement showing the June 2025 findings no longer hold. What does NOT falsify the profile: a vendor claim; a single anecdotal user report; a measurement from a different country; a paper about a different classifier class. Tests per threat are named in the §12 table (E1/E2/E7/AT-VPS/AT-GUI).

## 12. Final decision + evidence

**Decision (PROPOSED): Phase 1 §8 threat model: per-threat likelihood/impact/mitigation/test/residual, with the Iran profile as first-class input**

PROPOSED: adopt the Phase 1 threat model §8 as recorded below (per-threat likelihood/impact/mitigation/test/residual), with the Iran profile (`phase1/03-iran-profile.md`) as a first-class design input and the GFW model not transferred by default. The table is the decision's content — it is recorded here, not referenced away.

**Threat table (§8, Master Prompt):**

| # | Threat | Likelihood | Impact | Mitigation | Test | Residual |
|---|---|---|---|---|---|---|
| T1 | SNI-based TLS filtering (RST after ClientHello) — Iran V2 | HIGH (verified, observed 2025) | Session fails at handshake | Unremarkable donor SNI, genuinely hosted; SNI pool rotation on a profile schedule, never per-connection random | AT-E2-01–05 (probe equivalence); donor-health analysis (D8) | A donor domain can be blocklisted after adoption; rotation is the mitigation, not a cure |
| T2 | Protocol whitelist (only DNS/HTTP/HTTPS forwarded) — Iran V5 | HIGH (verified, in-country 2025) | Any non-443 transport dropped silently | TLS 1.3 over TCP/443 only as the Iran default; no custom ports; no UDP reliance | M2 UDP-blocked run (AT-E7-02); lab protocol-whitelist simulation | A whitelist change (narrowing) is outside PHOENIX's control; outage mode is the answer, not a bypass |
| T3 | QUIC/UDP blocking, 100% and sticky — Iran V6 | HIGH (verified, persistent post-shutdown) | Transport A unusable as a default in Iran | B-first default; A opportunistic standby, admitted only on measured health | AT-E7-02 (UDP blocked → TCP fallback); M2 benchmark UDP-blocked run | A remains unusable until a fresh measurement shows otherwise; the probe must expect failure |
| T4 | DNS poisoning / hijacking (10.10.34.34) — Iran V3 | HIGH (verified, >90% of censored domains) | Destinations forgeable; bootstrap fails | Resolve at the proxy (D10); no plaintext DNS; profile ships server IP + name | M2 DNS test (D10 validation) | Bootstrap resolver may be blocked per-ISP (U1-i) |
| T5 | DoH blocking — Iran V8 | HIGH (verified, Sept 2022) | Fixed-DoH clients fail by design | No fixed DoH resolver (D10); DNS-over-TCP bootstrap only | M2 DNS test | DoH-class interference could extend to DNS-over-TCP; re-tested at M2 |
| T6 | Throttling to ~130 kbps (protocol-differentiated, scheduled) — Iran V9 / R10 | HIGH (verified; Xue IMC 2021, 130–150 kbps on SNI alone, Fact 97%) | Tunnel 'connected' but unusable; user misled | Degraded mode, not failure: GUI shows throttling as degraded; session layer considers a transport switch; bulk shed before interactive | AT-E3 (rho under loss); AT-GUI degraded-mode signalling (AT-GUI-06) | Throttling cannot be defeated by wire format; survival, not victory, is the goal |
| T7 | Scheduled total outage / stealth blackout (BGP stays up) — Iran V7 | HIGH (verified: Nov 2019, Sept–Oct 2022 curfews ~100 h/13 days, June 2025 18–25 June) | No connectivity; aggressive retry burns endpoints | Queue / resume / no-identity-leak (G8); back off, don't hammer; liveness at the application layer only, never from routing | AT-E7-06 (whitelist-only, no identity leak); outage-mode tests at M2 | No transport survives absence; the requirement is G8, not connectivity through absence |
| T8 | Wu-class entropy/popcount first-packet tests | UNKNOWN (not verified in Iran; verified for GFW) | First packet flagged; session blocked | Real TLS 1.3 cover (D3/D8); first packet passes the exemption rules by construction | AT-E1-01 (Wu exemption heuristics) | UNKNOWN — design against the feasible, verify nothing (D1 matrix discipline) |
| T9 | Xue 2024 encapsulated-TLS (TLS-in-TLS) classifier | UNKNOWN in Iran (slides: VMess 77.14%, Shadowsocks 85.38%, VLESS 74.83%, Trojan 73.71%) | Nested handshake detected | Vision-style passthrough / multiplexing (D3); padding (U4/U5 unmeasured) | AT-E1-03 (nested-handshake classifier) | UNKNOWN until M3; if AT-E1-03 fails, B correctness 4→3 (D5 F2) |
| T10 | Xue 2025 cross-layer RTT classifier | UNKNOWN deployment everywhere; HIGH that the method works as published | Per-visit >70% TPR at 1% FPR | None in the MVP — R5 accepted limitation (D1); DNS resolved at the proxy (~2× susceptibility reduction, not a fix) | AT-E4-07 | FULL residual: the MVP is detectable by this classifier class; deployment scope limited to environments where it is not verified as deployed; F1 NO-GO if the owner requires RTT resistance |
| T11 | Active probing / handshake replay (GFW class) | UNKNOWN in Iran; HIGH for GFW | Server fingerprinted by its probe responses (silence is a fingerprint — Frolov 2020) | E2 probe equivalence, byte-for-byte, all seven input classes (D2/D7/D8); ServerFinished-length + NewSessionTicket probes explicit | AT-E2-01–05, AT-E2-06 (D2 audit) | UNKNOWN until M3; a class missing from D2 is an automatic E2 failure |
| T12 | Endpoint/IP blocking (cheapest censor move) — R12 | HIGH (GFW 120–180 s residual block, Wu 2023; Iran vendor claim of automated VPN-address blocking is LOW, unconfirmed) | Single server blocklisted; session dies | SNI/endpoint rotation (E7); server disposable and replaceable; endpoint-diversity requirement | AT-E7-03 (block primary SNI → rotation); AT-VPS reinstall/rotation tests at M5 | Rotation interval is designed against the GFW's 120–180 s figure, not an Iran-measured value (U1-h) |
| T13 | State-root MITM (Kazakhstan class) | HIGH for Kazakhstan (2019/2020); LOW elsewhere | TLS interception with a state root | Pinning + two-pin rotation + single-operation revocation (D9); CA key never leaves the operator machine | M5 rotation/revocation tests (AT-VPS series) | PQC does not mitigate a state root CA (authentication, not key exchange) |
| T14 | Distribution blocking (app stores, circumvention-tool sites) — R13 | HIGH (verified 2022; VPN sites blocked 78.6–90.1%, V11 secondary) | Owner cannot obtain or update the client | No Google Play dependence; sideloading / direct download; update checks over the tunnel only | AT-GUI distribution checks at M6 | A client the owner cannot obtain is not a client; distribution planning belongs in the Iran profile, not a Phase 4 afterthought |
| T15 | Collateral-tolerant censor breaks popular services anyway — Iran | HIGH (verified: ~400k Instagram businesses lost, ~8 T tomans/month) | 'Hide in popular traffic' buys little | Cover fidelity justified by classification difficulty, not collateral leverage (Iran profile §3.5); any architecture text claiming the censor 'won't block this because too many users depend on it' is struck | E1 (classification difficulty, not collateral) | The collateral lever does not exist in Iran; it is strongest in China/Russia, absent in Turkmenistan |

**Calculation that produced it:** S = 0.30·4 + 0.20·4 + 0.20·3 + 0.15·4 + 0.15·3 = 3.70, highest of the scored alternatives (GFW-port 2.85). Margin 0.85 points = 23.0% — outside the close-call band. The score records the judgment; the content is the table below.

**Confidence:** Opinion, HIGH (80%) for the blocking behaviours (V1–V12, verified 2025–26 in-country measurements); UNKNOWN for the classifiers (U1-a…U1-j) — the table states UNKNOWN where it is unknown, which is the honest confidence, not a gap in this record. Calibration p = 0.80 for the blocking rows; p is not assigned to the UNKNOWN classifier rows (they are unassigned by design).

**Status / date / deciders:** PROPOSED — 2026-11-25 — decided by Abolfazl (owner) at the Phase 1 gate; this record proposes, it does not accept. A decision without its M2/M3 evidence is PROPOSED, not ACCEPTED. Decided at: M0. Evidence that would settle it: Tier A pulls + Tier B reads (Iran profile §4.1–§4.2); the §5 falsifier (any measurement showing a deployed classifier the design does not survive).

**Related items:** U1, U19, R6, R10; AT-E1-01–06, AT-E2-01–05, AT-E4-07, AT-E7-02–06, AT-VPS-01–22, AT-GUI-01–20; M0, M2, M3, M4; P2; D1, D2, D3, D5, D7, D8, D10, D21.
