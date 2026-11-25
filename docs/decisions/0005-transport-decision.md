# Decision Record 0005 — transport-decision

- **Decision:** 0005 (D05 of 12 §6)
- **Status:** PROPOSED
- **Date:** 2026-11-25
- **Version:** V1.0
- **Owner:** Abolfazl (decides; owner of the phase-gate go-ahead); Phase 1 architecture work (this record); implementation owner: Phase 2 (pending owner go-ahead)
- **Source:** phase0/04-architecture-proposals.md §7.5 (recommendation); phase1/01-architecture-selection.md §1 (re-score), §2 (benchmark question), §3 (pick), §5.2 (draft D5); phase0/12-priority-roadmap.md §6 D5; phase0/09-mvp-roadmap.md §2 M2/M4

## 1. Problem definition

Which transport is the Phase 1 default — B (TLS 1.3 + real-browser mimicry over TCP/443), A (QUIC-native over UDP/443), or C (multipath) — and which transports are built and measured, in what order? The decision must be made now because M1 builds the default transport first, and building the wrong default first spends the scarcest Phase 2 milestone on a transport the owner's environment may not support.

## 2. Known facts

- Re-scored matrix: B 3.60 > A 3.40 > C 2.90; gap 5.56% (close call). [VERIFIED arithmetic; Opinion scores.] Independently re-verified by this record's generator.
- QUIC/HTTP-3 near zero in Iran from Sept 2022 while TCP/443 + TLS continued; QUIC still 100% blocked after the June 2025 shutdown lifted while TCP recovered (secondary, 70%). [Fact, HIGH / MEDIUM respectively.]
- B's cover is the best-measured in the set (REALITY/ShadowTLS/WebTunnel deployed, field-reported); A's cover has a demonstrated large-scale break (QUIC SNI decryption, Zohaib 2025) plus the unmitigated 1,216 B hybrid-share fingerprint. [Fact, HIGH.]
- B is the most reversible (4/5) and lowest-effort (4/5) alternative; C the least reversible (2/5) and highest-effort (2/5). [Fact of the matrix.]
- No benchmark ran in Phase 1 (01 §2): the minimal A/B prototype transports do not exist (Phase 2, hard-gated), and `tc netem` is unavailable on this VM (`sch_netem` absent, `/lib/modules/` nonexistent). No performance evidence supports or opposes the pick. [Fact, VERIFIED this session, 01 §2.2–§2.3.]
- The two TCP loss labs disagree sharply on magnitude (`04:§3.8`); the Hysteria2/QUIC lab shape is blog data (60%). [Fact about the literature.]
- Benchmark time-box reading is open (Q1, `05:§7`): ≤30 minutes — runtime-only or wall-clock including harness setup? This record uses runtime as the working reading and records the ambiguity; the question is answered before the M2 benchmark runs.

## 3. Assumptions

- Iran availability is the deciding factor for the default (not for the system's overall capability — A is retained precisely because B alone has no QUIC-grade loss resilience). [Opinion, HIGH — grounded in the owner's environment, `01:§4`.]
- A tied benchmark keeps B (more reversible wins, `03:§12.1`). [MEDIUM — method per the refined spec.]
- The session-layer design (01 §4.1) can make fallback session-preserving, so that 'B default, A in parallel' is one system rather than two side-by-side protocols. [MEDIUM — design claim from `05:§2.4`, unimplemented; falsified if the session layer cannot be built as specified at M2.]
- The owner does not require RTT resistance as a release criterion, or the RTT strategy (D1) is decided separately — if both fail together, the verdict flips to NO-GO (F1), which overrides this pick. [UNKNOWN — owner decision pending.]

## 4. Unknown variables

- The M2 benchmark result (A vs B on clean / 1% / 5% loss + UDP-blocked run) — the transports do not exist yet. Settles F1.
- AT-E1-03 at M3 (Vision padding vs Xue 2024) — settles F2 and can flip the matrix.
- AT-E1-06 at M3 (handshake-size KS per transport, PQC present) — settles F3 and the D6 placement decision.
- Whether the Phase 2 harness host will have `tc netem` (this VM does not) — settles whether the benchmark can run as designed at M2 or needs an equivalent impairment mechanism.

## 5. Constraints

- Phase 1 is docs only: this record proposes the pick; it authorises no code, no repo, no prototype. Phase 2 needs the owner's separate go-ahead, and the repo needs the owner's go-ahead on name + creation route (hard gate, `docs/operating-rules.md` §12).
- Standard vetted crypto only; hybrid X25519MLKEM768, never pure ML-KEM on the handshake path in Phase 1; no SLH-DSA on the handshake path (`03:§4.4`).
- No silent downgrade between transports, ever (`04:§2.3.8`); fallback is explicit and user-visible.
- Tier C: release publishing, keys/secrets, and any test against infrastructure not owned are out of scope for this decision.

## 6. Mathematical / computational model

The decision model is the D4 matrix: S = Σ(w·s); B 3.60, A 3.40, C 2.90. The pick additionally applies the close-call rule: gap <10% → ≤30-minute benchmark; tied → more reversible wins (B). No closed-form model of the RTT trade-off exists; the pick does not model Gap 1 and must not be read as addressing it (D1 is a separate decision). Sanity check: the 0.20 gap is smaller than any single criterion's weight, so the model cannot certify the pick — it can only rank Opinion scores, which is why the pick is provisional and evidence-gated.

## 7. Alternatives (≥3 when consequential, incl. "do nothing")

| # | Alternative | Pros | Cons |
|---|-------------|------|------|
| 1 | Do nothing — no default transport chosen; Phase 2 waits | No commitment on unmeasured scores. | M1 cannot start (it builds the default transport first); the P7 obligation is unmet; the decision is deferred, not avoided. |
| 2 | B default, A measured in parallel, C deferred (PICK — PROPOSED, provisional) | Only alternative with no UDP single point of failure in Iran; best-measured cover; most reversible / lowest effort; A retained so the close call is measured, not assumed. | B's core mitigation (Vision padding) unmeasured (U4); B degrades with TCP under loss (magnitude disputed between labs); no benchmark evidence yet — the pick is provisional. |
| 3 | A default, B as fallback | Best loss resilience in shape; lowest handshake latency; most natural RTT-shaping control (QUIC owns ACK timing); most testable state machine (quinn-proto sans-io). | UDP-only default fails outright where UDP is blocked — in Iran, the owner's environment, that is intermittent-to-total failure; QUIC SNI decryptable; 1,216 B share in the Initial; no in-protocol TCP fallback. |
| 4 | C default (multipath now) | Only alternative with session-preserving transport fallback (Gap 5); highest performance ceiling (5/5); transport diversity raises the censor's cost if session establishment is possible on any transport. | Highest effort, largest code surface, lowest maintainability (2/5) and reversibility (2/5); scheduler itself a fingerprint surface, unproven unclassifiable; no deployed precedent at this scope; commits the least reversible choice before A and B are measured. |

**Hard filters applied first** (license-compatible; MSRV set by Phase 1 and recorded; security: no known unpatched CVE, no custom crypto, audited libraries only — `03:§12.1`). A candidate failing any one is out, with no scoring. All three transports pass the hard filters (standard vetted crypto, audited libraries, no custom crypto). No alternative is filter-excluded; the choice is on the scored evidence, and the pick is explicitly provisional pending the M2 benchmark and M3 E1 evidence.

**Scoring.** Weights: correctness/safety .30, maintainability .20, performance .20, reversibility .15, effort .15; S = Σ(w·s); each criterion 0–5 (5 = best). Scores are **Opinion** unless a cell is labelled VERIFIED arithmetic or a cited Fact; the basis of each score is stated in the Pros/Cons above. Computed in Python (this generator):

| Alternative | Corr .30 | Maint .20 | Perf .20 | Rev .15 | Eff .15 | **S** |
|---|---:|---:|---:|---:|---:|---:|
| Do nothing — no default transport chosen; Phase 2 waits | 1 | 1 | 3 | 5 | 5 | **2.60** |
| B default, A measured in parallel, C deferred (PICK — PROPOSED, provisional) | 4 | 3 | 3 | 4 | 4 | **3.60** |
| A default, B as fallback | 3 | 4 | 4 | 3 | 3 | **3.40** |
| C default (multipath now) | 3 | 2 | 5 | 2 | 2 | **2.90** |

**Ranking:** 3.60 B default, A measured in parallel, C deferred (PICK — PROPOSED, provisional) > 3.40 A default, B as fallback > 2.90 C default (multipath now) > 2.60 Do nothing — no default transport chosen; Phase 2 waits. Gap between top two: 0.20 points = 5.56% of the leader's score.

**Sensitivity (±20% per weight, other weights renormalised to sum 1.0):**

| Weight varied | −20% → S | +20% → S | Leader changes? |
|---|---|---|---|
| Correctness/safety | Do nothing — no default transport chosen; Phase 2 waits: 2.7021 / B default, A measured in parallel, C deferred (PICK — PROPOSED, provisional): 3.5745 / A default, B as fallback: 3.4255 / C default (multipath now): 2.8936 | Do nothing — no default transport chosen; Phase 2 waits: 2.5094 / B default, A measured in parallel, C deferred (PICK — PROPOSED, provisional): 3.6226 / A default, B as fallback: 3.3774 / C default (multipath now): 2.9057 | No |
| Maintainability | Do nothing — no default transport chosen; Phase 2 waits: 2.6667 / B default, A measured in parallel, C deferred (PICK — PROPOSED, provisional): 3.6250 / A default, B as fallback: 3.3750 / C default (multipath now): 2.9375 | Do nothing — no default transport chosen; Phase 2 waits: 2.5385 / B default, A measured in parallel, C deferred (PICK — PROPOSED, provisional): 3.5769 / A default, B as fallback: 3.4231 / C default (multipath now): 2.8654 | No |
| Performance | Do nothing — no default transport chosen; Phase 2 waits: 2.5833 / B default, A measured in parallel, C deferred (PICK — PROPOSED, provisional): 3.6250 / A default, B as fallback: 3.3750 / C default (multipath now): 2.8125 | Do nothing — no default transport chosen; Phase 2 waits: 2.6154 / B default, A measured in parallel, C deferred (PICK — PROPOSED, provisional): 3.5769 / A default, B as fallback: 3.4231 / C default (multipath now): 2.9808 | No |
| Reversibility | Do nothing — no default transport chosen; Phase 2 waits: 2.5258 / B default, A measured in parallel, C deferred (PICK — PROPOSED, provisional): 3.5876 / A default, B as fallback: 3.4124 / C default (multipath now): 2.9278 | Do nothing — no default transport chosen; Phase 2 waits: 2.6699 / B default, A measured in parallel, C deferred (PICK — PROPOSED, provisional): 3.6117 / A default, B as fallback: 3.3883 / C default (multipath now): 2.8738 | No |
| Effort | Do nothing — no default transport chosen; Phase 2 waits: 2.5258 / B default, A measured in parallel, C deferred (PICK — PROPOSED, provisional): 3.5876 / A default, B as fallback: 3.4124 / C default (multipath now): 2.9278 | Do nothing — no default transport chosen; Phase 2 waits: 2.6699 / B default, A measured in parallel, C deferred (PICK — PROPOSED, provisional): 3.6117 / A default, B as fallback: 3.3883 / C default (multipath now): 2.8738 | No |

**Tipping weight:** none — no single ±20% weight variation changes the leader. The ranking is robust to weight choice (it may still be fragile to score choice; see §12).

**Close-call handling (<10% margin):** the margin is 5.56%, inside the <10% band, so the refined-spec rule fires: a ≤30-minute benchmark breaks the tie; if still tied, the more reversible option wins (`03:§12.1`). See §12 for the tie-break evidence.

## 8. Expected benefits

B default + A in parallel: a default that works in the owner's environment on day one; a second transport measured in parallel so the 5.56% close call is resolved by data at M2/M3 rather than by argument; C's session-layer ideas preserved (01 §4.1) without C's scheduler/reliability cost; the cheapest reversal if M3 falsifies B.

## 9. Failure modes

B's mitigation fails (F2): AT-E1-03 fails at M3 → B correctness 4→3 is the justified revision → A leads 3.40 > 3.30. Cost: M1 work on B partially wasted; mitigated by B's reversibility and the transport-agnostic session layer. The pick is re-opened, not silently kept. A's benchmark margin (F1): A meets E3 with a large margin and passes AT-E1-06 → revisit in an ADR; a tie keeps B. Both transports fail a loss scenario C would survive: C's revival condition (01 §3.4) — the pick's fallback (sequential session-preserving switch) is shown insufficient; C is revived as a Phase 2+ decision. RTT gate (F4): R5 accepted limitation + owner requires RTT resistance → NO-GO for Phase 2, overriding the pick. Harness host lacks netem: the M2 benchmark cannot run as designed on this VM; the harness needs a host with `sch_netem` or an equivalent. Cost: schedule, not correctness — the benchmark is deferred, and the pick stays provisional longer.

**Consequence if wrong:** If the pick is wrong (A should have been the default), the cost is one Phase 2 milestone of build order: M1 builds B first, and the M4 revisit-or-stand flips the default to A. The transports themselves are reusable (they are A and B); only the default order changes. If the pick is wrong in the worse direction (B is detectable and A is not, and the session layer cannot make the switch), the cost is the M1/M2 work on B plus the M3 falsification — which is exactly what the provisional status and the parallel-A measurement are designed to bound.

**Reversibility cost of undoing it:** Bounded by B's reversibility (4/5) and the session layer's transport-independence; the transports themselves are reusable, only the default order changes. The cost of not picking (do nothing) is a stalled Phase 2.

## 10. Cost / complexity

Implementation cost: B first (lowest effort, 4/5), A in parallel (one transport, mature crates quinn / rustls), C deferred (avoids the highest-effort build). Maintenance cost: two transports + a session layer, versus one — the price of measuring the close call instead of assuming it.

## 11. Experimental validation method

M2 ≤30-minute benchmark (AT-E3-01–03 + AT-E7-02): minimal A (quinn) vs minimal B (rustls), clean / 1% / 5% loss + UDP-blocked run; pass/fail = E3 thresholds (rho ≥ 0.90 clean / ≥ 1.0 lossy vs WireGuard) + UDP-blocked fallback; ≥10 runs, mean ± CI. NOT RUN — blocked in Phase 1 (01 §2: transports do not exist; `tc netem` unavailable on this VM). Runs at M2 on a host with netem. M3 E1/E2: AT-E1-01–03 (all three distinguishers, worst governs), AT-E1-06 (handshake-size KS per transport), AT-E2-01–05 (≥1,000 probes/type per transport). Calibration AT-E1-04 runs first. NOT RUN — no harness, no corpus, no prototype. M4: revisit-or-stand recorded as an ADR on the M2 + M3 evidence. Falsifiers F1–F4 in 01 §3.3 (benchmark margin; B's core mitigation fails; A's cover fails; RTT gate).

## 12. Final decision + evidence

**Decision (PROPOSED): B default, A measured in parallel, C deferred**

PROPOSED (provisional — NOT accepted): B as the Phase 1 default transport, A as the flagged second transport built and measured in parallel at M2, C deferred-not-rejected. Evidence: the re-scored matrix (01 §1: B 3.60 > A 3.40 > C 2.90, close call stands, no cell changes on current evidence); Iran availability (QUIC near zero since Sept 2022, still 100% blocked after June 2025, TCP recovered); cover maturity; reversibility/effort. No benchmark evidence exists — the pick is provisional by design. Revisit condition: F1–F4 in 01 §3.3; the pick is revisited at M4 (revisit-or-stand ADR) on the M2 benchmark + M3 E1 evidence, and re-opened earlier if AT-E1-03 fails.

**Calculation that produced it:** S(B) = 3.60, S(A) = 3.40, S(C) = 2.90 (see D4 for the full arithmetic, independently re-verified by this record's generator). Gap B–A = 0.20 = 5.56% < 10% → close call → ≤30-minute benchmark trigger fires; tied → more reversible wins (B). The score alone does not certify the pick (the 0.20 gap is smaller than any single criterion's weight); the pick is a statement about which default to measure first, not a claim that B is better.

**Confidence:** Opinion, HIGH (80%) that this is the correct provisional pick on current evidence; MEDIUM (60%) that it survives M2/M3 measurement — the §1.5 score fragility in 01 is the reason for the lower second confidence. Calibration p = 0.80 for the provisional pick; p = 0.60 for its survival to M4.

**Status / date / deciders:** PROPOSED — 2026-11-25 — decided by Abolfazl (owner) at the Phase 1 gate; this record proposes, it does not accept. A decision without its M2/M3 evidence is PROPOSED, not ACCEPTED. Decided at: M0 (ADR) → M2 (benchmark) → M4 (revisit-or-stand recorded). Evidence that would settle it: P7 benchmark at M2; AT-E1-03/06 at M3.

**Related items:** U4, U5, U8, R5, R7, C17, C18; AT-E1-01–03, AT-E1-06, AT-E2-01–05, AT-E3-01–03, AT-E7-02; M0, M2, M3, M4; P7; D4, D3, D6, D21, D22, D1 (F4 overrides the pick).
