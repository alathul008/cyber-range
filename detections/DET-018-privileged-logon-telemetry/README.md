# DET-018 — Privileged Logon Telemetry

## Objective

Validate telemetry for a successful interactive Windows logon followed by assignment of special privileges, using the Windows logon identifier to correlate the events.

## Environment

- Domain: `corp.home.arpa`
- Domain controller: `DC-01`
- Wazuh agent: `DC-01` (agent 002)
- Test account: `CORP\\admin`

## Technique / Telemetry

### Windows Security 4624

A controlled interactive logon was generated with:

- Account: `CORP\\admin`
- Logon Type: 2 (Interactive)
- Logon ID: `0x2c5499`
- Source address: `::1`
- Workstation: `DC-01`
- Time: 2026-09-28 09:02:03

### Windows Security 4672

The matching special-privilege event was verified in the Windows Security log:

- Event ID: 4672
- Time: 2026-09-28 09:02:03
- Record ID: 33066
- Logon ID: `0x2c5499`

The shared Logon ID provides the investigation pivot between the successful logon and the special-privilege assignment.

## Wazuh Validation

Wazuh retained the 4672 event in `wazuh-alerts-4.x-2026.09.28`:

- Rule: 67028
- Level: 3
- Description: `Special privileges assigned to new logon.`
- Account: `CORP\\admin`
- Logon ID: `0x2c5499`
- Event ID: 4672
- Agent: DC-01

The corresponding 4624 event was previously validated with native Wazuh rule 60118, and a Wazuh pivot on Logon ID `0x2c5499` returned the matching successful-logon event.

## ATT&CK Mapping

Wazuh native rule 67028 maps Event ID 4672 to **T1484**. This is vendor/native rule metadata and should not be interpreted as proof that Domain Policy Modification occurred.

The validated finding is **privileged-logon telemetry**, not evidence of malicious activity.

## Investigation Notes

- The source address `::1` indicates the controlled activity was local to DC-01.
- No remote-source claim is made.
- Event correlation is based on the shared Windows Logon ID.
- The 4672 event demonstrates special privileges assigned to the logon session.
- Additional evidence would be required to attribute a specific malicious technique or domain-policy modification.

## Result

**PASS** — Windows Security 4624 and 4672 telemetry was validated and correlated through Logon ID `0x2c5499`, with Wazuh retention and native alerting confirmed.
