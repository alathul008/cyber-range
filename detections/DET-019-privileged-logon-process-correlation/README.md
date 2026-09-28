# DET-019 — Privileged Logon → Process Creation Correlation

**Status:** CURRENT / NOT STARTED

## Objective

Investigate whether a privileged interactive logon is followed by process creation within the same Windows logon session, using stable session identifiers where the available telemetry supports correlation.

The scenario extends the validated DET-018 chain:

```
4624 Successful Logon
        ↓
4672 Special Privileges
        ↓
Sysmon Event ID 1
        ↓
Process Creation
        ↓
Wazuh
        ↓
Logon ID / session correlation
        ↓
Investigation
```

No DET-019 attack or test has been executed yet.

## Environment

Established project environment:

- DC-01 — Windows Server 2025 Active Directory domain controller
- WIN-01 — Windows enterprise endpoint
- WAZUH-01 — Ubuntu/Wazuh security server
- Wazuh agents:
  - WIN-01: agent 001
  - DC-01: agent 002
- Sysmon installed on DC-01 and WIN-01
- DC-01 Sysmon/Operational telemetry is explicitly collected by Wazuh
- AD domain: `corp.home.arpa`
- Domain NetBIOS name: `CORP`

DET-019 execution must remain within the established isolated lab architecture.

## Prerequisites

Before execution:

1. This README must exist as the persistent DET-019 artifact.
2. A pre-attack snapshot must be created for the relevant VM(s).
3. Existing telemetry collection must remain unchanged unless a documented validation requires otherwise.
4. The test must produce observable Windows logon, privilege, process-creation, and Wazuh telemetry.
5. The resulting evidence must be sufficient to determine whether the process can be correlated to the privileged logon session.

## Expected Telemetry

The expected conceptual telemetry chain is:

- Windows Security Event ID 4624 — successful logon
- Windows Security Event ID 4672 — special privileges assigned
- Sysmon Event ID 1 — process creation
- Wazuh ingestion of the relevant telemetry
- Windows Logon ID/session identifiers where available

These are expected telemetry sources, not observed DET-019 results.

## Detection Opportunity

Determine whether a privileged logon followed by process creation can be correlated using a stable session identifier, particularly the Windows Logon ID.

The detection question is:

> Can process execution be associated with the privileged session established by the preceding 4624/4672 events?

The test must distinguish:

- successful authentication,
- privileged-session establishment,
- process creation,
- and actual session correlation.

## Correlation Approach

The investigation should use the identifiers actually present in the collected telemetry.

DET-018 already validated:

```
4624 → 4672
```

using the Windows Logon ID.

DET-019 extends that investigation pivot toward Sysmon Event ID 1 process creation.

The exact correlation fields, query, and implementation must be determined from the telemetry observed during the controlled test.

No unsupported correlation syntax is pre-defined.

## Detection

**Status: NOT YET VALIDATED**

No DET-019 detection rule or persistent correlation logic is claimed at this stage.

Native Wazuh coverage should be evaluated first. Any custom detection/correlation logic must be based on actual observed telemetry.

## MITRE ATT&CK

**Status: TBD**

No ATT&CK technique is pre-claimed.

The final mapping must be determined from the actual behavior and telemetry observed during DET-019 execution.

## IOC / IOA / TTP

**Status: NOT YET OBSERVED**

No IOC, IOA, or TTP is recorded until the controlled test produces actual evidence.

## Triage

The eventual investigation should establish:

1. Which account created the privileged logon.
2. The relevant Windows Logon ID.
3. Whether Event 4672 corresponds to that logon.
4. Which Sysmon Event ID 1 process was created.
5. Whether the process can be associated with the same logon session.
6. What Wazuh telemetry and rule context were generated.
7. Whether the observed relationship is sufficient for reliable detection.

## Investigation

Investigation evidence will be recorded only after the controlled test is executed.

Potential investigation pivots include:

- Windows Logon ID
- account/domain
- event timestamps
- Sysmon ProcessGuid
- process image
- command line
- parent process
- Wazuh agent
- Wazuh rule
- Windows event record identifiers

Only fields actually present in the resulting telemetry will be documented.

## Evidence

**Status: NOT YET COLLECTED**

No screenshots, timestamps, event records, Wazuh alerts, queries, metrics, or findings are claimed before execution.

## Detection Gap

To be determined from the actual test.

The primary question is whether existing telemetry and Wazuh coverage provide sufficient linkage between:

```
Privileged Logon
      ↓
Process Creation
```

within the same Windows session.

## Improvement

To be determined after the initial controlled test.

Any improvement must address an evidence-backed detection gap rather than a hypothetical one.

## Replay

Not yet performed.

Replay will be considered only after the initial detection behavior and any improvement are established.

## Metrics

Not yet available.

No detection rate, timing, false-positive rate, or correlation metric is claimed before execution.

## Snapshot Gate

**Pre-attack snapshot: NOT CREATED**

The snapshot must be created and verified before executing the DET-019 test.

## Execution Gate

**DET-019 attack/test: NOT STARTED**

Do not execute until:

- the README is committed,
- the relevant pre-attack snapshot exists,
- and the snapshot state is verified.

## Status

**CURRENT / NOT STARTED**

This document establishes the DET-019 scenario and its evidence requirements. It does not represent execution results.
