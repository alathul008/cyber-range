# DET-002 Investigation

## 1. Objective

Validate that Wazuh can detect the parent-child process relationship:

`cmd.exe → powershell.exe`

using Windows Security Event ID 4688 telemetry.

## 2. Environment

- Endpoint: WIN-01
- Endpoint IP: 10.10.20.100
- Wazuh manager: WAZUH-01
- Wazuh custom rule: 100102
- Parent Wazuh rule: 67027
- Telemetry source: Windows Security Event Log
- Event ID: 4688

## 3. Controlled Test

The following benign command was executed on WIN-01:

```cmd
cmd.exe /c powershell.exe -NoProfile -Command "Write-Output DET002-Test"
```

The command was selected to create a clear and reproducible process lineage without performing malicious activity.

## 4. Raw Windows Telemetry

The captured Wazuh event for rule 67027 showed:

- Event ID: 4688
- Subject account: CORP\Administrator
- New Process ID: 0x12c8
- New Process Name: `C:\Windows\System32\WindowsPowerShell\v1.0\powershell.exe`
- Creator Process ID: 0x500
- Creator Process Name: `C:\Windows\System32\cmd.exe`
- Process Command Line: `powershell.exe -NoProfile -Command "Write-Output DET002-Test"`
- Computer: `DESKTOP-8PB40LJ.corp.home.arpa`

## 5. Detection Logic

The final deployed Wazuh rule is:

```xml
<rule id="100102" level="8">
  <if_sid>67027</if_sid>
  <field name="win.eventdata.newProcessName">powershell.exe</field>
  <field name="win.eventdata.parentProcessName">cmd.exe</field>
  <description>PowerShell spawned by Windows Command Shell on Windows endpoint.</description>
  <mitre>
    <id>T1059.001</id>
    <id>T1059.003</id>
  </mitre>
  <group>powershell,windows_command_shell,execution,</group>
</rule>
```

The rule passed `wazuh-analysisd -t`, the Wazuh manager was restarted successfully, and the service returned `active`.

## 6. Detection Result

The controlled retest generated Wazuh rule `100102` at level 8.

The Wazuh Dashboard showed:

- Rule ID: 100102
- Level: 8
- Description: PowerShell spawned by Windows Command Shell on Windows endpoint.
- MITRE ATT&CK: T1059.001 and T1059.003
- Agent: WIN-01

## 7. Investigation

The parent-child relationship was confirmed from the Windows 4688 telemetry:

```
cmd.exe
   └── powershell.exe
```

The command line was benign and intentionally generated for detection validation.

No malicious payload, exploitation, persistence, or external command-and-control activity was established by this exercise.

## 8. Detection Engineering Debugging

The first versions of the custom rule used:

```text
data.win.eventdata.newProcessName
data.win.eventdata.parentProcessName
```

Those fields are visible in the indexed Wazuh alert JSON, but the rule did not fire.

Inspection of the built-in Wazuh rule 67027 showed that the rule engine references Windows EventChannel fields using the `win.*` namespace. The final rule therefore uses:

```text
win.eventdata.newProcessName
win.eventdata.parentProcessName
```

After this change, the controlled retest successfully generated rule 100102.

## 9. Evidence Retention Limitation

At the time of documentation, the specific DET002-Test event was no longer present in the active:

```text
/var/ossec/logs/alerts/alerts.json
```

The expected archive path:

```text
/var/ossec/logs/archives/archives.json
```

was not configured/present.

The successful detection is therefore documented from the Dashboard evidence and raw 67027 event captured during the test, rather than claiming that the historical alert remains retrievable from the current alert file.

This is recorded as a telemetry-retention/evidence gap.

## 10. IOC / IOA / TTP Assessment

### IOC

No malicious IOC was established. The executable observed was the legitimate Windows PowerShell binary.

### IOA

The relevant behavioral indicator was:

```text
cmd.exe → powershell.exe
```

with a PowerShell command line.

### TTP

- T1059.001 — PowerShell
- T1059.003 — Windows Command Shell
- Tactic: Execution

## 11. Detection Gap

The detection identifies the process relationship but does not by itself establish malicious intent.

Legitimate administrative activity can also produce:

```text
cmd.exe → powershell.exe
```

Future detection improvements should add contextual signals such as command-line characteristics, user context, parent lineage, execution location, network behavior, and other telemetry before treating the activity as higher-confidence malicious behavior.

## 12. Conclusion

DET-002 successfully demonstrates a validated Wazuh parent-child process detection using Windows Event ID 4688 telemetry.

The exercise also identified and resolved an important Wazuh rule-engine field namespace issue and documented an evidence-retention limitation for historical alerts.
