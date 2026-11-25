# Decision Record 0010 — dns-resolution

- **Decision:** 0010 (D10 of 12 §6)
- **Status:** PROPOSED
- **Date:** 2026-11-25
- **Version:** V1.0
- **Owner:** Abolfazl (decides; owner of the phase-gate go-ahead); Phase 1 architecture work (this record); implementation owner: Phase 2 (pending owner go-ahead)
- **Source:** phase0/05-protocol-design-draft.md §7 Q8; phase0/01-threat-models.md §4 (Iran DoH blocking); phase1/03-iran-profile.md §3.2; phase1/02-rtt-strategy.md §2.2 (DNS-at-proxy halves RTT susceptibility)

## 1. Problem definition

DNS resolution method: how does the client resolve the server (and the user's destinations) without depending on a fixed DoH resolver or plaintext DNS? DoH endpoints are blocked in Iran (V8) and DNS answers are poisoned (V3); a client that depends on a fixed DoH resolver fails in Iran by design (`01:§4` #4).

## 2. Known facts

- DoH endpoints blocked 20 Sept 2022 by DNS tampering (bogons) + TLS interference (`01:§4`, OONI-IR22). [Fact, HIGH.] No fresh 2025–26 source contradicts it.
- DNS poisoning / hijacking verified: >90% of tested censored domains resolved to 10.10.34.0/24 via injected DNS replies (Aryapour 2025 §4.1). [Fact, HIGH.]
- Users who resolve DNS locally (at the client) are ~2× as susceptible to the cross-layer RTT classifier as users who resolve DNS at the proxy (Lantern corpus summary of Xue 2025 §VI-C-2). [Fact, MEDIUM.]
- Only DNS/HTTP/HTTPS are forwarded in Iran (V5); DNS over UDP/53 is the sole deliberate UDP exception (Nym June 2025 report).
- A correct resolution method is unverified from the Phase 0 sources: 'I cannot verify a correct method from the Phase 0 sources' (`05:§7` Q8).

## 3. Assumptions

- Resolving at the proxy (inside the tunnel) is implementable in the session layer without a separate DNS channel. [MEDIUM — design claim, unimplemented.]
- DNS-over-TCP to an unblocked resolver remains available where the profile permits it (TCP/443 is forwarded; DNS-over-TCP returned to baseline ~15% blocking after the June 2025 shutdown lifted, FOCI 2026). [MEDIUM.]

## 4. Unknown variables

- A correct resolution method — 'I cannot verify' from Phase 0 (Q8; `05` App. B item 3).
- Which resolvers (if any) are unblocked on the owner's network at the moment of use — measurements decay (`03:§5.2`).
- Whether DNS-over-TCP to a specific resolver is blocked on the owner's ISP (per-ISP heterogeneity, U1-i).

## 5. Constraints

- Hard filter — security/availability: no fixed DoH resolver (blocked in Iran); no plaintext DNS (poisoned in Iran). A candidate depending on either is out as the Iran method.
- Resolution must not leak the destination list: the UI never shows DNS query names (`06:§3.4`); the Statistics page shows a query count only.
- No test against infrastructure PHOENIX does not own (Tier C) — resolver reachability is tested in the lab (M2, `09:§5.3` Q8), not by probing live resolvers from Iran.

## 6. Mathematical / computational model

Resolution is a two-question decision: (a) where is the query answered (client vs proxy), and (b) over what channel. Resolving at the proxy halves RTT-fingerprint susceptibility (§2.2 of 02) and keeps the query inside the tunnel, so the censor sees no DNS query at all for tunnelled destinations. The fallback (DNS-over-TCP to an unblocked resolver) is for the pre-tunnel bootstrap — resolving the server's own name before the tunnel exists — where the proxy is not yet reachable.

## 7. Alternatives (≥3 when consequential, incl. "do nothing")

| # | Alternative | Pros | Cons |
|---|-------------|------|------|
| 1 | Do nothing — plaintext DNS via the system resolver | Zero work; the OS default. | FAILS the hard filter: plaintext DNS is poisoned in Iran (V3, >90% of censored domains to 10.10.34.34). The client's destinations are visible and forgeable. Out. |
| 2 | Fixed DoH resolver (e.g. a single hardcoded DoH endpoint) | Encrypted; simple to configure. | FAILS the hard filter: DoH endpoints are blocked in Iran (V8, Sept 2022); a fixed resolver is a single point of failure and a static signature. Out as the Iran method. |
| 3 | Resolve at the proxy (in-tunnel) for all tunnelled destinations; DNS-over-TCP to an unblocked resolver for bootstrap only (PICK) | No DNS query visible to the censor for tunnelled destinations; roughly halves RTT-fingerprint susceptibility; no fixed resolver to block; bootstrap uses DNS-over-TCP (which recovered to baseline after June 2025). | Bootstrap still needs a resolver before the tunnel exists (the residual exposure); the proxy becomes the DNS dependency (if the proxy is down, resolution is down — mitigated by the outage/queue behaviour, D21). |
| 4 | DNS-over-TCP to a rotating pool of public resolvers (no fixed resolver) | No single fixed resolver to block; encrypted-ish (TCP, not plaintext); works before the tunnel exists. | Queries are still visible to the censor (resolver names, query timing); the pool must be maintained against blocking; does not get the RTT-susceptibility halving that proxy resolution gives. |
| 5 | DNS-over-HTTPS to a rotating pool (DoH, no fixed endpoint) | Encrypted queries; no single endpoint. | DoH as a class is interfered with in Iran (V8 — DNS tampering + TLS interference against DoH endpoints); a rotating pool of DoH endpoints is still DoH, and the class-level interference applies. Weaker than proxy resolution on both counts. |

**Hard filters applied first** (license-compatible; MSRV set by Phase 1 and recorded; security: no known unpatched CVE, no custom crypto, audited libraries only — `03:§12.1`). A candidate failing any one is out, with no scoring. Plaintext DNS and fixed-DoH fail the hard filter (poisoned / blocked in Iran) and are out. All other candidates avoid a fixed resolver and plaintext, and pass the license/MSRV/CVE filters.

**Scoring.** Weights: correctness/safety .30, maintainability .20, performance .20, reversibility .15, effort .15; S = Σ(w·s); each criterion 0–5 (5 = best). Scores are **Opinion** unless a cell is labelled VERIFIED arithmetic or a cited Fact; the basis of each score is stated in the Pros/Cons above. Computed in Python (this generator):

| Alternative | Corr .30 | Maint .20 | Perf .20 | Rev .15 | Eff .15 | **S** |
|---|---:|---:|---:|---:|---:|---:|
| Do nothing — plaintext DNS via the system resolver | — | — | — | — | — | **EXCLUDED by hard filter — not scored** |
| Fixed DoH resolver (e.g. a single hardcoded DoH endpoint) | — | — | — | — | — | **EXCLUDED by hard filter — not scored** |
| Resolve at the proxy (in-tunnel) for all tunnelled destinations; DNS-over-TCP to an unblocked resolver for bootstrap only (PICK) | 4 | 4 | 3 | 4 | 3 | **3.65** |
| DNS-over-TCP to a rotating pool of public resolvers (no fixed resolver) | 3 | 3 | 3 | 3 | 3 | **3.00** |
| DNS-over-HTTPS to a rotating pool (DoH, no fixed endpoint) | 3 | 3 | 3 | 3 | 3 | **3.00** |

**Ranking:** 3.65 Resolve at the proxy (in-tunnel) for all tunnelled destinations; DNS-over-TCP to an unblocked resolver for bootstrap only (PICK) > 3.00 DNS-over-TCP to a rotating pool of public resolvers (no fixed resolver) > 3.00 DNS-over-HTTPS to a rotating pool (DoH, no fixed endpoint). Gap between top two: 0.65 points = 17.81% of the leader's score.

**Sensitivity (±20% per weight, other weights renormalised to sum 1.0):**

| Weight varied | −20% → S | +20% → S | Leader changes? |
|---|---|---|---|
| Correctness/safety | Resolve at the proxy (in-tunnel) for all tunnelled destinations; DNS-over-TCP to an unblocked resolver for bootstrap only (PICK): 3.6277 / DNS-over-TCP to a rotating pool of public resolvers (no fixed resolver): 3.0000 / DNS-over-HTTPS to a rotating pool (DoH, no fixed endpoint): 3.0000 | Resolve at the proxy (in-tunnel) for all tunnelled destinations; DNS-over-TCP to an unblocked resolver for bootstrap only (PICK): 3.6698 / DNS-over-TCP to a rotating pool of public resolvers (no fixed resolver): 3.0000 / DNS-over-HTTPS to a rotating pool (DoH, no fixed endpoint): 3.0000 | No |
| Maintainability | Resolve at the proxy (in-tunnel) for all tunnelled destinations; DNS-over-TCP to an unblocked resolver for bootstrap only (PICK): 3.6354 / DNS-over-TCP to a rotating pool of public resolvers (no fixed resolver): 3.0000 / DNS-over-HTTPS to a rotating pool (DoH, no fixed endpoint): 3.0000 | Resolve at the proxy (in-tunnel) for all tunnelled destinations; DNS-over-TCP to an unblocked resolver for bootstrap only (PICK): 3.6635 / DNS-over-TCP to a rotating pool of public resolvers (no fixed resolver): 3.0000 / DNS-over-HTTPS to a rotating pool (DoH, no fixed endpoint): 3.0000 | No |
| Performance | Resolve at the proxy (in-tunnel) for all tunnelled destinations; DNS-over-TCP to an unblocked resolver for bootstrap only (PICK): 3.6771 / DNS-over-TCP to a rotating pool of public resolvers (no fixed resolver): 3.0000 / DNS-over-HTTPS to a rotating pool (DoH, no fixed endpoint): 3.0000 | Resolve at the proxy (in-tunnel) for all tunnelled destinations; DNS-over-TCP to an unblocked resolver for bootstrap only (PICK): 3.6250 / DNS-over-TCP to a rotating pool of public resolvers (no fixed resolver): 3.0000 / DNS-over-HTTPS to a rotating pool (DoH, no fixed endpoint): 3.0000 | No |
| Reversibility | Resolve at the proxy (in-tunnel) for all tunnelled destinations; DNS-over-TCP to an unblocked resolver for bootstrap only (PICK): 3.6392 / DNS-over-TCP to a rotating pool of public resolvers (no fixed resolver): 3.0000 / DNS-over-HTTPS to a rotating pool (DoH, no fixed endpoint): 3.0000 | Resolve at the proxy (in-tunnel) for all tunnelled destinations; DNS-over-TCP to an unblocked resolver for bootstrap only (PICK): 3.6602 / DNS-over-TCP to a rotating pool of public resolvers (no fixed resolver): 3.0000 / DNS-over-HTTPS to a rotating pool (DoH, no fixed endpoint): 3.0000 | No |
| Effort | Resolve at the proxy (in-tunnel) for all tunnelled destinations; DNS-over-TCP to an unblocked resolver for bootstrap only (PICK): 3.6701 / DNS-over-TCP to a rotating pool of public resolvers (no fixed resolver): 3.0000 / DNS-over-HTTPS to a rotating pool (DoH, no fixed endpoint): 3.0000 | Resolve at the proxy (in-tunnel) for all tunnelled destinations; DNS-over-TCP to an unblocked resolver for bootstrap only (PICK): 3.6311 / DNS-over-TCP to a rotating pool of public resolvers (no fixed resolver): 3.0000 / DNS-over-HTTPS to a rotating pool (DoH, no fixed endpoint): 3.0000 | No |

**Tipping weight:** none — no single ±20% weight variation changes the leader. The ranking is robust to weight choice (it may still be fragile to score choice; see §12).

**Close-call handling (<10% margin):** the margin is 17.81%, outside the <10% band, so no benchmark is triggered by the close-call rule. See §12 for the tie-break evidence.

## 8. Expected benefits

Resolve-at-proxy + DNS-over-TCP bootstrap: the censor sees no DNS query for tunnelled destinations; RTT susceptibility roughly halves; no fixed resolver is a single point of failure; the bootstrap path uses a transport (TCP) that is forwarded and that recovered after the June 2025 shutdown.

## 9. Failure modes

Bootstrap resolver blocked on the owner's ISP (per-ISP heterogeneity, U1-i) → the client cannot resolve the server's name before the tunnel exists; mitigated by shipping the server's IP alongside its name in the profile (the profile is the credential — the IP is profile data, not a DNS query) and by the outage/queue behaviour. Proxy DNS fails → resolution fails with the tunnel; mitigated by treating it as a degraded mode, not a silent failure. A DoH-class interference extends to DNS-over-TCP → the bootstrap path is re-tested at M2 (`09:§5.3` Q8) and the method is revised by ADR.

**Consequence if wrong:** If resolve-at-proxy is wrong (the proxy's DNS answers are themselves poisoned or the proxy is a worse resolver than the client's), the cost is wrong answers for tunnelled destinations — detectable in the lab (M2 test against the Iran DoH-blocking evidence) and correctable by ADR before any release. If the bootstrap path is wrong, the cost is a client that cannot start in Iran — the highest-severity failure of this decision, which is why the profile ships the server IP as well as its name.

**Reversibility cost of undoing it:** High: the resolution method is a client configuration + session-layer rule. Switching methods costs an ADR + a client change; the profile data (server IP + name) is unchanged.

## 10. Cost / complexity

Specification cost: the method + bootstrap path in D2/Phase 2. Implementation cost: in-tunnel DNS (session layer) + DNS-over-TCP bootstrap. Test cost: M2 test against the Iran DoH-blocking evidence (`09:§5.3` Q8); no live probing from Iran (Tier C).

## 11. Experimental validation method

M2 test (`09:§5.3` Q8): in the lab, verify the client resolves tunnelled destinations through the proxy (no plaintext DNS leaves the client; no fixed DoH endpoint is contacted) and bootstraps via DNS-over-TCP. Falsifier: a packet capture showing a plaintext DNS query or a fixed-DoH connection from the client → the method is not implemented as specified. AT-E4-07 (RTT) secondary check: DNS-at-proxy vs DNS-local susceptibility (the ~2× factor) is confirmed in the lab topology where measurable. The method is re-tested whenever the Iran profile is revised (Tier A pulls, `phase1/03-iran-profile.md` §4.1).

## 12. Final decision + evidence

**Decision (PROPOSED): Resolve at the proxy (in-tunnel) for all tunnelled destinations; DNS-over-TCP to an unblocked resolver for bootstrap only**

PROPOSED: resolve at the proxy (inside the tunnel) for all tunnelled destinations; DNS-over-TCP to an unblocked resolver for bootstrap only (resolving the server's name before the tunnel exists); no fixed DoH resolver, no plaintext DNS. The profile ships the server's IP alongside its name so bootstrap does not depend on a resolver that may be blocked.

**Calculation that produced it:** S = 0.30·4 + 0.20·4 + 0.20·3 + 0.15·4 + 0.15·3 = 3.65, highest of the scored alternatives (rotating DoT pool and rotating DoH pool both 3.00). Margin 0.65 points = 17.8% — outside the close-call band.

**Confidence:** Opinion, MEDIUM (70%) — the constraint (no fixed DoH, no plaintext DNS) is Fact (HIGH, verified Iran capabilities); the method itself is 'I cannot verify a correct method from the Phase 0 sources' (Q8), so the pick is a judgment to be tested at M2. Calibration p = 0.70.

**Status / date / deciders:** PROPOSED — 2026-11-25 — decided by Abolfazl (owner) at the Phase 1 gate; this record proposes, it does not accept. A decision without its M2/M3 evidence is PROPOSED, not ACCEPTED. Decided at: M0 (specified) → M2 (tested against the Iran DoH-blocking evidence). Evidence that would settle it: M2 lab test; Tier A OONI pulls (Iran profile §4.1).

**Related items:** U1, U17; AT-E4-07 (secondary); M0, M2; P2; D1 (DNS-at-proxy halves RTT susceptibility), D21 (outage/queue when resolution fails).
