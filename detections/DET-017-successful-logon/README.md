# DET-017 — Successful Logon Investigation

## Objective

Validate Windows successful-logon telemetry for a controlled interactive domain logon and demonstrate an investigation pivot from the Windows 4624 Logon ID into Wazuh.

## Environment

- Domain: `corp.home.arpa`
- NetBIOS domain: `CORP`
- Domain Controller: `DC-01`
- Wazuh: `WAZUH-01`
- Wazuh agent: `DC-01` (agent 002)
- Test account: `CORP\\admin`

## Detection Coverage

Wazuh native rule:

- Rule ID: `60118`
- Description: `Windows Workstation Logon Success`
- Windows Event ID: `4624`

The existing native Windows Logon Success rule `60106` also covers Event ID 4624, while the observed `CORP\\admin` interactive logon was represented by rule `60118`. No duplicate custom detection rule was created.

## Controlled Test

A controlled local interactive logon was initiated on DC-01:

```powershell
runas /user:CORP\admin cmd.exe
```

Windows Security Event 4624 was generated at approximately `09:02:03` on 2026-09-28.

Relevant event fields:

- Account: `CORP\\admin`
- Logon Type: `2` (Interactive)
- Elevated Token: `Yes`
- Logon ID: `0x2C5499`
- Workstation: `DC-01`
- Source address: `::1`
- Process: `C:\Windows\System32\svchost.exe`
- Authentication package: `Negotiate`

The event was local to DC-01; the `::1` source must not be interpreted as a remote-origin logon.

## Wazuh Validation

The corresponding Wazuh event was observed in index `wazuh-alerts-4.x-2026.09.28`:

- Wazuh timestamp: `2026-09-28T03:33:20.070Z`
- Rule: `60118`
- Description: `Windows Workstation Logon Success`
- Event ID: `4624`
- Target user: `admin`
- Target domain: `CORP`
- Logon Type: `2`
- Target Logon ID: `0x2c5499`
- Source address: `::1`

## Investigation Pivot

A Wazuh query using the Windows Logon ID `0x2c5499` returned exactly one matching event.

This demonstrates that the 4624 Logon ID can be used as an investigation pivot from the Windows Security event into the Wazuh alert data.

## ATT&CK Context

The native Wazuh Windows Logon Success rule family maps successful authentication telemetry to MITRE ATT&CK T1078 (Valid Accounts).

This validation demonstrates authentication telemetry and investigation capability. The event itself does not establish malicious use of valid credentials.

## Limitations

- The controlled event originated locally on DC-01 (`::1`).
- It does not demonstrate a remote authentication source.
- Successful authentication alone does not establish malicious activity.
- No custom rule was required because native Wazuh coverage was already present.

## Validation Result

**PASS**

Validated:

- Windows 4624 successful-logon telemetry
- Human domain-account interactive logon
- Native Wazuh detection
- Windows-to-Wazuh Logon ID correlation
- Logon ID investigation pivot
- Local-source interpretation
