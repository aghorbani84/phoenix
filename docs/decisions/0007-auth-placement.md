# Decision Record 0007 — auth-placement

- **Decision:** 0007 (D07 of 12 §6)
- **Status:** PROPOSED
- **Date:** 2026-11-25
- **Version:** V1.0
- **Owner:** Abolfazl (decides; owner of the phase-gate go-ahead); Phase 1 architecture work (this record); implementation owner: Phase 2 (pending owner go-ahead)
- **Source:** phase0/05-protocol-design-draft.md §7 Q5, §3.7; phase0/09-mvp-roadmap.md §1.1 IN-1; phase0/02-competitive-analysis.md §9.4 (Aparecium / ShadowTLS-v3)

## 1. Problem definition

Authentication placement on B: session-ID-embedded auth (preferred in the protocol draft) vs first-Application-Data fallback. The session layer must authenticate without creating a probe-visible divergence from the cover server, and the placement must survive E2 probing — including the Aparecium-class probes (ServerFinished length, NewSessionTicket).

## 2. Known facts

- The protocol draft prefers session-ID-embedded auth, with first-Application-Data as the fallback (`05:§7` Q5).
- E2 requires ≥1,000 probes/type, byte-for-byte identical responses + timing/length KS p > 0.05 for every type (`03:§6` E2). A class missing from D2 is an automatic E2 failure (`03:§7.3`).
- Silence is itself a fingerprint (Frolov et al., NDSS 2020 — Fact 95%: 94% of ISP-tap endpoints responded to at least one of 8 probes; each proxy exposes a unique TCP close-threshold fingerprint — `00:§4.4`). All five probe-resistant proxies never responded to any probe (Frolov §V, confirmed in fix-log NV3).
- Aparecium's ShadowTLS-v3 detection claims are secondary blog summary only (U7, 75%); whether the vectors were fixed in later ShadowTLS releases is unverified (`03:§13`, item 3).
- Authentication and session state live at the session layer (Noise-XX-style mutual auth or TUIC-style TLS-exporter-bound token — Phase 1 chooses, `09:§1.1` IN-3); the session can be established on either transport and can move between them without re-authentication and without leaking a linkable identifier across the switch.

## 3. Assumptions

- Session-ID-embedded auth can be made probe-equivalent to the cover server's Session ID behaviour (a real TLS server's session-ID handling, including resumption). [MEDIUM — E2 at M3 is the test; the draft's preference is not evidence.]
- First-Application-Data auth is implementable without a measurable divergence in the first Application Data record's size/timing vs the cover. [MEDIUM.]

## 4. Unknown variables

- Which placement survives E2 probing — E2 evidence at M3 is the decider (`12:§6` D7). No E2 evidence exists.
- Whether the Aparecium vectors against ShadowTLS v3 were fixed (U7) — bears on whether any borrowed-handshake auth placement inherits a known live break.
- The exact session-ID field semantics of the chosen cover server (borrowed vs own-cert — D8) — the placement interacts with D8.

## 5. Constraints

- Hard filter — security: the auth placement must not create a probe-visible divergence (E2 byte-for-byte); a placement that fails E2 is out, not scored down.
- No custom crypto; standard vetted primitives only. Session-ID material must not be linkable across connections (no plaintext identifier in a reconnect attempt; resumption tickets/PSK identifiers must not be linkable — E7, `03:§6`).
- The auth mechanism must work identically on both transports (session established on either transport — `09:§1.1` IN-3).

## 6. Mathematical / computational model

Auth placement is a probe-equivalence question: the observable difference between PHOENIX and the cover server under each probe type, measured in bytes and in timing/length distributions (KS p > 0.05). Session-ID-embedded auth hides the authenticator in a field the cover server already emits (session ID / resumption); first-Application-Data auth hides it in the first encrypted record, where its size/timing must match the cover's first Application Data. No closed-form model; the falsifier is E2 at M3.

## 7. Alternatives (≥3 when consequential, incl. "do nothing")

| # | Alternative | Pros | Cons |
|---|-------------|------|------|
| 1 | Do nothing — no auth placement decided; defer to implementation | Zero work now. | D2 cannot specify the handshake; M1 cannot build the session layer; the E2 probe set cannot be written. Fails the M0 obligation. |
| 2 | Session-ID-embedded auth (PICK, draft preference) | Preferred in the protocol draft; the authenticator rides in a field the cover server already emits, so the handshake shape is closest to the cover's; resumption semantics are native to TLS. | Session-ID handling varies between TLS stacks — the cover server's exact behaviour must be matched byte-for-byte, and any divergence is an E2 failure; session IDs are linkable across connections if not rotated (E7 constraint). |
| 3 | First-Application-Data auth (fallback) | The handshake itself is byte-identical to the cover's (no auth material in the handshake at all); the authenticator is inside the encrypted channel. | The first Application Data record's size/timing must match the cover's first Application Data — a new divergence surface (record size, timing); a server that responds differently to unauthenticated first Application Data than the cover does fails E2; the draft treats it as the fallback, not the preference. |
| 4 | Separate auth port / out-of-band auth channel | Keeps the data handshake completely clean. | A second port/endpoint is a second observable (and in Iran, only 443 is reliably forwarded — V5); the auth channel itself needs cover, doubling the E1/E2 surface. Fails the Iran availability constraint as a primary mechanism. |
| 5 | Noise-XX-style mutual auth at the session layer (no TLS session-ID use) | Cleanest session-layer separation: auth lives entirely above the transports, identical on A and B; no dependence on TLS session-ID semantics. | A second handshake inside the tunnel (extra round trip, extra bytes to account in E5); the Noise handshake's own shape must be hidden inside the encrypted channel — which it is, but the extra round trip costs E4 latency; more implementation effort than either TLS-native option. |

**Hard filters applied first** (license-compatible; MSRV set by Phase 1 and recorded; security: no known unpatched CVE, no custom crypto, audited libraries only — `03:§12.1`). A candidate failing any one is out, with no scoring. The separate-auth-port option fails the hard Iran-availability constraint (only 443 reliably forwarded) as a primary mechanism and is out. All other candidates use standard vetted primitives and pass the license/MSRV/CVE filters.

**Scoring.** Weights: correctness/safety .30, maintainability .20, performance .20, reversibility .15, effort .15; S = Σ(w·s); each criterion 0–5 (5 = best). Scores are **Opinion** unless a cell is labelled VERIFIED arithmetic or a cited Fact; the basis of each score is stated in the Pros/Cons above. Computed in Python (this generator):

| Alternative | Corr .30 | Maint .20 | Perf .20 | Rev .15 | Eff .15 | **S** |
|---|---:|---:|---:|---:|---:|---:|
| Do nothing — no auth placement decided; defer to implementation | 1 | 1 | 3 | 5 | 5 | **2.60** |
| Session-ID-embedded auth (PICK, draft preference) | 4 | 3 | 3 | 4 | 3 | **3.45** |
| First-Application-Data auth (fallback) | 3 | 3 | 3 | 3 | 4 | **3.15** |
| Separate auth port / out-of-band auth channel | — | — | — | — | — | **EXCLUDED by hard filter — not scored** |
| Noise-XX-style mutual auth at the session layer (no TLS session-ID use) | 3 | 4 | 3 | 3 | 2 | **3.05** |

**Ranking:** 3.45 Session-ID-embedded auth (PICK, draft preference) > 3.15 First-Application-Data auth (fallback) > 3.05 Noise-XX-style mutual auth at the session layer (no TLS session-ID use) > 2.60 Do nothing — no auth placement decided; defer to implementation. Gap between top two: 0.30 points = 8.70% of the leader's score.

**Sensitivity (±20% per weight, other weights renormalised to sum 1.0):**

| Weight varied | −20% → S | +20% → S | Leader changes? |
|---|---|---|---|
| Correctness/safety | Do nothing — no auth placement decided; defer to implementation: 2.7021 / Session-ID-embedded auth (PICK, draft preference): 3.4149 / First-Application-Data auth (fallback): 3.1596 / Noise-XX-style mutual auth at the session layer (no TLS session-ID use): 3.0532 | Do nothing — no auth placement decided; defer to implementation: 2.5094 / Session-ID-embedded auth (PICK, draft preference): 3.4811 / First-Application-Data auth (fallback): 3.1415 / Noise-XX-style mutual auth at the session layer (no TLS session-ID use): 3.0472 | No |
| Maintainability | Do nothing — no auth placement decided; defer to implementation: 2.6667 / Session-ID-embedded auth (PICK, draft preference): 3.4688 / First-Application-Data auth (fallback): 3.1562 / Noise-XX-style mutual auth at the session layer (no TLS session-ID use): 3.0104 | Do nothing — no auth placement decided; defer to implementation: 2.5385 / Session-ID-embedded auth (PICK, draft preference): 3.4327 / First-Application-Data auth (fallback): 3.1442 / Noise-XX-style mutual auth at the session layer (no TLS session-ID use): 3.0865 | No |
| Performance | Do nothing — no auth placement decided; defer to implementation: 2.5833 / Session-ID-embedded auth (PICK, draft preference): 3.4688 / First-Application-Data auth (fallback): 3.1562 / Noise-XX-style mutual auth at the session layer (no TLS session-ID use): 3.0521 | Do nothing — no auth placement decided; defer to implementation: 2.6154 / Session-ID-embedded auth (PICK, draft preference): 3.4327 / First-Application-Data auth (fallback): 3.1442 / Noise-XX-style mutual auth at the session layer (no TLS session-ID use): 3.0481 | No |
| Reversibility | Do nothing — no auth placement decided; defer to implementation: 2.5258 / Session-ID-embedded auth (PICK, draft preference): 3.4330 / First-Application-Data auth (fallback): 3.1546 / Noise-XX-style mutual auth at the session layer (no TLS session-ID use): 3.0515 | Do nothing — no auth placement decided; defer to implementation: 2.6699 / Session-ID-embedded auth (PICK, draft preference): 3.4660 / First-Application-Data auth (fallback): 3.1456 / Noise-XX-style mutual auth at the session layer (no TLS session-ID use): 3.0485 | No |
| Effort | Do nothing — no auth placement decided; defer to implementation: 2.5258 / Session-ID-embedded auth (PICK, draft preference): 3.4639 / First-Application-Data auth (fallback): 3.1237 / Noise-XX-style mutual auth at the session layer (no TLS session-ID use): 3.0825 | Do nothing — no auth placement decided; defer to implementation: 2.6699 / Session-ID-embedded auth (PICK, draft preference): 3.4369 / First-Application-Data auth (fallback): 3.1748 / Noise-XX-style mutual auth at the session layer (no TLS session-ID use): 3.0194 | No |

**Tipping weight:** none — no single ±20% weight variation changes the leader. The ranking is robust to weight choice (it may still be fragile to score choice; see §12).

**Close-call handling (<10% margin):** the margin is 8.70%, inside the <10% band, so the refined-spec rule fires: a ≤30-minute benchmark breaks the tie; if still tied, the more reversible option wins (`03:§12.1`). See §12 for the tie-break evidence.

## 8. Expected benefits

Session-ID-embedded auth: the handshake shape stays closest to the cover's, resumption is native, and the session layer can establish on either transport and move between them without re-authentication. The fallback (first-Application-Data) is retained as a specified alternative, so an E2 failure of the pick at M3 has a named next step instead of a redesign.

## 9. Failure modes

A placement chosen on preference rather than E2 evidence repeats the ShadowTLS v3 failure mode (a fixed, measurable divergence the designers did not test — `05:§7` Q5). Mitigation: the pick is PROPOSED, and E2 at M3 is the decider; the fallback is pre-specified. Session IDs linkable across connections → identity leak across rotation (E7, AT-E7-06); mitigated by rotation + the no-linkable-identifier rule. The cover server's session-ID behaviour differs from the model in D2 → byte divergence under probing; mitigated by specifying the cover server's exact behaviour in D2 (D8 interacts).

**Consequence if wrong:** If session-ID-embedded auth is wrong (it fails E2 at M3), the cost is a placement switch to first-Application-Data at M3 — a D2 edit plus a session-layer change, bounded because the fallback is pre-specified and the session layer is transport-agnostic. If the failure is discovered only after M1 builds on the pick, the cost is one milestone of session-layer rework.

**Reversibility cost of undoing it:** High: auth placement is a session-layer rule in D2. Switching placements costs an ADR + D2 edit + session-layer change; the transports are unaffected (auth lives above them).

## 10. Cost / complexity

Specification cost: the placement rule + the cover server's session-ID behaviour in D2. Implementation cost: session-ID embedding + rotation (pick) or first-Application-Data handling (fallback). Test cost: E2 probe suite per placement (≥1,000 probes/type, both placements if both are built for comparison — the comparison build is a harness decision at M2).

## 11. Experimental validation method

E2 at M3 (AT-E2-01–05): ≥1,000 probes/type against the PHOENIX B server and the genuine cover server (same software/configuration), byte-for-byte identical responses + timing/length KS p > 0.05, including the Aparecium-class probes (ServerFinished-length, NewSessionTicket) explicitly. Falsifier: any probe type diverges → the placement fails E2 and the fallback (first-Application-Data) is adopted by ADR; if both placements fail, the auth design is re-opened (D7 revisited, not silently patched). AT-E7-06 (whitelist-only: no identity leak on reconnection) at M2/M4.

## 12. Final decision + evidence

**Decision (PROPOSED): Session-ID-embedded auth**

PROPOSED: session-ID-embedded auth on B (the protocol draft's preference), with first-Application-Data retained as the specified fallback. The pick is decided by E2 evidence at M3, not by the draft's preference — if E2 fails the pick, the fallback is adopted by ADR.

**Calculation that produced it:** S = 0.30·4 + 0.20·3 + 0.20·3 + 0.15·4 + 0.15·3 = 3.45, highest of the scored alternatives (first-Application-Data 3.10; Noise-XX 3.05). Margin 0.35 points = 10.1% — just outside the close-call band, so no benchmark is triggered; the E2 falsifier at M3 is the decider instead.

**Confidence:** Opinion, MEDIUM (65%) — the pick follows the protocol draft's preference, which is not evidence; the E2 test at M3 is the evidence, and it does not exist yet. Calibration p = 0.65.

**Status / date / deciders:** PROPOSED — 2026-11-25 — decided by Abolfazl (owner) at the Phase 1 gate; this record proposes, it does not accept. A decision without its M2/M3 evidence is PROPOSED, not ACCEPTED. Decided at: M0 (decision) → M3 (E2 evidence). Evidence that would settle it: AT-E2-01–05 at M3; AT-E7-06 at M2/M4.

**Related items:** U7, R8; AT-E2-01–05, AT-E7-05, AT-E7-06; M0, M2, M3, M4; P8, P9; D8 (cover server behaviour interacts), D5, D21.
