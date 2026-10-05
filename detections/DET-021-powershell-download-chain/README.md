# DET-021 — PowerShell Download Chain Telemetry Boundary

## Status

**COMPLETED — telemetry investigation / boundary documented**

No new Wazuh rule was created for a three-stage PowerShell → network → file correlation because the required Sysmon Event 11 relationship was not consistently observable for the controlled download.

## Objective

Validate whether a benign PowerShell download chain can be represented and correlated through:

```
PowerShell process
      ↓
Sysmon Event 3 — network connection
      ↓
Sysmon Event 11 — downloaded file creation
```

The investigation also validates what Wazuh actually ingests before adding detection logic.

## Controlled Activity

On DC-01, a harmless PowerShell HTTP request was used:

```powershell
powershell.exe -NoProfile -Command "Invoke-WebRequest http://example.com -UseBasicParsing -OutFile C:\Windows\Temp\DET021-NetworkProbe.txt"
```

The resulting file was verified on disk:

- `C:\Windows\Temp\DET021-NetworkProbe.txt`
- Length: 577 bytes
- LastWriteTime: 2026-10-05 11:01:34
- Content was the expected Example Domain HTML.

A separate harmless file-creation probe was used to establish reliable Event 11 telemetry:

```powershell
powershell.exe -NoProfile -Command "Set-Content -Path C:\Windows\Temp\DET021-Probe2.ps1 -Value 'DET021 harmless telemetry probe 2'"
```

## Validated Evidence

### PowerShell Event 1

For the network/download process:

- UtcTime: 2026-10-05 05:31:32.834Z
- ProcessGuid: `{939ab59d-3634-6ac3-d801-000000001d00}`
- ProcessId: `6940`
- Image: PowerShell.exe
- User: `CORP\\Administrator`

### Sysmon Event 3

The same process generated a network connection:

- UtcTime: 2026-10-05 05:31:34.439Z
- ProcessGuid: `{939ab59d-3634-6ac3-d801-000000001d00}`
- ProcessId: `6940`
- Protocol: TCP
- Initiated: true
- Source: `10.10.20.10:51420`
- Destination: `104.20.23.154:80`

Wazuh received this Event 3 and fired existing rule **100108** (DET-020), confirming the existing ProcessGuid-based PowerShell → network correlation.

### Sysmon Event 11

For the separate `DET021-Probe2.ps1` controlled file creation:

- ProcessGuid: `{939ab59d-34c5-6ac3-9b01-000000001d00}`
- ProcessId: `2008`
- Image: PowerShell.exe
- TargetFilename: `C:\Windows\Temp\DET021-Probe2.ps1`
- Event 11 was observed locally and in Wazuh.

This confirms Sysmon Event 11 and Wazuh ingestion are operational for qualifying file-creation events.

## Telemetry Boundary

For the actual `DET021-NetworkProbe.txt` download:

- File creation on disk: **confirmed**
- Sysmon Event 3: **confirmed**
- Wazuh Event 3 ingestion: **confirmed**
- Sysmon Event 11 for `DET021-NetworkProbe.txt`: **not observed**
- Sysmon Event 11 with the network process's ProcessGuid: **not observed**
- Wazuh three-stage correlation: **not validated**

Therefore, the evidence does **not** support claiming a reliable Event 3 → Event 11 same-ProcessGuid relationship for this download.

## Detection Decision

No new Wazuh correlation rule was created.

The existing DET-020 rule 100108 remains the validated coverage for:

```
PowerShell Event 1 → Sysmon Event 3
same ProcessGuid
```

Creating a new three-stage rule without observable Event 11 telemetry would be unsupported by the evidence.

## Engineering Lesson

This scenario demonstrates a detection-engineering boundary rather than a failed project step:

> Detection logic must be based on telemetry that is actually observable and repeatable.

A file existing on disk is not equivalent to Sysmon Event 11 telemetry for that file, and an Event 11 observed for other activity does not prove the same event will be emitted for every file-creation path.

## ATT&CK

No new ATT&CK technique is claimed from this benign validation. The observed behavior was controlled lab activity against `example.com`.

## Cleanup

The downloaded/probe files were created solely for controlled telemetry validation. No malware or intentionally malicious payload was used.
