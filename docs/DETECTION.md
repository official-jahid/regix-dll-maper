# docs/DETECTION.md — Defensive Detection & Blue-Team Guide

> 🛡️ This document is for **defensive security researchers, blue teams, and detection engineers**.
> It describes how to detect the techniques used by `regix-dll-maper` and how to respond to incidents.

---

## 🎯 Overview

`regix-dll-maper` performs several actions that generate detectable telemetry:

1. Service stops (`WSearch`, `cbdhsvc`, `VSS`, `fhsvc`)
2. Registry changes to PowerShell logging policies
3. Temp folder cleanup (recently created files)
4. Hidden console window (`ShowWindow SW_HIDE`)
5. DLL download from remote URL
6. Manual DLL mapping (RWX memory, PE parsing, import resolution)

---

## 📡 Windows Event Log Indicators

### Service Stop Events

| Event ID | Source                  | Description                   |
| -------- | ----------------------- | ----------------------------- |
| 7036     | Service Control Manager | Service entered stopped state |
| 7040     | Service Control Manager | Service start type changed    |
| 7045     | Service Control Manager | Service installed             |

**Relevant services:** `WSearch`, `cbdhsvc`, `VSS`, `fhsvc`, `EventLog`

**PowerShell query:**

```powershell
Get-WinEvent -FilterHashtable @{
    LogName = 'System'
    Id = 7036, 7040, 7045
} | Where-Object { $_.Message -match 'WSearch|cbdhsvc|VSS|fhsvc|EventLog' }
```

### Registry Changes (PowerShell Logging)

| Path                                                                     | Value                      | Expected (secure) | Expected (tool) |
| ------------------------------------------------------------------------ | -------------------------- | ----------------- | --------------- |
| `HKLM\SOFTWARE\Policies\Microsoft\Windows\PowerShell\ModuleLogging`      | `EnableModuleLogging`      | `1`               | `0`             |
| `HKLM\SOFTWARE\Policies\Microsoft\Windows\PowerShell\ScriptBlockLogging` | `EnableScriptBlockLogging` | `1`               | `0`             |
| `HKLM\SOFTWARE\WOW6432Node\...\PowerShell\ModuleLogging`                 | `EnableModuleLogging`      | `1`               | `0`             |
| `HKLM\SOFTWARE\WOW6432Node\...\PowerShell\ScriptBlockLogging`            | `EnableScriptBlockLogging` | `1`               | `0`             |

**Detection:** Monitor for `RegSetValue` on these paths with value `0`.

### Event Log File Deletion

Files targeted (under `C:\Windows\System32\winevt\Logs\`):

- `System.evtx`
- `Windows PowerShell.evtx`
- `Microsoft-Windows-PowerShell%4Operational.evtx`
- `Security.evtx`

**Detection:** File system auditing for delete/rename/truncate on `.evtx` files. Event ID 4656/4658/4660 (object access) if auditing enabled.

### PowerShell History Clearing

File: `%APPDATA%\Microsoft\Windows\PowerShell\PSReadLine\ConsoleHost_history.txt`

**Detection:** File modification/deletion on the PSReadLine history file; `Clear-Content` invocation. 7. Event Log service stop + `.evtx` file deletion/truncation 8. PowerShell history clearing

---

## 🔍 Sigma Rules

> Sigma is an open standard for SIEM detection rules. Below are example rules.

### Rule 1: PowerShell Logging Disabled via Registry

```yaml
title: PowerShell Logging Disabled via Registry
id: regix-dll-maper-ps-logging-disabled
status: experimental
description: Detects disabling of PowerShell ModuleLogging or ScriptBlockLogging via registry
references:
  - https://github.com/{username}/regix-dll-maper
author: Blue Team
date: 2026-10-25
tags:
  - attack.defense_evasion
  - attack.t1089
logsource:
  category: registry_set
  product: windows
detection:
  selection:
    TargetObject|contains|all:
      - 'Microsoft\Windows\PowerShell'
      - 'ModuleLogging'
    TargetObject|contains|all:
      - 'Microsoft\Windows\PowerShell'
      - 'ScriptBlockLogging'
    Details|contains: 'EnableModuleLogging'
    Details|contains: 'EnableScriptBlockLogging'
    NewValue: 0
  condition: selection
level: high
```

### Rule 2: Event Log Service Stopped

```yaml
title: Event Log Service Stopped
id: regix-dll-maper-eventlog-stopped
status: experimental
description: Detects stopping of the Windows EventLog service
logsource:
  category: service_state
  product: windows
detection:
  selection:
    ServiceName: "EventLog"
    State: "stopped"
  condition: selection
level: high
```

### Rule 3: PowerShell Hidden Console Window

````yaml
title: PowerShell Hidden Console Window
id: regix-dll-maper-hidden-console
status: experimental
description: Detects PowerShell processes that hide their console window via ShowWindow SW_HIDE
logsource:
  category: process_creation

---

## 🧬 YARA Rules

### Rule: Detect regix-dll-maper Script Patterns

```yara
rule regix_dll_mapper_powershell
{
    meta:
        description = "Detects regix-dll-maper PowerShell script patterns"
        author = "Blue Team"
        date = "2026-10-25"
    strings:
        $s1 = "EnableModuleLogging" nocase
        $s2 = "EnableScriptBlockLogging" nocase
        $s3 = "WOW6432Node" nocase
        $s4 = "ShowWindow" nocase
        $s5 = "SW_HIDE" nocase
        $s6 = "NativeLoader" nocase
        $s7 = "LogWiper" nocase
        $s8 = "raw.githubusercontent.com" nocase
        $s9 = "MOVEFILE_DELAY_UNTIL_REBOOT" nocase
        $s10 = "ConsoleHost_history.txt" nocase
    condition:
        any of ($s*) and filesize < 500KB
}
````

### Rule: Detect Manual DLL Mapping Artifacts

```yara
rule manual_dll_mapping_rwx
{
    meta:
        description = "Detects RWX memory allocations with PE headers (manual mapping)"
    strings:
        $mz = { 4D 5A }
        $pe = { 50 45 00 00 }
    condition:
        $mz at 0 and $pe and (uint16(0) == 0x5A4D)
}
```

---

## 🔍 EDR / Hunting Queries

### PowerShell Script Block Logging (if re-enabled)

```powershell
# Search for regix-dll-maper indicators in ScriptBlock logs
Get-WinEvent -FilterHashtable @{
    LogName = 'Microsoft-Windows-PowerShell/Operational'
    Id = 4104
} | Where-Object { $_.Message -match 'NativeLoader|LogWiper|EnableModuleLogging|raw.githubusercontent' }
```

### Network Hunting

---

## 🚨 Blue-Team Incident Response Playbook

### Phase 1 — Detect

- Alert on PowerShell logging disabled (Sigma Rule 1)
- Alert on EventLog service stopped (Sigma Rule 2)
- Alert on `.evtx` file deletion/truncation
- Alert on DLL download from `raw.githubusercontent.com`

### Phase 2 — Contain

- Isolate the affected host from the network
- Capture memory dump (for RWX mapping analysis)
- Preserve remaining event logs (copy to centralized SIEM immediately)

### Phase 3 — Eradicate

- Remove the `main.ps1` script and any downloaded DLLs
- Re-enable PowerShell ModuleLogging and ScriptBlockLogging
- Restart the EventLog service
- Restore deleted `.evtx` files from backup/SIEM if possible

### Phase 4 — Recover

- Re-image the host if integrity is in doubt
- Re-enable `WSearch`, `VSS`, and other stopped services
- Review user accounts and credentials used on the host

### Phase 5 — Lessons Learned

- Deploy registry monitoring for PowerShell logging policies
- Enable file auditing on `%WINDIR%\System32\winevt\Logs\`
- Centralize event logs to a SIEM (so local deletion doesn't remove evidence)
- Enable PowerShell ScriptBlock Logging and ModuleLogging by GPO

---

## 📎 References

- MITRE ATT&CK: T1089 (Disabling Security Tools), T1562 (Impair Defenses), T1027 (Obfuscated Files), T1055 (Process Injection)
- Sigma: https://github.com/SigmaHQ/sigma
- YARA: https://github.com/VirusTotal/yara
- Windows Event Log channels: System, Security, Microsoft-Windows-PowerShell/Operational

```powershell
# Look for outbound HTTPS to raw.githubusercontent.com with .dll in URL
# Use your EDR network logs or proxy logs
```

### Process Hunting

| Indicator                         | Query                                                     |
| --------------------------------- | --------------------------------------------------------- |
| PowerShell spawning hidden window | Process start with `ShowWindow` / `SW_HIDE`               |
| PowerShell downloading DLL        | `WebClient.DownloadData` or `Invoke-WebRequest` to `.dll` |
| RWX allocation                    | Memory allocation with `PAGE_EXECUTE_READWRITE`           |
| EventLog service stop             | Service control: `EventLog` stop                          |

product: windows
detection:
selection:
Image|endswith: '\powershell.exe'
ParentImage|endswith: '\powershell.exe'
condition: selection
level: medium

````

### Rule 4: DLL Download from GitHub Raw

```yaml
title: DLL Download from GitHub Raw URL
id: regix-dll-maper-github-download
status: experimental
description: Detects download of .dll files from githubusercontent.com
logsource:
  category: proxy
  product: windows
detection:
  selection:
    URL|contains: 'raw.githubusercontent.com'
    URL|endswith: '.dll'
  condition: selection
level: medium
````
