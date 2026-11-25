# Decision Record 0014 — admin-panel-access

- **Decision:** 0014 (D14 of 12 §6)
- **Status:** PROPOSED
- **Date:** 2026-11-25
- **Version:** V1.0
- **Owner:** Abolfazl (decides; owner of the phase-gate go-ahead); Phase 1 architecture work (this record); implementation owner: Phase 2 (pending owner go-ahead)
- **Source:** phase0/03-feasibility-assessment.md §10.3, §11.3 decision 6; phase0/07-vps-architecture.md; phase0/09-mvp-roadmap.md §2 M5 (installer verification)

## 1. Problem definition

VPS admin-panel access model: how does the operator reach the admin panel (`phoenix-admin` backend)? The default in `03:§10.3` is localhost + SSH tunnel; public exposure needs a written ADR justification. The decision is needed now because the installer, the firewall rules, and the M5 verification (panel not publicly reachable — a hard stop if it is) all depend on it.

## 2. Known facts

- The panel must not listen on a public interface. Options in order of preference (`03:§10.3`, Fact 90% — standard practice): (1) bind to localhost only + SSH tunnel; (2) bind to a private network / WireGuard interface reachable only via the operator's own tunnel; (3) if public exposure is unavoidable, mTLS client certificates + IP allowlist + fail2ban + no password-only auth. Opinion (90%): option (1) is the only default Phase 1 should accept; (3) needs a written justification in an ADR.
- Default-deny inbound firewall (only 22/SSH, 80/443 as required) is the baseline in every fetched guide.
- SSH baseline (Fact, 90%): key-only authentication, PasswordAuthentication no, PermitRootLogin prohibit-password (or no), non-root sudo user, fail2ban, automatic security patches, encrypted off-server backups with a tested restore.
- M5 installer verification (`09:§2`): the panel is NOT reachable from the public internet (a positive test — its failure is a hard stop); the panel IS reachable through the SSH tunnel with the admin client certificate; a backup can be taken and its integrity verified.
- VPS feasibility risks: the provider control plane can reset the server, replace SSH keys, and open a console — provider-account security (phishing-resistant MFA) is part of the threat model (`03:§10.3`). Docker can bypass the host firewall — container port publishing must be audited (`ss -tlnp`), not assumed.

## 3. Assumptions

- The operator has SSH access to the VPS (key-only) and can establish a tunnel. [HIGH — the SSH baseline is the installer's prerequisite.]
- The admin client certificate can be issued to the operator's machine at install time and rotated by the Phase 3 procedure. [MEDIUM.]

## 4. Unknown variables

- VPS specifics ungrounded (U21): provider ranking by blocklist risk, testable console APIs, rotation intervals/quota values/alert thresholds, reinstall time, SSH reachability from Iran to arbitrary VPS ports, Ubuntu LTS version to pin.
- Whether SSH from Iran to the VPS's port 22 is reachable at the moment of use — measurements decay; the installer must verify it at install time, not assume it.

## 5. Constraints

- Hard filter — security: the panel must not listen on a public interface by default; public exposure needs a written ADR justification (`03:§10.3`). A candidate that exposes the panel publicly by default is out.
- No password-only auth; no secrets in any repo/log/public file (AT-VPS-22).
- Tier C: keys/secrets stop for one explicit owner ask; the admin client certificate's issuance is a key operation.
- The free-VPS route is closed for PHOENIX hosting — Altare's ToS explicitly forbids VPN/proxy (`steps:§2.1` B4). Phase 3 must source a host that permits VPN/proxy use; no purchase is authorised by this roadmap; the owner provisions the VPS (`09:§2` M5).

## 6. Mathematical / computational model

Access model as a network-exposure question: which interfaces the panel binds to, and which authentication the path requires. Localhost + SSH tunnel exposes zero panel surface to the internet (the only internet-reachable service is SSH, key-only, fail2ban-protected). Every other option adds an internet-reachable surface and must justify it. The M5 verification tests the exposure directly (panel not reachable publicly = pass; reachable = hard stop).

## 7. Alternatives (≥3 when consequential, incl. "do nothing")

| # | Alternative | Pros | Cons |
|---|-------------|------|------|
| 1 | Do nothing — no access model decided; the installer picks whatever is convenient | Zero work now. | The installer could bind the panel publicly (the convenient default in many installers); the M5 hard-stop test has no model to test against. Fails the M0 obligation and the hard security filter. |
| 2 | Localhost-only bind + SSH tunnel + admin client certificate (PICK) | Zero panel surface on the internet; the only reachable service is SSH (key-only, fail2ban); the admin client certificate adds a second factor on the tunnel path; the M5 test is a direct positive test (not reachable publicly). | The operator must establish an SSH tunnel for every admin session (one extra step); if SSH from Iran to the VPS is blocked, admin access is blocked with it (the tunnel is the single path — mitigated by the VPS being replaceable and by SSH on 443 as a documented fallback only if the operator approves it). |
| 3 | Bind to a private network / WireGuard interface reachable only via the operator's own tunnel | No public panel surface; no per-session SSH tunnel step (the operator's tunnel is persistent); the panel is reachable whenever the operator's tunnel is up. | Requires the operator to run and maintain their own WireGuard tunnel to the VPS (a second system to operate); the WireGuard endpoint itself is an internet-reachable UDP service with its own fingerprint (148 B handshake signature); more moving parts than the SSH tunnel for the same security property. |
| 4 | Public exposure with mTLS client certificates + IP allowlist + fail2ban, no password-only auth | Direct access with no tunnel step; mTLS is strong authentication. | An internet-reachable admin panel is an internet-reachable attack surface (scanning, CVE exposure, allowlist bypass risk); needs a written ADR justification (`03:§10.3`) — none exists; the M5 hard-stop test (panel not reachable publicly) would fail by design. Least reversible once exposed (2/5). |
| 5 | Public exposure with password-only auth (or password + TOTP) | Most convenient; no tunnel, no certificates. | FAILS the hard security filter: password-only auth is explicitly forbidden (`03:§10.3` option 3 requires no password-only auth); a public panel with password auth is the standard compromise path. Out. |

**Hard filters applied first** (license-compatible; MSRV set by Phase 1 and recorded; security: no known unpatched CVE, no custom crypto, audited libraries only — `03:§12.1`). A candidate failing any one is out, with no scoring. Password-only public exposure fails the hard security filter (no password-only auth — `03:§10.3`) and is out. Public mTLS exposure is not filter-failed but requires a written ADR justification that does not exist, and it fails the M5 hard-stop test by design — it is scored to show why it loses. 'Do nothing' fails the M0 obligation.

**Scoring.** Weights: correctness/safety .30, maintainability .20, performance .20, reversibility .15, effort .15; S = Σ(w·s); each criterion 0–5 (5 = best). Scores are **Opinion** unless a cell is labelled VERIFIED arithmetic or a cited Fact; the basis of each score is stated in the Pros/Cons above. Computed in Python (this generator):

| Alternative | Corr .30 | Maint .20 | Perf .20 | Rev .15 | Eff .15 | **S** |
|---|---:|---:|---:|---:|---:|---:|
| Do nothing — no access model decided; the installer picks whatever is convenient | 1 | 1 | 3 | 5 | 5 | **2.60** |
| Localhost-only bind + SSH tunnel + admin client certificate (PICK) | 4 | 4 | 3 | 4 | 3 | **3.65** |
| Bind to a private network / WireGuard interface reachable only via the operator's own tunnel | 4 | 3 | 3 | 3 | 3 | **3.30** |
| Public exposure with mTLS client certificates + IP allowlist + fail2ban, no password-only auth | 3 | 3 | 3 | 2 | 3 | **2.85** |
| Public exposure with password-only auth (or password + TOTP) | — | — | — | — | — | **EXCLUDED by hard filter — not scored** |

**Ranking:** 3.65 Localhost-only bind + SSH tunnel + admin client certificate (PICK) > 3.30 Bind to a private network / WireGuard interface reachable only via the operator's own tunnel > 2.85 Public exposure with mTLS client certificates + IP allowlist + fail2ban, no password-only auth > 2.60 Do nothing — no access model decided; the installer picks whatever is convenient. Gap between top two: 0.35 points = 9.59% of the leader's score.

**Sensitivity (±20% per weight, other weights renormalised to sum 1.0):**

| Weight varied | −20% → S | +20% → S | Leader changes? |
|---|---|---|---|
| Correctness/safety | Do nothing — no access model decided; the installer picks whatever is convenient: 2.7021 / Localhost-only bind + SSH tunnel + admin client certificate (PICK): 3.6277 / Bind to a private network / WireGuard interface reachable only via the operator's own tunnel: 3.2553 / Public exposure with mTLS client certificates + IP allowlist + fail2ban, no password-only auth: 2.8404 | Do nothing — no access model decided; the installer picks whatever is convenient: 2.5094 / Localhost-only bind + SSH tunnel + admin client certificate (PICK): 3.6698 / Bind to a private network / WireGuard interface reachable only via the operator's own tunnel: 3.3396 / Public exposure with mTLS client certificates + IP allowlist + fail2ban, no password-only auth: 2.8585 | No |
| Maintainability | Do nothing — no access model decided; the installer picks whatever is convenient: 2.6667 / Localhost-only bind + SSH tunnel + admin client certificate (PICK): 3.6354 / Bind to a private network / WireGuard interface reachable only via the operator's own tunnel: 3.3125 / Public exposure with mTLS client certificates + IP allowlist + fail2ban, no password-only auth: 2.8438 | Do nothing — no access model decided; the installer picks whatever is convenient: 2.5385 / Localhost-only bind + SSH tunnel + admin client certificate (PICK): 3.6635 / Bind to a private network / WireGuard interface reachable only via the operator's own tunnel: 3.2885 / Public exposure with mTLS client certificates + IP allowlist + fail2ban, no password-only auth: 2.8558 | No |
| Performance | Do nothing — no access model decided; the installer picks whatever is convenient: 2.5833 / Localhost-only bind + SSH tunnel + admin client certificate (PICK): 3.6771 / Bind to a private network / WireGuard interface reachable only via the operator's own tunnel: 3.3125 / Public exposure with mTLS client certificates + IP allowlist + fail2ban, no password-only auth: 2.8438 | Do nothing — no access model decided; the installer picks whatever is convenient: 2.6154 / Localhost-only bind + SSH tunnel + admin client certificate (PICK): 3.6250 / Bind to a private network / WireGuard interface reachable only via the operator's own tunnel: 3.2885 / Public exposure with mTLS client certificates + IP allowlist + fail2ban, no password-only auth: 2.8558 | No |
| Reversibility | Do nothing — no access model decided; the installer picks whatever is convenient: 2.5258 / Localhost-only bind + SSH tunnel + admin client certificate (PICK): 3.6392 / Bind to a private network / WireGuard interface reachable only via the operator's own tunnel: 3.3093 / Public exposure with mTLS client certificates + IP allowlist + fail2ban, no password-only auth: 2.8763 | Do nothing — no access model decided; the installer picks whatever is convenient: 2.6699 / Localhost-only bind + SSH tunnel + admin client certificate (PICK): 3.6602 / Bind to a private network / WireGuard interface reachable only via the operator's own tunnel: 3.2913 / Public exposure with mTLS client certificates + IP allowlist + fail2ban, no password-only auth: 2.8252 | No |
| Effort | Do nothing — no access model decided; the installer picks whatever is convenient: 2.5258 / Localhost-only bind + SSH tunnel + admin client certificate (PICK): 3.6701 / Bind to a private network / WireGuard interface reachable only via the operator's own tunnel: 3.3093 / Public exposure with mTLS client certificates + IP allowlist + fail2ban, no password-only auth: 2.8454 | Do nothing — no access model decided; the installer picks whatever is convenient: 2.6699 / Localhost-only bind + SSH tunnel + admin client certificate (PICK): 3.6311 / Bind to a private network / WireGuard interface reachable only via the operator's own tunnel: 3.2913 / Public exposure with mTLS client certificates + IP allowlist + fail2ban, no password-only auth: 2.8544 | No |

**Tipping weight:** none — no single ±20% weight variation changes the leader. The ranking is robust to weight choice (it may still be fragile to score choice; see §12).

**Close-call handling (<10% margin):** the margin is 9.59%, inside the <10% band, so the refined-spec rule fires: a ≤30-minute benchmark breaks the tie; if still tied, the more reversible option wins (`03:§12.1`). See §12 for the tie-break evidence.

## 8. Expected benefits

Localhost + SSH tunnel + admin client certificate: zero internet panel surface; the M5 verification is a direct, falsifiable test; the operator's workflow is one SSH command; the model is the only default Phase 1 should accept (Opinion 90%, `03:§10.3`).

## 9. Failure modes

SSH from Iran to the VPS blocked → admin access blocked; mitigated by the VPS being disposable/replaceable (new endpoint, new SSH path) and by the documented SSH-on-443 fallback (operator-approved only). Admin client certificate lost or compromised → re-issue from the operator machine (CA key never leaves it — D9); a certificate found in a log/repo is compromised by definition (AT-VPS-22). Docker bypasses the host firewall and publishes the panel port → mitigated by auditing `ss -tlnp` at install and at M5 verification, not assuming the firewall holds.

**Consequence if wrong:** If localhost + SSH tunnel is wrong (SSH proves unusable from Iran to the chosen VPS), the cost is admin inaccessibility — an operational failure, not a security failure — and the fallback is the private-network option (D14 revisited by ADR) or a new VPS endpoint. If the choice is wrong in the other direction (a public panel is exposed), the cost is an internet-reachable admin surface — a security failure, which is why the default is the closed option and the open option needs a justification that does not exist.

**Reversibility cost of undoing it:** High: the bind address and the access path are installer configuration. Moving from localhost + SSH tunnel to the private-network option costs an installer change; moving to public exposure costs an ADR justification plus the M5 test change. Nothing in the protocol depends on the access model.

## 10. Cost / complexity

Specification cost: the access model + the M5 verification steps in Phase 3. Implementation cost: installer (bind localhost, firewall default-deny, SSH key-only, fail2ban, client-certificate issuance) — standard practice, Opinion 85% feasible. Test cost: AT-VPS-01–22 at M5, including the positive 'panel not reachable publicly' test.

## 11. Experimental validation method

M5 installer verification (`09:§2` M5): data-plane port answers a valid test credential and refuses an invalid one; the panel is NOT reachable from the public internet (positive test — failure is a hard stop); the panel IS reachable through the SSH tunnel with the admin client certificate; a backup can be taken and its integrity verified. AT-VPS-01–22 (`11:§11`), including AT-VPS-22 (zero secrets in any repo, log, or public file). Falsifier: the panel is reachable from the public internet at M5 → the install fails (hard stop), the access model is re-opened, and no Phase 4 work starts on a publicly-exposed panel.

## 12. Final decision + evidence

**Decision (PROPOSED): Localhost-only bind + SSH tunnel + admin client certificate**

PROPOSED: localhost-only bind + SSH tunnel + admin client certificate as the default (and only default) access model. Public exposure is not adopted and would need a written ADR justification that does not exist. The private-network/WireGuard option is the named fallback if SSH proves unusable from Iran to the chosen VPS.

**Calculation that produced it:** S = 0.30·4 + 0.20·4 + 0.20·3 + 0.15·4 + 0.15·3 = 3.65, highest of the scored alternatives (private-network 3.30; public mTLS 2.85). Margin 0.35 points = 9.6% — inside the <10% close-call band, so the close-call rule fires: the tie-break is the M5 evidence (the positive 'not reachable publicly' test, which only the localhost option passes by design) and, if still tied, the more reversible option — localhost + SSH tunnel (no second system to operate, no extra internet-reachable service) is the more reversible, so the pick stands.

**Confidence:** Opinion, HIGH (85%) — localhost + SSH tunnel is the only default Phase 1 should accept (Opinion 90%, `03:§10.3`), and the M5 test is a direct positive test. Calibration p = 0.85.

**Status / date / deciders:** PROPOSED — 2026-11-25 — decided by Abolfazl (owner) at the Phase 1 gate; this record proposes, it does not accept. A decision without its M2/M3 evidence is PROPOSED, not ACCEPTED. Decided at: M0 (decision) → M5 (implemented and verified). Evidence that would settle it: AT-VPS-01–22 at M5, including the panel-reachability tests.

**Related items:** U21, R12, R13; AT-VPS-01–22 (esp. AT-VPS-22); M0, M5; P11; D9 (client-certificate lifecycle), D15 (admin-panel gate mechanism).
