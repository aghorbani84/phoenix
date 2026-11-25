# Decision Record 0019 — cover-corpus

- **Decision:** 0019 (D19 of 12 §6)
- **Status:** PROPOSED
- **Date:** 2026-11-25
- **Version:** V1.0
- **Owner:** Abolfazl (decides; owner of the phase-gate go-ahead); Phase 1 architecture work (this record); implementation owner: Phase 2 (pending owner go-ahead)
- **Source:** phase1/04-cover-corpus-plan.md (§10 decision record); phase0/03-feasibility-assessment.md §1.3, §1.4 F2, §2.3, §11.2 item 1; phase0/12-priority-roadmap.md §1 P3, §6 D19; phase1/05-threshold-validation-plan.md §3 (power analysis)

## 1. Problem definition

Cover-corpus collection (lawful): how is the cover corpus (real browsers to real sites) collected lawfully on owned infrastructure, and what are its representativeness limits? E1 cannot run without a corpus; the corpus does not exist; lawful collection is unproven (U3). The plan must exist at M0 and execute at M3 — and flip F2 if no lawful source exists.

## 2. Known facts

- E1 consumers (04 §1.1): distinguisher 1 (Wu exemption heuristics — real first packets), distinguisher 2 (ML classifier — ≥5,000 flows/class in the test set), distinguisher 3 (Xue 2024 nested-handshake — session-level grouping preserved), AT-E1-06 (handshake sizes — ≥1,000, recommended ≥10,000), AT-E1-05 (FPR — cover-only traffic never used in training), E2 (genuine cover server baseline), AT-E7-05 (rotation events labelled), AT-ZEN-RT-01 (frozen baseline).
- Sources B and C are clearly lawful (ordinary browsing of public pages; owned endpoints); Source A is lawful with the owner's consent (04 §2). Opinion (HIGH) that F2 is unlikely to trigger in full.
- Sizing (mathematical truth, 05 §3): collection target 10,000 flows/class (5,000 train + 5,000 test) at 5σ separation (CI ±0.020 on Adv); ≥1,000 complete visits (≥60 packets per visit); ≥10,000 handshake captures (KS D ≈ 0.019).
- Site list: Tranco top 100 (head) + 200 domains from ranks 1,000–100,000 (long tail); total 300 domains per corpus version, frozen (content-hash recorded). Browsers: Chromium + Firefox headless (current stable, versions pinned per corpus version) + the owner's own browser (Source A).
- Representativeness limits L1–L9 (04 §6), headline: the corpus measures PHOENIX against lab browsing to public sites; the censor measures PHOENIX against the owner's real traffic on his real network. Every E1 claim must carry that sentence or its equivalent.

## 3. Assumptions

- The owner consents to Source A (or the partial-F2 fallback applies, 04 §7). [MEDIUM — consent is his to give or withhold.]
- Headless-browser traffic is an adequate volume source despite L7 (synthetic timing). [MEDIUM — mitigated by stratifying the test set: Source A flows are always in the test set, never only in training.]
- 10,000 flows/class is collectable in the M3 window (to be verified by the pilot, 04 §9.2). [MEDIUM.]

## 4. Unknown variables

- Achieved Source A volume (bounded by one person's browsing); per-site flow yields; ML distinguisher variance (Phase 2).
- Whether a lawful corpus source exists at all — I cannot verify this from Phase 0 (12 §11, item 2); the plan answers it, and flip F2 is the consequence if the answer is no.
- Storage volume for ~20,000 flows of pcaps — no figure is asserted (to be measured by the pilot).

## 5. Constraints

- Hard filter — lawfulness: no third-party traffic, ever; no credentials, no logins, no personal accounts; politeness (Source B: one page at a time, dwell ≥5 s, respect robots.txt, stop on errors rather than retrying aggressively); no probing of any censorship system; no data leaves the lab; consent recorded before capture, not after (04 §2.4).
- Hard filter — method: a corpus version is a directory that can be copied, hashed, and archived as a unit (manifest + checksums); an unversioned capture (no manifest, no checksums) is deleted, not used.
- The corpus is not published (04 §8.3): publishing it would give any censor a labelled training set for the exact cover PHOENIX uses. (See D20 for the shareability decision.)
- No traffic generation for its own sake; no inference from vendor marketing (`phase1/03-iran-profile.md` §4.4).

## 6. Mathematical / computational model

Three strata (A human / B synthetic / C control); flow = unit; visit = page load; test set 5,000/class at 5σ. Power analysis (05 §3): SE = √(0.5/n) worst case; n = 1,000 → 2.2σ; n = 1,800 → 3σ; n = 5,000 → 5σ; n = 19,200 → 9.8σ (CI ±0.010 — recorded as the price of finer resolution, not targeted). KS D_crit = 1.36·√(2/n): n = 1,000 → 0.061; n = 10,000 → 0.019. Per-flow packet floor: flows with <10 packets are excluded from the ML test set (pre-registered, not chosen after seeing results).

## 7. Alternatives (≥3 when consequential, incl. "do nothing")

| # | Alternative | Pros | Cons |
|---|-------------|------|------|
| 1 | Do nothing — no corpus, E1 abandoned | Zero work. | E1 cannot run; the evaluation framework collapses (F2); every cover-fidelity claim is untestable. Fails the M0 obligation. |
| 2 | This plan: lawful collection on owned infrastructure, Sources A + B + C, 10,000 flows/class, versioned/dated/documented (PICK) | E1 becomes runnable and resolvable at 5σ; every E1 claim carries a versioned, dated corpus; Sources B/C clearly lawful, Source A lawful with consent; representativeness limits stated in advance (L1–L9). | Source A volume is bounded by one person's browsing; the corpus is lab browsing, not the owner's real traffic (L1–L9 travel with every result); collection executes at M3, so E1 cannot run before then. |
| 3 | Synthetic-only corpus (Sources B + C, no Source A) as the plan (not the fallback) | No consent needed; volume is fully schedulable; simplest to execute. | The ML distinguisher's test set is entirely synthetic — the claim weakens precisely where it matters (human timing, L7 promoted to the headline limit); every E1 pass must carry 'against a synthetic-only cover corpus'. Accepted as the partial-F2 fallback (04 §7), not the plan. |
| 4 | Public pcap dataset as cover | Volume immediately; no collection work. | FAILS the hard lawfulness filter: provenance unknown, may contain third-party traffic, not the lab's own. Prohibited by §2.4 and the operating rules (§10). Out. |
| 5 | Traffic-generator cover (invent cover traffic with a generator, call it 'real-browser traffic') | Unlimited volume on demand. | FAILS the hard honesty filter: generated traffic is not real-browser traffic; an E1 pass against it means nothing. Prohibited by 04 §7 ('what is NOT a fallback'). Out. |

**Hard filters applied first** (license-compatible; MSRV set by Phase 1 and recorded; security: no known unpatched CVE, no custom crypto, audited libraries only — `03:§12.1`). A candidate failing any one is out, with no scoring. The public-pcap option fails the hard lawfulness filter (third-party traffic, unknown provenance) and is out. The traffic-generator option fails the hard honesty filter and is out. All other candidates are lawful; synthetic-only is admissible but adopted only as the partial-F2 fallback, not the plan.

**Scoring.** Weights: correctness/safety .30, maintainability .20, performance .20, reversibility .15, effort .15; S = Σ(w·s); each criterion 0–5 (5 = best). Scores are **Opinion** unless a cell is labelled VERIFIED arithmetic or a cited Fact; the basis of each score is stated in the Pros/Cons above. Computed in Python (this generator):

| Alternative | Corr .30 | Maint .20 | Perf .20 | Rev .15 | Eff .15 | **S** |
|---|---:|---:|---:|---:|---:|---:|
| Do nothing — no corpus, E1 abandoned | 1 | 1 | 3 | 5 | 5 | **2.60** |
| This plan: lawful collection on owned infrastructure, Sources A + B + C, 10,000 flows/class, versioned/dated/documented (PICK) | 4 | 4 | 3 | 4 | 3 | **3.65** |
| Synthetic-only corpus (Sources B + C, no Source A) as the plan (not the fallback) | 3 | 3 | 3 | 4 | 4 | **3.30** |
| Public pcap dataset as cover | — | — | — | — | — | **EXCLUDED by hard filter — not scored** |
| Traffic-generator cover (invent cover traffic with a generator, call it 'real-browser traffic') | — | — | — | — | — | **EXCLUDED by hard filter — not scored** |

**Ranking:** 3.65 This plan: lawful collection on owned infrastructure, Sources A + B + C, 10,000 flows/class, versioned/dated/documented (PICK) > 3.30 Synthetic-only corpus (Sources B + C, no Source A) as the plan (not the fallback) > 2.60 Do nothing — no corpus, E1 abandoned. Gap between top two: 0.35 points = 9.59% of the leader's score.

**Sensitivity (±20% per weight, other weights renormalised to sum 1.0):**

| Weight varied | −20% → S | +20% → S | Leader changes? |
|---|---|---|---|
| Correctness/safety | Do nothing — no corpus, E1 abandoned: 2.7021 / This plan: lawful collection on owned infrastructure, Sources A + B + C, 10,000 flows/class, versioned/dated/documented (PICK): 3.6277 / Synthetic-only corpus (Sources B + C, no Source A) as the plan (not the fallback): 3.3191 | Do nothing — no corpus, E1 abandoned: 2.5094 / This plan: lawful collection on owned infrastructure, Sources A + B + C, 10,000 flows/class, versioned/dated/documented (PICK): 3.6698 / Synthetic-only corpus (Sources B + C, no Source A) as the plan (not the fallback): 3.2830 | No |
| Maintainability | Do nothing — no corpus, E1 abandoned: 2.6667 / This plan: lawful collection on owned infrastructure, Sources A + B + C, 10,000 flows/class, versioned/dated/documented (PICK): 3.6354 / Synthetic-only corpus (Sources B + C, no Source A) as the plan (not the fallback): 3.3125 | Do nothing — no corpus, E1 abandoned: 2.5385 / This plan: lawful collection on owned infrastructure, Sources A + B + C, 10,000 flows/class, versioned/dated/documented (PICK): 3.6635 / Synthetic-only corpus (Sources B + C, no Source A) as the plan (not the fallback): 3.2885 | No |
| Performance | Do nothing — no corpus, E1 abandoned: 2.5833 / This plan: lawful collection on owned infrastructure, Sources A + B + C, 10,000 flows/class, versioned/dated/documented (PICK): 3.6771 / Synthetic-only corpus (Sources B + C, no Source A) as the plan (not the fallback): 3.3125 | Do nothing — no corpus, E1 abandoned: 2.6154 / This plan: lawful collection on owned infrastructure, Sources A + B + C, 10,000 flows/class, versioned/dated/documented (PICK): 3.6250 / Synthetic-only corpus (Sources B + C, no Source A) as the plan (not the fallback): 3.2885 | No |
| Reversibility | Do nothing — no corpus, E1 abandoned: 2.5258 / This plan: lawful collection on owned infrastructure, Sources A + B + C, 10,000 flows/class, versioned/dated/documented (PICK): 3.6392 / Synthetic-only corpus (Sources B + C, no Source A) as the plan (not the fallback): 3.2784 | Do nothing — no corpus, E1 abandoned: 2.6699 / This plan: lawful collection on owned infrastructure, Sources A + B + C, 10,000 flows/class, versioned/dated/documented (PICK): 3.6602 / Synthetic-only corpus (Sources B + C, no Source A) as the plan (not the fallback): 3.3204 | No |
| Effort | Do nothing — no corpus, E1 abandoned: 2.5258 / This plan: lawful collection on owned infrastructure, Sources A + B + C, 10,000 flows/class, versioned/dated/documented (PICK): 3.6701 / Synthetic-only corpus (Sources B + C, no Source A) as the plan (not the fallback): 3.2784 | Do nothing — no corpus, E1 abandoned: 2.6699 / This plan: lawful collection on owned infrastructure, Sources A + B + C, 10,000 flows/class, versioned/dated/documented (PICK): 3.6311 / Synthetic-only corpus (Sources B + C, no Source A) as the plan (not the fallback): 3.3204 | No |

**Tipping weight:** none — no single ±20% weight variation changes the leader. The ranking is robust to weight choice (it may still be fragile to score choice; see §12).

**Close-call handling (<10% margin):** the margin is 9.59%, inside the <10% band, so the refined-spec rule fires: a ≤30-minute benchmark breaks the tie; if still tied, the more reversible option wins (`03:§12.1`). See §12 for the tie-break evidence.

## 8. Expected benefits

E1 becomes runnable and resolvable at 5σ; every E1 claim carries a versioned, dated corpus (v1.0.0); the pilot (§9.2) validates the pipeline end-to-end (capture → pcapng → flow CSV → manifest → checksums) before the full run; the partial-F2 fallback is pre-decided, so a missing Source A does not improvise a new plan at M3.

## 9. Failure modes

Source A unavailable (owner withholds consent, or browses too little) → partial-F2 fallback (04 §7): corpus proceeds with B + C, every E1 claim downgraded one level ('synthetic-only cover', L7 promoted to headline). Collection shortfall (<10,000/class) → downgraded resolution (achieved n and SE reported; results downgraded, not silently accepted at the 1,000-flow resolution). Live-web drift between versions (L8) → results are dated to their corpus version; a result older than one corpus generation is re-run, not re-cited. Full F2 (no lawful source at all) → E1 cannot run, AT-E1-01–06 not reportable, NO-GO for the measurement-dependent parts of the MVP (project-level decision for the owner); the protocol work that does not depend on E1 (D2, E2 against the lab's own cover server, E3/E4/E5/E6) can proceed.

**Consequence if wrong:** If the plan is wrong (headless-browser traffic proves an inadequate volume source, or the pilot shows the sizing targets are uncollectable in the M3 window), the cost is a sizing revision by decision with the pilot evidence (more time, fewer flows with downgraded resolution, or a revised site list) — bounded because the pilot runs before the full collection. If the plan is wrong about lawfulness (a source proves unlawful after all), the cost is deletion of the affected stratum and a re-collection — which is why consent and the lawful-basis notes are recorded in the manifest before capture, not after.

**Reversibility cost of undoing it:** High: the corpus is versioned. A bad version is deleted (Source A pcaps deleted by default after the E1 runs that need them are complete); the manifest + flow CSV + results (aggregates) are retained permanently as the audit trail. A new version is a new capture, not a patch to the old one.

## 10. Cost / complexity

One capture script + one pilot (10 sites, 1 browser) + the full collection run (Phase 2, M3); storage for ~20,000 flows of pcaps (volume to be measured by the pilot — no storage figure asserted). Source A retention: pcaps deleted after the E1 runs that need them are complete unless the owner explicitly asks to keep them.

## 11. Experimental validation method

Pilot (04 §9.2): Source B pilot run (10 sites, 1 browser) to validate the pipeline end-to-end before the full run. Version cut (04 §9.4): corpus v1.0.0 frozen (manifest + checksums); sizing verified against §5 (flows/class, visits, handshakes); representativeness limits (§6) recorded as measured, not as targeted. Handoff to calibration: corpus v1.0.0 handed to the threshold-validation plan — AT-E1-04 runs first, before any E1 result is reportable. Falsifier (F2): no lawful source exists → E1 cannot run and the verdict flips for the measurement-dependent parts of the MVP (04 §7).

## 12. Final decision + evidence

**Decision (PROPOSED): This plan: lawful collection on owned infrastructure, Sources A + B + C, 10,000 flows/class, versioned/dated/documented**

PROPOSED: adopt the lawful collection plan in `phase1/04-cover-corpus-plan.md`; collection executes in Phase 2 (M3). No corpus exists today. Sources A (owner's lab browsing, with recorded consent) + B (headless-browser synthetic browsing to a frozen 300-domain Tranco list) + C (lab-owned origin control); collection target 10,000 flows/class (5,000 train + 5,000 test) at 5σ; ≥1,000 complete visits; ≥10,000 handshake captures. Partial-F2 fallback pre-decided: if Source A is unavailable, the corpus proceeds with B + C and every E1 claim is downgraded one level ('synthetic-only cover').

**Calculation that produced it:** S = 0.30·4 + 0.20·4 + 0.20·3 + 0.15·4 + 0.15·3 = 3.65, highest of the scored alternatives (synthetic-only 3.30). Margin 0.35 points = 9.6% — inside the <10% close-call band, so the close-call rule fires: the tie-break is the evidence (Source A is the only source of genuine human timing, and the ML distinguisher trains on inter-arrival times — the synthetic-only corpus weakens the claim precisely where it matters), and if still tied, the more reversible option wins — both are equally reversible, so the pick stands on the evidence argument, recorded here.

**Confidence:** Opinion, HIGH (80%) that Sources B/C are lawful and sufficient for volume; MEDIUM (70%) that Source A reaches its target (depends on the owner's browsing volume and consent). Calibration p = 0.80 / 0.70 respectively.

**Status / date / deciders:** PROPOSED — 2026-11-25 — decided by Abolfazl (owner) at the Phase 1 gate; this record proposes, it does not accept. A decision without its M2/M3 evidence is PROPOSED, not ACCEPTED. Decided at: M0 (plan) → M3 (corpus exists). Evidence that would settle it: the pilot (§9.2) for collectability; the version cut (§9.4) for sizing; the consent record for Source A.

**Related items:** U3, R2, C3, C4, C14; AT-E1-01–06, AT-E2-01–05, AT-E7-05, AT-ZEN-RT-01; M0, M3; P3; D11 (calibration the corpus feeds), D20 (shareability of the corpus), D5 (E1 evidence for the transport pick).
