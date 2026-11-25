# Decision Record 0008 — server-variant

- **Decision:** 0008 (D08 of 12 §6)
- **Status:** PROPOSED
- **Date:** 2026-11-25
- **Version:** V1.0
- **Owner:** Abolfazl (decides; owner of the phase-gate go-ahead); Phase 1 architecture work (this record); implementation owner: Phase 2 (pending owner go-ahead)
- **Source:** phase0/05-protocol-design-draft.md §7 Q6, §5.2 (donor health / SNI pool); phase0/09-mvp-roadmap.md §1.1 IN-1; phase0/02-competitive-analysis.md (REALITY / ShadowTLS / WebTunnel)

## 1. Problem definition

Server variant on B: borrowed-handshake (REALITY-style, no own certificate — the server borrows a real site's handshake) vs own-cert + real-site server (WebTunnel-style — the server holds its own certificate and serves a real site). Phase 1 must choose one: maintaining both variants doubles the E2 test surface and the ClientHello maintenance burden (`04:§7.2`, B maintainability 3/5).

## 2. Known facts

- REALITY/ShadowTLS/WebTunnel are deployed and field-reported working, and pass the Wu 2023 exemption (Ex5) by construction (01 §4.2 context; `04` cover maturity).
- E2 requires byte-for-byte probe equivalence against the donor/origin server, per variant, plus the donor-health and SNI-pool analysis (`05:§5.2`).
- A fixed SNI is a static (IP, SNI) signature (`02:§2.5`); rotation draws from domains genuinely hosted on the same CDN/origin, on a profile schedule, never per-connection random (`05:§2.4.4`).
- The SNI is read and connections to forbidden domains are RST immediately after ClientHello in Iran (Aryapour 2025 §4.3 — V2, VERIFIED mechanism). The borrowed/donor SNI must be a domain that is not blocklisted, is genuinely hosted where the connection goes, and is unremarkable.
- Aparecium's ShadowTLS-v3 vectors are secondary-only (U7); whether fixed in later releases is unverified.

## 3. Assumptions

- A borrowed-handshake server can be made byte-identical to the donor site's handshake under all seven E2 probe classes (a–g in `03:§7.3`). [MEDIUM — E2 at M3 is the test.]
- An own-cert + real-site server can be operated by PHOENIX (the operator runs a real site on the same server, with its own certificate). [MEDIUM — operational, not yet demonstrated.]

## 4. Unknown variables

- Which variant survives E2 probing — E2 evidence at M3 is the decider (`12:§6` D8). No E2 evidence exists.
- Donor health: whether the chosen donor domains stay unblocked and genuinely hosted at the same origin over time (donor-health analysis, `05:§5.2`).
- Whether the Aparecium vectors apply to the borrowed-handshake variant specifically (U7).

## 5. Constraints

- Hard filter — security: the variant must survive E2 byte-for-byte probing against its donor/origin; a variant that fails E2 is out.
- Only one variant is maintained (maintaining both doubles the E2 surface and the ClientHello maintenance burden).
- No secrets in any repo/log; the server's private key (own-cert variant) or the borrowed-handshake configuration must not leak into any public artefact.

## 6. Mathematical / computational model

Server variant is a probe-equivalence question per variant: the observable difference between the PHOENIX server and the donor (borrowed) or the origin (own-cert) under each E2 probe class. Borrowed-handshake shifts the trust to the donor's handshake (no cert to manage, but the donor's behaviour must be tracked); own-cert shifts it to PHOENIX's own server (full control, but a real site must be operated and its certificate managed). No closed-form model; the falsifier is E2 at M3.

## 7. Alternatives (≥3 when consequential, incl. "do nothing")

| # | Alternative | Pros | Cons |
|---|-------------|------|------|
| 1 | Do nothing — no server variant chosen; both remain possible | Zero work now. | Maintaining both variants doubles the E2 test surface and the ClientHello maintenance burden; D2 cannot specify the server's probe behaviour. Fails the M0 obligation. |
| 2 | Borrowed-handshake (REALITY-style, no own cert) (PICK) | No certificate to manage or leak; the handshake is the donor site's real handshake by construction; REALITY is deployed and field-reported working; the server needs no real site of its own. | Depends on the donor's continued health and unblocked status (donor-health analysis required); the donor's handshake behaviour must be tracked (if the donor changes its TLS stack, PHOENIX must follow); Aparecium-class risk (U7) attaches to this variant specifically. |
| 3 | Own-cert + real-site server (WebTunnel-style) | Full control: the server is PHOENIX's own, serving a real site with its own certificate; no donor dependency; the probe comparison is against PHOENIX's own origin, which it controls exactly. | PHOENIX must operate a real site (content, maintenance, certificate lifecycle); the site itself becomes an observable (its content, its popularity, its blocklist status); a real site with no traffic is itself suspicious — the site must be genuinely used or it is a Potemkin cover. |
| 4 | Both variants maintained (borrowed + own-cert, chosen per profile) | Per-profile flexibility; a donor failure has a fallback variant. | Doubles the E2 test surface and the ClientHello maintenance burden (`04:§7.2`); two probe-equivalence proofs to maintain; the draft and the roadmap both require Phase 1 to choose one. |

**Hard filters applied first** (license-compatible; MSRV set by Phase 1 and recorded; security: no known unpatched CVE, no custom crypto, audited libraries only — `03:§12.1`). A candidate failing any one is out, with no scoring. No candidate fails a license/MSRV/CVE/custom-crypto filter. 'Both variants' is not filter-failed but is rejected by the explicit Phase 1 obligation to choose one (`05:§7` Q6) and by the doubled E2 surface.

**Scoring.** Weights: correctness/safety .30, maintainability .20, performance .20, reversibility .15, effort .15; S = Σ(w·s); each criterion 0–5 (5 = best). Scores are **Opinion** unless a cell is labelled VERIFIED arithmetic or a cited Fact; the basis of each score is stated in the Pros/Cons above. Computed in Python (this generator):

| Alternative | Corr .30 | Maint .20 | Perf .20 | Rev .15 | Eff .15 | **S** |
|---|---:|---:|---:|---:|---:|---:|
| Do nothing — no server variant chosen; both remain possible | 1 | 1 | 3 | 5 | 5 | **2.60** |
| Borrowed-handshake (REALITY-style, no own cert) (PICK) | 4 | 3 | 3 | 4 | 3 | **3.45** |
| Own-cert + real-site server (WebTunnel-style) | 3 | 4 | 3 | 3 | 3 | **3.20** |
| Both variants maintained (borrowed + own-cert, chosen per profile) | 3 | 2 | 3 | 2 | 2 | **2.50** |

**Ranking:** 3.45 Borrowed-handshake (REALITY-style, no own cert) (PICK) > 3.20 Own-cert + real-site server (WebTunnel-style) > 2.60 Do nothing — no server variant chosen; both remain possible > 2.50 Both variants maintained (borrowed + own-cert, chosen per profile). Gap between top two: 0.25 points = 7.25% of the leader's score.

**Sensitivity (±20% per weight, other weights renormalised to sum 1.0):**

| Weight varied | −20% → S | +20% → S | Leader changes? |
|---|---|---|---|
| Correctness/safety | Do nothing — no server variant chosen; both remain possible: 2.7021 / Borrowed-handshake (REALITY-style, no own cert) (PICK): 3.4149 / Own-cert + real-site server (WebTunnel-style): 3.2128 / Both variants maintained (borrowed + own-cert, chosen per profile): 2.4681 | Do nothing — no server variant chosen; both remain possible: 2.5094 / Borrowed-handshake (REALITY-style, no own cert) (PICK): 3.4811 / Own-cert + real-site server (WebTunnel-style): 3.1887 / Both variants maintained (borrowed + own-cert, chosen per profile): 2.5283 | No |
| Maintainability | Do nothing — no server variant chosen; both remain possible: 2.6667 / Borrowed-handshake (REALITY-style, no own cert) (PICK): 3.4688 / Own-cert + real-site server (WebTunnel-style): 3.1667 / Both variants maintained (borrowed + own-cert, chosen per profile): 2.5208 | Do nothing — no server variant chosen; both remain possible: 2.5385 / Borrowed-handshake (REALITY-style, no own cert) (PICK): 3.4327 / Own-cert + real-site server (WebTunnel-style): 3.2308 / Both variants maintained (borrowed + own-cert, chosen per profile): 2.4808 | No |
| Performance | Do nothing — no server variant chosen; both remain possible: 2.5833 / Borrowed-handshake (REALITY-style, no own cert) (PICK): 3.4688 / Own-cert + real-site server (WebTunnel-style): 3.2083 / Both variants maintained (borrowed + own-cert, chosen per profile): 2.4792 | Do nothing — no server variant chosen; both remain possible: 2.6154 / Borrowed-handshake (REALITY-style, no own cert) (PICK): 3.4327 / Own-cert + real-site server (WebTunnel-style): 3.1923 / Both variants maintained (borrowed + own-cert, chosen per profile): 2.5192 | No |
| Reversibility | Do nothing — no server variant chosen; both remain possible: 2.5258 / Borrowed-handshake (REALITY-style, no own cert) (PICK): 3.4330 / Own-cert + real-site server (WebTunnel-style): 3.2062 / Both variants maintained (borrowed + own-cert, chosen per profile): 2.5155 | Do nothing — no server variant chosen; both remain possible: 2.6699 / Borrowed-handshake (REALITY-style, no own cert) (PICK): 3.4660 / Own-cert + real-site server (WebTunnel-style): 3.1942 / Both variants maintained (borrowed + own-cert, chosen per profile): 2.4854 | No |
| Effort | Do nothing — no server variant chosen; both remain possible: 2.5258 / Borrowed-handshake (REALITY-style, no own cert) (PICK): 3.4639 / Own-cert + real-site server (WebTunnel-style): 3.2062 / Both variants maintained (borrowed + own-cert, chosen per profile): 2.5155 | Do nothing — no server variant chosen; both remain possible: 2.6699 / Borrowed-handshake (REALITY-style, no own cert) (PICK): 3.4369 / Own-cert + real-site server (WebTunnel-style): 3.1942 / Both variants maintained (borrowed + own-cert, chosen per profile): 2.4854 | No |

**Tipping weight:** none — no single ±20% weight variation changes the leader. The ranking is robust to weight choice (it may still be fragile to score choice; see §12).

**Close-call handling (<10% margin):** the margin is 7.25%, inside the <10% band, so the refined-spec rule fires: a ≤30-minute benchmark breaks the tie; if still tied, the more reversible option wins (`03:§12.1`). See §12 for the tie-break evidence.

## 8. Expected benefits

Borrowed-handshake: no certificate management, no real site to operate, the handshake is the donor's real handshake, and the deployed precedent (REALITY) is field-reported working. The own-cert variant is retained as the named fallback if E2 or donor-health fails the pick at M3.

## 9. Failure modes

Donor blocked or donor changes its TLS stack → the borrowed handshake diverges from the donor or the SNI is blocklisted; mitigated by the donor-health analysis (`05:§5.2`) and the SNI pool (rotation from domains genuinely hosted on the same CDN/origin). Aparecium-class probe divergence (ServerFinished length, NewSessionTicket) → E2 failure at M3; mitigated by including those probes explicitly in the E2 set and by the own-cert fallback. Own-cert fallback risk: the real site must be genuinely operated, or it is a Potemkin cover — the fallback is not free.

**Consequence if wrong:** If borrowed-handshake is wrong (it fails E2 at M3, or the donor becomes unusable), the cost is a variant switch to own-cert + real-site at M3 — a server-side change plus a real site to operate, bounded because the fallback is pre-specified. If the failure is discovered only after M1 builds on the pick, the cost is one milestone of server-side rework.

**Reversibility cost of undoing it:** Medium: switching variants is a server-side change (no client protocol change if the handshake shape is preserved), but the own-cert fallback requires operating a real site — the reversal cost is operational, not just an ADR edit.

## 10. Cost / complexity

Specification cost: the variant rule + donor-health/SNI-pool analysis in D2. Implementation cost: borrowed-handshake server (no cert management) vs own-cert server + real site (content + certificate lifecycle). Test cost: E2 probe suite per variant (≥1,000 probes/type).

## 11. Experimental validation method

E2 at M3 (AT-E2-01–05): ≥1,000 probes/type against the PHOENIX B server and the donor/origin server, byte-for-byte identical responses + timing/length KS p > 0.05, with ServerFinished-length and NewSessionTicket probes explicit (Aparecium class, U7). Donor-health analysis (`05:§5.2`) at M0/M3: donor domains genuinely hosted, unblocked, unremarkable. Falsifier: any probe type diverges, or the donor becomes blocked/unhealthy → the variant fails and the own-cert fallback is adopted by ADR.

## 12. Final decision + evidence

**Decision (PROPOSED): Borrowed-handshake (REALITY-style, no own cert)**

PROPOSED: borrowed-handshake (REALITY-style, no own certificate) on B, with own-cert + real-site retained as the specified fallback. The pick is decided by E2 evidence at M3, not by the deployed precedent alone — if E2 or donor-health fails the pick, the fallback is adopted by ADR.

**Calculation that produced it:** S = 0.30·4 + 0.20·3 + 0.20·3 + 0.15·4 + 0.15·3 = 3.45, highest of the scored alternatives (own-cert 3.15; both variants 2.50). Margin 0.30 points = 8.7% — inside the <10% close-call band, so the close-call rule fires: the tie-break is the M3 E2 evidence (not a benchmark — the difference between the variants is probe equivalence, measured by E2, not throughput). If E2 ties, the more reversible option wins — borrowed-handshake (no cert/site to unwind) is the more reversible, so the pick stands.

**Confidence:** Opinion, MEDIUM (65%) — the pick follows the deployed precedent (REALITY field-reported working), which is evidence of deployability, not of E2 equivalence; the E2 test at M3 is the evidence and does not exist yet. Calibration p = 0.65.

**Status / date / deciders:** PROPOSED — 2026-11-25 — decided by Abolfazl (owner) at the Phase 1 gate; this record proposes, it does not accept. A decision without its M2/M3 evidence is PROPOSED, not ACCEPTED. Decided at: M0 (decision) → M3 (E2 evidence). Evidence that would settle it: AT-E2-01–05 at M3 (ServerFinished-length + NewSessionTicket probes explicit); donor-health analysis.

**Related items:** U7, R8; AT-E2-01–05, AT-E2-06 (D2 probe-behaviour audit); M0, M3; P9; D7 (auth placement interacts — session-ID behaviour depends on the server variant), D3, D5.
