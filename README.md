# regix-dll-maper

[![License: MIT](https://img.shields.io/badge/License-MIT-yellow.svg)](./LICENSE)
[![PowerShell](https://img.shields.io/badge/PowerShell-5.1%20%7C%207+-blue.svg)](https://docs.microsoft.com/powershell/)
[![Windows](https://img.shields.io/badge/Windows-10%2F11%2C%20Server%202019%2F2022-green.svg)](https://www.microsoft.com/windows)
[![Educational](https://img.shields.io/badge/Purpose-Educational%2FTesting-orange.svg)](./docs/DISCLAIMER.md)
[![Authorized Use Only](https://img.shields.io/badge/Status-Authorized%20Use%20Only-red.svg)](./docs/DISCLAIMER.md)

> ⚠️ **regix-dll-maper** is an offensive-security style PowerShell toolkit intended **strictly for authorized red-team testing, security research, and educational purposes**. It disables Windows logging, wipes event logs, and performs manual DLL mapping/injection. **Never run this on systems you do not own or have explicit written permission to test.**

---

## 🚨 AUTHORIZED USE ONLY

> [!DANGER]
> **This tool performs actions that can be detected by EDR/AV and may violate laws if used without authorization.**
>
> - Only run on systems **you own** or have **written authorization** to test.
> - Unauthorized use may violate computer crime laws (e.g., CFAA in the US, Computer Misuse Act in the UK, GDPR-related statutes in the EU).
> - The author accepts **no liability** for misuse. See [`docs/DISCLAIMER.md`](./docs/DISCLAIMER.md) for the full legal notice.

---

## 📋 Project Overview

**regix-dll-maper** is a single-file PowerShell tool (`main.ps1`) that demonstrates:

| Capability             | Description                                                                                                                                    |
| ---------------------- | ---------------------------------------------------------------------------------------------------------------------------------------------- |
| 🔇 Service Disruption  | Stops `WSearch`, `cbdhsvc*`, `VSS*`, `fhsvc*` services                                                                                         |
| 📛 Logging Suppression | Disables PowerShell ModuleLogging & ScriptBlockLogging via registry                                                                            |
| 🧹 Temp Cleanup        | Removes recently created `.cs`, `.dll`, `.pdb`, `.tmp`, `.ps1` from `%TEMP%`                                                                   |
| 🛡️ Admin Gate          | Exits unless running as Administrator                                                                                                          |
| 👻 Hidden Console      | Hides the console window via `ShowWindow(SW_HIDE)`                                                                                             |
| 🗑️ Event Log Wiper     | Stops the EventLog service, deletes/truncates `.evtx` files (System, PowerShell, Security), then restarts the service                          |
| 📦 Manual DLL Mapper   | Downloads a DLL from a remote URL, parses PE/MZ headers, allocates RWX memory, maps sections, resolves imports, relocates, and calls `DllMain` |
| 🧼 Final Cleanup       | Wipes event logs again, clears PowerShell history, nulls variables, forces GC                                                                  |

---

## 🎛️ Status Dashboard

| Component          | Status           | Details                                     |
| ------------------ | ---------------- | ------------------------------------------- |
| **Runner**         | `main.ps1`       | PowerShell script (hidden console)          |
| **Console**        | 👻 Hidden        | `ShowWindow(..., 0)` — no visible window    |
| **Privilege**      | 🔐 Administrator | Required; exits if not elevated             |
| **DLL Source**     | 🌐 Remote URL    | Configurable via `config.ps1`               |
| **Architecture**   | ⚖️ x86 / x64     | Must match host PowerShell architecture     |
| **Logging**        | 🔇 Suppressed    | ModuleLogging & ScriptBlockLogging disabled |
| **Event Logs**     | 🗑️ Wiped         | System, PowerShell, Security `.evtx` files  |
| **Temp Artifacts** | 🧹 Cleaned       | `%TEMP%` artifacts removed                  |

---

## 🧭 Visual Flow Diagram

### Mermaid Flowchart (rendered in GitHub / VS Code / GitLab)

```mermaid
flowchart TD
    Start([Start main.ps1]) --> AdminCheck{Administrator?}
    AdminCheck -->|No| Exit1([Exit silently])
    AdminCheck -->|Yes| StopServices[Stop WSearch, cbdhsvc, VSS, fhsvc]
    StopServices --> DisableLogging[Disable ModuleLogging & ScriptBlockLogging via Registry]
    DisableLogging --> CleanTemp[Clean %TEMP% artifacts]
    CleanTemp --> HideConsole[Hide console window SW_HIDE]
    HideConsole --> DotSource[Dot-source config.ps1 → set $dllUrl]
    DotSource --> DownloadDLL[Download DLL from $dllUrl via WebClient]
    DownloadDLL --> ParsePE{Valid MZ + PE?}
    ParsePE -->|No| Exit2([Exit: Invalid payload])
    ParsePE -->|Yes| AllocMem[VirtualAlloc RWX memory]
    AllocMem --> MapSections[Copy PE headers + sections into memory]
    MapSections --> Relocate[Apply base relocation (delta)]
    Relocate --> ResolveImports[Resolve DLL imports via GetProcAddress]
    ResolveImports --> Protect[VirtualProtect sections to original characteristics]
    Protect --> Flush[FlushInstructionCache]
    Flush --> CallEntry{callEntry = $true?}
    CallEntry -->|Yes| CallDllMain[Call DllMain / CRT_INIT]
    CallEntry -->|No| SkipEntry
    CallDllMain --> SkipEntry
    SkipEntry --> WipeLogs[Wipe Event Logs: System, PowerShell, Security]
    WipeLogs --> FinalCleanup[Clean %TEMP%, clear PS history, null vars, GC]
    FinalCleanup --> End([Done])
```

### ASCII Art Fallback (for terminals / print)

```
┌─────────────────┐
│   Start main.ps1 │
└─────────┬───────┘
          ▼
  ┌───────────────┐
  │  Admin?       │──No──► Exit
  └───────┬───────┘
          │ Yes
          ▼
  ┌─────────────────────────┐
  │ Stop services + disable │

---

## 🖥️ Visual Flow Dashboard

```

┌─────────────────────────────────────────────────────────────────────────┐
│ regix-dll-maper — EXECUTION DASHBOARD │
├─────────────────────────────────────────────────────────────────────────┤
│ │
│ ┌──────────────┐ ┌──────────────┐ ┌──────────────┐ │
│ │ 1. SERVICES │ │ 2. LOGGING │ │ 3. CONSOLE │ │
│ │ ⛔ STOPPED │ │ 🔇 DISABLED │ │ 👻 HIDDEN │ │
│ └──────────────┘ └──────────────┘ └──────────────┘ │
│ │
│ ┌──────────────┐ ┌──────────────┐ ┌──────────────┐ │
│ │ 4. TEMP │ │ 5. DLL DOWN │ │ 6. MAP DLL │ │
│ │ 🧹 CLEANED │ │ 🌐 FETCHED │ │ 📦 MAPPED │ │
│ └──────────────┘ └──────────────┘ └──────────────┘ │
│ │
│ ┌──────────────┐ ┌──────────────┐ ┌──────────────┐ │
│ │ 7. ENTRY │ │ 8. LOG WIPE │ │ 9. CLEANUP │ │
│ │ ▶️ CALLED │ │ 🗑️ WIPED │ │ ✅ DONE │ │
│ └──────────────┘ └──────────────┘ └──────────────┘ │
│ │
└─────────────────────────────────────────────────────────────────────────┘

```

---

## 📁 Repository Structure

```

regix-dll-maper/
├── README.md # Human-facing docs (this file)
├── AGENTS.md # AI coding agent documentation
├── LICENSE # MIT License
├── main.ps1 # Core script — DO NOT MODIFY
├── config.ps1.example # Configuration template (copy to config.ps1)
├── CHANGELOG.md # Version history
└── docs/
├── DETECTION.md # Defensive detection rules (Sigma, EventLog, YARA)
└── DISCLAIMER.md # Full legal / authorization / RoE

````

---

## ✅ Prerequisites

| Requirement | Details |
|-------------|---------|
| **OS** | Windows 10 / 11, Windows Server 2019 / 2022 |
| **PowerShell** | Windows PowerShell 5.1 (built-in) or PowerShell 7+ |
| **Privileges** | Administrator (required; script exits if not elevated) |
| **Execution Policy** | `Set-ExecutionPolicy Bypass -Scope Process` |
| **File Trust** | `Unblock-File` on downloaded scripts |
| **.NET Framework** | 4.7.2+ (required for `Add-Type` C# compilation) |
| **Temp Access** | Write access to `%TEMP%` |
| **Architecture** | DLL bitness must match PowerShell host (x86 DLL ↔ x86 PS, x64 DLL ↔ x64 PS) |
| **Network** | Outbound HTTPS to download the DLL from `$dllUrl` |

---

## ⚙️ Configuring Guide

The DLL source URL is **not** hardcoded in a way that requires editing `main.ps1`. Instead, use a companion `config.ps1` file that is dot-sourced before running.

### Step 1 — Create `config.ps1`

Copy the example:
```powershell
Copy-Item config.ps1.example config.ps1
````

### Step 2 — Edit `config.ps1`

Open `config.ps1` and replace the placeholder URL:

```powershell
# config.ps1 — regix-dll-maper configuration
# Edit this file to set your DLL source URL

# Remote DLL URL — replace {username}, {repo}, {branch}, and {DLL Name}
$dllUrl = "https://raw.githubusercontent.com/{username}/{repo name}/refs/heads/{branch name}/{DLL Name e.g. example.dll}"

# Optional: additional settings can be added here
```

### Step 3 — Dot-source and run

```powershell
# Dot-source the config so $dllUrl is available in the session
. ./config.ps1

# Run main.ps1 (must be Administrator)
.\main.ps1
```

> [!IMPORTANT]
> `main.ps1` contains the placeholder `$dllUrl` line as documentation, but the **working** value should come from `config.ps1`. Do not edit `main.ps1` — keep it pristine for auditing and supply-chain integrity.

### Configuration Reference

| Setting          | File         | Description                              |
| ---------------- | ------------ | ---------------------------------------- |
| `$dllUrl`        | `config.ps1` | HTTPS URL to the DLL to download and map |
| Admin rights     | OS           | Must run elevated                        |
| Execution Policy | PowerShell   | `Bypass -Scope Process`                  |
| Architecture     | DLL + PS     | x86/x64 must match                       |

---

## 🚀 Usage / Sample Run Walkthrough

### Example Session

```powershell
PS C:\regix-dll-maper> Copy-Item config.ps1.example config.ps1
PS C:\regix-dll-maper> notepad config.ps1        # edit $dllUrl
PS C:\regix-dll-maper> Set-ExecutionPolicy Bypass -Scope Process -Force

---

## 🔒 Safety & Ethical Guardrails

> [!DANGER]
> **Maximum guardrails apply.** This tool disables defensive logging and wipes audit trails. Misuse can cause legal liability, termination, or criminal charges.

### Authorization Requirements
- [ ] Written permission from the system owner / organization
- [ ] Defined scope (which systems, which time window)
- [ ] Signed Rules of Engagement (RoE) — see [`docs/DISCLAIMER.md`](./docs/DISCLAIMER.md)
- [ ] Pre-run confirmation checklist completed

### Dual-Use Framing
This tool can be used by **red teams** (authorized adversarial simulation) and **blue teams** (defensive research, detection engineering). The same techniques that bypass logging can be studied to build better detection.

### Defensive Detection
Defenders can detect this activity via:
- Service stop events (Event ID 7036, 7040)
- Registry changes to PowerShell logging policies
- `.evtx` file deletion/truncation
- Hidden console window creation (`ShowWindow SW_HIDE`)
- Outbound HTTPS downloads of DLLs to `%TEMP%`

See [`docs/DETECTION.md`](./docs/DETECTION.md) for Sigma rules, EventLog queries, YARA rules, and a blue-team incident-response playbook.

---

## 🛡️ Detection & Defensive Notes

Short summary — full details in [`docs/DETECTION.md`](./docs/DETECTION.md):

| Indicator | Detection Method |
|-----------|-----------------|
| Service stops | Windows Event Log (System, Security) |
| Logging policy changes | Registry monitoring (HKLM\...\PowerShell\ModuleLogging) |
| Event log deletion | File system monitoring on `C:\Windows\System32\winevt\Logs\` |
| Hidden console | Process monitoring for `ShowWindow` / `SW_HIDE` |
| DLL download | Network monitoring for GitHub raw / HTTPS DLL retrieval |
| PE mapping in RWX | Memory scanning for RWX allocations with PE headers |

---

## ⚖️ Legal Disclaimer

Short summary — full details in [`docs/DISCLAIMER.md`](./docs/DISCLAIMER.md):

- **Authorized use only.** Written permission required.
- **No warranty.** Provided "AS IS" under the MIT License.
- **No liability.** The author is not responsible for misuse or damages.
- **Jurisdiction.** Laws vary by country; consult legal counsel before testing.
- **Responsible disclosure.** Report findings through proper channels.

---

## 🤝 Contributing

1. **Docs only:** `main.ps1` must not be modified. Propose changes to documentation, configuration templates, and detection/disclaimer files.
2. **Preserve warnings:** All `AUTHORIZED USE ONLY`, `[!DANGER]`, and educational disclaimers must remain intact.
3. **Educational framing:** New content must reinforce the educational / authorized-testing purpose.
4. **Submit via Pull Request** with a clear description of the change.

---

## 📄 License

MIT License. See [`LICENSE`](./LICENSE) for details.

> This project is released under the MIT License for **educational and authorized security-testing purposes only**. The license does not authorize illegal or unethical use.

---

**Built for authorized red-team research & education.**
Use responsibly. Obtain written permission. Stay within scope.
PS C:\regix-dll-maper> . ./config.ps1
PS C:\regix-dll-maper> .\main.ps1
```

### Expected Behavior

1. Console window disappears (`SW_HIDE`).
2. Services `WSearch`, `cbdhsvc*`, `VSS*`, `fhsvc*` are stopped.
3. PowerShell ModuleLogging and ScriptBlockLogging are disabled via registry.
4. Recent `%TEMP%` artifacts are removed.
5. DLL is downloaded from `$dllUrl`.
6. PE headers validated; DLL mapped into RWX memory.
7. Imports resolved, relocations applied, `DllMain` called.
8. Event logs (System, PowerShell, Security) wiped.
9. Final temp cleanup, PS history cleared, variables nulled, GC forced.
10. Script exits silently.

> [!TIP]
> For safe testing, use a **known benign DLL** (e.g., a simple MessageBox DLL you compiled yourself) and verify behavior in an isolated VM.

> [!NOTE]
> OS / PowerShell compatibility depends on the DLL you map. The runner requirements above are the baseline; your payload may have additional dependencies.

---

## 📥 Installation

1. **Clone the repository**

   ```powershell
   git clone https://github.com/{username}/regix-dll-maper.git
   cd regix-dll-maper
   ```

2. **Unblock the script** (required if downloaded from the internet)

   ```powershell
   Unblock-File -Path ./main.ps1
   ```

3. **Set execution policy for this process**

   ```powershell
   Set-ExecutionPolicy Bypass -Scope Process -Force
   ```

4. **Create the configuration file**

   ```powershell
   Copy-Item config.ps1.example config.ps1
   ```

5. **Edit `config.ps1`** and set `$dllUrl` to your remote DLL URL (see Configuring Guide below).
   │ logging + clean temp │
   └───────────┬─────────────┘
   ▼
   ┌─────────────────────────┐
   │ Hide console window │
   └───────────┬─────────────┘
   ▼
   ┌─────────────────────────┐
   │ config.ps1 → $dllUrl │
   └───────────┬─────────────┘
   ▼
   ┌─────────────────────────┐
   │ Download DLL from URL │
   └───────────┬─────────────┘
   ▼
   ┌─────────────────────────┐
   │ Validate MZ + PE header│
   └───────────┬─────────────┘
   │ Invalid ──► Exit
   │ Valid
   ▼
   ┌─────────────────────────┐
   │ VirtualAlloc RWX mem │
   └───────────┬─────────────┘
   ▼
   ┌─────────────────────────┐
   │ Map PE sections │
   └───────────┬─────────────┘
   ▼
   ┌─────────────────────────┐
   │ Relocate (delta) │
   └───────────┬─────────────┘
   ▼
   ┌─────────────────────────┐
   │ Resolve imports │
   └───────────┬─────────────┘
   ▼
   ┌─────────────────────────┐
   │ Protect + Flush cache │
   └───────────┬─────────────┘
   ▼
   ┌─────────────────────────┐
   │ Call DllMain / CRT_INIT│
   └───────────┬─────────────┘
   ▼
   ┌─────────────────────────┐
   │ Wipe Event Logs │
   └───────────┬─────────────┘
   ▼
   ┌─────────────────────────┐
   │ Final cleanup + GC │
   └───────────┬─────────────┘
   ▼
   ┌─────────────────┐
   │ End │
   └─────────────────┘

```

```
