# Decision Record Template (V2.0, 12 fields)

File: `docs/decisions/NNNN-title.md` (NNNN = zero-padded sequence; title in
kebab-case). One decision per file. Copy this template; delete nothing; write
"I cannot verify this" or "not applicable (reason)" rather than leaving a
field blank.

- **Status:** PROPOSED / ACCEPTED / REJECTED / SUPERSEDED (by NNNN) / REVISITED
- **Date:** YYYY-MM-DD
- **Version:** V1.0
- **Owner:** (who decides; who implements)
- **Source:** (governing doc / research file:line / measurement)

## 1. Problem definition

What decision is this, and why must it be made now?

## 2. Known facts

Verified facts only, each with a source (URL or file:line).

## 3. Assumptions

Explicitly labelled; each with a confidence level
(VERIFIED / HIGH / MEDIUM / LOW / UNKNOWN).

## 4. Unknown variables

What is not known, and what would settle it.

## 5. Constraints

Hard constraints first (license, MSRV, security, scope, Tier C).

## 6. Mathematical / computational model

The model used (or: no useful model exists, and why). Include dimensional
analysis / sanity checks where applicable.

## 7. Alternatives (≥3 when consequential, incl. "do nothing")

| # | Alternative | Pros | Cons |
|---|-------------|------|------|
| 1 | Do nothing | | |
| 2 | | | |
| 3 | | | |

Weights: correctness/safety .30, maintainability .20, perf .20,
reversibility .15, effort .15; S = Σ(w·s); ±20% sensitivity; close call
(<10%) = ≤30 min benchmark.

## 8. Expected benefits

Per alternative, measurably stated where possible.

## 9. Failure modes

How each alternative can fail; what the failure costs.

## 10. Cost / complexity

Implementation cost, maintenance cost, risk, reversibility.

## 11. Experimental validation method

How the decision will be tested (harness, test IDs from
`phase0/11-acceptance-tests.md`, benchmark design). "Not measured" until run.

## 12. Final decision + evidence

The decision, the evidence for it, and the conditions under which it is
revisited. A decision without evidence is PROPOSED, not ACCEPTED.
