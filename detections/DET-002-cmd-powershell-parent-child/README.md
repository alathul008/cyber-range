# DET-002 — CMD Spawning PowerShell

## Objective

Detect a Windows process lineage in which `cmd.exe` spawns `powershell.exe`, then improve the detection with command-line context.

## Detection Logic

### Baseline — Wazuh rule 100102

The custom Wazuh rule:

- Uses parent rule `67027` (Windows process creation / Event ID 4688).
- Matches `win.eventdata.newProcessName` containing `powershell.exe`.
- Matches `win.eventdata.parentProcessName` containing `cmd.exe`.
- Generates a level 8 alert.
- Maps the behavior to MITRE ATT&CK T1059.001 (PowerShell) and T1059.003 (Windows Command Shell).

### Improved detection — Wazuh rule 100103

Rule 100103 preserves the baseline relationship and adds command-line context:

- Child process: `powershell.exe`
- Parent process: `cmd.exe`
- Command line contains `-EncodedCommand` or `-enc`
- Generates a level 14 alert.
- Maps the behavior to MITRE ATT&CK T1059.001 and T1059.003.

This creates a higher-confidence behavioral signal without replacing the broader baseline rule.

## Validation

Controlled benign tests were executed on WIN-01.

### Baseline test

```cmd
cmd.exe /c powershell.exe -NoProfile -Command "Write-Output DET002-Baseline"
```

Expected and observed:

- Wazuh rule `100102`
- Level 8
- Parent: `cmd.exe`
- Child: `powershell.exe`
- No `100103` match

### Encoded test

```cmd
cmd.exe /c powershell.exe -NoProfile -EncodedCommand VwByAGkAdABlAC0ATwB1AHQAcAB1AHQAIABEAFAAVAAwADAAMgAtAEkAbQBwAHIAbwB2AGUAbQBlAG4AdA==
```

Expected and observed:

- Wazuh rule `100102`
- Wazuh rule `100103`
- Wazuh rule `100103` at level 14
- Command line contained `-EncodedCommand`

The same encoded execution also matched existing rule `100101`, which independently detects encoded PowerShell.

## Detection Comparison

| Behavior | 100101 | 100102 | 100103 |
|---|---:|---:|---:|
| Normal CMD → PowerShell | — | Yes | No |
| CMD → PowerShell + encoded command | Yes | Yes | Yes |

This demonstrates layered detection:

```
100101 — encoded PowerShell
100102 — CMD → PowerShell baseline
100103 — CMD → encoded PowerShell contextual correlation
```

## Detection Engineering Lesson

The Wazuh rule engine uses `win.eventdata.*` field names for rule matching, while the resulting alert JSON is represented under `data.win.eventdata.*`. Using the indexed alert namespace in the custom rule did not produce a match; switching to the decoder/rule-engine namespace resolved the detection.

The improvement also demonstrates why a baseline behavioral rule should not simply be replaced by a more specific rule. The baseline captures the broader process relationship, while the contextual rule provides additional signal for a narrower behavior.

## Scope

This is a controlled, benign detection exercise. The test commands only write test strings and do not establish malicious activity.

No malicious IOC was established.

## Evidence

See [INVESTIGATION.md](./INVESTIGATION.md).

The Sigma artifacts in this directory represent both the baseline and improved detection logic.
