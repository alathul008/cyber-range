# DET-002 — CMD Spawning PowerShell

## Objective

Detect a Windows process lineage in which `cmd.exe` spawns `powershell.exe`.

## Detection Logic

The custom Wazuh rule:

- Uses parent rule `67027` (Windows process creation / Event ID 4688).
- Matches `win.eventdata.newProcessName` containing `powershell.exe`.
- Matches `win.eventdata.parentProcessName` containing `cmd.exe`.
- Generates a level 8 alert.
- Maps the behavior to MITRE ATT&CK T1059.001 (PowerShell) and T1059.003 (Windows Command Shell).

## Validation

Controlled benign test executed on WIN-01:

```cmd
cmd.exe /c powershell.exe -NoProfile -Command "Write-Output DET002-Test"
```

The resulting Windows Security Event ID 4688 contained:

- New Process Name: `C:\Windows\System32\WindowsPowerShell\v1.0\powershell.exe`
- Creator Process Name: `C:\Windows\System32\cmd.exe`
- Process Command Line: `powershell.exe -NoProfile -Command "Write-Output DET002-Test"`

Wazuh generated custom rule `100102` at level 8.

## Detection Engineering Lesson

The Wazuh rule engine uses `win.eventdata.*` field names for rule matching, while the resulting alert JSON is represented under `data.win.eventdata.*`. Using the indexed alert namespace in the custom rule did not produce a match; switching to the decoder/rule-engine namespace resolved the detection.

## Scope

This is a controlled, benign detection exercise. The test command only writes the string `DET002-Test`.

## Evidence

See [INVESTIGATION.md](./INVESTIGATION.md).
