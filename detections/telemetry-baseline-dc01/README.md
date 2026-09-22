# DC-01 Telemetry Baseline

## Objective

Validate that DC-01 provides the Windows security and process telemetry required for Active Directory detection engineering, threat hunting, purple-team exercises, investigation, and incident response.

## Environment

| Component | Value |
|---|---|
| Host | DC-01 |
| OS | Windows Server 2025 |
| IP | 10.10.20.10 |
| Domain | corp.home.arpa |
| Wazuh Agent | 4.14.7 |
| Wazuh Agent ID | 002 |
| Wazuh Server | WAZUH-01 |
| Wazuh Server IP | 10.10.40.10 |
| Sysmon | 15.22 |
| Sysmon channel | Microsoft-Windows-Sysmon/Operational |

## Telemetry Sources

Wazuh is configured to collect:

- Application
- Security
- System
- Microsoft-Windows-Sysmon/Operational

Sysmon is installed and running on DC-01.

## Sysmon Process Creation Validation

A fresh Sysmon Event ID 1 was generated on DC-01 and validated locally using the Windows Event Log.

Validated event characteristics included:

- Provider: Microsoft-Windows-Sysmon
- Event ID: 1 (Process Create)
- Channel: Microsoft-Windows-Sysmon/Operational
- Process image
- Command line
- Parent process image
- Parent command line
- User
- Integrity level
- SHA256 hash
- Process GUID

Example validated event:

- Process: C:\Windows\System32\conhost.exe
- Parent: C:\Windows\SysWOW64\WindowsPowerShell\v1.0\powershell.exe
- User: NT AUTHORITY\SYSTEM
- Integrity: System
- SHA256: 2DBE51CBCBEFA60341AF1AA968DB8825D398FFFC40FE730022AE6F4174B44ADA

## Wazuh Ingestion Validation

The Sysmon Operational channel was added to the DC-01 Wazuh agent configuration and the agent was restarted.

A Sysmon Event ID 1 was subsequently observed in Wazuh alerts.

Validated Wazuh fields included:

- providerName: Microsoft-Windows-Sysmon
- eventID: 1
- channel: Microsoft-Windows-Sysmon/Operational
- computer: DC-01.corp.home.arpa
- agent.name: DC-01
- agent.id: 002
- agent.ip: 10.10.20.10
- decoder: windows_eventchannel
- Process image
- Command line
- Parent process
- Parent command line
- User
- Integrity level
- SHA256 hash

The observed Wazuh alert was associated with Wazuh rule 92021 and Sysmon Event ID 1.

## Validation Chain

```text
DC-01 Process
     |
     v
Sysmon
     |
     v
Microsoft-Windows-Sysmon/Operational
     |
     v
Wazuh Agent 002
     |
     v
WAZUH-01
     |
     v
Wazuh Alert
```

## Result

The DC-01 Sysmon telemetry pipeline is operational and validated end-to-end.

This establishes the process telemetry foundation required for:

- Detection Engineering
- MITRE ATT&CK mapping
- Threat Hunting
- Purple Team exercises
- Investigation
- Incident Response
- Detection validation and replay

## Evidence Notes

Validation was performed on 2026-09-22.

The Wazuh host did not have `/var/ossec/logs/archives/archives.json` at the time of validation. This did not prevent validation because the corresponding Sysmon event was present in `/var/ossec/logs/alerts/alerts.json`.

No claim is made here about telemetry not directly validated during this test.

## Status

**PASS — DC-01 Sysmon telemetry is ingested by Wazuh.**
