# DET-018 Investigation Evidence

## Validation Date

2026-09-28

## Evidence Chain

```
4624 Successful Interactive Logon
        |
        | Logon ID = 0x2c5499
        v
4672 Special Privileges Assigned
        |
        v
Wazuh Rule 67028
```

## Windows Evidence

### Event 4624

Controlled local interactive logon:

- Account: `CORP\\admin`
- Event ID: 4624
- Logon Type: 2
- Logon ID: `0x2c5499`
- Workstation: DC-01
- Source: `::1`
- Time: 2026-09-28 09:02:03

### Event 4672

Original Windows Security log query:

```powershell
Get-WinEvent -FilterHashtable @{
    LogName='Security'
    Id=4672
} -MaxEvents 500 |
Where-Object { $_.Message -match '2c5499' } |
Select-Object TimeCreated, Id, RecordId, Message
```

Observed:

```
TimeCreated            Id RecordId
-----------            -- --------
9/28/2026 9:02:03 AM  4672 33066
```

The event message was `Special privileges assigned to new logon....`.

## Wazuh Evidence

Wazuh query against `wazuh-alerts-4.x-2026.09.28` returned one matching 4672 event:

- Rule ID: 67028
- Description: Special privileges assigned to new logon.
- Event ID: 4672
- Subject: `CORP\\admin`
- Subject Logon ID: `0x2c5499`
- Agent: DC-01
- Wazuh timestamp: 2026-09-28T03:33:20.132Z

A Wazuh pivot on `targetLogonId=0x2c5499` previously returned the corresponding 4624 event under native rule 60118.

## Assessment

**PASS.**

The Windows Security log and Wazuh telemetry independently preserve the same Logon ID, allowing an analyst to pivot from a successful interactive logon to the associated special-privilege assignment.

This test does not establish maliciousness, remote access, or Domain Policy Modification. The activity was controlled and local to DC-01.
