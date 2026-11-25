# PHOENIX

**Private. Resilient. Self-Hosted.** A research project on a censorship-resistant, self-hosted secure networking stack.

![Status](https://img.shields.io/badge/status-Phase%201%20%C2%B7%20architecture-blue)
![Content](https://img.shields.io/badge/content-design%20docs%20only-lightgrey)
[![License](https://img.shields.io/badge/license-PHOENIX%20Noncommercial%20v1.0-orange)](LICENSE)

## Overview

PHOENIX is an early-stage research and design effort for a secure networking ecosystem: an independent
protocol core, VPN clients (Android, Windows, Linux), and a self-hosted VPS server. The guiding principles are
standard, vetted cryptography, reproducible measurement, and **no unprovable claims**: every resilience
property is meant to be stated with the test that would falsify it and the residual risk it leaves.

The work is defensive in purpose: helping people keep private, reliable access to the open internet on
infrastructure they run themselves.

> **Status: Phase 1, architecture.** This repository currently contains design documents only. There is no
> code, binary, or release yet, and nothing here should be read as a working tool or as a performance or
> reliability guarantee.

## What's in the repository

- **Architecture Decision Records** in [`docs/decisions/`](docs/decisions/), each using a 12-field template
  (problem, known facts, assumptions, unknowns, constraints, alternatives, failure modes, validation method, final decision, and more). Topics include:
  - Threat model and the adversary capability matrix
  - Transport choice (TLS 1.3 over TCP, QUIC, or multipath) and cover / fallback order
  - Round-trip-time (RTT) fingerprinting strategy
  - Post-quantum posture (hybrid X25519 + ML-KEM) and where the hybrid key share is placed
  - Authentication placement, server variant, key pinning, rotation, and compromise recovery
  - DNS resolution without relying on a fixed resolver
  - Session-layer retransmission on transport switch and scheduler classifiability
  - Automatic regeneration of a blocked instance ("rebirth")
  - Measurement thresholds and how they get calibrated before anything is judged against them
  - Lawful collection of the measurement corpus and whether the test harness is public
  - GUI framework, local IPC, and admin-panel access for the self-hosted server
  - Telemetry (none by default is the stated starting position) and licence
- **Calibration log** in [`docs/calibration.md`](docs/calibration.md) for tracking forecast confidence over time.

Most records are currently **Proposed**; the RTT strategy and licence records are **Accepted**.

## Planned components

These are design targets described in the records, not existing software:

| Component | Notes |
| --- | --- |
| Protocol core | Built on standard libraries (rustls / quinn are the candidates under evaluation) |
| Android client | Kotlin + Jetpack Compose |
| Windows / Linux client | GUI framework under evaluation (ADR 0013) |
| Self-hosted server | VPS install with a locally restricted admin panel (ADR 0014) |

## License

PHOENIX is released under the **PHOENIX Noncommercial License v1.0**. Non-commercial use, modification, and
sharing are free with attribution; commercial use requires a paid licence from the author. This is a
source-available licence, not an OSI-approved open-source licence. See [LICENSE](LICENSE) for the full terms.

## Author

[Abolfazl Ghorbani](https://github.com/aghorbani84)
