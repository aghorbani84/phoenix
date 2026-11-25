# PHOENIX — Calibration Log (skeleton, Phase 0)

Purpose: track the calibration of confidence claims across PHOENIX work, per the
V2.0 scientific layer and the standing rule (attach p to every non-trivial
claim; Brier target < 0.20).

## Confidence levels (V2.0)

- VERIFIED — reproducible evidence.
- HIGH — strong supporting evidence, not fully verified.
- MEDIUM — reasonable inference.
- LOW — insufficient evidence.
- UNKNOWN — not established.

Mapping to the existing labels used in the Phase 0 documents: VERIFIED ≈ Fact
with reproducible evidence; HIGH ≈ Fact with strong support; MEDIUM ≈ Theory;
LOW ≈ weak Theory/Opinion; UNKNOWN ≈ "I cannot verify this".

## Brier score

Brier = mean((p − outcome)²) over resolved claims, where p is the stated
probability (0–1) and outcome is 1 if the claim proved true, 0 if false.
Standing target: Brier < 0.20. A claim resolves only when evidence settles it;
unresolved claims stay open and are not scored.

## Entry format

| Date | Claim (file:line) | p | Level | Resolved (date) | Outcome (1/0) | (p−outcome)² | Notes |
|------|-------------------|---|-------|-----------------|---------------|--------------|-------|
| — | — | — | — | — | — | — | — |

## Log

### Phase 1 predictions (recorded 2026-11-25; resolve at M2–M4)

| Date | Claim (file:line) | p | Level | Resolved (date) | Outcome (1/0) | (p−outcome)² | Notes |
|------|-------------------|---|-------|-----------------|---------------|--------------|-------|
| 2026-11-25 | Matrix ranking is robust to ±20% weight changes (B leads in all 10 variations) — `phase1/01-architecture-selection.md` §1.4 | 0.75 | HIGH | — | — | — | Recomputed in Python in Phase 1; resolves when M2/M3 evidence justifies (or fails to justify) a cell revision. |
| 2026-11-25 | Matrix ranking is fragile to scores: a single ±1 change on any correctness cell flips the leader; ±1 on any .20-weight cell ties it — `phase1/01-architecture-selection.md` §1.5 | 0.70 | MEDIUM | — | — | — | Arithmetic (VERIFIED); the prediction is that M2/M3 evidence WILL move at least one cell. Resolves at M4 revisit-or-stand. |
| 2026-11-25 | B's core mitigation (Vision padding) survives the Xue 2024 encapsulated-TLS classifier (AT-E1-03 at M3) — `phase1/01-architecture-selection.md` §3.3 F2 | 0.60 | MEDIUM | — | — | — | If AT-E1-03 fails, B correctness 4→3 flips the matrix to A. Falsifier pre-registered. |
| 2026-11-25 | A (QUIC-native) meets E3 with a large margin at M2 AND AT-E1-06 passes → D5 revisited — `phase1/01-architecture-selection.md` §3.3 F1 | 0.50 | MEDIUM | — | — | — | Tie at M2 keeps B. Resolves at M2/M4. |
| 2026-11-25 | Cross-layer RTT: no deployable mitigation exists for the MVP (D1 = R5) — `phase1/02-rtt-strategy.md` §1 | 0.85 | HIGH | — | — | — | Xue 2025's own evaluation found delayed ACK and multiplexing limited. Resolves only if R1/R2/R3 falsifier experiments pass. |
| 2026-11-25 | Iran: QUIC remains 100% blocked in steady state after June 2025 (no recovery) — `phase1/03-iran-profile.md` V6 | 0.90 | HIGH | — | — | — | FOCI 2026 (Scans 2–3, June–July 2025). Resolves on the next fresh Iran measurement. |
| 2026-11-25 | A permissive licence (Apache-2.0 or MIT) fits better than copyleft/dual for PHOENIX — `phase1/10-license-recommendation.md` | 0.70 | MEDIUM | — | — | — | Owner's decision (D18, Tier C); resolves when the owner picks. |

No claims recorded yet from Phase 0. Phase 0 claims live in the documents under
`~/workspace/phoenix/phase0/`; resolution is a Phase 1+ activity (measurements
from the D4 harness and E1–E7 tests settle them).
