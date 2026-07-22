# 03 — Windows Agent Enrollment

Enrolled a Windows endpoint as a monitored agent against the Wazuh Manager, using the Dashboard's "Deploy new agent" wizard.

## Steps

From the Dashboard: **Server Management → Endpoints Summary → Deploy new agent**, selecting Windows / MSI, entering the Manager's IP, and an agent name (`windows-target`). The wizard generates a PowerShell one-liner:

```powershell
Invoke-WebRequest -Uri https://packages.wazuh.com/4.x/windows/wazuh-agent-4.9.2-1.msi `
  -OutFile $env:tmp\wazuh-agent; `
  msiexec.exe /i $env:tmp\wazuh-agent /q WAZUH_MANAGER='<manager-ip>' WAZUH_AGENT_NAME='windows-target'
```

Run as **Administrator** on the Windows endpoint, then start the service:
```powershell
net start WazuhSvc
Get-Service WazuhSvc   # Status: Running
```

## Verification

**Dashboard → Endpoints Summary** shows `windows-target` as **Active**, correctly identifying the OS (Windows 10) and matching agent/manager versions (both 4.9.2).

**Real telemetry confirmed** — the agent immediately began running Security Configuration Assessment (SCA) scans against the CIS Microsoft Windows 10 Enterprise Benchmark, generating hundreds of real events within the first 24 hours (not just a heartbeat).

## Scope note

This agent was enrolled on a **real Windows host**, not a disposable VM. See the [module README](../README.md) for the reasoning and how later sections handle this.
