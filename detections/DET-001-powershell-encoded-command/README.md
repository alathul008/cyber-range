# DET-001 — PowerShell Encoded Command Execution

## Objective

Detect PowerShell execution where the command line contains
`-EncodedCommand` or its abbreviated form `-enc`.

This analytic identifies PowerShell executions that warrant investigation.
The presence of an encoded command is not, by itself, proof of malicious activity.

## Data Source

- Host: WIN-01
- Operating System: Windows 10
- Telemetry: Sysmon
- Event ID: 1 — Process Create
- Field: Image
- Field: CommandLine

## Detection Logic

Trigger when:

1. The process image is:
   `C:\Windows\System32\WindowsPowerShell\v1.0\powershell.exe`

AND

2. The command line contains:
   - `-EncodedCommand`
   - OR `-enc`

## MITRE ATT&CK

- Technique: T1059.001
- Name: Command and Scripting Interpreter: PowerShell

## Validation

### Positive Test

A controlled PowerShell command was executed using:

`-EncodedCommand`

The resulting Sysmon Event ID 1 contained:

- PowerShell image path
- Full command line
- Parent process information
- User information
- Process GUID information

The detection successfully matched the event.

### Negative / Baseline Activity

Normal PowerShell activity was also observed, including controlled executions
using:

- `Get-Date`
- `Get-Process`

These commands do not contain the encoded-command indicator.

## Assessment

The analytic is intended as an investigation trigger rather than a definitive
malware verdict.

Further tuning should consider:

- Parent process
- User
- Host
- Encoded command frequency
- Script content
- Network activity
- Other correlated telemetry

## Status

Validated prototype detection.

## Next Validation

Replay the behavior after SIEM ingestion and verify that the same analytic
can identify the event centrally.
