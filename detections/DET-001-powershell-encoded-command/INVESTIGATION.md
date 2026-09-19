# DET-001 — PowerShell Encoded Command Detection

## Objective

Detect PowerShell execution using `-EncodedCommand` / `-enc` on the Windows endpoint WIN-01 and map the behavior to MITRE ATT&CK T1059.001 (PowerShell).

This exercise validates the full detection-engineering workflow:

**Execution → Telemetry → Detection → Investigation → ATT&CK mapping → Retest**

## Environment

- Endpoint: WIN-01
- Endpoint IP: 10.10.20.100
- Domain: CORP / corp.home.arpa
- SIEM/XDR: Wazuh 4.14.7
- Detection rule: 100101
- Detection level: 12
- Telemetry sources:
  - Windows Security Event ID 4688
  - Sysmon Event ID 1
  - Sysmon Event ID 3
  - Sysmon Event ID 22

## Detection Logic

The custom Wazuh rule:

- Chains from Wazuh rule 67027 (Windows process creation).
- Matches the Windows process command line field for `-EncodedCommand` or `-enc`.
- Maps the behavior to MITRE ATT&CK T1059.001.
- Generates a Level 12 alert.

Rule file:

`DET-001-powershell-encoded-command.yml`

Rule ID:

`100101`

## Test Activity

A benign encoded PowerShell command was executed on WIN-01:

```powershell
powershell.exe -NoProfile -EncodedCommand VwByAGkAdABlAC0ATwB1AHQAcAB1AHQAIABQAG8AdwBlAHIAUwBoAGUAbABsACAAdABlAHMAdAAgAGMAbwBtAG0AYQBuAGQA
```

The Base64 payload was decoded using PowerShell and produced:

```text
Write-Output PowerShell test command
```

The payload was therefore a controlled, benign test payload.

## Investigation

### Windows Security Event ID 4688

The process creation event established:

- Account: `CORP\\Administrator`
- New process: `C:\\Windows\\System32\\WindowsPowerShell\\v1.0\\powershell.exe`
- New PID: `0x126c` (4716)
- Creator PID: `0x740` (1856)
- Creator process: PowerShell
- Command line contained `-NoProfile -EncodedCommand`
- Mandatory label: `S-1-16-12288` (High integrity)

### Sysmon Event ID 1

Sysmon independently recorded the process creation:

- Image: `C:\\Windows\\System32\\WindowsPowerShell\\v1.0\\powershell.exe`
- Process ID: 4716
- Parent Process ID: 1856
- Parent Image: PowerShell
- User: `CORP\\Administrator`
- SHA256:
  `9785001B0DCF755EDDB8AF294A373C0B87B2498660F724E76C4D53F9C217C7A3`

The available Sysmon telemetry did not contain the creation event for parent PID 1856, so the original launcher could not be established from retained Event ID 1 telemetry.

### Network investigation

A Sysmon Event ID 3 search for PID 4716 during the detection window returned no matching events.

This is documented as:

> No matching Sysmon Event ID 3 network connection was observed for PID 4716 during the investigated window.

This does **not** establish that the process made no network connection; it establishes that no matching Sysmon network event was available in the searched window.

### DNS investigation

A Sysmon Event ID 22 search for the investigated window returned no events.

This is documented as:

> No Sysmon DNS-query event was observed in the investigated window.

## IOC / IOA / TTP Assessment

### IOC

No malicious IOC was established during this controlled test.

The observed PowerShell SHA256 belongs to the Windows PowerShell executable recorded during the test and was not treated as a malicious IOC.

### IOA

The primary indicator of attack/behavior (IOA) was:

```text
powershell.exe -NoProfile -EncodedCommand <Base64>
```

### TTP

MITRE ATT&CK:

- Tactic: Execution
- Technique: T1059.001
- Technique name: PowerShell

## Detection Result

Wazuh generated:

- Rule ID: 100101
- Level: 12
- Description: PowerShell encoded command detected on Windows endpoint.
- MITRE ID: T1059.001
- MITRE tactic: Execution
- MITRE technique: PowerShell

## Retest

The same benign encoded PowerShell behavior was executed again after the initial investigation.

The retest generated a fresh Wazuh alert at approximately:

```text
2026-09-19 15:55:37
```

The Wazuh Threat Hunting view showed:

- Agent: WIN-01
- Rule ID: 100101
- Rule level: 12
- Detection description: PowerShell encoded command detected on Windows endpoint.
- MITRE ID: T1059.001
- MITRE tactic: Execution
- MITRE technique: PowerShell

The rule's `firedtimes` value for the fresh alert was 1.

## Detection Gap

The detection successfully identifies the encoded PowerShell behavior, but this exercise also exposed an investigation limitation:

1. The original launcher of parent PID 1856 could not be established from retained Sysmon Event ID 1 telemetry.
2. No matching Sysmon network event was observed for the detected process.
3. No Sysmon DNS event was observed in the investigated window.
4. The detection is behavior-focused and does not by itself determine whether the encoded payload is malicious.

These are investigation/telemetry limitations, not failures of the DET-001 rule itself.

## Conclusion

DET-001 successfully detected and mapped controlled encoded PowerShell execution on WIN-01.

The detection was independently supported by Windows Security 4688 and Sysmon Event ID 1 telemetry, investigated through process lineage and network/DNS telemetry, mapped to MITRE ATT&CK T1059.001, and successfully reproduced during retesting.

The test payload was benign by design. No malicious IOC was established.

## Evidence

Evidence collected during the exercise included:

- Wazuh alert for rule 100101
- Windows Security Event ID 4688
- Sysmon Event ID 1
- Sysmon Event ID 3 search result
- Sysmon Event ID 22 search result
- PowerShell Base64 decoding result
- Wazuh retest alert showing T1059.001
