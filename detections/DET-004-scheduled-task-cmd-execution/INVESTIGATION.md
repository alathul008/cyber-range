# DET-004 Investigation

## 1. Objective

Validate detection of a controlled Windows scheduled task creation where the task action executes `cmd.exe` on WIN-01.

## 2. Test Action

A temporary scheduled task named `DET004-TestTask` was created on WIN-01 using:

```cmd
schtasks /create /tn "DET004-TestTask" /tr "cmd.exe /c echo DET004-Test" /sc once /st 23:59 /f
```

This was an authorized lab activity.

No credentials or secrets are recorded in this report.

## 3. Expected Telemetry

Expected Windows Security telemetry:

- Event ID 4698 — A scheduled task was created.
- TaskName identifying the created task.
- SubjectUserName identifying the creator.
- TaskContent containing the scheduled task XML.

## 4. Observed Telemetry

Wazuh received Event ID 4698 from WIN-01.

Observed fields included:

| Field | Observed value |
|---|---|
| Agent | WIN-01 |
| Agent IP | 10.10.20.100 |
| Event ID | 4698 |
| TaskName | \\DET004-TestTask |
| SubjectUserName | Administrator |
| SubjectDomainName | CORP |
| Computer | DESKTOP-8PB40LJ.corp.home.arpa |
| Wazuh native rule | 60228 |
| Native description | A scheduled task was created |

The task XML included:

```xml
<Command>cmd.exe</Command>
<Arguments>/c echo DET004-Test</Arguments>
```

## 5. Detection

Custom Wazuh rule:

- Rule ID: 100105
- Level: 12
- Parent rule: 60228
- Description: Scheduled task created with cmd.exe execution.
- MITRE: T1053.005 and T1059.003

The rule was syntax-validated with `wazuh-analysisd -t` and the manager restarted successfully.

## 6. Positive Validation

The controlled CMD-based scheduled task generated Event 4698.

Wazuh Threat Hunting showed rule 100105 firing with:

- Rule ID: 100105
- Level: 12
- Agent: WIN-01
- Description: Scheduled task created with cmd.exe execution.
- MITRE: T1053.005
- MITRE: T1059.003

Result: **PASS**.

## 7. Negative Validation

A second scheduled task was created with:

```cmd
powershell.exe -NoProfile -Command echo DET004-FP
```

The search:

`agent.name:WIN-01 AND rule.id:100105`

showed only the earlier positive detection.

No new 100105 alert was observed for the PowerShell task.

Result: **PASS**.

## 8. IOC / IOA / TTP

### IOC

No malicious IOC was identified.

### IOA

Scheduled task creation with a CMD execution action.

### TTP

- T1053.005 — Scheduled Task/Job: Scheduled Task
- T1059.003 — Command and Scripting Interpreter: Windows Command Shell

## 9. Investigation Assessment

The activity was expected because it was intentionally generated as a controlled detection test.

The detection identifies a behavior that can be used legitimately or for persistence/execution. It should therefore be investigated in context rather than treated as proof of compromise.

Relevant investigation questions:

- Who created the task?
- Is the task authorized?
- What command and arguments does it execute?
- What identity does it run as?
- Is the task hidden?
- Does the task execute after creation?
- What process tree follows execution?
- Does execution generate network activity?
- Does the task persist across reboot or logon?

## 10. Detection Gap

The current rule identifies `cmd.exe` inside the scheduled-task XML.

It does not distinguish legitimate administrative tasks from malicious scheduled-task persistence.

It also does not detect scheduled tasks whose action uses other interpreters or binaries.

## 11. Future Improvement

Potential next-stage enrichment:

1. Correlate Event 4698 with subsequent task execution.
2. Correlate task creation with process creation telemetry.
3. Add creator and privilege context.
4. Detect suspicious command-line patterns.
5. Detect execution from unusual paths.
6. Correlate with network activity.
7. Measure false positives from legitimate enterprise automation.

## 12. Cleanup

Both temporary tasks were deleted after testing:

```cmd
schtasks /delete /tn "DET004-TestTask" /f
schtasks /delete /tn "DET004-FP-Test" /f
```

Verification confirmed that neither task remained on WIN-01.

## 13. Validation Result

**PASS — controlled Event ID 4698 generated, Wazuh ingested it, native rule 60228 detected it, custom rule 100105 fired for CMD execution, the PowerShell negative test did not trigger 100105, and the test tasks were removed.**

Validation date: 2026-09-21.
