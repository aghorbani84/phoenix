# Decision Record 0024 — phoenix-rebirth

- **Decision:** 0024 (D24 — addendum to the 23 Phase 1 decisions; added after Phase 1 completion, per the owner's words 2026-11-25 23:40 Tehran)
- **Status:** PROPOSED
- **Date:** 2026-11-25
- **Version:** V1.0
- **Owner:** Abolfazl (decides; stated the requirement); implementation owner: Phase 2 (pending owner go-ahead)
- **Source:** owner's message 2026-11-25 23:40 Tehran: «مثل اسمش باید اینجوری باشه که هربار پروتکل رو dpi بکشه دوباره از تخم در بیاد از دیوار اتشین به راحتی بتونه عبور کنه» — "like its name, every time DPI kills the protocol it must rise from the egg again and pass through the firewall easily." Related: phase1/02-rtt-strategy.md (D1/R5), phase1/07-harness-design.md (E7), decisions 0003 (D3 cover+fallback), 0005 (D5 transport), 0017 (D17 E7 budget), 0021 (D21 session retransmission).

## 1. Problem definition

PHOENIX must behave like its namesake: when DPI (or any censor mechanism) kills a protocol instance — RST injection, IP:port block, SNI block, probe-triggered block — the system must automatically regenerate (a new observable identity: transport, cover, session parameters) and re-establish the connection through the firewall, without the user doing anything. The question is whether "rebirth" is a first-class Phase 2 requirement with a measured budget and falsifier, or only an aspiration. It must be decided now because it shapes the session layer (D21), the fallback design (D3), the E7 budget (D17), and the harness (new kill→rebirth tests).

## 2. Known facts

- Censor kills are real and documented: a single 1-byte packet can trigger a 120–180 s IP:port block (Wu 2023, phase0/01 §7); RST injection, SNI filtering, and probe-triggered blocking are in the D1 adversary matrix (phase0/01 §8).
- Iran: QUIC dropped to near zero in Sept 2022 and is still 100% blocked after the June 2025 shutdown lifted while TCP recovered (phase1/03). A kill that takes down UDP does not take down TCP — transport diversity is already the reason B is the default (D5).
- Fallback exists as design, not as a measured capability: D3 (real TLS 1.3 default, real QUIC standby, fallback B→A→queue), D5 (B default, A in parallel), D21 (sequence numbers + idempotent receiver, bounded, inside the D17 budget). None of it has run — no prototype, no harness, no corpus (Phase 1 is docs only).
- Rebirth does NOT cure the RTT gap: cross-layer RTT (Xue NDSS 2025, >70% per-visit TPR at 1% FPR) is independent of all payload obfuscation, and D1 records it as an ACCEPTED LIMITATION (R5) with the F1 NO-GO condition. A reborn session is reborn into the same timing side-channel. [Fact, VERIFIED from the cited sources.]
- "Easily" is the owner's word for the goal, not a measured property. No measurement of rebirth exists. [Fact.]

## 3. Assumptions

- A kill is detectable by the client (RST, timeout, probe failure, sudden throughput collapse) quickly enough for rebirth to matter. [MEDIUM — detection is designed in D21/E7, unimplemented; falsified if kills are silent and slow to detect.]
- Regeneration changes the observable signature enough that the reborn session is not killed by the same rule immediately (new transport/cover/session identity ≠ same fingerprint). [UNKNOWN — the core claim of this decision; must be measured, not assumed. Falsified if the censor's rule keys on something regeneration does not change (e.g. destination IP, RTT profile, or a whitelist).]
- The user tolerates automatic regeneration without asking (the client retries on its own, within a budget). [HIGH — the owner's words imply automatic rebirth; the budget itself is the D17 question.]
- Rebirth is compatible with "no silent downgrade" (D5): regeneration must not silently drop to a weaker transport or weaker crypto. [HIGH — stated as a hard constraint in §5; a rebirth that weakens security is not a rebirth, it is a downgrade.]

## 4. Unknown variables

- Time-to-rebirth and success rate against real kills — no numbers exist; set by the M2/M3 harness (see §11).
- Which kills trigger rebirth vs. which trigger "wait and tell the user" (e.g. whitelist-only shutdown, where no transport helps — phase0/01 Iran finding). The trigger set is undefined.
- The rebirth budget (max attempts, max time, backoff) — D17 deliberately did not set the E7 number; this decision inherits that gap.
- Whether the censor's kill rule survives regeneration (the §3 UNKNOWN above) — settled only by kill→rebirth tests in the harness.

## 5. Constraints

- Phase 1 is docs only: this record proposes the requirement; it authorises no code, no prototype. Phase 2 needs the owner's separate go-ahead.
- Standard vetted crypto only; no custom crypto; rebirth must never weaken crypto, authentication, or forward secrecy to "get through faster" (Zen invariants I1–I8, phase0/08).
- No silent downgrade between transports, ever (D5); fallback/rebirth is explicit and user-visible (log/diagnostics), even when automatic.
- Rebirth is NOT a release criterion by this record alone — whether it gates the MVP is the owner's word (it interacts with F1: if the owner requires RTT resistance as a release criterion, Phase 2 does not start, and rebirth does not change that).
- Tier C: any test against infrastructure not owned is out of scope; rebirth is validated against simulated kills in the lab harness only.

## 6. Mathematical / computational model

Rebirth as a loop: detect kill (time t_d) → regenerate identity (t_r) → re-establish session (t_e) → total recovery T = t_d + t_r + t_e. Success per attempt p; attempts bounded by budget B (max attempts N, max wall time T_max, backoff). Expected time to a working session ≈ T / p for small N (geometric), and the requirement is falsifiable as: P(working session within T_max and N attempts) ≥ threshold, measured over simulated kills in the harness. No closed-form model of p exists — p depends on the censor's rule, which is UNKNOWN (§3). The model therefore does not predict p; it defines what to measure. Sanity check: if p ≈ 0 (the kill rule keys on something regeneration does not change), no budget makes rebirth work — the loop must then stop and report, not retry forever (Zen: no endless reconnect loops, phase0/08).

## 7. Alternatives (≥3 when consequential, incl. "do nothing")

| # | Alternative | Pros | Cons |
|---|-------------|------|------|
| 1 | Do nothing — fallback only, as D3/D5 specify (transport switch on failure, no identity regeneration) | No new mechanism; already designed; cheapest. | A kill that targets the session/identity (not just the transport) kills the fallback too — the client switches transport and is killed again by the same rule; no "rise again" property. |
| 2 | Rebirth as specified (PICK — PROPOSED): automatic kill detection → identity regeneration (transport + cover + session params) → re-establish, session-preserving where possible, inside the D17 budget, never weakening crypto | Directly implements the owner's stated identity for the project; turns each kill into a measured event with a recovery number; bounds cost via the D17 budget. | New mechanism (detection + regeneration + re-establishment) to design, test, and maintain; the core claim (regeneration defeats the kill rule) is UNKNOWN until measured; risks endless retry loops if the budget is wrong. |
| 3 | Rebirth with manual user trigger (client reports the kill, user taps reconnect) | No automatic retry risk; simplest to reason about; user stays in control. | Not "rise again" — it is a manual reconnect; useless when the user is asleep or the kill repeats every few minutes; contradicts the owner's words. |
| 4 | Rebirth limited to transport rotation only (switch B↔A, no cover/session regeneration) | Smallest new mechanism; reuses D3 fallback. | Fails exactly when the kill keys on identity rather than transport — which is the common DPI case (SNI, fingerprint, IP:port); barely different from alternative 1. |

**Hard filters applied first** (license-compatible; MSRV set by Phase 1 and recorded; security: no known unpatched CVE, no custom crypto, audited libraries only — `03:§12.1`). All alternatives pass the hard filters (no custom crypto is involved in any of them). No alternative is filter-excluded; the choice is on the scored evidence and the owner's stated requirement.

**Scoring.** Weights: correctness/safety .30, maintainability .20, performance .20, reversibility .15, effort .15; S = Σ(w·s); each criterion 0–5 (5 = best). Scores are **Opinion** unless labelled otherwise.

| Alternative | Corr .30 | Maint .20 | Perf .20 | Rev .15 | Eff .15 | **S** |
|---|---:|---:|---:|---:|---:|---:|
| Do nothing — fallback only (D3/D5) | 2 | 4 | 3 | 5 | 5 | **3.50** |
| Rebirth as specified (PICK — PROPOSED) | 4 | 3 | 4 | 3 | 3 | **3.50** |
| Rebirth with manual user trigger | 3 | 4 | 2 | 4 | 4 | **3.30** |
| Rebirth limited to transport rotation only | 3 | 4 | 3 | 4 | 4 | **3.50** |

**Ranking:** tie at 3.50 between do-nothing, rebirth-as-specified, and transport-rotation-only; manual trigger 3.30. The score does NOT pick the winner — the pick rests on the owner's stated identity requirement (the project's name and his words), not on the arithmetic. Sensitivity: ±20% on the correctness weight flips the leader between do-nothing (correctness −20% → 3.63 vs rebirth 3.46) and rebirth (correctness +20% → rebirth 3.54 vs do-nothing 3.37). The ranking is fragile to the correctness score — which is expected: the value of rebirth is exactly the correctness question (does regeneration defeat the kill rule?), and that is UNKNOWN until measured.

**Close-call handling (<10% margin):** the margin is 0% — a tie. The refined-spec tie-break (≤30-minute benchmark; tied → more reversible wins) does not apply cleanly here because there is no prototype to benchmark; the tie is broken by the owner's direction instead, and recorded as such. The M2/M3 kill→rebirth measurements are the evidence that will re-open or confirm the pick.

## 8. Expected benefits

Rebirth as specified: each censor kill becomes a bounded, measured recovery (time-to-working-session, attempts used) instead of a dead connection; the system degrades gracefully under active blocking rather than failing once; the property is named, specified, and testable, so "like its name" is a requirement with a falsifier, not a slogan. Do nothing: zero new cost, but no rise-again property. Manual trigger: control, but not automatic. Transport-only rotation: cheap, but fails the common DPI case.

## 9. Failure modes

Regeneration does not defeat the kill rule (the §3 UNKNOWN resolves against us): rebirth burns the budget and the session stays dead — cost is the wasted attempts plus user-visible flapping; mitigated by the budget cap and the stop-and-report rule (§6). Kill detection is slow or silent: T is dominated by t_d and the user experiences a long outage before rebirth even starts; mitigated by detection design in D21/E7. Rebirth loops forever: budget mis-set or kill rule persistent (e.g. whitelist-only shutdown, where no transport helps) — mitigated by the hard stop (max attempts/time, then report; Zen: no endless reconnect loops). Rebirth weakens security to get through (e.g. drops to a weaker cover or skips authentication): forbidden by §5 — a rebirth that does this is a defect, not a feature; caught by the harness (E1/E2 run against reborn sessions, not just fresh ones). RTT gate (F1): rebirth does not address Gap 1; if the owner requires RTT resistance as a release criterion, Phase 2 does not start, and this requirement does not override that.

**Consequence if wrong:** If rebirth is wrong (regeneration never defeats a real kill rule), the cost is the Phase 2 design + harness work on detection/regeneration, bounded by the D17 budget discipline and the M2/M3 measurements that falsify it early. If rebirth is right but unmeasured (shipped without the kill→rebirth tests), the cost is a false claim — "rises again" asserted without evidence — which the V2.0 rules forbid. The requirement therefore ships with its tests or not at all.

**Reversibility cost of undoing it:** Low — rebirth is a session-layer behaviour on top of D3/D21; removing it leaves the D3 fallback intact. The cost of not adopting it is the project's defining property staying a slogan.

## 10. Cost / complexity

Implementation cost: a detection + regeneration + re-establishment loop in the session layer (D21), plus harness kill→rebirth tests (new AT-E7 tests, see §11). Maintenance cost: the trigger set and budget need tuning against real kills; the mechanism must be re-validated whenever a transport or cover changes. Risk: the UNKNOWN (§3) — the mechanism may not work against the kill rules that matter most, which is discovered at M2/M3, not at release.

## 11. Experimental validation method

New harness tests (to be added to the AT-E7 family in phase0/11-acceptance-tests.md at M2): simulated kills in the lab — RST injection, IP:port block, SNI block, probe-triggered block — followed by rebirth attempts; measure per kill type: detection time t_d, regeneration time t_r, re-establishment time t_e, attempts used, P(working session within T_max and N attempts). Pass/fail = the thresholds set by the follow-up ADR that sets the D16/D17 numbers before the first M2 run (anti-fitting rule: thresholds are set before measurement, not after). E1/E2 distinguishers run against reborn sessions as well as fresh ones (a reborn session must not be more classifiable than a fresh one). NOT RUN — no harness, no prototype, no corpus. Runs at M2/M3. Falsifier: if P(working session within budget) is not better than a fresh manual connection against the same simulated kills, the requirement fails and is re-opened (or the trigger set/budget is revised on evidence).

## 12. Final decision + evidence

**Decision (PROPOSED): rebirth as a first-class Phase 2 requirement**

PROPOSED (not accepted): on a detected kill, the client automatically regenerates its observable identity (transport + cover + session parameters) and re-establishes the session, session-preserving where possible, inside the D17 budget, never weakening crypto, with a hard stop-and-report when the budget is exhausted. Evidence: the owner's stated requirement (2026-11-25 23:40 — the project's name and his words); the documented reality of censor kills (Wu 2023; D1 matrix); the existing fallback design (D3/D5/D21) it builds on. No measurement evidence exists — the requirement is provisional by design and its falsifier is in §11. Revisit condition: the M2/M3 kill→rebirth measurements; if they falsify the core claim (§3 UNKNOWN), the requirement is re-opened and the pick revisits alternatives 1/3/4 on evidence. It does NOT override F1 (RTT as a release criterion still gates Phase 2) and it does NOT become a release criterion by this record alone — that is the owner's word.

**Calculation that produced it:** S = 3.50 (tie) — see §7. The pick does not follow from the score; it follows from the owner's direction. The score's tie is itself informative: rebirth costs real complexity for a real benefit, and the deciding evidence is the measurement in §11, not the arithmetic.

**Confidence:** Opinion, HIGH (85%) that this is the correct requirement to record (it is the owner's words + the project's name); LOW (35%) that regeneration defeats the kill rules that matter most in Iran until measured — the §3 UNKNOWN is the reason for the lower second confidence. Calibration p = 0.85 for the requirement as stated; p = 0.35 for its effectiveness against real kills pre-measurement.

**Status / date / deciders:** PROPOSED — 2026-11-25 — decided by Abolfazl (owner); this record proposes, it does not accept. Decided at: Phase 1 addendum (owner direction) → M2 (harness) → M3 (kill→rebirth evidence) → M4 (revisit-or-stand). Evidence that would settle it: AT-E7 kill→rebirth tests at M2/M3.

**Related items:** D1 (F1 overrides), D3, D5, D17, D21, D22; E7; AT-E7-02 (extend with kill→rebirth tests); M2, M3, M4; phase0/01 §7–§8 (kill mechanisms); phase0/08 (Zen: no endless reconnect loops).

**Push manifest note:** addendum to Phase 1 — include this file in the push manifest (§8 of 12-phase1-report.md) alongside the 23 Phase 1 decision records.
