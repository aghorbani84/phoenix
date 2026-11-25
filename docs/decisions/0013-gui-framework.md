# Decision Record 0013 — gui-framework

- **Decision:** 0013 (D13 of 12 §6)
- **Status:** PROPOSED
- **Date:** 2026-11-25
- **Version:** V1.0
- **Owner:** Abolfazl (decides; owner of the phase-gate go-ahead); Phase 1 architecture work (this record); implementation owner: Phase 2 (pending owner go-ahead)
- **Source:** phase0/03-feasibility-assessment.md §10.2, §11.3 decision 5; phase0/06-gui-architecture.md §10 (decision deferred to Phase 1 with explicit criteria); phase0/10-repo-structure.md §5.4 (Windows/Linux TBD by Phase 1 ADR)

## 1. Problem definition

GUI framework: which framework builds the Windows/Linux GUI (the Android client is already decided — Kotlin + Jetpack Compose, native, `06:§10`)? Tauri 2 is recommended by `03:§10.2` feasibility, but the decision needs ≥3 options including 'do nothing' (`03:§11.3` decision 5). The three-layer client (thin native UI → Go daemon → Rust core) and the IPC boundary (`06:§3`) constrain the choice: the UI never talks to the core, the core never talks to the UI, and the UI process never holds profile contents or key material (AT-GUI-02).

## 2. Known facts

- Android is decided: Kotlin + Jetpack Compose (native, maintained path; VpnService integration is Kotlin-first — `06:§10`). This ADR is about Windows/Linux only; it does not re-open Android.
- Tauri 2.0 went stable in October 2024 and targets Windows, macOS, Linux, iOS, and Android from one codebase — web frontend + Rust backend over a typed IPC bridge, using the system webview (WebView2 on Windows, WebKitGTK on Linux, System WebView on Android). The PHOENIX Rust core is a natural Tauri backend: the same Rust library serves desktop and mobile. [Fact, 90% — fetched 2026-11-25, `03:§10.2`.]
- Caveat (Fact, 85%): Tauri on mobile is still maturing — mobile needs native code in places (plugins with Kotlin/Swift). The VPN tunnel itself cannot live in a webview: Android requires a native VpnService (`03:§10.2`). (Not load-bearing here: Android is native anyway; this ADR is Windows/Linux.)
- Windows/Linux desktop VPN clients use a TUN device (Windows: wintun driver; Linux: /dev/net/tun) with admin/root for setup; Tauri desktop covers packaging (.msi/.exe, .deb/.rpm/.AppImage). No OS-level always-on equivalent to Android's lockdown exists on desktop — the kill switch is firewall-rule-based and must be designed per OS. [Fact, 85% / Theory 80% — `03:§10.2`.]
- The UI must be able to run unprivileged and talk to the daemon over the §3.5 IPC with no networking code of its own (`06:§10`, criterion 3). Persian-first, RTL-correct, English-second (`06:§1.1` G5). 10 main pages (Dashboard, Connection, Servers, Profiles, Zen Mode, Statistics, Security, Settings, Diagnostics, About — AT-GUI-05).

## 3. Assumptions

- Tauri 2's typed IPC bridge can carry the daemon↔UI message schema (`06:§3`) without the UI process holding secrets — the bridge is a transport for the §3.5 IPC, not a replacement for it. [MEDIUM — the IPC boundary is specified in `06:§3.5` (named pipe / Unix-domain socket), and Tauri's bridge sits above it; the layering must be verified in Phase 4.]
- A web frontend (HTML/CSS/JS) can be made RTL-correct and Persian-first to the same standard as a native UI. [MEDIUM — the design tokens / strings live in `clients/shared-ui/`, which is framework-independent.]

## 4. Unknown variables

- Whether Tauri 2's WebView2/WebKitGTK webview renders the Premium Dark / Obsidian / Champagne Gold design and the RTL layout identically on Windows and Linux — unverified (no prototype, no render).
- Windows kill-switch WFP filter set, Linux secret-storage canonical mechanism, Jalali library per platform, Android 14 foreground-service type (U21 — Phase 1 surveys/verifications).
- Whether the Tauri backend (Rust) talking to the Go daemon over the §3.5 IPC introduces a second IPC hop with its own failure modes — a Phase 4 design question.

## 5. Constraints

- Hard filter — security: the UI process never holds profile contents or key material (AT-GUI-02); the UI never talks to the core; localhost TCP for UI↔daemon IPC is forbidden (`06:§3.5`). A framework that requires the UI to hold secrets or to use localhost TCP is out.
- Hard filter — license/MSRV: the framework and its webview must be license-compatible with the project's open-source intent and on stable, maintained releases (no alpha/beta/RC/deprecated in production).
- The tunnel cannot live in a webview (Android VpnService; desktop TUN device with admin/root for setup).
- Persian-first, RTL-correct, English-second (`06:§1.1` G5); distribution must not rely on Google Play (`06:§6.10`).

## 6. Mathematical / computational model

Framework choice is a layering question: the UI is a thin shell over the §3.5 IPC (named pipe on Windows, Unix-domain socket 0600 on Linux), and the framework is judged on (a) whether it preserves the three-layer separation, (b) whether it can be unprivileged with no networking code of its own, (c) packaging for Windows/Linux, and (d) RTL/Persian rendering. The Rust core is the same in every option; only the shell changes.

## 7. Alternatives (≥3 when consequential, incl. "do nothing")

| # | Alternative | Pros | Cons |
|---|-------------|------|------|
| 1 | Do nothing — no GUI framework chosen; Phase 4 waits | Zero work now; no commitment. | M6 cannot start (it builds the GUI); the `03:§11.3` decision-5 obligation is unmet; the daemon API cannot be finalised against a UI that does not exist. Fails the M0 obligation. |
| 2 | Tauri 2 (Rust backend + system webview) for Windows/Linux (PICK) | Recommended by `03:§10.2` feasibility; one codebase for Windows/Linux (and potentially macOS later); the Rust core is a natural Tauri backend (same Rust library serves desktop and mobile); small binaries (system webview, no bundled Chromium); typed IPC bridge; desktop packaging covered (.msi/.exe, .deb/.rpm/.AppImage). | Webview rendering of RTL/Persian and the premium theme is unverified on both platforms; Tauri on mobile is still maturing (not load-bearing — Android is native); the Tauri backend talking to the Go daemon adds a second IPC hop to verify; webview CVEs are a dependency surface (system webview, patched by the OS). |
| 3 | Native per-OS shells: WinUI 3 (Windows) + GTK/Qt (Linux), thin shells over the same IPC | Truly native rendering and RTL on each OS; no webview dependency surface; thinnest possible shell (least code between the IPC and the pixels). | Two codebases to maintain (WinUI 3 + GTK/Qt) for one product; WinUI 3 and GTK have different string/layout idioms, so the shared-ui tokens must be re-implemented per shell; highest effort of the maintained options; no shared frontend code with any future platform. |
| 4 | Electron (Chromium + Node) for Windows/Linux | Mature; largest ecosystem; RTL/Persian rendering well understood in Chromium; one codebase. | Bundled Chromium (large binaries, own CVE surface, separate patching burden); Node backend duplicates what the Rust core + Go daemon already do — the Electron main process would be a third runtime in the stack; heaviest runtime cost on the user's machine; no advantage over Tauri that justifies the weight. |
| 5 | Flutter (Dart) for Windows/Linux | One codebase; good RTL support; small-to-medium binaries; mature desktop support. | A fourth language/runtime (Dart) in a Rust + Go + Kotlin stack; the Dart side cannot use the Rust core directly (FFI bridge needed, same as Tauri's, but without the Rust-backend advantage); desktop Linux support is less mature than Windows; no benefit over Tauri that justifies a new runtime. |

**Hard filters applied first** (license-compatible; MSRV set by Phase 1 and recorded; security: no known unpatched CVE, no custom crypto, audited libraries only — `03:§12.1`). A candidate failing any one is out, with no scoring. All candidates pass the license/MSRV/CVE filters at the framework level (stable, maintained releases). No candidate requires the UI to hold secrets or to use localhost TCP — any framework that did would be out. Electron and Flutter are scored, not filtered, and lose on the scored criteria (weight, runtime count, no Rust-backend advantage).

**Scoring.** Weights: correctness/safety .30, maintainability .20, performance .20, reversibility .15, effort .15; S = Σ(w·s); each criterion 0–5 (5 = best). Scores are **Opinion** unless a cell is labelled VERIFIED arithmetic or a cited Fact; the basis of each score is stated in the Pros/Cons above. Computed in Python (this generator):

| Alternative | Corr .30 | Maint .20 | Perf .20 | Rev .15 | Eff .15 | **S** |
|---|---:|---:|---:|---:|---:|---:|
| Do nothing — no GUI framework chosen; Phase 4 waits | 1 | 1 | 3 | 5 | 5 | **2.60** |
| Tauri 2 (Rust backend + system webview) for Windows/Linux (PICK) | 4 | 4 | 3 | 4 | 3 | **3.65** |
| Native per-OS shells: WinUI 3 (Windows) + GTK/Qt (Linux), thin shells over the same IPC | 4 | 3 | 3 | 3 | 2 | **3.15** |
| Electron (Chromium + Node) for Windows/Linux | 3 | 3 | 3 | 3 | 3 | **3.00** |
| Flutter (Dart) for Windows/Linux | 3 | 3 | 3 | 3 | 3 | **3.00** |

**Ranking:** 3.65 Tauri 2 (Rust backend + system webview) for Windows/Linux (PICK) > 3.15 Native per-OS shells: WinUI 3 (Windows) + GTK/Qt (Linux), thin shells over the same IPC > 3.00 Electron (Chromium + Node) for Windows/Linux > 3.00 Flutter (Dart) for Windows/Linux > 2.60 Do nothing — no GUI framework chosen; Phase 4 waits. Gap between top two: 0.50 points = 13.70% of the leader's score.

**Sensitivity (±20% per weight, other weights renormalised to sum 1.0):**

| Weight varied | −20% → S | +20% → S | Leader changes? |
|---|---|---|---|
| Correctness/safety | Do nothing — no GUI framework chosen; Phase 4 waits: 2.7021 / Tauri 2 (Rust backend + system webview) for Windows/Linux (PICK): 3.6277 / Native per-OS shells: WinUI 3 (Windows) + GTK/Qt (Linux), thin shells over the same IPC: 3.0957 / Electron (Chromium + Node) for Windows/Linux: 3.0000 / Flutter (Dart) for Windows/Linux: 3.0000 | Do nothing — no GUI framework chosen; Phase 4 waits: 2.5094 / Tauri 2 (Rust backend + system webview) for Windows/Linux (PICK): 3.6698 / Native per-OS shells: WinUI 3 (Windows) + GTK/Qt (Linux), thin shells over the same IPC: 3.1981 / Electron (Chromium + Node) for Windows/Linux: 3.0000 / Flutter (Dart) for Windows/Linux: 3.0000 | No |
| Maintainability | Do nothing — no GUI framework chosen; Phase 4 waits: 2.6667 / Tauri 2 (Rust backend + system webview) for Windows/Linux (PICK): 3.6354 / Native per-OS shells: WinUI 3 (Windows) + GTK/Qt (Linux), thin shells over the same IPC: 3.1562 / Electron (Chromium + Node) for Windows/Linux: 3.0000 / Flutter (Dart) for Windows/Linux: 3.0000 | Do nothing — no GUI framework chosen; Phase 4 waits: 2.5385 / Tauri 2 (Rust backend + system webview) for Windows/Linux (PICK): 3.6635 / Native per-OS shells: WinUI 3 (Windows) + GTK/Qt (Linux), thin shells over the same IPC: 3.1442 / Electron (Chromium + Node) for Windows/Linux: 3.0000 / Flutter (Dart) for Windows/Linux: 3.0000 | No |
| Performance | Do nothing — no GUI framework chosen; Phase 4 waits: 2.5833 / Tauri 2 (Rust backend + system webview) for Windows/Linux (PICK): 3.6771 / Native per-OS shells: WinUI 3 (Windows) + GTK/Qt (Linux), thin shells over the same IPC: 3.1562 / Electron (Chromium + Node) for Windows/Linux: 3.0000 / Flutter (Dart) for Windows/Linux: 3.0000 | Do nothing — no GUI framework chosen; Phase 4 waits: 2.6154 / Tauri 2 (Rust backend + system webview) for Windows/Linux (PICK): 3.6250 / Native per-OS shells: WinUI 3 (Windows) + GTK/Qt (Linux), thin shells over the same IPC: 3.1442 / Electron (Chromium + Node) for Windows/Linux: 3.0000 / Flutter (Dart) for Windows/Linux: 3.0000 | No |
| Reversibility | Do nothing — no GUI framework chosen; Phase 4 waits: 2.5258 / Tauri 2 (Rust backend + system webview) for Windows/Linux (PICK): 3.6392 / Native per-OS shells: WinUI 3 (Windows) + GTK/Qt (Linux), thin shells over the same IPC: 3.1546 / Electron (Chromium + Node) for Windows/Linux: 3.0000 / Flutter (Dart) for Windows/Linux: 3.0000 | Do nothing — no GUI framework chosen; Phase 4 waits: 2.6699 / Tauri 2 (Rust backend + system webview) for Windows/Linux (PICK): 3.6602 / Native per-OS shells: WinUI 3 (Windows) + GTK/Qt (Linux), thin shells over the same IPC: 3.1456 / Electron (Chromium + Node) for Windows/Linux: 3.0000 / Flutter (Dart) for Windows/Linux: 3.0000 | No |
| Effort | Do nothing — no GUI framework chosen; Phase 4 waits: 2.5258 / Tauri 2 (Rust backend + system webview) for Windows/Linux (PICK): 3.6701 / Native per-OS shells: WinUI 3 (Windows) + GTK/Qt (Linux), thin shells over the same IPC: 3.1856 / Electron (Chromium + Node) for Windows/Linux: 3.0000 / Flutter (Dart) for Windows/Linux: 3.0000 | Do nothing — no GUI framework chosen; Phase 4 waits: 2.6699 / Tauri 2 (Rust backend + system webview) for Windows/Linux (PICK): 3.6311 / Native per-OS shells: WinUI 3 (Windows) + GTK/Qt (Linux), thin shells over the same IPC: 3.1165 / Electron (Chromium + Node) for Windows/Linux: 3.0000 / Flutter (Dart) for Windows/Linux: 3.0000 | No |

**Tipping weight:** none — no single ±20% weight variation changes the leader. The ranking is robust to weight choice (it may still be fragile to score choice; see §12).

**Close-call handling (<10% margin):** the margin is 13.70%, outside the <10% band, so no benchmark is triggered by the close-call rule. See §12 for the tie-break evidence.

## 8. Expected benefits

Tauri 2: one Windows/Linux codebase; the Rust core serves as the Tauri backend directly (no FFI bridge for a fourth runtime); small binaries via the system webview; desktop packaging covered; the three-layer separation is preserved (Tauri backend talks to the Go daemon over the §3.5 IPC; the webview talks only to the Tauri backend). Android stays native Kotlin + Compose, so the mobile-maturity caveat does not apply.

## 9. Failure modes

Webview renders RTL/Persian or the premium theme incorrectly on Windows or Linux → caught at M6 (AT-GUI-01–20, incl. AT-GUI-05 page coverage and the RTL checks); the fallback is a native shell for the affected platform (D13 revisited by ADR — the per-platform shell is the escape hatch, not a redesign). System-webview CVE → patched by the OS, not by PHOENIX; the UI holds no secrets, so a webview compromise cannot reach key material (AT-GUI-02 is the containment). Second IPC hop (Tauri backend ↔ Go daemon) fails → the layering is verified mechanically at M6 (AT-GUI-13, IPC audit).

**Consequence if wrong:** If Tauri 2 is wrong (the webview cannot render the required UI, or the second IPC hop is unworkable), the cost is a per-platform native shell rebuild at M6 — bounded because the daemon, the IPC schema, and `clients/shared-ui/` (design tokens, icons, strings) are framework-independent and survive the switch. The UI is the thinnest layer by design (`06:§3.1`), so the blast radius of a wrong framework choice is the shell, not the system.

**Reversibility cost of undoing it:** High: the UI is a thin shell; the daemon, the IPC schema, and the shared-ui assets are framework-independent. Replacing Tauri with native shells costs one M6 rebuild of the shell, not a system redesign.

## 10. Cost / complexity

Specification cost: the framework choice + the Tauri-backend↔daemon layering in Phase 4. Implementation cost: one Windows/Linux shell (Tauri) vs two native shells; the Tauri option is the lowest-effort maintained alternative (4/5 effort is the matrix score — here scored 3/5 effort because the second IPC hop is real work). Test cost: AT-GUI-01–20 at M6 on Windows + Linux.

## 11. Experimental validation method

AT-GUI-01–20 at M6 (`11:§10`), including AT-GUI-02 (UI process never holds profile contents or key material), AT-GUI-04 (state derived only from the daemon's StateChanged), AT-GUI-05 (10 pages), AT-GUI-06 (outage classification honest), AT-GUI-13 (IPC audit — the boundary in `06:§3.4` holds mechanically). Falsifier: any AT-GUI failure attributable to the framework (rendering, IPC layering, privilege) → D13 is revisited by ADR with the native-shell fallback. `bench/` runs begin at M6 (E3 targets measured).

## 12. Final decision + evidence

**Decision (PROPOSED): Tauri 2 (Rust backend + system webview) for Windows/Linux**

PROPOSED: Tauri 2 for the Windows/Linux GUI (thin shell over the §3.5 IPC, Rust backend talking to the Go daemon); Android stays native Kotlin + Jetpack Compose (already decided, `06:§10`). The framework choice is decided now so Phase 4 can build; it is PROPOSED until the M6 AT-GUI evidence exists.

**Calculation that produced it:** S = 0.30·4 + 0.20·4 + 0.20·3 + 0.15·4 + 0.15·3 = 3.65, highest of the scored alternatives (native shells 3.15; Electron 3.00; Flutter 3.00). Margin 0.50 points = 13.7% — outside the close-call band.

**Confidence:** Opinion, HIGH (75%) — Tauri 2's fit is Fact (90%, fetched 2026-11-25) for the desktop targets, and the Rust-core-as-backend argument is structural; the RTL/webview rendering is MEDIUM (unverified, no prototype) and is the M6 test. Calibration p = 0.75.

**Status / date / deciders:** PROPOSED — 2026-11-25 — decided by Abolfazl (owner) at the Phase 1 gate; this record proposes, it does not accept. A decision without its M2/M3 evidence is PROPOSED, not ACCEPTED. Decided at: M0 (decision; build in Phase 4, M6). Evidence that would settle it: AT-GUI-01–20 at M6 on Windows + Linux.

**Related items:** U21; AT-GUI-01–20 (esp. AT-GUI-02, AT-GUI-05, AT-GUI-13); M0, M6; P12; D15 (IPC mechanism the shell uses), D23 (no telemetry in the UI).
