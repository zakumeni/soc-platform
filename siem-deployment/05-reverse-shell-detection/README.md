# 05 — Reverse Shell Detection Infrastructure

> **Scope note:** this section builds and verifies the detection *infrastructure* for reverse shell / suspicious process detection using real, safe commands. Actual offensive payloads (msfvenom-generated shells, Netcat listeners) were **not executed** against this real host — see [the module README](../README.md) for why. What each detection would look like against a real payload is described conceptually below.

## Detection theory

Msfvenom payloads and Netcat reverse shells reliably produce two correlated events within milliseconds of each other:
1. A **process creation** event (the shell/payload launching)
2. A **network connection** event (that same process opening an outbound connection to the attacker's listener)

That pairing — not either signal alone — is the reliable detection signature this section builds toward.

## Infrastructure built and verified

**1. Process creation auditing (Windows native)**
```powershell
auditpol /set /subcategory:"Process Creation" /success:enable /failure:enable
```
Confirmed: `Process Creation — Success and Failure`.

**2. Full command-line capture** (off by default — Windows only logs the process name otherwise)
```powershell
reg add "HKLM\SOFTWARE\Microsoft\Windows\CurrentVersion\Policies\System\Audit" /v ProcessCreationIncludeCmdLine_Enabled /t REG_DWORD /d 1 /f
```
Confirmed populated: alerts show full `commandLine` strings (e.g. `backgroundTaskHost.exe -ServerName:App.AppXmtcanOh2tfbfy7k9kn8hbxb6dmzz1zh0.mca`), not just bare process names.

**3. Sysmon — process creation (Event ID 1) and network connections (Event ID 3)**
```powershell
Invoke-WebRequest -Uri https://download.sysinternals.com/files/Sysmon.zip -OutFile Sysmon.zip
Expand-Archive Sysmon.zip -DestinationPath C:\Sysmon\
C:\Sysmon\sysmon64.exe -accepteula -i -n   # -n enables network connection logging
```

Confirmed both event types logging correctly:
```powershell
Get-WinEvent -LogName "Microsoft-Windows-Sysmon/Operational" -MaxEvents 20 | Select-Object TimeCreated, Id
```

## Gotcha: network monitoring isn't on by default

The standard Sysmon install (`-accepteula -i`) does **not** enable Event ID 3 — that requires the `-n` flag at install time, or `-c -n` to reconfigure afterward. A reconfigure-in-place attempt on the running service didn't take effect reliably (kernel driver network filters can require a full service reinstall to apply cleanly); a clean uninstall (`-u`) + reinstall with `-n` baked in resolved it immediately. Confirmed via a real, safe outbound connection:
```powershell
Test-NetConnection -ComputerName 208.67.222.222 -Port 443
```
producing genuine Event ID 3 entries with full `Image`, `User`, `SourceIp/Port`, `DestinationIp/Port`, and `Initiated` fields.

## What a real detection would show (not executed)

| Signal | Conceptual expected value |
|---|---|
| Wazuh Rule 92000-92999 | Fires when Sysmon Event 1 matches a suspicious-process decoder pattern (encoded PowerShell, unexpected parent process) |
| `rule.level` | 12 (critical) |
| `data.win.eventdata.commandLine` | Full payload command line, e.g. base64-encoded PowerShell |
| `data.win.eventdata.parentProcessName` | Unexpected parent (e.g. `winword.exe` spawning `cmd.exe`) |
| Sysmon Event 3 (same `ProcessGuid`) | Outbound connection to attacker IP, non-standard port (4444, 8080, etc.) |
| tshark on the Blue Team VM | Lone `SYN` packet (no `ACK`) from the Windows endpoint to the C2 IP, visible before any shell traffic |
| `rule.mitre.technique` | T1059 (Command and Scripting Interpreter) |

## jq query pattern (documented, not run against real alert data)

```bash
docker exec single-node-wazuh.manager-1 cat /var/ossec/logs/alerts/alerts.json | \
jq -r 'select((.rule.id|tonumber) >= 92000 and (.rule.id|tonumber) <= 92999) |
{rule: .rule.description, cmdline: .data.win.eventdata.commandLine,
 user: .data.win.eventdata.subjectUserName, time: .timestamp}'
```
