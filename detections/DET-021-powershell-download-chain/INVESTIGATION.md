# DET-021 Investigation Evidence

## Scope

Controlled benign investigation on DC-01 to determine whether a PowerShell HTTP download can be represented as a correlated Event 1 → Event 3 → Event 11 chain using Sysmon ProcessGuid.

## Validation 1 — FileCreate telemetry

The active Sysmon configuration was inspected. Sysmon 15.22 reported FileCreate monitoring enabled. DC-01 generated Event 11 records during the investigation. A controlled DET021-Probe2.ps1 creation produced a local Event 11, and Wazuh received the corresponding Event 11.

## Validation 2 — PowerShell network process

Controlled command:

```powershell
powershell.exe -NoProfile -Command "Invoke-WebRequest http://example.com -UseBasicParsing -OutFile C:\Windows\Temp\DET021-NetworkProbe.txt"
```

Observed Sysmon Event 1:
- ProcessGuid: `{939ab59d-3634-6ac3-d801-000000001d00}`
- PID: `6940`
- PowerShell
- User: `CORP\\Administrator`
- Time: `2026-10-05 05:31:32.834Z`

Observed Sysmon Event 3:
- Same ProcessGuid
- Same PID
- TCP
- Initiated: true
- Source: `10.10.20.10:51420`
- Destination: `104.20.23.154:80`
- Time: `2026-10-05 05:31:34.439Z`

Wazuh received the exact Event 3 and generated rule 100108, the existing DET-020 PowerShell process → network correlation.

## Validation 3 — Downloaded file

The file was verified on DC-01:

```
C:\Windows\Temp\DET021-NetworkProbe.txt
577 bytes
LastWriteTime: 2026-10-05 11:01:34
```

The content was Example Domain HTML.

However, local Sysmon queries for Event 11 returned no event for this exact filename. A time-window query around the file creation also returned no Event 11, and a ProcessGuid search returned no Event 11 containing `{939ab59d-3634-6ac3-d801-000000001d00}`.

## Result

### Positive
- PowerShell Event 1: **PASS**
- Event 1 → Event 3 same ProcessGuid: **PASS**
- Wazuh Event 3 ingestion: **PASS**
- Existing Wazuh 100108 correlation: **PASS**
- File exists on disk: **PASS**
- Event 11 exists for other controlled file creation: **PASS**

### Boundary
- Downloaded file → Event 11: **NOT OBSERVED**
- Event 3 → Event 11 same ProcessGuid: **NOT VALIDATED**
- New three-stage Wazuh rule: **NOT CREATED**

## Interpretation

The investigation does not establish that Sysmon or Wazuh universally misses downloaded files. It establishes only that this particular controlled download did not produce an observable Event 11 in the tested window, while other FileCreate events did.

No unsupported correlation or ATT&CK claim is made.
