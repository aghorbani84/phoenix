# Decision Record 0009 — key-rotation

- **Decision:** 0009 (D09 of 12 §6)
- **Status:** PROPOSED
- **Date:** 2026-11-25
- **Version:** V1.0
- **Owner:** Abolfazl (decides; owner of the phase-gate go-ahead); Phase 1 architecture work (this record); implementation owner: Phase 2 (pending owner go-ahead)
- **Source:** phase0/05-protocol-design-draft.md §7 Q7, §3.7; phase0/07-vps-architecture.md §6 (key lifecycle); phase0/03-feasibility-assessment.md §10.3; phase0/01-threat-models.md §7 (Kazakhstan state-root MITM)

## 1. Problem definition

Pinning, key rotation, and compromise recovery: how is the server-key pin set distributed and rotated, and how does a client recover after a server-key compromise without breaking all reconnections? The procedure is specified in Phase 1 and implemented in Phase 3 (`05:§7` Q7; `09:§5.3`).

## 2. Known facts

- The pin set must support at least two pins (current + next) during rotation (`05:§3.7`).
- CA key never leaves the operator machine; revocation is a single operation taking effect in minutes (`07:§6` — Opinion 70% target for the timing).
- No secrets in any repo, log, or public file ever (AT-VPS-22: any violation is a failed install; a key found in a log/repo/public file is compromised by definition — revoke and re-issue).
- Kazakhstan's state-root MITM (2019/2020, `01:§7`) makes pinning a first-class question, not an operational detail — a pinning design that cannot rotate is a future outage; a rotation design without a compromise path is a future breach with no recovery (`05:§7` Q7).
- The exact mechanism is unverified from the Phase 0 sources: 'I cannot verify a correct mechanism from the Phase 0 sources' (`05:§7` Q7).

## 3. Assumptions

- Two-pin rotation (current + next) is sufficient: clients pin both pins during the rotation window, so a server key change does not break reconnections. [MEDIUM — standard practice, not verified against a PHOENIX prototype.]
- Revocation taking effect 'in minutes' is achievable via the control channel (daemon → core profile update). [MEDIUM — Opinion 70% target in `07:§6`, unmeasured.]

## 4. Unknown variables

- A correct pin-distribution / key-rotation mechanism — 'I cannot verify' from Phase 0 (`05:§7` Q7; `05` App. B item 4). No mechanism is verified; this record specifies the procedure, and Phase 3 implementation + AT-VPS tests are the validation.
- Rotation intervals, quota values, alert thresholds, and reinstall time (U21 — VPS specifics ungrounded).
- Whether the revocation-in-minutes target holds on the real control path (unmeasured).

## 5. Constraints

- Hard filter — security: CA key never leaves the operator machine; no secrets in any repo/log/public file (AT-VPS-22); revocation must be a single operation.
- Tier C: keys/secrets stop Phase 1 work for one explicit owner ask each (`03:§12.5`). Rotation execution is a Tier C-adjacent operation — Phase 1 defines who approves it (`03:§10.3`).
- The server is disposable and replaceable: fast verified reinstall, restorable state, no irreplaceable secret that cannot be re-issued (`07:§2.3`).

## 6. Mathematical / computational model

Pin lifecycle as a state machine: pin set = {current, next} during rotation; clients accept either pin during the window; after the window, {next} becomes {current} and a new {next} is issued. Compromise: revoke the compromised pin (single operation, effect in minutes), issue a new pin, clients that pinned the compromised pin alone must re-fetch the pin set through the authenticated control channel — the recovery path must not require the compromised key. No closed-form model; the falsifier is the harness + VPS tests (AT-VPS series) at M5.

## 7. Alternatives (≥3 when consequential, incl. "do nothing")

| # | Alternative | Pros | Cons |
|---|-------------|------|------|
| 1 | Do nothing — no pinning, no rotation procedure; trust the TLS PKI alone | Zero work; no pin management. | No defence against a state-root MITM (Kazakhstan class); a server-key compromise has no recovery path; a key change breaks all clients. Fails the hard security filter (pinning is a first-class requirement, `05:§7` Q7). |
| 2 | Two-pin rotation (current + next), CA key on operator machine only, single-operation revocation (PICK) | Standard, auditable procedure; rotation without outage (clients accept either pin during the window); compromise recovery is a single revocation + re-issue; CA key never leaves the operator machine. | Pin distribution itself needs an authenticated channel (the control API); a client that misses the rotation window must re-fetch the pin set — the re-fetch path must be specified and must not depend on the compromised key. |
| 3 | Single-pin, hard cutover (rotate by replacing the pin, all clients update at once) | Simplest to specify; no window logic. | Any client that misses the cutover cannot reconnect (outage for that client); a compromise forces a flag day. Least reversible in practice (2/5). |
| 4 | No pinning; rely on DANE / CT-log monitoring instead | No pin distribution problem; leverages the public PKI's transparency. | DANE is not deployed for the donor/own-cert names PHOENIX uses; CT monitoring detects a mis-issued cert after the fact, it does not prevent a MITM in real time; does not answer the rotation question. I cannot verify a DANE deployment for PHOENIX's names. |
| 5 | Trust-on-first-use (TOFU) with no rotation procedure | Simple; no pin distribution infrastructure. | First connection is unauthenticated (MITM on first use succeeds silently); no defined rotation or compromise path — the exact failure Q7 names. Fails the hard security filter as a primary mechanism. |

**Hard filters applied first** (license-compatible; MSRV set by Phase 1 and recorded; security: no known unpatched CVE, no custom crypto, audited libraries only — `03:§12.1`). A candidate failing any one is out, with no scoring. 'Do nothing' and TOFU fail the hard security filter (pinning is a first-class requirement; a compromise without a recovery path is a future breach with no recovery). All other candidates use standard mechanisms and pass the license/MSRV/CVE filters.

**Scoring.** Weights: correctness/safety .30, maintainability .20, performance .20, reversibility .15, effort .15; S = Σ(w·s); each criterion 0–5 (5 = best). Scores are **Opinion** unless a cell is labelled VERIFIED arithmetic or a cited Fact; the basis of each score is stated in the Pros/Cons above. Computed in Python (this generator):

| Alternative | Corr .30 | Maint .20 | Perf .20 | Rev .15 | Eff .15 | **S** |
|---|---:|---:|---:|---:|---:|---:|
| Do nothing — no pinning, no rotation procedure; trust the TLS PKI alone | — | — | — | — | — | **EXCLUDED by hard filter — not scored** |
| Two-pin rotation (current + next), CA key on operator machine only, single-operation revocation (PICK) | 4 | 4 | 3 | 4 | 3 | **3.65** |
| Single-pin, hard cutover (rotate by replacing the pin, all clients update at once) | 3 | 3 | 3 | 2 | 4 | **3.00** |
| No pinning; rely on DANE / CT-log monitoring instead | 2 | 3 | 3 | 3 | 3 | **2.70** |
| Trust-on-first-use (TOFU) with no rotation procedure | — | — | — | — | — | **EXCLUDED by hard filter — not scored** |

**Ranking:** 3.65 Two-pin rotation (current + next), CA key on operator machine only, single-operation revocation (PICK) > 3.00 Single-pin, hard cutover (rotate by replacing the pin, all clients update at once) > 2.70 No pinning; rely on DANE / CT-log monitoring instead. Gap between top two: 0.65 points = 17.81% of the leader's score.

**Sensitivity (±20% per weight, other weights renormalised to sum 1.0):**

| Weight varied | −20% → S | +20% → S | Leader changes? |
|---|---|---|---|
| Correctness/safety | Two-pin rotation (current + next), CA key on operator machine only, single-operation revocation (PICK): 3.6277 / Single-pin, hard cutover (rotate by replacing the pin, all clients update at once): 3.0000 / No pinning; rely on DANE / CT-log monitoring instead: 2.7447 | Two-pin rotation (current + next), CA key on operator machine only, single-operation revocation (PICK): 3.6698 / Single-pin, hard cutover (rotate by replacing the pin, all clients update at once): 3.0000 / No pinning; rely on DANE / CT-log monitoring instead: 2.6604 | No |
| Maintainability | Two-pin rotation (current + next), CA key on operator machine only, single-operation revocation (PICK): 3.6354 / Single-pin, hard cutover (rotate by replacing the pin, all clients update at once): 3.0000 / No pinning; rely on DANE / CT-log monitoring instead: 2.6875 | Two-pin rotation (current + next), CA key on operator machine only, single-operation revocation (PICK): 3.6635 / Single-pin, hard cutover (rotate by replacing the pin, all clients update at once): 3.0000 / No pinning; rely on DANE / CT-log monitoring instead: 2.7115 | No |
| Performance | Two-pin rotation (current + next), CA key on operator machine only, single-operation revocation (PICK): 3.6771 / Single-pin, hard cutover (rotate by replacing the pin, all clients update at once): 3.0000 / No pinning; rely on DANE / CT-log monitoring instead: 2.6875 | Two-pin rotation (current + next), CA key on operator machine only, single-operation revocation (PICK): 3.6250 / Single-pin, hard cutover (rotate by replacing the pin, all clients update at once): 3.0000 / No pinning; rely on DANE / CT-log monitoring instead: 2.7115 | No |
| Reversibility | Two-pin rotation (current + next), CA key on operator machine only, single-operation revocation (PICK): 3.6392 / Single-pin, hard cutover (rotate by replacing the pin, all clients update at once): 3.0309 / No pinning; rely on DANE / CT-log monitoring instead: 2.6907 | Two-pin rotation (current + next), CA key on operator machine only, single-operation revocation (PICK): 3.6602 / Single-pin, hard cutover (rotate by replacing the pin, all clients update at once): 2.9709 / No pinning; rely on DANE / CT-log monitoring instead: 2.7087 | No |
| Effort | Two-pin rotation (current + next), CA key on operator machine only, single-operation revocation (PICK): 3.6701 / Single-pin, hard cutover (rotate by replacing the pin, all clients update at once): 2.9691 / No pinning; rely on DANE / CT-log monitoring instead: 2.6907 | Two-pin rotation (current + next), CA key on operator machine only, single-operation revocation (PICK): 3.6311 / Single-pin, hard cutover (rotate by replacing the pin, all clients update at once): 3.0291 / No pinning; rely on DANE / CT-log monitoring instead: 2.7087 | No |

**Tipping weight:** none — no single ±20% weight variation changes the leader. The ranking is robust to weight choice (it may still be fragile to score choice; see §12).

**Close-call handling (<10% margin):** the margin is 17.81%, outside the <10% band, so no benchmark is triggered by the close-call rule. See §12 for the tie-break evidence.

## 8. Expected benefits

Two-pin rotation: rotation without outage, compromise recovery as a single operation, CA key containment (operator machine only), and a server that stays disposable/replaceable (no irreplaceable secret). The procedure is specified in Phase 1 and implemented in Phase 3, so M5 tests it (AT-VPS series) instead of discovering it.

## 9. Failure modes

Pin distribution channel compromised → attacker distributes their own pin set; mitigated by the control API's authentication (D15) and by the CA key never leaving the operator machine. Client misses the rotation window → cannot reconnect; mitigated by the re-fetch path (specified in D2/Phase 3, not depending on the compromised key). Revocation does not take effect in minutes → the Opinion-70% target fails; measured at M5, and the target is revised by decision with the measured evidence, not silently. Secret leaks into a repo/log → AT-VPS-22: failed install, revoke and re-issue by definition.

**Consequence if wrong:** If the two-pin procedure is wrong (e.g. the rotation window is too short for the owner's clients, or the re-fetch path depends on the compromised key), the cost is client outages on rotation or an unrecoverable compromise — both are Phase 3/M5 test findings, not production failures, because the procedure is specified now and tested at M5 before any release.

**Reversibility cost of undoing it:** High: the pin set is profile data, not protocol. Changing the rotation procedure costs an ADR + a Phase 3 implementation change; issued pins can be revoked and re-issued. The CA key itself is never in the protocol, so rotating the CA is an operator action, not a protocol change.

## 10. Cost / complexity

Specification cost: the procedure in D2/Phase 3 (pin set format, rotation window, re-fetch path, revocation operation, approver). Implementation cost: Phase 3 (key lifecycle in `07:§6`, control API). Test cost: AT-VPS series at M5, including AT-VPS-22 (zero secrets).

## 11. Experimental validation method

Harness + VPS tests at M5: AT-VPS-01–22 (`11:§11`), including installer verification (data-plane port answers a valid test credential and refuses an invalid one; panel not publicly reachable — hard stop if it is; backup integrity verified). Rotation test (Phase 3): rotate the server key with a client holding the old pin set — the client must reconnect without user action during the window; then revoke the old pin — the client must move to the new pin. Compromise test: revoke a pin as a single operation and measure time-to-effect (target: minutes, Opinion 70% — measured, not assumed). Falsifier: a rotation that breaks reconnections, or a revocation that does not take effect, fails the procedure and re-opens D9.

## 12. Final decision + evidence

**Decision (PROPOSED): Two-pin rotation (current + next), CA key on operator machine only, single-operation revocation**

PROPOSED: two-pin rotation (current + next) during the rotation window; CA key never leaves the operator machine; revocation is a single operation taking effect in minutes (target, Opinion 70%, to be measured at M5); pin distribution through the authenticated control API (D15); re-fetch path specified in Phase 3 and not dependent on the compromised key; rotation approval defined (Tier C-adjacent — who approves it is recorded in the Phase 3 procedure).

**Calculation that produced it:** S = 0.30·4 + 0.20·4 + 0.20·3 + 0.15·4 + 0.15·3 = 3.65, highest of the scored alternatives (single-pin hard cutover 3.05; DANE/CT 2.75). Margin 0.60 points = 16.4% — outside the close-call band.

**Confidence:** Opinion, MEDIUM (70%) — the procedure is standard practice, but 'I cannot verify a correct mechanism from the Phase 0 sources' (`05:§7` Q7); the revocation-in-minutes target is Opinion 70% (`07:§6`). The M5 tests are the evidence. Calibration p = 0.70.

**Status / date / deciders:** PROPOSED — 2026-11-25 — decided by Abolfazl (owner) at the Phase 1 gate; this record proposes, it does not accept. A decision without its M2/M3 evidence is PROPOSED, not ACCEPTED. Decided at: M0 (specified) → M5 (implemented and tested). Evidence that would settle it: AT-VPS-01–22 at M5, including the rotation and revocation tests.

**Related items:** U17, U21, R13; AT-VPS-01–22 (esp. AT-VPS-22); M0, M5; P11; D15 (control API carries pin distribution), D14 (admin panel), D8 (server variant — the pinned key belongs to the chosen variant).
