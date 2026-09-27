# DET-013 — Investigation Evidence

## Investigation Objective

Determine whether the controlled scheduled-task creation on DC-01 generated the expected Windows telemetry and whether the existing Wazuh Rule 100105 detected it.

## Test Preparation

VMware snapshot:

`PHASE2-DET013-PREATTACK-DC01`

Initial audit check:

```powershell
auditpol /get /subcategory:"Other Object Access Events"
```

Initial result:

```
Other Object Access Events                  No Auditing
```

Because Event 4698 was not being generated, the targeted audit setting was enabled:

```powershell
auditpol /set /subcategory:"Other Object Access Events" /success:enable
```

Verification:

```
Other Object Access Events                  Success
```

## Controlled Activity

The test task was recreated:

```powershell
schtasks /Delete /TN "DET013-TestTask" /F

schtasks /Create /TN "DET013-TestTask" /TR "cmd.exe /c echo DET013-ScheduledTask-Test" /SC ONCE /ST 23:59 /F
```

The creation command generated a supporting Security Event 4688 for `schtasks.exe`.

Observed command line:

```
"C:\WINDOWS\system32\schtasks.exe" /Create /TN DET013-TestTask /TR "cmd.exe /c echo DET013-ScheduledTask-Test" /SC ONCE /ST 23:59 /F
```

## Event 4698

The subsequent Security Event 4698 contained:

- Event ID: **4698**
- Task name: `\\DET013-TestTask`
- Subject account: `CORP\\admin`
- Task content with `cmd.exe`
- Arguments: `/c echo DET013-ScheduledTask-Test`

Observed Windows event time:

`2026-09-27 09:40:18`

The event was successfully received by Wazuh agent **DC-01**.

## Wazuh Rule 100105

Wazuh generated:

```
Rule: 100105
Level: 12
Description: Scheduled task created with cmd.exe execution.
```

Observed alert timestamp:

`2026-09-27T04:11:04.966+0000`

ATT&CK mappings:

- **T1053.005 — Scheduled Task/Job: Scheduled Task**
- **T1059.003 — Windows Command Shell**

The alert retained the scheduled-task XML, including the command:

```text
cmd.exe
```

and arguments:

```text
/c echo DET013-ScheduledTask-Test
```

## Supporting Process Event

Immediately before the 4698 alert, Wazuh recorded Security Event 4688 for `schtasks.exe`.

The event showed:

- User: `CORP\\admin`
- Image: `C:\Windows\System32\schtasks.exe`
- Parent: `C:\Windows\System32\WindowsPowerShell\v1.0\powershell.exe`
- Command line containing the `/Create` operation and `DET013-TestTask`

This provides a useful investigation chain:

```
PowerShell
   ↓
schtasks.exe /Create
   ↓
Security Event 4698
   ↓
Wazuh Rule 100105
```

## Initial Negative Result

Before enabling the audit prerequisite, the Wazuh Indexer search for `DET013-TestTask` returned zero results.

The Windows Security log also showed no 4698 event at that stage.

This was not treated as a rule failure because the source telemetry had not yet been established.

After enabling the targeted audit policy and recreating the task, Event 4698 appeared and Rule 100105 fired.

## Detection Engineering Conclusion

The detection itself was valid.

The initial gap was telemetry generation rather than rule logic.

This establishes the following dependency:

```
Source audit policy
      ↓
Windows Event 4698
      ↓
Wazuh EventChannel ingestion
      ↓
Rule 100105
      ↓
SOC alert
```

The existing Rule 100105 was therefore retained unchanged.

## Investigation Result

**PASS**

The controlled scheduled-task creation was reconstructed from Windows and Wazuh telemetry, and the existing detection successfully identified the intended `cmd.exe` scheduled-task behavior.

## Limitations

- Controlled benign activity only.
- No malicious payload was used.
- Validation covers the specific `cmd.exe` scheduled-task pattern represented by Rule 100105.
- The exercise does not establish coverage for other scheduled-task actions or trigger types.
