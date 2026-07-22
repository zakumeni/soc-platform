# 04 — FIM: Windows Registry & File Monitoring

Configured Wazuh's File Integrity Monitoring (`<syscheck>`) to watch registry keys and directories relevant to common attack techniques: unquoted service paths, persistence via Run keys, and DLL hijacking/payload drops.

## What was already there

The default Wazuh 4.9.2 Windows agent config already monitors `HKLM\...\Services` and the `Run`/`RunOnce` autorun keys, plus dozens of restricted-pattern file watches (`cmd.exe`, `powershell.exe`, `reg.exe`, etc. by filename). Newer agent defaults are considerably richer than older Wazuh curriculum material assumes — worth checking the shipped config before assuming something needs adding.

## What was added

Full-directory, real-time integrity monitoring (not just named-file watching) for two attack-relevant paths, added to `ossec.conf`'s `<syscheck>` block:

```xml
<!-- DLL Hijacking + payload drops: realtime full-integrity monitoring -->
<directories check_all="yes" realtime="yes">C:\Program Files</directories>

<!-- DLL replacement attacks: System32 tampering -->
<directories check_all="yes" realtime="yes">C:\Windows\System32</directories>
```

Applied via:
```powershell
NET STOP WazuhSvc
NET START WazuhSvc
```

## Verification

Created a test file in a monitored directory:
```powershell
New-Item -Path "C:\Program Files\TestFile.exe" -ItemType File
```

Confirmed in the Dashboard's Events view — **5 alerts fired within seconds**, `rule.id: 550` ("Integrity checksum changed"), matching the timestamp of the test file creation.

## Known limitation (expected, not a bug)

The agent log shows repeated `ERROR (6716): Could not open handle for 'c:\windows\system32\config\sam'` (and similarly for `SECURITY`, `SYSTEM`, `DEFAULT`). These are live Windows registry hive files, permanently OS-locked while running — **no process, including Wazuh, can ever hash them on a live system.** This recurs on every scan and is a platform limitation, not a misconfiguration.

## A baseline-vs-realtime timing lesson

The first scan of a newly-added monitored directory establishes a silent baseline — it does **not** generate "file added" alerts, since there's nothing yet to compare against. Alerts only fire for changes *after* a baseline exists. This tripped up initial testing (first test file produced no alert) until a second test file, created after the baseline pass completed, triggered real detections.
