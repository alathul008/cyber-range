# DET-013 — Scheduled Task Detection

## Objective

Validate the existing scheduled-task detection rule against a controlled benign Windows scheduled-task creation on DC-01.

Validation path:

`Scheduled Task creation → Windows Security Event 4698 → Wazuh ingestion → Rule 100105 → MITRE ATT&CK mapping`

The exercise also validates the Windows auditing prerequisite required to generate Event ID 4698.

## Scope

- Host: `DC-01`
- Domain: `corp.home.arpa`
- User: `CORP\\admin`
- Telemetry: Windows Security Event Log + Wazuh
- Detection: Wazuh custom rule **100105**
- Test task: `DET013-TestTask`
- Validation date: **2026-09-27**
- Test type: controlled benign scheduled-task creation

## Existing Detection

Rule **100105** is already present in Wazuh local rules:

- Level: **12**
- Detection: scheduled task created with `cmd.exe` execution
- MITRE ATT&CK:
  - **T1053.005 — Scheduled Task/Job: Scheduled Task**
  - **T1059.003 — Windows Command Shell**

The exercise was designed to validate this existing rule rather than create a duplicate.

## Initial Telemetry Check

The first controlled task creation succeeded, but no matching Wazuh alert was observed.

Windows Security Event 4698 was also initially absent.

The Windows audit policy was checked:

```powershell
auditpol /get /subcategory:"Other Object Access Events"
```

Observed state:

```
Other Object Access Events                  No Auditing
```

The available Object Access audit policy showed the relevant subcategory disabled.

## Audit Policy Prerequisite

To enable the required scheduled-task creation telemetry, the following targeted audit setting was enabled on DC-01:

```powershell
auditpol /set /subcategory:"Other Object Access Events" /success:enable
```

Verification:

```
auditpol /get /subcategory:"Other Object Access Events"
```

Observed:

```
Other Object Access Events                  Success
```

A VMware snapshot had already been taken before the DET-013 test:

`PHASE2-DET013-PREATTACK-DC01`

This provides a rollback point for the test-system policy change.

## Controlled Test

The task was deleted and recreated after enabling the audit policy:

```powershell
schtasks /Delete /TN "DET013-TestTask" /F

schtasks /Create /TN "DET013-TestTask" /TR "cmd.exe /c echo DET013-ScheduledTask-Test" /SC ONCE /ST 23:59 /F
```

The task was created successfully.

The task was configured with:

- Task name: `DET013-TestTask`
- Action: `cmd.exe /c echo DET013-ScheduledTask-Test`
- Trigger: one-time scheduled execution at 23:59
- Run level: least privilege
- User: `CORP\\admin`

The test validated task **creation telemetry**. The scheduled action itself was not required to execute for this detection test.

## Windows Event 4698

After the audit-policy change and task recreation, Windows Security Event **4698 — A scheduled task was created** was observed.

Observed event time:

`2026-09-27 09:40:18` local DC-01 time

The event contained:

- Account: `CORP\\admin`
- Task Name: `\\DET013-TestTask`
- Task Content containing:
  - `<Command>cmd.exe</Command>`
  - `<Arguments>/c echo DET013-ScheduledTask-Test</Arguments>`

The event was successfully ingested by Wazuh.

## Wazuh Detection

Wazuh generated the existing custom rule:

- Rule: **100105**
- Level: **12**
- Description: **Scheduled task created with cmd.exe execution**
- Agent: **DC-01**
- Agent IP: `10.10.20.10`
- Event ID: **4698**
- ATT&CK: **T1053.005**, **T1059.003**

Observed Wazuh alert timestamp:

`2026-09-27T04:11:04.966+0000`

The alert contained the full 4698 event and the scheduled-task XML, including the `cmd.exe` action.

## Supporting Process Telemetry

The scheduled-task creation also generated Windows Security Event **4688** for:

```
C:\Windows\System32\schtasks.exe
```

The captured command line included:

```
"C:\WINDOWS\system32\schtasks.exe" /Create /TN DET013-TestTask /TR "cmd.exe /c echo DET013-ScheduledTask-Test" /SC ONCE /ST 23:59 /F
```

Wazuh ingested this as native rule **67027 — A process was created**.

This provides supporting process telemetry immediately preceding the 4698 detection.

## Detection Validation

| Validation | Result |
|---|---|
| Scheduled task created | PASS |
| Windows Event 4698 generated | PASS |
| 4698 ingested by Wazuh | PASS |
| Task content exposed `cmd.exe` | PASS |
| Existing rule 100105 fired | PASS |
| Rule level 12 | PASS |
| T1053.005 mapping | PASS |
| T1059.003 mapping | PASS |
| Supporting 4688 telemetry | PASS |

## Detection Engineering Finding

The initial failure to produce Event 4698 was caused by the relevant Windows audit setting being disabled.

After enabling:

`Other Object Access Events — Success`

and recreating the task, the expected 4698 telemetry appeared and Rule 100105 fired.

This is an important detection-engineering dependency:

**A correct detection rule is ineffective when its required source telemetry is not generated or collected.**

No change was made to Rule 100105 because the rule successfully detected the intended behavior once telemetry was available.

## Current Status

**PASS — scheduled-task detection validated.**

Validated:

- Windows scheduled-task creation.
- Required Windows audit-policy prerequisite.
- Security Event 4698.
- Wazuh ingestion.
- Existing custom Rule 100105.
- MITRE ATT&CK mapping.
- Supporting Event 4688 process telemetry.

## Limitations

- This was a controlled benign scheduled-task creation.
- Only the `cmd.exe` scheduled-task behavior covered by Rule 100105 was tested.
- No claim is made that every scheduled-task variant is detected.
- The audit-policy change was made specifically to establish the required telemetry path.
- No modification was made to the existing detection rule.
