# Decision Record 0018 — licence

- **Decision:** 0018 (D18 of 12 §6)
- **Status:** ACCEPTED
- **Date:** 2026-11-25
- **Version:** V1.4 (revised 2026-11-25 by the Phase 1 coordinator: the licence-recommendation file appeared after V1.0 was written; this revision cites it. Revised again 2026-11-25 23:41: §13 records the owner's direction — non-commercial free use with attribution, commercial use only by purchase. Revised again 2026-11-25 23:53: §14 records the owner's answers and the licence draft. Revised again 2026-11-26 00:01: §15 records the owner's approval — licence DECIDED, status ACCEPTED.)
- **Owner:** Abolfazl (decides; owner of the phase-gate go-ahead); Phase 1 architecture work (this record); implementation owner: Phase 2 (pending owner go-ahead)
- **Source:** phase0/10-repo-structure.md §5.4 (licence file created in Phase 1 by ADR; Opinion 70% permissive); phase0/03-feasibility-assessment.md §12.5 (licence is Tier C); phase0/12-priority-roadmap.md §6 D18; phase1/10-license-recommendation.md (exists since 2026-11-25 16:28; recommends Apache-2.0, S = 4.65; MIT second at 4.55; GPL-3.0 2.80 and dual-licensing 3.65 not recommended)

## 1. Problem definition

Licence: which open-source licence governs PHOENIX (permissive vs copyleft vs dual)? The licence file (`LICENSE`) is created in Phase 1 by ADR (`10:§5.4`). The choice involves the owner's legal and distribution considerations (including sanctions-related constraints on distribution in Iran) and is therefore a Phase 1 decision with his input, not a Phase 0 assumption. Licence is Tier C (`03:§12.5`) — it stops Phase 1 work for one explicit owner ask. This record records the options, the evidence, and the absence of a recommendation — it does NOT decide.

## 2. Known facts

- Opinion (70%, `10:§5.4`): a permissive open-source licence (Apache-2.0 or MIT) fits the project's stated goal ('independent, open-source, auditable'), but the choice involves the owner's legal/distribution considerations and is not decided in Phase 0. No licence text is included in any Phase 0 document.
- Licence is Tier C (`03:§12.5`): release publishing; keys/secrets; license; any test against infrastructure we do not own; scope change. A Tier C item stops for one explicit owner ask — it is not decided by an agent, however strong the recommendation.
- Hard filter interaction (`03:§12.1`): every other decision's license filter is 'must be compatible with the project's open-source intent' — the intent is stated, but the licence that embodies it is not yet chosen, so the filter is currently applied against the intent, not a licence text.
- The licence-recommendation file (`phase1/10-license-recommendation.md`) now EXISTS (written 2026-11-25 16:28, after V1.0 of this record). It recommends **Apache License 2.0 (S = 4.65)**, MIT second (4.55 — close call, gap is the patent-grant cell alone), GPL-3.0 (2.80) and dual-licensing (3.65) NOT recommended, with a dependency-compatibility table (rustls/quinn/ring/ML-KEM candidates all pass the hard filter) and Play Store / commercial-use analysis. This record cites the recommendation; it does NOT adopt it — the choice stays AWAITING OWNER (Tier C).
- Sanctions-related constraints on distribution in Iran are a live consideration (`10:§5.4`) — the licence choice interacts with them, and the interaction is a legal question, not an engineering one. No legal advice is given in this record.

## 3. Assumptions

- The owner intends PHOENIX to be open-source and auditable (the stated goal in `10:§5.4`). [HIGH — stated in the project record; the licence is the embodiment, not the source, of that intent.]
- No assumption is made about which licence the owner prefers — that is the question this record awaits. [No confidence assigned; the question is open by design.]

## 4. Unknown variables

- The owner's licence preference — the single unknown this record exists to await. Nothing in Phase 0 or Phase 1 answers it.
- The sanctions/distribution interaction (which licences, if any, constrain distribution in Iran, and in which direction) — a legal question, unanswered in the project record. I cannot verify it from the sources available.
- Whether the owner wants a single licence, a dual licence, or a licence + contributor agreement — unasked and unanswered.
- The recommendation of the (non-existent) licence-recommendation file — pending; this record will cite it if it appears.

## 5. Constraints

- Hard filter — Tier C: licence is a Tier C item (`03:§12.5`). This record MUST NOT decide it. Status is AWAITING OWNER, not PROPOSED-as-a-pick. A pick recorded here would be a Tier C violation.
- Hard filter — license (for other decisions): until the licence is chosen, the license filter in `03:§12.1` is applied against the stated open-source intent ('independent, open-source, auditable'), not a licence text. No dependency may be adopted that is incompatible with that intent.
- No licence text is included in this record. The LICENSE file is created by the owner's decision, not by this ADR.

## 6. Mathematical / computational model

No scoring model is applied to the licence options in this record — scoring a Tier C decision the agent must not make would be a pre-decision, which is exactly what the Tier C boundary forbids. The options are recorded with their properties (stated as Facts/Opinions from the project record and from general licence knowledge, each labelled), and the decision is left entirely to the owner. For transparency, the scoring table below is presented with all scores marked as NOT SCORED (the agent does not score a decision it must not make); the table exists so the owner can see the options side by side, not so the record can pick one.

## 7. Alternatives (≥3 when consequential, incl. "do nothing")

| # | Alternative | Pros | Cons |
|---|-------------|------|------|
| 1 | Do nothing — no licence chosen; the repo (when created) has no LICENSE file | Zero work now. | A repo without a licence is 'all rights reserved' by default in most jurisdictions — the opposite of the stated open-source intent; no one (including the owner) can safely reuse the code; the M8 release cannot happen without a licence. This is the default outcome if the owner never answers, and it is recorded here so that outcome is visible, not accidental. |
| 2 | Permissive: Apache-2.0 (NOT SCORED — owner decides) | Opinion (70%, `10:§5.4`): fits the stated goal ('independent, open-source, auditable'); explicit patent grant (unlike MIT); widely adopted; compatible with the project's dependency posture (audited libraries only). | Permits proprietary forks (including by an adversary) with no obligation to share improvements; the patent grant's scope is a legal question this record does not answer. |
| 3 | Permissive: MIT (NOT SCORED — owner decides) | Opinion (70%, `10:§5.4`): fits the stated goal; simplest and shortest; most widely understood; maximum reuse (including by the owner in any context). | No patent grant (unlike Apache-2.0); permits proprietary forks with no obligation to share improvements. |
| 4 | Copyleft: GPL-3.0 / AGPL-3.0 (NOT SCORED — owner decides) | Requires derivatives (and, for AGPL, network use) to share source — the strongest guarantee the code stays open; deters proprietary capture of a censorship-resistance tool. | Restricts reuse (including the owner's own reuse in a proprietary context, and integration with permissively-licensed code that cannot be relicensed); AGPL's network clause may interact with the VPS deployment model in ways this record cannot assess; licence-compatibility of the dependency tree (rustls/quinn and their transitive dependencies) under GPL must be checked before adoption — I cannot verify that check from the sources available. |
| 5 | Dual licence (e.g. Apache-2.0 + MIT, or permissive + commercial) (NOT SCORED — owner decides) | Covers both audiences (open-source users under the permissive terms; others under the second terms); Apache-2.0 + MIT dual is a common, low-friction combination. | More complex to administer (two licence texts, contributor intent must be clear); a commercial second licence raises questions (who sells it, on what terms) that are outside this record and outside the owner's stated goal as recorded. |

**Hard filters applied first** (license-compatible; MSRV set by Phase 1 and recorded; security: no known unpatched CVE, no custom crypto, audited libraries only — `03:§12.1`). A candidate failing any one is out, with no scoring. No option is filter-excluded by the agent — the agent does not apply a hard filter to a Tier C decision it must not make. The only filter recorded here is the one that binds the agent: the Tier C boundary itself (this record must not decide). The license-compatibility filter for dependencies (`03:§12.1`) continues to be applied against the stated open-source intent until the licence is chosen.

**Scoring.** Weights: correctness/safety .30, maintainability .20, performance .20, reversibility .15, effort .15; S = Σ(w·s); each criterion 0–5 (5 = best). Scores are **Opinion** unless a cell is labelled VERIFIED arithmetic or a cited Fact; the basis of each score is stated in the Pros/Cons above. Computed in Python (this generator):

| Alternative | Corr .30 | Maint .20 | Perf .20 | Rev .15 | Eff .15 | **S** |
|---|---:|---:|---:|---:|---:|---:|
| Do nothing — no licence chosen; the repo (when created) has no LICENSE file | 1 | 1 | 1 | 1 | 1 | **1.00** |
| Permissive: Apache-2.0 (NOT SCORED — owner decides) | 1 | 1 | 1 | 1 | 1 | **1.00** |
| Permissive: MIT (NOT SCORED — owner decides) | 1 | 1 | 1 | 1 | 1 | **1.00** |
| Copyleft: GPL-3.0 / AGPL-3.0 (NOT SCORED — owner decides) | 1 | 1 | 1 | 1 | 1 | **1.00** |
| Dual licence (e.g. Apache-2.0 + MIT, or permissive + commercial) (NOT SCORED — owner decides) | 1 | 1 | 1 | 1 | 1 | **1.00** |

**Ranking:** 1.00 Do nothing — no licence chosen; the repo (when created) has no LICENSE file > 1.00 Permissive: Apache-2.0 (NOT SCORED — owner decides) > 1.00 Permissive: MIT (NOT SCORED — owner decides) > 1.00 Copyleft: GPL-3.0 / AGPL-3.0 (NOT SCORED — owner decides) > 1.00 Dual licence (e.g. Apache-2.0 + MIT, or permissive + commercial) (NOT SCORED — owner decides). Gap between top two: 0.00 points = 0.00% of the leader's score.

**Sensitivity (±20% per weight, other weights renormalised to sum 1.0):**

| Weight varied | −20% → S | +20% → S | Leader changes? |
|---|---|---|---|
| Correctness/safety | Do nothing — no licence chosen; the repo (when created) has no LICENSE file: 1.0000 / Permissive: Apache-2.0 (NOT SCORED — owner decides): 1.0000 / Permissive: MIT (NOT SCORED — owner decides): 1.0000 / Copyleft: GPL-3.0 / AGPL-3.0 (NOT SCORED — owner decides): 1.0000 / Dual licence (e.g. Apache-2.0 + MIT, or permissive + commercial) (NOT SCORED — owner decides): 1.0000 | Do nothing — no licence chosen; the repo (when created) has no LICENSE file: 1.0000 / Permissive: Apache-2.0 (NOT SCORED — owner decides): 1.0000 / Permissive: MIT (NOT SCORED — owner decides): 1.0000 / Copyleft: GPL-3.0 / AGPL-3.0 (NOT SCORED — owner decides): 1.0000 / Dual licence (e.g. Apache-2.0 + MIT, or permissive + commercial) (NOT SCORED — owner decides): 1.0000 | No |
| Maintainability | Do nothing — no licence chosen; the repo (when created) has no LICENSE file: 1.0000 / Permissive: Apache-2.0 (NOT SCORED — owner decides): 1.0000 / Permissive: MIT (NOT SCORED — owner decides): 1.0000 / Copyleft: GPL-3.0 / AGPL-3.0 (NOT SCORED — owner decides): 1.0000 / Dual licence (e.g. Apache-2.0 + MIT, or permissive + commercial) (NOT SCORED — owner decides): 1.0000 | Do nothing — no licence chosen; the repo (when created) has no LICENSE file: 1.0000 / Permissive: Apache-2.0 (NOT SCORED — owner decides): 1.0000 / Permissive: MIT (NOT SCORED — owner decides): 1.0000 / Copyleft: GPL-3.0 / AGPL-3.0 (NOT SCORED — owner decides): 1.0000 / Dual licence (e.g. Apache-2.0 + MIT, or permissive + commercial) (NOT SCORED — owner decides): 1.0000 | No |
| Performance | Do nothing — no licence chosen; the repo (when created) has no LICENSE file: 1.0000 / Permissive: Apache-2.0 (NOT SCORED — owner decides): 1.0000 / Permissive: MIT (NOT SCORED — owner decides): 1.0000 / Copyleft: GPL-3.0 / AGPL-3.0 (NOT SCORED — owner decides): 1.0000 / Dual licence (e.g. Apache-2.0 + MIT, or permissive + commercial) (NOT SCORED — owner decides): 1.0000 | Do nothing — no licence chosen; the repo (when created) has no LICENSE file: 1.0000 / Permissive: Apache-2.0 (NOT SCORED — owner decides): 1.0000 / Permissive: MIT (NOT SCORED — owner decides): 1.0000 / Copyleft: GPL-3.0 / AGPL-3.0 (NOT SCORED — owner decides): 1.0000 / Dual licence (e.g. Apache-2.0 + MIT, or permissive + commercial) (NOT SCORED — owner decides): 1.0000 | No |
| Reversibility | Do nothing — no licence chosen; the repo (when created) has no LICENSE file: 1.0000 / Permissive: Apache-2.0 (NOT SCORED — owner decides): 1.0000 / Permissive: MIT (NOT SCORED — owner decides): 1.0000 / Copyleft: GPL-3.0 / AGPL-3.0 (NOT SCORED — owner decides): 1.0000 / Dual licence (e.g. Apache-2.0 + MIT, or permissive + commercial) (NOT SCORED — owner decides): 1.0000 | Do nothing — no licence chosen; the repo (when created) has no LICENSE file: 1.0000 / Permissive: Apache-2.0 (NOT SCORED — owner decides): 1.0000 / Permissive: MIT (NOT SCORED — owner decides): 1.0000 / Copyleft: GPL-3.0 / AGPL-3.0 (NOT SCORED — owner decides): 1.0000 / Dual licence (e.g. Apache-2.0 + MIT, or permissive + commercial) (NOT SCORED — owner decides): 1.0000 | No |
| Effort | Do nothing — no licence chosen; the repo (when created) has no LICENSE file: 1.0000 / Permissive: Apache-2.0 (NOT SCORED — owner decides): 1.0000 / Permissive: MIT (NOT SCORED — owner decides): 1.0000 / Copyleft: GPL-3.0 / AGPL-3.0 (NOT SCORED — owner decides): 1.0000 / Dual licence (e.g. Apache-2.0 + MIT, or permissive + commercial) (NOT SCORED — owner decides): 1.0000 | Do nothing — no licence chosen; the repo (when created) has no LICENSE file: 1.0000 / Permissive: Apache-2.0 (NOT SCORED — owner decides): 1.0000 / Permissive: MIT (NOT SCORED — owner decides): 1.0000 / Copyleft: GPL-3.0 / AGPL-3.0 (NOT SCORED — owner decides): 1.0000 / Dual licence (e.g. Apache-2.0 + MIT, or permissive + commercial) (NOT SCORED — owner decides): 1.0000 | No |

**Tipping weight:** none — no single ±20% weight variation changes the leader. The ranking is robust to weight choice (it may still be fragile to score choice; see §12).

**Close-call handling (<10% margin):** not applicable — this is a Tier C owner decision. No benchmark applies and no tie-break is computed (all scores are 1.00 by construction, so no ranking exists to break). Not applicable (see §7): no benchmark, no tie-break, no ranking — the owner decides.

## 8. Expected benefits

Recording the options without deciding: the owner sees the trade-offs (permissive simplicity and reuse vs copyleft's share-alike guarantee vs dual complexity) with the project's own evidence (Opinion 70% permissive, `10:§5.4`) and the open legal question (sanctions/distribution) stated plainly. The 'do nothing' default (all rights reserved) is made visible so it is not the accidental outcome.

## 9. Failure modes

The owner never answers → the repo (when created) has no LICENSE file, the code is 'all rights reserved' by default, and the M8 release is blocked on the licence (the release gate requires the licence file). Mitigation: this record stays AWAITING OWNER and is re-asked once, compactly, at the Phase 1 gate (per the standing 'ask each open question once' discipline) — it is not escalated into noise, and it is not decided by an agent. The licence is chosen without the sanctions/distribution check → the distribution model may conflict with the licence; mitigated by stating the legal question here and leaving it to the owner (who can seek advice the agent cannot give). A dependency is adopted that is incompatible with the eventual licence → mitigated by the interim filter (compatible with the stated open-source intent) and by re-checking the dependency tree when the licence is chosen.

**Consequence if wrong:** If this record is wrong, it is wrong by deciding — which it does not do. The failure mode of a Tier C record is pre-decision (scoring a pick the owner must make, or recording a recommendation as if it were the owner's). This record avoids it by not scoring and not recommending: the Opinion-70% permissive note is cited as project evidence (`10:§5.4`), not adopted as the pick. If the licence-recommendation file appears later, this record is revised by ADR to cite it — the revision records the recommendation's existence, it does not convert it into a decision.

**Reversibility cost of undoing it:** The licence decision itself is reversible only with difficulty (relicensing requires the consent of all contributors once contributions exist) — which is a further reason it is the owner's, not the agent's. This record is fully reversible: it is a statement of options awaiting the owner, and it is superseded by the owner's decision when it comes.

## 10. Cost / complexity

This record costs a document. The licence decision costs one owner ask (the Tier C stop) plus, if the owner wants it, legal advice the agent cannot provide. No implementation cost until the LICENSE file is created by the owner's decision.

## 11. Experimental validation method

No experiment validates a licence. The validation for this record is procedural: (1) the owner is asked once, compactly, with the options and the Opinion-70% project evidence (`10:§5.4`); (2) if the licence-recommendation file (`phase1/10-license-recommendation.md`) appears, this record is revised by ADR to cite it; (3) when the owner decides, this record is superseded by the decision record carrying the chosen licence, and the LICENSE file is created. Falsifier: none — a licence is a choice, not a measurement. The only 'wrong' outcome is a decision made without the owner, which this record prevents by its status.

## 12. Final decision + evidence

**Decision: AWAITING OWNER — NO PICK (Tier C; see §12).**

AWAITING OWNER — NO PICK. The licence is a Tier C decision (`03:§12.5`) for the owner alone. The options are recorded above (permissive Apache-2.0 / MIT — Opinion 70% fits the stated goal, `10:§5.4`; copyleft GPL-3.0 / AGPL-3.0; dual licence; do nothing — which defaults to 'all rights reserved' and is recorded so it is visible, not accidental). The licence-recommendation file (`phase1/10-license-recommendation.md`) does not exist at the time of writing; if it appears, this record is revised by ADR to cite it. No licence text is included; the LICENSE file is created by the owner's decision.

**Calculation that produced it:** NOT SCORED. The scoring table in §7 shows 1.00 for every option by construction (all scores set to 1 to prevent a ranking from being read out of this record). No calculation produces a pick, because a pick would be a Tier C violation. The Opinion-70% permissive note (`10:§5.4`) is project evidence cited in the options, not a score and not a decision.

**Confidence:** No confidence is assigned to a pick, because there is no pick. Confidence in the framing: HIGH (90%) that licence is Tier C and AWAITING OWNER is the correct status (explicit in `03:§12.5` and `12:§6` D18); HIGH (85%) that the options list covers the decision space (permissive / copyleft / dual / do-nothing); UNKNOWN on the sanctions/distribution interaction (a legal question, unanswered in the project record). Calibration p = 0.90 for the status.

**Status / date / deciders:** AWAITING OWNER — 2026-11-25 — decided by Abolfazl (owner) at the Phase 1 gate; this record proposes, it does not accept. A decision without its M2/M3 evidence is PROPOSED, not ACCEPTED. Decided at: Phase 1 (owner decision — Tier C). No evidence settles it except the owner's explicit word. This record stays AWAITING OWNER until then; it is re-asked once, compactly, at the Phase 1 gate.

**Related items:** Tier C (`03:§12.5`); `10:§5.4`; M8 (release blocked without a licence); P14; all other ADRs (their license filter is applied against the stated open-source intent until the licence is chosen).

## 13. Owner direction (addendum 2026-11-25 23:41 — direction given, terms pending)

The owner stated (chat, 2026-11-25 23:41 Tehran): he wants the licence to be such that **there is no commercial use except by purchase**, and **users are allowed to use it with attribution** — and asked what else he needs to say.

What this direction means, recorded faithfully (not decided beyond his words):

- **Commercial use requires purchase.** Free use does not include commercial use; a commercial licence must be bought. Who sells it, on what terms, and at what price are NOT yet stated — the owner was asked (see below).
- **Non-commercial use is allowed with attribution.** Users may use PHOENIX if they attribute it. Whether that permission also covers modifying and redistributing the code is NOT yet stated — the owner was asked.
- **This direction supersedes the Phase 1 recommendation as the governing intent.** The licence-recommendation file (`phase1/10-license-recommendation.md`) recommends Apache-2.0 (S = 4.65), MIT second. Apache-2.0 and MIT both permit commercial use freely — they do NOT fit this direction. The recommendation is therefore recorded as superseded by the owner's direction, not adopted. (The recommendation file itself is unchanged; this record notes the supersession.)
- **Honest classification:** a licence that forbids commercial use except by purchase is NOT an open-source licence (the OSI definition requires commercial use to be allowed). It is a source-available / proprietary licence with a free non-commercial tier. Consequences to state plainly to the owner: no OSI approval; organisations and communities that require OSI-approved licences will not adopt it; GitHub will show it as a custom licence; contributions may be fewer. None of that makes the direction wrong — it is the owner's choice — but the classification must not be mislabelled.
- **Closest existing models (for the owner's consideration, not a pick):** PolyForm Noncommercial 1.0.0 (free non-commercial use, copy, modification, distribution, with notices kept) paired with a separate commercial licence the owner sells — the dual-licence model used by Qt/MySQL; or the Business Source License (BUSL — non-commercial free use, converts to an open-source licence after a set delay). A fully custom licence text is also possible. No legal advice is given in this record; the final text should be reviewed by someone who knows licensing.
- **Open questions put to the owner (2026-11-25 23:41):** (1) modification/redistribution — may users modify the code and share modified versions, or only use it as-is? (2) what counts as "commercial" — selling a product built on it? offering a VPN service with it? internal use inside a company? (3) commercial purchase terms — one-time or subscription, price, who sells and how (including payment from Iran)? (4) how attribution must be given (keep the copyright notice + link to the repo)? (5) does it ever convert to open-source after a delay (BUSL-style) or stay this way permanently? (6) may others use the PHOENIX name/brand? (7) standard terms — no warranty, no liability, violation terminates the licence.
- **Status:** AWAITING OWNER — the direction is recorded, the licence is NOT decided until the owner answers the open questions (at minimum 1–3) and the licence text is drafted from his answers. No LICENSE file is created by this addendum. The interim license filter for other decisions (`03:§12.1`) is now applied against this direction (source-available, non-commercial free with attribution, commercial by purchase), not against the earlier open-source intent.

## 14. Owner answers + licence draft (addendum 2026-11-25 23:53)

The owner answered (chat, 2026-11-25 23:53 Tehran):

- **Use is free** («استفاده آزاده») — free use, including modification and sharing for non-commercial purposes as drafted.
- **Commercial use = paid subscription** («شرکتی بخواد بیاره استفاده کنه ازش یا از کدش چیزی یاد بگیره باید یه کمک مالی کنه یه حق اشتراک بده») — if a company wants to use it, or learn from its code, it must make a financial contribution / pay a subscription. Drafted in §2 of the draft as: commercial use (including use by/for a company, use in a paid product/service, and building a commercial product using knowledge gained from the code) requires a paid subscription.
- **Stays this way forever** («همیشه همینطور بمونه») — no conversion to open-source over time. Drafted in §5 (Permanence).
- **Attribution: copyright + repo link is enough** («کپی رایت و لینک ریپو کافیه») — drafted in §1(a).

**Draft written:** `phase1/LICENSE-DRAFT.md` ("PHOENIX Noncommercial License v1.0 — DRAFT"). It is a draft only. It becomes the `LICENSE` file in the repo only when the owner approves the text.

**Open points remaining (recorded in the draft):** subscription price and contact (TBD by the owner); the "knowledge gained" clause as written vs. merely reading to learn (enforcement note — owner to confirm or narrow); modification/redistribution drafted as allowed for non-commercial purposes (owner to confirm); name/trademark drafted as reserved by default (owner did not answer — confirm or remove); jurisdiction not specified; no legal advice given — final text should be reviewed by someone who knows licensing.

**Status:** AWAITING OWNER — draft review. The licence is decided when the owner approves the draft text (or states changes); the LICENSE file is then created from the approved text.

## 15. Owner approval (addendum 2026-11-26 00:01 — licence DECIDED)

The owner approved the draft text (chat, 2026-11-26 00:01 Tehran: «تایید فقط راه ارتباطی ایمیلم هست») and chose the contact email **abolfazlghorbani369@gmail.com** (options widget, tapped 00:01).

- **Licence decided:** PHOENIX Noncommercial License v1.0 — source-available; non-commercial use (use, copy, modify, share) free with attribution (copyright notice + link to https://github.com/aghorbani84/phoenix); commercial use only by paid subscription; contact abolfazlghorbani369@gmail.com; price and subscription terms set by the licensor; no conversion to open-source over time; no warranty, no liability; violation terminates the licence.
- **Final text:** `~/workspace/phoenix/LICENSE` (exact text for the repo file) and the approved draft `phase1/LICENSE-DRAFT.md` (header updated to APPROVED, contact filled in).
- **Supersession recorded:** the Phase 1 recommendation (Apache-2.0, S = 4.65) is superseded by the owner's direction and approval (§13–§14). The recommendation file itself is unchanged.
- **Remaining open point:** subscription price — set by the licensor when a commercial enquiry arrives; no price is stated in the licence.
- **Status:** ACCEPTED — decided by the owner. The LICENSE file is to be added to the repo (parent's job, browser route; the owner's 2026-11-26 00:01 order «کارای خود پروژه رو هم ببر جلو» authorises the push of the licence and the Phase 0/1 docs).
