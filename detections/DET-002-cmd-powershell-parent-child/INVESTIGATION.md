# DET-002 Investigation

## 1. Objective

Validate that Wazuh can detect the parent-child process relationship:

`cmd.exe → powershell.exe`

using Windows Security Event ID 4688 telemetry, then improve the detection by adding command-line context.

## 2. Environment

- Endpoint: WIN-01
- Endpoint IP: 10.10.20.100
- Wazuh manager: WAZUH-01
- Wazuh baseline rule: 100102
- Wazuh improved rule: 100103
- Existing encoded PowerShell rule: 100101
- Parent Wazuh rule: 67027
- Telemetry source: Windows Security Event Log
- Event ID: 4688

## 3. Baseline Controlled Test

The following benign command was executed on WIN-01:

```cmd
cmd.exe /c powershell.exe -NoProfile -Command "Write-Output DET002-Baseline"
```

The command was selected to create a clear and reproducible process lineage without performing malicious activity.

## 4. Baseline Telemetry

The captured Wazuh event contained:

- Event ID: 4688
- Agent: WIN-01
- New Process Name: `C:\Windows\System32\WindowsPowerShell\v1.0\powershell.exe`
- Parent Process Name: `C:\Windows\System32\cmd.exe`
- Process Command Line: `powershell.exe -NoProfile -Command "Write-Output DET002-Baseline"`

Wazuh generated custom rule 100102 at level 8.

## 5. Baseline Detection Logic

The deployed Wazuh rule is:

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

## 6. Detection Improvement

The baseline rule identifies the process relationship but does not distinguish normal CMD-launched PowerShell from a narrower behavior such as encoded PowerShell execution.

A second rule was therefore added without changing 100102:

```xml
<rule id="100103" level="14">
  <if_sid>67027</if_sid>
  <field name="win.eventdata.newProcessName">powershell.exe</field>
  <field name="win.eventdata.parentProcessName">cmd.exe</field>
  <field name="win.eventdata.commandLine" type="pcre2">(?i)(-encodedcommand|-enc)(\s|$)</field>
  <description>PowerShell spawned by CMD with encoded command execution.</description>
  <mitre>
    <id>T1059.001</id>
    <id>T1059.003</id>
  </mitre>
  <group>powershell,windows_command_shell,encoded_command,execution,mitre_t1059.001,mitre_t1059.003,</group>
</rule>
```

All three field conditions must match for 100103 to fire.

## 7. Improved Controlled Test

The following benign command was executed on WIN-01:

```cmd
cmd.exe /c powershell.exe -NoProfile -EncodedCommand VwByAGkAdABlAC0ATwB1AHQAcAB1AHQAIABEAFAAVAAwADAAMgAtAEkAbQBwAHIAbwB2AGUAbQBlAG4AdA==
```

The encoded payload performs a simple test output and was used only to validate detection logic.

## 8. Improved Detection Result

The Wazuh Dashboard confirmed:

- Rule 100101 — PowerShell encoded command detected, level 12
- Rule 100102 — PowerShell spawned by Windows Command Shell, level 8
- Rule 100103 — PowerShell spawned by CMD with encoded command execution, level 14

The 100103 alert contained:

- New Process Name: `C:\Windows\System32\WindowsPowerShell\v1.0\powershell.exe`
- Parent Process Name: `C:\Windows\System32\cmd.exe`
- Command Line containing `-EncodedCommand`
- Event ID: 4688
- Agent: WIN-01

## 9. Validation Matrix

| Controlled behavior | 100101 | 100102 | 100103 |
|---|---:|---:|---:|
| Normal CMD → PowerShell | No | Yes | No |
| CMD → PowerShell + encoded command | Yes | Yes | Yes |

The results demonstrate that 100102 remains the broad parent-child detection while 100103 adds contextual specificity.

## 10. Detection Engineering Debugging

The first versions of the custom rule used:

```text
data.win.eventdata.newProcessName
data.win.eventdata.parentProcessName
```

Those fields are visible in the indexed Wazuh alert JSON, but the rule did not fire.

Inspection of built-in Wazuh rule 67027 showed that the rule engine references Windows EventChannel fields using the `win.*` namespace. The final rules therefore use:

```text
win.eventdata.newProcessName
win.eventdata.parentProcessName
win.eventdata.commandLine
```

After this change, the baseline and improved detections fired as expected.

## 11. Relationship to DET-001

Existing DET-001 rule 100101 independently detects encoded PowerShell.

DET-002 therefore does not replace DET-001. Instead:

- DET-001 identifies encoded PowerShell execution.
- DET-002/100102 identifies the CMD → PowerShell process relationship.
- DET-002/100103 combines the parent-child relationship with encoded-command context.

This provides layered telemetry and demonstrates contextual detection engineering.

## 12. IOC / IOA / TTP Assessment

### IOC

No malicious IOC was established.

The executable observed was the legitimate Windows PowerShell binary, and the tests were intentionally benign.

### IOA

The relevant behavioral indicators were:

```text
cmd.exe → powershell.exe
cmd.exe → powershell.exe + -EncodedCommand
```

### TTP

- T1059.001 — PowerShell
- T1059.003 — Windows Command Shell
- Tactic: Execution

## 13. Detection Gap

The detections identify behaviors but do not establish malicious intent.

Legitimate administrative activity can produce:

```text
cmd.exe → powershell.exe
```

Encoded PowerShell can also have legitimate administrative uses.

Future improvements can add contextual signals such as:

- User context
- Parent lineage
- Execution location
- Command-line characteristics
- Network behavior
- File reputation/hash context
- Additional endpoint telemetry

## 14. Evidence Retention Limitation

The previous DET-002-Test event was no longer present in the active:

```text
/var/ossec/logs/alerts/alerts.json
```

The expected archive path:

```text
/var/ossec/logs/archives/archives.json
```

was not configured/present.

The improvement validation is therefore documented from the Wazuh Dashboard evidence captured during the controlled tests rather than claiming persistent retrieval from the current alert file.

This remains a telemetry-retention/evidence gap.

## 15. Conclusion

DET-002 successfully demonstrates a validated Wazuh parent-child process detection using Windows Event ID 4688 telemetry.

The detection was improved from a broad process relationship (100102) to a contextual higher-confidence signal (100103) by requiring encoded PowerShell command-line characteristics.

The exercise also identified and resolved a Wazuh rule-engine field namespace issue and documented an evidence-retention limitation.

All observed activity in this exercise was controlled and benign; no malicious IOC was established.
