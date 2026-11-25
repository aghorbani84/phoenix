# Decision Record 0020 — harness-publicity

- **Decision:** 0020 (D20 of 12 §6)
- **Status:** PROPOSED
- **Date:** 2026-11-25
- **Version:** V1.0
- **Owner:** Abolfazl (decides; owner of the phase-gate go-ahead); Phase 1 architecture work (this record); implementation owner: Phase 2 (pending owner go-ahead)
- **Source:** phase0/03-feasibility-assessment.md §8.4 (independently verified resilience — feasible, not yet done); phase0/12-priority-roadmap.md §1 P14, §6 D20; phase1/04-cover-corpus-plan.md §1.2, §8.3 (corpus is not published)

## 1. Problem definition

Harness publicity + corpus shareability: is the harness public, and is the corpus shareable, so that a third party can reproduce E1–E7 (the 'independently verified resilience' success criterion, `03:§8.4`)? The harness must be public for reproduction; the corpus must not be published openly (it would hand a censor a labelled training set for the exact cover PHOENIX uses — 04 §1.2/§8.3). The decision must reconcile those two facts.

## 2. Known facts

- Third-party reproduction of E1–E7 from the published harness + corpus is the 'independently verified resilience' success criterion (`03:§8.4`); it is feasible, not yet done — it requires the harness to be public and the corpus to be shareable, which are Phase 1 architecture decisions (12 §1 P14).
- The corpus is not published (04 §8.3): publishing it would give any censor a labelled training set for the exact cover PHOENIX uses. If a future decision ever favours publication, it requires a separate owner decision and a re-anonymisation review. This plan assumes no publication.
- The corpus contains the owner's browsing pattern (Source A: destination + timing is his private data — 04 §8.1/§8.2). Source A pcaps are retained only until the E1 runs that need them are complete, then deleted unless the owner explicitly asks to keep them.
- The harness is the instrument every later claim is measured with; an unvalidated instrument invalidates every measurement downstream (12 §1 P4). Its validity is demonstrated by AT-E1-04 (reproducing a known break), which a third party can only check if the harness is public and runnable.
- Reproducible builds and third-party reproduction are M8 (release) concerns; the publicity decision is Phase 1 because the harness is built in Phase 2 and its publicity shapes how it is built (no secrets, no owner data, versioned inputs).

## 3. Assumptions

- A third party can reproduce E1–E7 from a public harness + a controlled-access corpus without the corpus being public — i.e. the corpus can be shared with named verifiers under a written agreement without becoming public. [MEDIUM — the agreement's terms are a Phase 2/owner matter; the mechanism (controlled sharing) is standard practice.]
- The harness can be built with no secrets and no owner data in it (it runs against a corpus version + a prototype SHA, both versioned inputs). [HIGH — the harness consumes the corpus; it does not contain it.]

## 4. Unknown variables

- Whether any independent verifier will actually reproduce E1–E7 (the success criterion is 'if achieved' — P14). No verifier is lined up; this record creates the possibility, not the reproduction.
- The terms under which the corpus can be shared with a verifier (what is shared — flow CSV + aggregates vs raw pcaps; what the verifier may publish) — a Phase 2/owner matter, not decided here.
- Whether the owner wants the harness public at all (the harness is PHOENIX's measurement instrument; its publicity is an architecture decision, but the owner is the publisher — the M8 release is Tier C).

## 5. Constraints

- Hard filter — privacy/security: the corpus (in particular Source A pcaps, which contain the owner's browsing pattern) is never published openly and never leaves the lab except under the controlled-sharing mechanism in §12. A candidate that publishes the corpus is out.
- Hard filter — honesty: a reproduction claim is only made if a third party actually reproduces E1–E7; 'public harness' is not itself a reproduction.
- Tier C: release publishing is Tier C (`03:§12.5`) — the harness's publication as part of a release needs the owner's explicit go-ahead as a separate act, not merely the phase-gate go-ahead.
- No secrets in any repo/log/public file (AT-VPS-22 logic applies to the harness repo as well).

## 6. Mathematical / computational model

Publicity as a two-axis decision: harness (public vs private) × corpus (public vs controlled-access vs private). The axes are independent: the harness can be public while the corpus is controlled-access, because the harness consumes a corpus version as an input — a verifier runs the public harness against their own corpus (or the controlled-access one) and compares results. The reproduction claim requires both axes to be satisfied for the verifier, not for the public.

## 7. Alternatives (≥3 when consequential, incl. "do nothing")

| # | Alternative | Pros | Cons |
|---|-------------|------|------|
| 1 | Do nothing — harness private, corpus private; no third-party reproduction possible | Zero work; no privacy risk; no publication decisions. | The 'independently verified resilience' success criterion (`03:§8.4`) is unachievable by construction; every E1–E7 claim rests on PHOENIX's own harness alone. Fails the P14 obligation to decide. |
| 2 | Public harness + controlled-access corpus (named verifiers, written agreement, no open publication) (PICK) | Third-party reproduction is possible (a verifier runs the public harness against the controlled-access corpus, or their own corpus built to the same plan); the corpus is not published, so no censor gets a labelled training set for PHOENIX's exact cover; the owner's browsing pattern (Source A) never leaves the lab except under agreement (and Source A pcaps are deleted by default after the runs that need them). | Controlled access needs an agreement mechanism (who the verifiers are, what they may publish) — a Phase 2/owner matter not specified here; a verifier's own corpus may differ from PHOENIX's, so reproduction is 'same harness, equivalent corpus', not byte-identical. |
| 3 | Public harness + public corpus | Maximum reproducibility: anyone can reproduce E1–E7 byte-for-byte; the strongest possible 'independently verified' claim. | FAILS the hard privacy/security filter: publishing the corpus hands any censor a labelled training set for the exact cover PHOENIX uses (04 §1.2/§8.3) and publishes the owner's browsing pattern. Out. |
| 4 | Private harness + shareable corpus | The corpus is available for others to measure against. | A corpus without the harness that measured PHOENIX against it is not a reproduction — the verifier cannot run PHOENIX's distinguishers, only their own, so the comparison is not 'did PHOENIX pass E1' but 'does our distinguisher agree'. Backwards: the harness is the instrument, and the instrument is what must be public. |
| 5 | Public harness + synthetic-only public corpus (publish a generated cover, keep the real corpus private) | Anyone can run the harness end-to-end on a public input; the real corpus stays private. | A synthetic corpus is not the cover PHOENIX was judged against — reproduction against it does not reproduce the E1 result (04 §7: generated traffic is not real-browser traffic). It demonstrates the harness runs, not that PHOENIX passes E1. Misleading if presented as reproduction. |

**Hard filters applied first** (license-compatible; MSRV set by Phase 1 and recorded; security: no known unpatched CVE, no custom crypto, audited libraries only — `03:§12.1`). A candidate failing any one is out, with no scoring. Public corpus fails the hard privacy/security filter (labelled training set for the censor + the owner's browsing pattern) and is out. All other candidates are admissible; the synthetic-public-corpus option is rejected on honesty (it does not reproduce the E1 result), not on a filter.

**Scoring.** Weights: correctness/safety .30, maintainability .20, performance .20, reversibility .15, effort .15; S = Σ(w·s); each criterion 0–5 (5 = best). Scores are **Opinion** unless a cell is labelled VERIFIED arithmetic or a cited Fact; the basis of each score is stated in the Pros/Cons above. Computed in Python (this generator):

| Alternative | Corr .30 | Maint .20 | Perf .20 | Rev .15 | Eff .15 | **S** |
|---|---:|---:|---:|---:|---:|---:|
| Do nothing — harness private, corpus private; no third-party reproduction possible | 1 | 2 | 3 | 5 | 5 | **2.80** |
| Public harness + controlled-access corpus (named verifiers, written agreement, no open publication) (PICK) | 4 | 4 | 3 | 4 | 3 | **3.65** |
| Public harness + public corpus | — | — | — | — | — | **EXCLUDED by hard filter — not scored** |
| Private harness + shareable corpus | 2 | 3 | 3 | 3 | 4 | **2.85** |
| Public harness + synthetic-only public corpus (publish a generated cover, keep the real corpus private) | 3 | 3 | 3 | 3 | 3 | **3.00** |

**Ranking:** 3.65 Public harness + controlled-access corpus (named verifiers, written agreement, no open publication) (PICK) > 3.00 Public harness + synthetic-only public corpus (publish a generated cover, keep the real corpus private) > 2.85 Private harness + shareable corpus > 2.80 Do nothing — harness private, corpus private; no third-party reproduction possible. Gap between top two: 0.65 points = 17.81% of the leader's score.

**Sensitivity (±20% per weight, other weights renormalised to sum 1.0):**

| Weight varied | −20% → S | +20% → S | Leader changes? |
|---|---|---|---|
| Correctness/safety | Do nothing — harness private, corpus private; no third-party reproduction possible: 2.9149 / Public harness + controlled-access corpus (named verifiers, written agreement, no open publication) (PICK): 3.6277 / Private harness + shareable corpus: 2.9043 / Public harness + synthetic-only public corpus (publish a generated cover, keep the real corpus private): 3.0000 | Do nothing — harness private, corpus private; no third-party reproduction possible: 2.6981 / Public harness + controlled-access corpus (named verifiers, written agreement, no open publication) (PICK): 3.6698 / Private harness + shareable corpus: 2.8019 / Public harness + synthetic-only public corpus (publish a generated cover, keep the real corpus private): 3.0000 | No |
| Maintainability | Do nothing — harness private, corpus private; no third-party reproduction possible: 2.8333 / Public harness + controlled-access corpus (named verifiers, written agreement, no open publication) (PICK): 3.6354 / Private harness + shareable corpus: 2.8438 / Public harness + synthetic-only public corpus (publish a generated cover, keep the real corpus private): 3.0000 | Do nothing — harness private, corpus private; no third-party reproduction possible: 2.7692 / Public harness + controlled-access corpus (named verifiers, written agreement, no open publication) (PICK): 3.6635 / Private harness + shareable corpus: 2.8558 / Public harness + synthetic-only public corpus (publish a generated cover, keep the real corpus private): 3.0000 | No |
| Performance | Do nothing — harness private, corpus private; no third-party reproduction possible: 2.7917 / Public harness + controlled-access corpus (named verifiers, written agreement, no open publication) (PICK): 3.6771 / Private harness + shareable corpus: 2.8438 / Public harness + synthetic-only public corpus (publish a generated cover, keep the real corpus private): 3.0000 | Do nothing — harness private, corpus private; no third-party reproduction possible: 2.8077 / Public harness + controlled-access corpus (named verifiers, written agreement, no open publication) (PICK): 3.6250 / Private harness + shareable corpus: 2.8558 / Public harness + synthetic-only public corpus (publish a generated cover, keep the real corpus private): 3.0000 | No |
| Reversibility | Do nothing — harness private, corpus private; no third-party reproduction possible: 2.7320 / Public harness + controlled-access corpus (named verifiers, written agreement, no open publication) (PICK): 3.6392 / Private harness + shareable corpus: 2.8454 / Public harness + synthetic-only public corpus (publish a generated cover, keep the real corpus private): 3.0000 | Do nothing — harness private, corpus private; no third-party reproduction possible: 2.8641 / Public harness + controlled-access corpus (named verifiers, written agreement, no open publication) (PICK): 3.6602 / Private harness + shareable corpus: 2.8544 / Public harness + synthetic-only public corpus (publish a generated cover, keep the real corpus private): 3.0000 | No |
| Effort | Do nothing — harness private, corpus private; no third-party reproduction possible: 2.7320 / Public harness + controlled-access corpus (named verifiers, written agreement, no open publication) (PICK): 3.6701 / Private harness + shareable corpus: 2.8144 / Public harness + synthetic-only public corpus (publish a generated cover, keep the real corpus private): 3.0000 | Do nothing — harness private, corpus private; no third-party reproduction possible: 2.8641 / Public harness + controlled-access corpus (named verifiers, written agreement, no open publication) (PICK): 3.6311 / Private harness + shareable corpus: 2.8835 / Public harness + synthetic-only public corpus (publish a generated cover, keep the real corpus private): 3.0000 | No |

**Tipping weight:** none — no single ±20% weight variation changes the leader. The ranking is robust to weight choice (it may still be fragile to score choice; see §12).

**Close-call handling (<10% margin):** the margin is 17.81%, outside the <10% band, so no benchmark is triggered by the close-call rule. See §12 for the tie-break evidence.

## 8. Expected benefits

Public harness + controlled-access corpus: third-party reproduction becomes possible without publishing the corpus; the harness's validity (AT-E1-04) is checkable by anyone; the corpus's privacy (Source A, the owner's browsing pattern) is preserved; the M8 'independently verified resilience' criterion is achievable, not merely aspirational.

## 9. Failure modes

No verifier ever comes forward → the criterion stays 'feasible, not done' (P14) — the decision creates the possibility, not the reproduction; the failure is a null result, not a security failure. A verifier leaks the controlled-access corpus → the labelled-training-set risk materialises through the sharing channel; mitigated by the written agreement (what is shared — flow CSV + aggregates preferred over raw pcaps; what the verifier may publish) and by Source A pcaps being deleted by default. The harness is public but contains a secret or owner data → AT-VPS-22 logic: the publication is a failed release; mitigated by secret scanning in CI (gitleaks) and by the harness consuming the corpus as a versioned input, not containing it.

**Consequence if wrong:** If the split is wrong (controlled access proves unworkable — no verifier will sign an agreement, or the agreement's terms cannot be settled), the cost is that reproduction stays 'feasible, not done' — the same outcome as 'do nothing', but with a public harness that at least lets others inspect the instrument. If the split is wrong the other way (the corpus should have been public for full reproducibility), the cost is weaker reproduction claims — bounded, because a verifier with controlled access can still reproduce E1–E7, and the corpus can be re-decided (by owner decision + re-anonymisation review, 04 §8.3) if the owner ever favours publication.

**Reversibility cost of undoing it:** Medium: publishing the harness is reversible only by unpublishing it (which does not retract copies); sharing the corpus under agreement is reversible per verifier (access revoked). The decision is recorded so the publicity posture can be revisited by ADR — but a published harness cannot be unpublished from the internet, so the M8 Tier C release approval is the real gate, not this record.

## 10. Cost / complexity

Specification cost: this record + the controlled-sharing terms (Phase 2/owner matter). Implementation cost: the harness built for publicity from the start (no secrets, no owner data, versioned inputs — a design constraint on the Phase 2 harness, not a separate build). Test cost: none beyond AT-E1-04 (which is the harness-validity proof a verifier would re-run).

## 11. Experimental validation method

The harness's publicity is validated by AT-E1-04 itself: a third party runs the public harness + the controlled-access corpus and checks that AT-E1-04 reproduces the known break (Wu 2023 exemption rules flagging RB-1 at the published rate). If the third party cannot reproduce AT-E1-04, the harness is invalid (05 §5) — the same falsifier, applied by an independent party. M7 re-calibration re-runs AT-E1-04 on the current harness + corpus version. Falsifier: a verifier demonstrates the public harness cannot reproduce AT-E1-04 → the harness is invalid and no E1 result is reportable (05 §5), regardless of who found it.

## 12. Final decision + evidence

**Decision (PROPOSED): Public harness + controlled-access corpus (named verifiers, written agreement, no open publication)**

PROPOSED: public harness + controlled-access corpus. The harness is public (open-source, reproducible, no secrets, no owner data; corpus consumed as a versioned input). The corpus is shareable with named independent verifiers under a written agreement (flow CSV + aggregates preferred; raw pcaps only if the agreement covers them and Source A pcaps still exist), and is NOT published openly. If the owner ever favours open publication of the corpus, that requires a separate owner decision and a re-anonymisation review (04 §8.3). Publication of the harness as part of a release remains Tier C (owner's explicit go-ahead, M8).

**Calculation that produced it:** S = 0.30·4 + 0.20·4 + 0.20·3 + 0.15·4 + 0.15·3 = 3.70, highest of the scored alternatives (private harness + shareable corpus 2.85; synthetic-public 3.00). Margin 0.70 points = 18.9% — outside the close-call band. The public-corpus option is excluded by the hard filter before scoring.

**Confidence:** Opinion, HIGH (80%) that the split (public harness, controlled corpus) is the correct reconciliation — both halves are grounded in the project record (harness publicity required for reproduction, `03:§8.4`; corpus non-publication required for privacy/security, 04 §8.3). MEDIUM (70%) that controlled access will in practice attract a verifier (no verifier is lined up). Calibration p = 0.80 / 0.70 respectively.

**Status / date / deciders:** PROPOSED — 2026-11-25 — decided by Abolfazl (owner) at the Phase 1 gate; this record proposes, it does not accept. A decision without its M2/M3 evidence is PROPOSED, not ACCEPTED. Decided at: Phase 1 (architecture decision). Evidence that would settle it: a third party reproducing E1–E7 from the public harness + controlled-access corpus (M7/M8); the controlled-sharing terms (Phase 2/owner).

**Related items:** U20 (economic-model unknowns — reproduction is the 'independently verified resilience' criterion); AT-E1-04 (the validity proof a verifier re-runs); M7, M8; P14; D19 (the corpus this decision governs), D11 (calibration the harness performs), D18 (licence — the harness's licence is part of its publicity).
