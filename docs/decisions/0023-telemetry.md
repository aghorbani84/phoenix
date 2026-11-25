# Decision Record 0023 — telemetry

- **Decision:** 0023 (D23 of 12 §6)
- **Status:** PROPOSED
- **Date:** 2026-11-25
- **Version:** V1.0
- **Owner:** Abolfazl (decides; owner of the phase-gate go-ahead); Phase 1 architecture work (this record); implementation owner: Phase 2 (pending owner go-ahead)
- **Source:** phase0/03-feasibility-assessment.md §8.4 row 7 (no telemetry by default — Opinion 90%); phase0/12-priority-roadmap.md §6 D23; phase0/06-gui-architecture.md §3.4, §7 (Zen invariants; AT-ZEN-I07 — no secrets in logs/reports)

## 1. Problem definition

Telemetry: does PHOENIX collect any telemetry (usage, diagnostics, crash reports) by default? The Phase 0 position is no telemetry by default (`03:§8.4` row 7, Opinion 90%) — a Phase 1 architecture decision to record (D23). The decision must cover the client, the daemon, the VPS, and diagnostics — and the boundary between 'no telemetry' and 'diagnostics the user explicitly shares'.

## 2. Known facts

- No telemetry by default (`03:§8.4` row 7, Opinion 90%) — a Phase 1 architecture decision to record.
- AT-ZEN-I07: no secrets in logs/reports. The UI never shows DNS query names, server IPs, or full key fingerprints (`06:§3.4`); the Statistics page shows a query count only; diagnostics scrubbing is a daemon responsibility (`06:§3.1` — the Go daemon does diagnostics scrubbing).
- The daemon classifies outage signals; the core reports signals (e.g. 'handshake timed out after N attempts', 'unexpected RST pattern'), and the daemon classifies them into outage kinds — classification is policy; the core must not decide what the user is told (`06:§3.6`).
- No secrets in any repo, log, or public file ever (AT-VPS-22 logic). Log redaction for tokens/keys is part of the VPS security process (`03:§10.3`).
- User safety depends on enforcement practice, not packets (`03:§2.2`, Opinion 90%) — telemetry that leaves the device is a user-safety question, not merely a privacy preference, in the owner's environment.

## 3. Assumptions

- Local-only diagnostics (on-device, never transmitted) are not 'telemetry' — the distinction is transmission, not collection. [HIGH — the standard meaning of the term; the record states it explicitly so the boundary is testable.]
- A user who explicitly shares a diagnostic bundle understands what it contains because the bundle is shown to them before sharing (scrubbed, with secrets redacted). [MEDIUM — the sharing UI is a Phase 4 deliverable.]

## 4. Unknown variables

- Whether the owner ever wants any opt-in telemetry (e.g. anonymous crash reports) — unasked; the default is no, and any opt-in is a future decision, not part of this record.
- What a diagnostic bundle contains exactly (which counters, which logs, which scrubbing rules) — a Phase 4 (GUI/diagnostics) and Phase 3 (VPS) detail; this record sets the boundary (local-only by default; explicit sharing only), not the bundle's contents.

## 5. Constraints

- Hard filter — privacy/security: no telemetry by default; no transmission of usage, diagnostics, or crash data without the user's explicit, per-instance consent. A candidate that transmits by default is out.
- Hard filter — security: no secrets in logs/reports (AT-ZEN-I07); no DNS query names, server IPs, or full key fingerprints in any UI surface or diagnostic output (`06:§3.4`).
- Tier C: any test against infrastructure PHOENIX does not own is out of scope; telemetry infrastructure would be PHOENIX-owned or not exist — there is no telemetry endpoint in the MVP.

## 6. Mathematical / computational model

Telemetry as a transmission question, not a collection question: local-only diagnostics (counters, scrubbed logs, on-device statistics) are collected and never transmitted; telemetry is transmission to any party (including PHOENIX). The default is: collect locally, transmit never, share only when the user explicitly shares a diagnostic bundle they have seen. The boundary is testable: a packet capture of the client must show no telemetry traffic; the diagnostics bundle must be inspectable before sharing.

## 7. Alternatives (≥3 when consequential, incl. "do nothing")

| # | Alternative | Pros | Cons |
|---|-------------|------|------|
| 1 | Do nothing — no telemetry decision recorded; each component decides for itself | Zero work now. | A component (crash reporter, analytics SDK, update checker) could transmit by default without a decision — the exact failure the record exists to prevent. The UI process could acquire a dependency that phones home (the `06:§10` reasoning: the profile is the credential, and every library in the UI process is one dependency away from the keys). Fails the M0 obligation. |
| 2 | No telemetry by default; local-only diagnostics; explicit, per-instance, user-reviewed sharing only (PICK) | Strongest privacy posture for the owner's environment; the boundary (transmission, not collection) is testable by packet capture; diagnostics still exist for debugging (local-only, scrubbed); sharing is explicit, per-instance, and user-reviewed — not a standing consent. | No aggregate usage data to guide development (no install counts, no failure rates across the user base); debugging a remote user's problem requires them to share a bundle (a manual step). |
| 3 | Opt-out telemetry (collect and transmit usage/diagnostics by default; user can disable) | Aggregate data for development; crash reports arrive without user action. | FAILS the hard filter: transmits by default in an environment where transmission is a user-safety question (`03:§2.2`); an opt-out that the user never finds is consent that was never given. Out. |
| 4 | Opt-in telemetry (transmit only with explicit standing consent) | Respects consent; still yields aggregate data from consenting users. | A standing consent, once given, is easily forgotten — and in the owner's environment the cost of a forgotten consent is user safety, not merely privacy; the telemetry endpoint itself becomes infrastructure PHOENIX must operate and secure (a new attack surface and a new data-protection obligation). Weaker than the pick, which requires per-instance, user-reviewed sharing. |
| 5 | No diagnostics at all (not even local-only) | Smallest possible surface; nothing to leak, nothing to scrub. | Debugging becomes guesswork (no counters, no scrubbed logs, no outage classification history); the daemon's diagnostics-scrubbing responsibility (`06:§3.1`) has nothing to scrub; the outage-classification feature (AT-GUI-06, honest outage signalling) loses its evidence base. Overkill: local-only diagnostics carry no transmission risk. |

**Hard filters applied first** (license-compatible; MSRV set by Phase 1 and recorded; security: no known unpatched CVE, no custom crypto, audited libraries only — `03:§12.1`). A candidate failing any one is out, with no scoring. Opt-out telemetry fails the hard privacy/security filter (transmits by default; consent never given) and is out. All other candidates are admissible; opt-in telemetry is scored, not filtered, and loses to per-instance sharing on the safety analysis.

**Scoring.** Weights: correctness/safety .30, maintainability .20, performance .20, reversibility .15, effort .15; S = Σ(w·s); each criterion 0–5 (5 = best). Scores are **Opinion** unless a cell is labelled VERIFIED arithmetic or a cited Fact; the basis of each score is stated in the Pros/Cons above. Computed in Python (this generator):

| Alternative | Corr .30 | Maint .20 | Perf .20 | Rev .15 | Eff .15 | **S** |
|---|---:|---:|---:|---:|---:|---:|
| Do nothing — no telemetry decision recorded; each component decides for itself | 1 | 1 | 3 | 5 | 5 | **2.60** |
| No telemetry by default; local-only diagnostics; explicit, per-instance, user-reviewed sharing only (PICK) | 4 | 4 | 3 | 5 | 4 | **3.95** |
| Opt-out telemetry (collect and transmit usage/diagnostics by default; user can disable) | — | — | — | — | — | **EXCLUDED by hard filter — not scored** |
| Opt-in telemetry (transmit only with explicit standing consent) | 3 | 4 | 3 | 4 | 3 | **3.35** |
| No diagnostics at all (not even local-only) | 2 | 3 | 3 | 4 | 4 | **3.00** |

**Ranking:** 3.95 No telemetry by default; local-only diagnostics; explicit, per-instance, user-reviewed sharing only (PICK) > 3.35 Opt-in telemetry (transmit only with explicit standing consent) > 3.00 No diagnostics at all (not even local-only) > 2.60 Do nothing — no telemetry decision recorded; each component decides for itself. Gap between top two: 0.60 points = 15.19% of the leader's score.

**Sensitivity (±20% per weight, other weights renormalised to sum 1.0):**

| Weight varied | −20% → S | +20% → S | Leader changes? |
|---|---|---|---|
| Correctness/safety | Do nothing — no telemetry decision recorded; each component decides for itself: 2.7021 / No telemetry by default; local-only diagnostics; explicit, per-instance, user-reviewed sharing only (PICK): 3.9468 / Opt-in telemetry (transmit only with explicit standing consent): 3.3723 / No diagnostics at all (not even local-only): 3.0638 | Do nothing — no telemetry decision recorded; each component decides for itself: 2.5094 / No telemetry by default; local-only diagnostics; explicit, per-instance, user-reviewed sharing only (PICK): 3.9528 / Opt-in telemetry (transmit only with explicit standing consent): 3.3302 / No diagnostics at all (not even local-only): 2.9434 | No |
| Maintainability | Do nothing — no telemetry decision recorded; each component decides for itself: 2.6667 / No telemetry by default; local-only diagnostics; explicit, per-instance, user-reviewed sharing only (PICK): 3.9479 / Opt-in telemetry (transmit only with explicit standing consent): 3.3229 / No diagnostics at all (not even local-only): 3.0000 | Do nothing — no telemetry decision recorded; each component decides for itself: 2.5385 / No telemetry by default; local-only diagnostics; explicit, per-instance, user-reviewed sharing only (PICK): 3.9519 / Opt-in telemetry (transmit only with explicit standing consent): 3.3750 / No diagnostics at all (not even local-only): 3.0000 | No |
| Performance | Do nothing — no telemetry decision recorded; each component decides for itself: 2.5833 / No telemetry by default; local-only diagnostics; explicit, per-instance, user-reviewed sharing only (PICK): 3.9896 / Opt-in telemetry (transmit only with explicit standing consent): 3.3646 / No diagnostics at all (not even local-only): 3.0000 | Do nothing — no telemetry decision recorded; each component decides for itself: 2.6154 / No telemetry by default; local-only diagnostics; explicit, per-instance, user-reviewed sharing only (PICK): 3.9135 / Opt-in telemetry (transmit only with explicit standing consent): 3.3365 / No diagnostics at all (not even local-only): 3.0000 | No |
| Reversibility | Do nothing — no telemetry decision recorded; each component decides for itself: 2.5258 / No telemetry by default; local-only diagnostics; explicit, per-instance, user-reviewed sharing only (PICK): 3.9175 / Opt-in telemetry (transmit only with explicit standing consent): 3.3299 / No diagnostics at all (not even local-only): 2.9691 | Do nothing — no telemetry decision recorded; each component decides for itself: 2.6699 / No telemetry by default; local-only diagnostics; explicit, per-instance, user-reviewed sharing only (PICK): 3.9806 / Opt-in telemetry (transmit only with explicit standing consent): 3.3689 / No diagnostics at all (not even local-only): 3.0291 | No |
| Effort | Do nothing — no telemetry decision recorded; each component decides for itself: 2.5258 / No telemetry by default; local-only diagnostics; explicit, per-instance, user-reviewed sharing only (PICK): 3.9485 / Opt-in telemetry (transmit only with explicit standing consent): 3.3608 / No diagnostics at all (not even local-only): 2.9691 | Do nothing — no telemetry decision recorded; each component decides for itself: 2.6699 / No telemetry by default; local-only diagnostics; explicit, per-instance, user-reviewed sharing only (PICK): 3.9515 / Opt-in telemetry (transmit only with explicit standing consent): 3.3398 / No diagnostics at all (not even local-only): 3.0291 | No |

**Tipping weight:** none — no single ±20% weight variation changes the leader. The ranking is robust to weight choice (it may still be fragile to score choice; see §12).

**Close-call handling (<10% margin):** the margin is 15.19%, outside the <10% band, so no benchmark is triggered by the close-call rule. See §12 for the tie-break evidence.

## 8. Expected benefits

No telemetry by default + local-only diagnostics + explicit per-instance sharing: the strongest privacy posture for the owner's environment; diagnostics still exist where they are needed (on-device, scrubbed, for debugging and honest outage signalling); the boundary is testable (packet capture shows no telemetry traffic; the bundle is inspectable before sharing); no telemetry endpoint to operate, secure, or defend.

## 9. Failure modes

A dependency phones home (analytics/crash SDK in the UI process) → mitigated by the dependency discipline (no third-party UI dependencies in the daemon; the UI's dependencies are reviewed — `06:§10` reasoning) and by the packet-capture test (any telemetry traffic is a failed build). A diagnostic bundle contains a secret or an identifier despite scrubbing → mitigated by AT-ZEN-I07 (no secrets in logs/reports) and by the user reviewing the bundle before sharing (the review step is the last line of defence, not the first). The user shares a bundle under pressure (e.g. asked to share diagnostics by a third party) → mitigated by the bundle being scrubbed by default (the sensitive fields are absent whether or not the user checks) and by the sharing being per-instance (no standing consent to exploit).

**Consequence if wrong:** If the pick is wrong (aggregate telemetry would have materially improved the product — e.g. crash rates across the user base), the cost is slower, anecdote-driven debugging — bounded, because local diagnostics + explicit sharing still give the team the data for any specific problem, just not the aggregate. If the pick is wrong the other way (even local-only diagnostics prove too risky), the cost is a further restriction to 'no diagnostics at all' — a one-ADR change, since the boundary is a policy, not an architecture.

**Reversibility cost of undoing it:** Highest (5/5): telemetry is a policy, not an architecture. Changing it costs an ADR; no code change is needed to stop transmitting (there is no transmitter to remove), and adding an opt-in later costs a transmitter + an endpoint + a consent UI — which is why the default is set now, before any transmitter exists.

## 10. Cost / complexity

Specification cost: this record (the boundary) + the diagnostics-scrubbing rules (Phase 4 GUI/diagnostics, Phase 3 VPS). Implementation cost: local-only diagnostics (counters, scrubbed logs — already the daemon's responsibility, `06:§3.1`); no telemetry endpoint, no transmitter, no consent UI for a standing opt-in. Test cost: packet-capture test (no telemetry traffic) at M6; AT-ZEN-I07 (no secrets in logs/reports) at M7.

## 11. Experimental validation method

Packet-capture test at M6: the client (and daemon, and VPS admin panel) emit no telemetry traffic — any transmission to a non-user destination that is not the tunnel itself or an explicit user action is a failed build. AT-ZEN-I07 at M7 (no secrets in logs/reports). Diagnostics-bundle review: the bundle is shown to the user (scrubbed) before sharing; the scrubbing rules are tested by grep (no secret, DNS name, server IP, or full key fingerprint in the bundle). Falsifier: any telemetry traffic, or any secret/identifier in a diagnostic bundle, fails the decision and re-opens D23.

## 12. Final decision + evidence

**Decision (PROPOSED): No telemetry by default; local-only diagnostics; explicit, per-instance, user-reviewed sharing only**

PROPOSED: no telemetry by default. Local-only diagnostics (counters, scrubbed logs, on-device statistics) are collected and never transmitted. Sharing happens only when the user explicitly shares a diagnostic bundle, per instance, after reviewing it (scrubbed: no secrets, no DNS query names, no server IPs, no full key fingerprints). There is no telemetry endpoint in the MVP, no standing opt-in, and no aggregate usage collection. Any future opt-in telemetry is a separate decision, not part of this record.

**Calculation that produced it:** S = 0.30·4 + 0.20·4 + 0.20·3 + 0.15·5 + 0.15·4 = 3.95, highest of the scored alternatives (opt-in telemetry 3.35; no diagnostics at all 3.00). Margin 0.60 points = 15.2% — outside the close-call band.

**Confidence:** Opinion, HIGH (90%) — the 'no telemetry by default' position is the project's own (`03:§8.4` row 7, Opinion 90%), and the transmission-vs-collection boundary is the standard meaning of the term. Calibration p = 0.90.

**Status / date / deciders:** PROPOSED — 2026-11-25 — decided by Abolfazl (owner) at the Phase 1 gate; this record proposes, it does not accept. A decision without its M2/M3 evidence is PROPOSED, not ACCEPTED. Decided at: M0. Evidence that would settle it: the M6 packet-capture test (no telemetry traffic); AT-ZEN-I07 at M7. Any future opt-in telemetry is a separate decision.

**Related items:** AT-ZEN-I07; AT-GUI-02, AT-GUI-06; M0, M6, M7; P12, P13; D13 (the UI must not acquire a phoning-home dependency), D15 (diagnostics scrubbing is the daemon's responsibility).
