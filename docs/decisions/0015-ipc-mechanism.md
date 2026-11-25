# Decision Record 0015 — ipc-mechanism

- **Decision:** 0015 (D15 of 12 §6)
- **Status:** PROPOSED
- **Date:** 2026-11-25
- **Version:** V1.0
- **Owner:** Abolfazl (decides; owner of the phase-gate go-ahead); Phase 1 architecture work (this record); implementation owner: Phase 2 (pending owner go-ahead)
- **Source:** phase0/05-protocol-design-draft.md §7 Q13; phase0/06-gui-architecture.md §3.4–§3.5 (IPC mechanism per platform; localhost TCP forbidden); phase0/04-architecture-proposals.md §1.2 (IPC, 'I cannot verify a best IPC mechanism')

## 1. Problem definition

Local IPC mechanism + admin-panel gate: how do the core, the daemon (control plane), and the UI communicate on the local machine, and how is the admin panel gated? Localhost TCP for UI↔daemon IPC is FORBIDDEN (`06:§3.5`, Opinion 90%). Assuming an IPC mechanism (e.g. an unauthenticated local socket) would put keys within reach of any local process (`05:§7` Q13).

## 2. Known facts

- Per-platform mechanisms specified in `06:§3.5`: Android — bound service + AIDL / Messenger, or Kotlin StateFlow over a ServiceConnection (the VpnService is already a bound service; no network socket, no localhost TCP). Windows — named pipe (\\.\pipe\phoenix-daemon) with an integrity-level ACL, or COM (DACL admits only the interactive user's UI process). Linux — Unix-domain socket in the runtime directory ($XDG_RUNTIME_DIR/phoenix.sock), 0600 (filesystem permissions are the access control; abstract sockets avoided — no filesystem ACL).
- Localhost TCP for UI↔daemon IPC is the single most common architectural mistake in desktop VPN clients (Opinion 90%, `06:§3.5`): convenient, works in demos, exposes the daemon to every other process on the machine (and, on misconfigured hosts, to the network). The specified designs cost one extra day each and remove the entire class.
- The §3.4 invariant (security, not guideline): the UI process never holds profile contents or key material; the UI never shows DNS query names, server IPs, or full key fingerprints; state is derived only from the daemon's StateChanged. A Phase 1 review must be able to test each item mechanically (grep the IPC schema; grep the UI log calls).
- Handles, not data (`06:§3.6`): the daemon passes profile handles (references to material the core loaded through a separate, one-time, memory-only handoff at session start). The core never receives a profile as a JSON blob over the command channel, and the daemon never retains core session keys after handoff.
- The admin-panel gate is the D14 access model (localhost + SSH tunnel + admin client certificate); the gate mechanism for the panel's own API is part of this decision.

## 3. Assumptions

- The per-platform mechanisms in `06:§3.5` are implementable as specified (bound service / named pipe with ACL / UDS 0600) without a localhost-TCP fallback. [HIGH — each is a standard OS mechanism.]
- The IPC schema (message tables in `06:§3`) is platform-independent; only the transport differs. [MEDIUM — `06:§5` states this; the schema itself is a Phase 1 deliverable.]

## 4. Unknown variables

- A best IPC mechanism — 'I cannot verify a best IPC mechanism from the Phase 0 sources' (`04:§1.2`; `05` App. B item 6). This record adopts the `06:§3.5` mechanisms, which are specified but unimplemented; AT-GUI-13 at M6 is the mechanical test.
- The exact admin-panel gate mechanism beyond the D14 access model (which authentication the panel's API requires once the tunnel is established) — specified here as the admin client certificate (D14) + a per-session token issued by the panel backend; the token's format is a Phase 3 detail.

## 5. Constraints

- Hard filter — security: localhost TCP for UI↔daemon IPC is FORBIDDEN (`06:§3.5`). Any candidate using it is out, no scoring.
- Hard filter — security: the UI process never holds profile contents or key material (AT-GUI-02); the §3.4 invariant holds mechanically (AT-GUI-13).
- The daemon is small, has no third-party UI dependencies, and is the only component besides the core that needs key material (`06:§10` reasoning: the profile is the credential).

## 6. Mathematical / computational model

IPC as a trust-boundary question: which OS mechanism gives the daemon a channel that (a) only the intended UI process can connect to, (b) carries no secrets in a form any local process can read, and (c) is auditable mechanically (AT-GUI-13). Filesystem permissions (Linux UDS 0600), a DACL (Windows named pipe), and the Android binder (bound service) each provide (a) by an OS-enforced mechanism; localhost TCP provides none of them. The admin-panel gate is the same question one hop up: the panel's API is reachable only through the D14 tunnel, and the tunnel's authentication (SSH key + admin client certificate) is the gate.

## 7. Alternatives (≥3 when consequential, incl. "do nothing")

| # | Alternative | Pros | Cons |
|---|-------------|------|------|
| 1 | Do nothing — no IPC mechanism decided; each platform picks whatever is convenient | Zero work now. | The convenient default is localhost TCP — the forbidden option. The §3.4 invariant cannot be tested mechanically without a specified mechanism. Fails the M0 obligation. |
| 2 | Per-platform OS mechanisms from 06 §3.5 (Android bound service/AIDL; Windows named pipe + ACL; Linux UDS 0600) + admin-panel gate via D14 tunnel + admin client certificate (PICK) | Each mechanism is OS-enforced (binder / DACL / filesystem permissions); no network socket anywhere; the §3.4 invariant is mechanically testable (AT-GUI-13); the admin-panel gate reuses the D14 authentication (no second credential system). | Three mechanisms to implement and test (one per platform) instead of one; the IPC schema must be platform-independent while the transports differ (a Phase 1 schema deliverable). |
| 3 | Localhost TCP for UI↔daemon IPC (with an auth token) | One mechanism everywhere; simplest to implement and debug; an auth token adds a check. | FORBIDDEN (`06:§3.5`, Opinion 90%): exposes the daemon to every other process on the machine (and, on misconfigured hosts, to the network); an auth token on localhost is not an access control (any local process can read it from the UI's memory or config). FAILS the hard filter. Listed in the table but not scored. |
| 4 | Single cross-platform IPC: gRPC over a Unix-domain socket everywhere (incl. Windows via npipe) | One IPC stack everywhere; schema-first (protobuf); mature tooling. | Adds a gRPC dependency to the daemon (the daemon must stay small, with no third-party UI dependencies — a gRPC runtime is a new dependency class); on Android the bound-service model is still the native path (gRPC over UDS does not replace the VpnService binding); the schema benefit is real but the daemon-size cost is not justified when the message tables in `06:§3` are already the schema. |
| 5 | Unauthenticated local socket (no access control; 'localhost is trusted') | Simplest possible; zero configuration. | FAILS the hard filter: puts keys within reach of any local process (`05:§7` Q13) — the exact failure the question names. Out. |

**Hard filters applied first** (license-compatible; MSRV set by Phase 1 and recorded; security: no known unpatched CVE, no custom crypto, audited libraries only — `03:§12.1`). A candidate failing any one is out, with no scoring. Localhost TCP and the unauthenticated local socket both fail the hard security filter (`06:§3.5` forbids localhost TCP; `05:§7` Q13 forbids an unauthenticated socket) and are out, no scoring counted. All other candidates pass the license/MSRV/CVE filters.

**Scoring.** Weights: correctness/safety .30, maintainability .20, performance .20, reversibility .15, effort .15; S = Σ(w·s); each criterion 0–5 (5 = best). Scores are **Opinion** unless a cell is labelled VERIFIED arithmetic or a cited Fact; the basis of each score is stated in the Pros/Cons above. Computed in Python (this generator):

| Alternative | Corr .30 | Maint .20 | Perf .20 | Rev .15 | Eff .15 | **S** |
|---|---:|---:|---:|---:|---:|---:|
| Do nothing — no IPC mechanism decided; each platform picks whatever is convenient | 1 | 1 | 3 | 5 | 5 | **2.60** |
| Per-platform OS mechanisms from 06 §3.5 (Android bound service/AIDL; Windows named pipe + ACL; Linux UDS 0600) + admin-panel gate via D14 tunnel + admin client certificate (PICK) | 4 | 4 | 3 | 4 | 4 | **3.80** |
| Localhost TCP for UI↔daemon IPC (with an auth token) | — | — | — | — | — | **EXCLUDED by hard filter — not scored** |
| Single cross-platform IPC: gRPC over a Unix-domain socket everywhere (incl. Windows via npipe) | 3 | 4 | 3 | 3 | 3 | **3.20** |
| Unauthenticated local socket (no access control; 'localhost is trusted') | — | — | — | — | — | **EXCLUDED by hard filter — not scored** |

**Ranking:** 3.80 Per-platform OS mechanisms from 06 §3.5 (Android bound service/AIDL; Windows named pipe + ACL; Linux UDS 0600) + admin-panel gate via D14 tunnel + admin client certificate (PICK) > 3.20 Single cross-platform IPC: gRPC over a Unix-domain socket everywhere (incl. Windows via npipe) > 2.60 Do nothing — no IPC mechanism decided; each platform picks whatever is convenient. Gap between top two: 0.60 points = 15.79% of the leader's score.

**Sensitivity (±20% per weight, other weights renormalised to sum 1.0):**

| Weight varied | −20% → S | +20% → S | Leader changes? |
|---|---|---|---|
| Correctness/safety | Do nothing — no IPC mechanism decided; each platform picks whatever is convenient: 2.7021 / Per-platform OS mechanisms from 06 §3.5 (Android bound service/AIDL; Windows named pipe + ACL; Linux UDS 0600) + admin-panel gate via D14 tunnel + admin client certificate (PICK): 3.7872 / Single cross-platform IPC: gRPC over a Unix-domain socket everywhere (incl. Windows via npipe): 3.2128 | Do nothing — no IPC mechanism decided; each platform picks whatever is convenient: 2.5094 / Per-platform OS mechanisms from 06 §3.5 (Android bound service/AIDL; Windows named pipe + ACL; Linux UDS 0600) + admin-panel gate via D14 tunnel + admin client certificate (PICK): 3.8113 / Single cross-platform IPC: gRPC over a Unix-domain socket everywhere (incl. Windows via npipe): 3.1887 | No |
| Maintainability | Do nothing — no IPC mechanism decided; each platform picks whatever is convenient: 2.6667 / Per-platform OS mechanisms from 06 §3.5 (Android bound service/AIDL; Windows named pipe + ACL; Linux UDS 0600) + admin-panel gate via D14 tunnel + admin client certificate (PICK): 3.7917 / Single cross-platform IPC: gRPC over a Unix-domain socket everywhere (incl. Windows via npipe): 3.1667 | Do nothing — no IPC mechanism decided; each platform picks whatever is convenient: 2.5385 / Per-platform OS mechanisms from 06 §3.5 (Android bound service/AIDL; Windows named pipe + ACL; Linux UDS 0600) + admin-panel gate via D14 tunnel + admin client certificate (PICK): 3.8077 / Single cross-platform IPC: gRPC over a Unix-domain socket everywhere (incl. Windows via npipe): 3.2308 | No |
| Performance | Do nothing — no IPC mechanism decided; each platform picks whatever is convenient: 2.5833 / Per-platform OS mechanisms from 06 §3.5 (Android bound service/AIDL; Windows named pipe + ACL; Linux UDS 0600) + admin-panel gate via D14 tunnel + admin client certificate (PICK): 3.8333 / Single cross-platform IPC: gRPC over a Unix-domain socket everywhere (incl. Windows via npipe): 3.2083 | Do nothing — no IPC mechanism decided; each platform picks whatever is convenient: 2.6154 / Per-platform OS mechanisms from 06 §3.5 (Android bound service/AIDL; Windows named pipe + ACL; Linux UDS 0600) + admin-panel gate via D14 tunnel + admin client certificate (PICK): 3.7692 / Single cross-platform IPC: gRPC over a Unix-domain socket everywhere (incl. Windows via npipe): 3.1923 | No |
| Reversibility | Do nothing — no IPC mechanism decided; each platform picks whatever is convenient: 2.5258 / Per-platform OS mechanisms from 06 §3.5 (Android bound service/AIDL; Windows named pipe + ACL; Linux UDS 0600) + admin-panel gate via D14 tunnel + admin client certificate (PICK): 3.7938 / Single cross-platform IPC: gRPC over a Unix-domain socket everywhere (incl. Windows via npipe): 3.2062 | Do nothing — no IPC mechanism decided; each platform picks whatever is convenient: 2.6699 / Per-platform OS mechanisms from 06 §3.5 (Android bound service/AIDL; Windows named pipe + ACL; Linux UDS 0600) + admin-panel gate via D14 tunnel + admin client certificate (PICK): 3.8058 / Single cross-platform IPC: gRPC over a Unix-domain socket everywhere (incl. Windows via npipe): 3.1942 | No |
| Effort | Do nothing — no IPC mechanism decided; each platform picks whatever is convenient: 2.5258 / Per-platform OS mechanisms from 06 §3.5 (Android bound service/AIDL; Windows named pipe + ACL; Linux UDS 0600) + admin-panel gate via D14 tunnel + admin client certificate (PICK): 3.7938 / Single cross-platform IPC: gRPC over a Unix-domain socket everywhere (incl. Windows via npipe): 3.2062 | Do nothing — no IPC mechanism decided; each platform picks whatever is convenient: 2.6699 / Per-platform OS mechanisms from 06 §3.5 (Android bound service/AIDL; Windows named pipe + ACL; Linux UDS 0600) + admin-panel gate via D14 tunnel + admin client certificate (PICK): 3.8058 / Single cross-platform IPC: gRPC over a Unix-domain socket everywhere (incl. Windows via npipe): 3.1942 | No |

**Tipping weight:** none — no single ±20% weight variation changes the leader. The ranking is robust to weight choice (it may still be fragile to score choice; see §12).

**Close-call handling (<10% margin):** the margin is 15.79%, outside the <10% band, so no benchmark is triggered by the close-call rule. See §12 for the tie-break evidence.

## 8. Expected benefits

Per-platform OS mechanisms: the IPC boundary holds mechanically (AT-GUI-13); the UI never holds secrets (AT-GUI-02); the admin-panel gate reuses D14's authentication instead of inventing a second credential system; the daemon stays small and dependency-light.

## 9. Failure modes

A platform's mechanism is misconfigured (e.g. the Windows named-pipe DACL admits more than the interactive user's UI process, or the Linux socket is created with wider-than-0600 permissions) → any local process can reach the daemon; mitigated by AT-GUI-13 (mechanical audit of the boundary) and by the installer setting the permissions (M5/M6 verification). The IPC schema drifts per platform → the message tables diverge; mitigated by the schema being platform-independent (`06:§5`) and reviewed as one artefact. The admin client certificate is compromised → the panel gate fails open to the certificate holder; mitigated by D9 (revocation, single operation) and AT-VPS-22.

**Consequence if wrong:** If the per-platform choice is wrong (e.g. the Windows named pipe proves unworkable and COM is needed instead), the cost is a transport swap on one platform — the IPC schema is unchanged, so the UI and daemon logic are unaffected. If the choice is wrong in the worse direction (a mechanism admits an unintended local process), the cost is a local privilege-escalation path to key material — which is why AT-GUI-13 audits the boundary mechanically at M6 before any release, and why localhost TCP (the most likely such failure) is forbidden outright.

**Reversibility cost of undoing it:** High: the IPC transport is per-platform configuration; the schema is platform-independent. Swapping a transport costs an ADR + a platform implementation change; the UI, daemon, and core logic are unchanged.

## 10. Cost / complexity

Specification cost: the IPC schema (message tables, `06:§3`) as a Phase 1 deliverable + the transport table (`06:§3.5`, adopted here). Implementation cost: three transports (one per platform), each 'one extra day' over localhost TCP (`06:§3.5`). Test cost: AT-GUI-13 (mechanical IPC audit) at M6; AT-VPS panel-gate tests at M5.

## 11. Experimental validation method

AT-GUI-13 at M6 (`11:§10`): the IPC boundary in `06:§3.4` holds mechanically (grep the IPC schema; grep the UI log calls; verify the transports are the specified ones, not localhost TCP). AT-GUI-02 (UI process never holds profile contents or key material). M5 panel-gate tests (D14 validation): the panel is reachable only through the SSH tunnel with the admin client certificate. Falsifier: any AT-GUI-13 finding (a secret crossing the boundary, a localhost-TCP listener, a permission wider than specified) → the mechanism is wrong and D15 is re-opened; a localhost-TCP listener found anywhere is a failed build, not a finding to waive.

## 12. Final decision + evidence

**Decision (PROPOSED): Per-platform OS mechanisms from 06 §3.5 (Android bound service/AIDL; Windows named pipe + ACL; Linux UDS 0600) + admin-panel gate via D14 tunnel + admin client certificate**

PROPOSED: adopt the per-platform OS mechanisms from `06:§3.5` — Android bound service + AIDL / Messenger (or Kotlin StateFlow over a ServiceConnection); Windows named pipe (\\.\pipe\phoenix-daemon) with an integrity-level ACL (or COM); Linux Unix-domain socket ($XDG_RUNTIME_DIR/phoenix.sock), 0600, no abstract sockets. Localhost TCP for UI↔daemon IPC is forbidden. Admin-panel gate: the D14 access model (localhost + SSH tunnel) with the admin client certificate as the gate authentication, plus a per-session token issued by the panel backend (format a Phase 3 detail).

**Calculation that produced it:** S = 0.30·4 + 0.20·4 + 0.20·3 + 0.15·4 + 0.15·4 = 3.80, highest of the scored alternatives (gRPC-over-UDS 3.20). Margin 0.60 points = 15.8% — outside the close-call band. Localhost TCP and the unauthenticated socket are excluded by the hard filter before scoring.

**Confidence:** Opinion, HIGH (85%) — the mechanisms are standard OS primitives and the forbidden option is explicitly named in `06:§3.5` (Opinion 90%); the residual uncertainty is implementation (permissions set correctly), which AT-GUI-13 tests mechanically. Calibration p = 0.85.

**Status / date / deciders:** PROPOSED — 2026-11-25 — decided by Abolfazl (owner) at the Phase 1 gate; this record proposes, it does not accept. A decision without its M2/M3 evidence is PROPOSED, not ACCEPTED. Decided at: M0/M1 (decision) → M5 (panel gate implemented) → M6 (AT-GUI-13 audit). Evidence that would settle it: AT-GUI-13 at M6; M5 panel-gate tests.

**Related items:** U17; AT-GUI-02, AT-GUI-13, AT-VPS-01–22; M0, M1, M5, M6; P11, P12; D14 (admin-panel access model — the gate reuses it), D13 (GUI framework — the shell uses this IPC), D9 (certificate lifecycle).
