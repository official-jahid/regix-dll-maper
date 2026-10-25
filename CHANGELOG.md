# CHANGELOG

All notable changes to the `regix-dll-maper` documentation project are documented in this file.

## [v1.0.0] — 2026-10-25

### Added

- `README.md` — Full human-facing documentation with:
  - Authorized-use banner and educational disclaimer
  - Status dashboard (emoji indicators)
  - Visual flow diagram (Mermaid + ASCII fallback)
  - Visual flow dashboard (ASCII panel)
  - Repository structure
  - Prerequisites table
  - Installation steps
  - Configuring guide (config.ps1 dot-sourcing pattern)
  - Sample run walkthrough
  - Safety & ethical guardrails (maximum)
  - Detection & defensive notes summary
  - Legal disclaimer summary
  - Contributing guidelines
  - MIT license reference
- `AGENTS.md` — AI coding agent documentation with:
  - Strict governance update rules and guardrails
  - Pre-edit checklist (7 items)
  - Versioning and changelog policy
  - File map with edit permissions
  - Complete technical reference of `main.ps1` (read-only):
    - Service disruption
    - Logging suppression
    - Temp cleanup
    - Strict mode and preference silencing
    - Admin gate and hidden console
    - LogWiper C# class (event log wiper)
    - NativeLoader C# class (manual DLL mapper)
    - DLL download and mapping
    - Final wipe and cleanup
  - Configurable elements table
  - AI contribution checklist
  - Prohibited AI actions
  - Related documentation links
- `config.ps1.example` — Configuration template with `$dllUrl` placeholder
- `docs/DETECTION.md` — Comprehensive defensive detection:
  - Windows Event Log indicators
  - Sigma rules (4 rules: PS logging disabled, EventLog stopped, hidden console, GitHub DLL download)
  - YARA rules (2 rules: script patterns, RWX mapping artifacts)
  - EDR/hunting queries (PowerShell, network, process)
  - Blue-team incident response playbook (5 phases)
  - MITRE ATT&CK references
- `docs/DISCLAIMER.md` — Full legal documentation:
  - Authorized use only banner
  - Written permission requirement table
  - Per-jurisdiction notes (US, UK, EU, Canada, Australia, other)
  - No warranty / no liability statement
  - Rules of Engagement (RoE) template
  - Pre-run confirmation checklist (9 items)
  - Responsible disclosure guidance
  - License summary
- `CHANGELOG.md` — This file

### Constraints

- `main.ps1` is NOT modified — remains as the audited core script
- All documentation framed for educational and authorized security-testing purposes only
