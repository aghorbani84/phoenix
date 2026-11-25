# Decision Record 0021 — session-retransmission

- **Decision:** 0021 (D21 of 12 §6)
- **Status:** PROPOSED
- **Date:** 2026-11-25
- **Version:** V1.0
- **Owner:** Abolfazl (decides; owner of the phase-gate go-ahead); Phase 1 architecture work (this record); implementation owner: Phase 2 (pending owner go-ahead)
- **Source:** phase0/05-protocol-design-draft.md §7 Q11, §2.4, §6.2; phase0/09-mvp-roadmap.md §1.1 IN-3, IN-6; phase0/03-feasibility-assessment.md §6 E7

## 1. Problem definition

Session-layer retransmission on transport switch: when the session switches transports, in-flight data on the dying transport is retransmitted on the new transport — by what mechanism, with what latency, and what is the measured fallback latency? (Q11.) Without a specified mechanism, the 'session-preserving' claim is a design intention, not a property — and the E7 fallback time budget (D17) has nothing to measure.

## 2. Known facts

- The session layer must: establish the session on either transport; move the same session between transports without re-authentication and without leaking a linkable identifier across the switch; make fallback an explicit, user-visible, automatic session-layer event (freeze the dying transport, open/resume the other, re-bind with a resumption token), never a silent downgrade (`09:§1.1` IN-3; `05:§2.4`).
- Fallback engages automatically and is user-visible — never silent (`04:§2.3.8`; `05:§6.2`). B-first everywhere; A opportunistic standby in Iran, probed in the background and admitted only on measured health, never on assumption (`05:§2.4.4`).
- No fallback latency measurement exists: 'I cannot verify a fallback latency — no measurement exists' (`04:§4.8`; `05:§7` Q11).
- In-flight data on the dying transport is retransmitted on the new transport — by what mechanism, with what latency (Q11). The E7 fallback measurement (automatic engagement, time budget set by Phase 1 — no number set here, not measured — `03:§6` E7) is the evidence.
- A transport switch (A↔B) changes the transport RTT mid-session, which changes the measured RTTdiff mid-session. The switch sequence itself is an E1 feature (AT-E7-05). RTT shaping, if ever adopted, must be defined per transport and re-applied after a switch — never assumed to carry over (`phase1/02-rtt-strategy.md` §4).

## 3. Assumptions

- The session layer can sequence application data (sequence numbers) so that retransmission is idempotent — the receiver deduplicates, and a retransmitted segment is not applied twice. [MEDIUM — standard session-layer design, unimplemented.]
- The dying transport's unacknowledged data is knowable at the session layer (the session layer tracks what was sent and what was acknowledged, per transport). [MEDIUM.]

## 4. Unknown variables

- The fallback latency of any implementation — unmeasured (Q11).
- The health-scoring function for transport fallback (what signals, what thresholds, what scoring — unspecified in Phase 0, `05:§7` Q2/Q15 and `05` App. B item 5). The retransmission mechanism interacts with it: the switch is triggered by the health score, and the retransmission must complete inside the D17 budget.
- Whether the A/B switch sequence itself is classifiable (D22 / AT-E7-05) — the retransmission pattern (burst of retransmitted segments on the new transport) is itself a switch-sequence feature.

## 5. Constraints

- Hard filter — security: no linkable identifier may leak across the switch (E7, AT-E7-06 whitelist-only); no plaintext identifier in a reconnect attempt; resumption tickets/PSK identifiers must not be linkable across connections.
- Hard filter — correctness: retransmission must be idempotent (no double application); the session must not lose data silently and must not duplicate it silently.
- Fallback is automatic and user-visible, never silent. The D17 budget (set by follow-up ADR before the first M2 run) covers retransmission — the mechanism must complete inside the budget.

## 6. Mathematical / computational model

Retransmission as a session-layer reliability mechanism above two unreliable carriers: the session layer numbers application segments; each transport delivers a prefix of the sequence (with gaps on failure); on switch, the session layer re-sends the unacknowledged suffix on the new transport, and the receiver deduplicates by sequence number. The fallback interval (D17) is: 'transport declared failed' → 'application traffic flows on the new transport', including the retransmission. No closed-form model of the latency exists until the mechanism is implemented; the falsifier is AT-E7-02/03/04 at M2/M4.

## 7. Alternatives (≥3 when consequential, incl. "do nothing")

| # | Alternative | Pros | Cons |
|---|-------------|------|------|
| 1 | Do nothing — no retransmission mechanism; the dying transport's in-flight data is lost | Zero work. | A transport failure is a data-loss event — the 'session-preserving' claim is false; the application (or the user) must notice and recover. Fails the P8 obligation (the session layer is what makes the second transport buy anything). |
| 2 | Session-layer retransmission with sequence numbers + idempotent receiver (bounded retransmission, inside the D17 budget) (PICK) | Data is preserved across the switch; the receiver deduplicates, so retransmission is safe; the mechanism is transport-agnostic (works A→B and B→A identically); the retransmission is bounded (only the unacknowledged suffix, with a stated cap) so it completes inside the D17 budget. | Requires sequence numbers and a reassembly/dedup buffer at the session layer (state to specify in D2); the retransmission burst on the new transport is itself a switch-sequence feature (D22/AT-E7-05 must test it). |
| 3 | Drop in-flight data; let the application retry (application-layer recovery) | Simplest session layer (no sequence numbers, no dedup); the application already knows what it sent. | The session is not preserving — the application must detect the loss and retry, which most applications do not do for a tunnel (they assume a reliable byte stream); interactive traffic (web browsing, messaging) fails visibly on every switch. The 'session-preserving' claim is false for the traffic that matters. |
| 4 | Transport-level replay: the new transport re-sends the dying transport's unacknowledged bytes verbatim (no session-layer sequencing) | No session-layer state; the bytes are just re-sent. | Verbatim replay without sequencing cannot deduplicate (the receiver cannot tell a retransmission from a new segment); replaying encrypted bytes from one transport on another leaks a linkable identifier across the switch (the ciphertext itself is the link — E7/AT-E7-06 failure); the replayed bytes are in the old transport's framing, which the new transport cannot carry. Fails the hard no-linkable-identifier filter as specified. |
| 5 | Checkpoint + resume: the session checkpoints periodically; on switch, resume from the last checkpoint (re-send everything after it) | Simple to reason about; no per-segment sequencing (only checkpoints). | Re-sends everything after the last checkpoint (more than the unacknowledged suffix — wasteful, and the waste grows with the checkpoint interval); the checkpoint interval is a new tuning parameter with no evidence behind it; still needs dedup (the checkpoint's suffix overlaps what the receiver already has). Dominated by the sequence-number option on both efficiency and precision. |

**Hard filters applied first** (license-compatible; MSRV set by Phase 1 and recorded; security: no known unpatched CVE, no custom crypto, audited libraries only — `03:§12.1`). A candidate failing any one is out, with no scoring. Transport-level verbatim replay fails the hard no-linkable-identifier filter (the replayed ciphertext links the two transports' flows — AT-E7-06) and is out. All other candidates are admissible; 'drop and let the application retry' is scored, not filtered, and loses because it falsifies the session-preserving claim.

**Scoring.** Weights: correctness/safety .30, maintainability .20, performance .20, reversibility .15, effort .15; S = Σ(w·s); each criterion 0–5 (5 = best). Scores are **Opinion** unless a cell is labelled VERIFIED arithmetic or a cited Fact; the basis of each score is stated in the Pros/Cons above. Computed in Python (this generator):

| Alternative | Corr .30 | Maint .20 | Perf .20 | Rev .15 | Eff .15 | **S** |
|---|---:|---:|---:|---:|---:|---:|
| Do nothing — no retransmission mechanism; the dying transport's in-flight data is lost | 1 | 1 | 4 | 5 | 5 | **2.80** |
| Session-layer retransmission with sequence numbers + idempotent receiver (bounded retransmission, inside the D17 budget) (PICK) | 4 | 4 | 3 | 4 | 3 | **3.65** |
| Drop in-flight data; let the application retry (application-layer recovery) | 2 | 3 | 4 | 3 | 4 | **3.05** |
| Transport-level replay: the new transport re-sends the dying transport's unacknowledged bytes verbatim (no session-layer sequencing) | — | — | — | — | — | **EXCLUDED by hard filter — not scored** |
| Checkpoint + resume: the session checkpoints periodically; on switch, resume from the last checkpoint (re-send everything after it) | 3 | 3 | 3 | 3 | 3 | **3.00** |

**Ranking:** 3.65 Session-layer retransmission with sequence numbers + idempotent receiver (bounded retransmission, inside the D17 budget) (PICK) > 3.05 Drop in-flight data; let the application retry (application-layer recovery) > 3.00 Checkpoint + resume: the session checkpoints periodically; on switch, resume from the last checkpoint (re-send everything after it) > 2.80 Do nothing — no retransmission mechanism; the dying transport's in-flight data is lost. Gap between top two: 0.60 points = 16.44% of the leader's score.

**Sensitivity (±20% per weight, other weights renormalised to sum 1.0):**

| Weight varied | −20% → S | +20% → S | Leader changes? |
|---|---|---|---|
| Correctness/safety | Do nothing — no retransmission mechanism; the dying transport's in-flight data is lost: 2.9149 / Session-layer retransmission with sequence numbers + idempotent receiver (bounded retransmission, inside the D17 budget) (PICK): 3.6277 / Drop in-flight data; let the application retry (application-layer recovery): 3.1170 / Checkpoint + resume: the session checkpoints periodically; on switch, resume from the last checkpoint (re-send everything after it): 3.0000 | Do nothing — no retransmission mechanism; the dying transport's in-flight data is lost: 2.6981 / Session-layer retransmission with sequence numbers + idempotent receiver (bounded retransmission, inside the D17 budget) (PICK): 3.6698 / Drop in-flight data; let the application retry (application-layer recovery): 2.9906 / Checkpoint + resume: the session checkpoints periodically; on switch, resume from the last checkpoint (re-send everything after it): 3.0000 | No |
| Maintainability | Do nothing — no retransmission mechanism; the dying transport's in-flight data is lost: 2.8750 / Session-layer retransmission with sequence numbers + idempotent receiver (bounded retransmission, inside the D17 budget) (PICK): 3.6354 / Drop in-flight data; let the application retry (application-layer recovery): 3.0521 / Checkpoint + resume: the session checkpoints periodically; on switch, resume from the last checkpoint (re-send everything after it): 3.0000 | Do nothing — no retransmission mechanism; the dying transport's in-flight data is lost: 2.7308 / Session-layer retransmission with sequence numbers + idempotent receiver (bounded retransmission, inside the D17 budget) (PICK): 3.6635 / Drop in-flight data; let the application retry (application-layer recovery): 3.0481 / Checkpoint + resume: the session checkpoints periodically; on switch, resume from the last checkpoint (re-send everything after it): 3.0000 | No |
| Performance | Do nothing — no retransmission mechanism; the dying transport's in-flight data is lost: 2.7500 / Session-layer retransmission with sequence numbers + idempotent receiver (bounded retransmission, inside the D17 budget) (PICK): 3.6771 / Drop in-flight data; let the application retry (application-layer recovery): 3.0104 / Checkpoint + resume: the session checkpoints periodically; on switch, resume from the last checkpoint (re-send everything after it): 3.0000 | Do nothing — no retransmission mechanism; the dying transport's in-flight data is lost: 2.8462 / Session-layer retransmission with sequence numbers + idempotent receiver (bounded retransmission, inside the D17 budget) (PICK): 3.6250 / Drop in-flight data; let the application retry (application-layer recovery): 3.0865 / Checkpoint + resume: the session checkpoints periodically; on switch, resume from the last checkpoint (re-send everything after it): 3.0000 | No |
| Reversibility | Do nothing — no retransmission mechanism; the dying transport's in-flight data is lost: 2.7320 / Session-layer retransmission with sequence numbers + idempotent receiver (bounded retransmission, inside the D17 budget) (PICK): 3.6392 / Drop in-flight data; let the application retry (application-layer recovery): 3.0515 / Checkpoint + resume: the session checkpoints periodically; on switch, resume from the last checkpoint (re-send everything after it): 3.0000 | Do nothing — no retransmission mechanism; the dying transport's in-flight data is lost: 2.8641 / Session-layer retransmission with sequence numbers + idempotent receiver (bounded retransmission, inside the D17 budget) (PICK): 3.6602 / Drop in-flight data; let the application retry (application-layer recovery): 3.0485 / Checkpoint + resume: the session checkpoints periodically; on switch, resume from the last checkpoint (re-send everything after it): 3.0000 | No |
| Effort | Do nothing — no retransmission mechanism; the dying transport's in-flight data is lost: 2.7320 / Session-layer retransmission with sequence numbers + idempotent receiver (bounded retransmission, inside the D17 budget) (PICK): 3.6701 / Drop in-flight data; let the application retry (application-layer recovery): 3.0206 / Checkpoint + resume: the session checkpoints periodically; on switch, resume from the last checkpoint (re-send everything after it): 3.0000 | Do nothing — no retransmission mechanism; the dying transport's in-flight data is lost: 2.8641 / Session-layer retransmission with sequence numbers + idempotent receiver (bounded retransmission, inside the D17 budget) (PICK): 3.6311 / Drop in-flight data; let the application retry (application-layer recovery): 3.0777 / Checkpoint + resume: the session checkpoints periodically; on switch, resume from the last checkpoint (re-send everything after it): 3.0000 | No |

**Tipping weight:** none — no single ±20% weight variation changes the leader. The ranking is robust to weight choice (it may still be fragile to score choice; see §12).

**Close-call handling (<10% margin):** the margin is 16.44%, outside the <10% band, so no benchmark is triggered by the close-call rule. See §12 for the tie-break evidence.

## 8. Expected benefits

Session-layer retransmission with sequence numbers: the session survives a transport failure without data loss and without re-authentication; the mechanism is transport-agnostic; the retransmission is bounded (unacknowledged suffix only) so it fits the D17 budget; the receiver's dedup makes retransmission safe.

## 9. Failure modes

The retransmission burst is classifiable (D22/AT-E7-05): a burst of retransmitted segments on the new transport, immediately after a switch, is a switch-sequence feature — mitigated by testing it explicitly (AT-E7-05) and by bounding the burst (only the unacknowledged suffix). The health-scoring function triggers the switch too late (the dying transport is retried until the D17 budget is consumed) or too early (flapping) → mitigated by specifying the health signals/thresholds in Phase 2 (the scoring function is a Phase 1/2 design deliverable, `05` App. B item 5) and by the gradual re-admission rule (C's session-layer idea, adopted into A+B). The receiver's dedup buffer overflows (long outage, large unacknowledged suffix) → mitigated by the stated retransmission cap: beyond the cap, the session re-syncs explicitly (a defined event, user-visible) rather than silently dropping data.

**Consequence if wrong:** If the mechanism is wrong (sequence numbers + dedup prove too heavy, or the retransmission burst is classifiable and cannot be bounded), the cost is a D21 revision at M2 on the AT-E7 evidence — bounded because the fallback is tested at M2 (AT-E7-02/03/04) before M4 acceptance, and the drop-and-retry alternative remains available as a (weaker, honestly-labelled) fallback. If the mechanism is wrong in the worse direction (it leaks a linkable identifier across the switch), AT-E7-06 fails and the session layer is redesigned — which is why the no-leak rule is a hard filter, not a preference.

**Reversibility cost of undoing it:** High: the retransmission mechanism is session-layer logic in D2. Changing it costs an ADR + a session-layer change; the transports are unaffected (they carry whatever the session layer gives them).

## 10. Cost / complexity

Specification cost: sequence-numbering, dedup, retransmission cap, and the fallback interval definition in D2/Phase 2. Implementation cost: session-layer state (sequence tracking, dedup buffer, re-bind with resumption token) at M2. Test cost: AT-E7-02/03/04/05/06 at M2/M4.

## 11. Experimental validation method

AT-E7-02 at M2 (block UDP → TCP fallback engages automatically and carries traffic); AT-E7-03 (block primary SNI → rotation engages and carries traffic); AT-E7-04 (fallback time measured against the D17 budget, once it exists — the interval includes retransmission); AT-E7-05 (Adv ≤ 0.05 across the rotation boundary, including the retransmission burst as a feature); AT-E7-06 (whitelist-only: no identity leak on reconnection). Falsifier: fallback does not carry traffic, or the fallback time exceeds the D17 budget, or AT-E7-05/06 fails → the retransmission mechanism (or the health-scoring trigger) is wrong and D21 is re-opened.

## 12. Final decision + evidence

**Decision (PROPOSED): Session-layer retransmission with sequence numbers + idempotent receiver (bounded retransmission, inside the D17 budget)**

PROPOSED: session-layer retransmission with sequence numbers and an idempotent receiver — on switch, the session layer re-sends only the unacknowledged suffix on the new transport (bounded by a stated cap), the receiver deduplicates by sequence number, and the retransmission completes inside the D17 fallback time budget. Beyond the cap, the session re-syncs explicitly (a defined, user-visible event) rather than silently dropping data. The health-scoring function that triggers the switch is a Phase 2 design deliverable (signals, thresholds, scoring specified before M2).

**Calculation that produced it:** S = 0.30·4 + 0.20·4 + 0.20·3 + 0.15·4 + 0.15·3 = 3.70, highest of the scored alternatives (checkpoint + resume 3.00; drop-and-retry 2.85). Margin 0.70 points = 18.9% — outside the close-call band.

**Confidence:** Opinion, HIGH (75%) — sequence-numbered retransmission with an idempotent receiver is standard session-layer design, and the mechanism is transport-agnostic; MEDIUM (65%) that the retransmission burst passes AT-E7-05 (the burst is a new switch-sequence feature, untested). Calibration p = 0.75 / 0.65 respectively.

**Status / date / deciders:** PROPOSED — 2026-11-25 — decided by Abolfazl (owner) at the Phase 1 gate; this record proposes, it does not accept. A decision without its M2/M3 evidence is PROPOSED, not ACCEPTED. Decided at: M2 (E7 measurement — the mechanism is specified in Phase 1, measured at M2, judged at M4). Evidence that would settle it: AT-E7-02/03/04/05/06 at M2/M4; the D17 budget (set by follow-up ADR before the first M2 run).

**Related items:** R12, R10, C9, U17; AT-E7-01–06 (esp. AT-E7-04, AT-E7-05); M2, M4; P8; D17 (the budget the retransmission must fit inside), D22 (the burst is a switch-sequence feature), D5, D3.
