# DET-014 — Investigation Evidence

## Validation Timeline

| Time | Source | Event | Evidence |
|---|---|---|---|
| 2026-09-27 11:04:25 | DC-01 Security | 4624 | CORP\admin, Logon Type 2, Elevated Token Yes, Logon ID 0x3F868B |
| 2026-09-27 11:04:25 | DC-01 Security | 4672 | CORP\admin, same Logon ID 0x3F868B, special privileges assigned |
| 2026-09-27T05:34:36.249Z | Wazuh | 67028 | Native alert for Event 4672, eventRecordID 29662 |

## Windows Correlation

The 4624 event established:

```
Account: CORP\admin
Logon Type: 2
Elevated Token: Yes
Logon ID: 0x3F868B
```

The corresponding 4672 event contained:

```
Account: CORP\admin
Logon ID: 0x3F868B
```

The shared Logon ID is the correlation key.

## Wazuh Evidence

The validated Wazuh alert contained:

```
rule.id = 67028
rule.level = 3
win.system.eventID = 4672
win.system.eventRecordID = 29662
agent.name = DC-01
agent.id = 002
subjectUserName = admin
subjectDomainName = CORP
subjectLogonId = 0x3f868b
```

The alert description was:

```
Special privileges assigned to new logon.
```

## Commands Used

Windows:

```powershell
auditpol /get /subcategory:"Special Logon"

whoami
whoami /groups

Get-WinEvent -FilterHashtable @{
    LogName='Security'
    Id=4624
} -MaxEvents 100

Get-WinEvent -FilterHashtable @{
    LogName='Security'
    Id=4672
} -MaxEvents 500
```

Wazuh:

```bash
sudo grep '"id":"67028"' /var/ossec/logs/alerts/alerts.json | tail -5
sudo grep '"eventRecordID":"29662"' /var/ossec/logs/alerts/alerts.json
```

## Interpretation

The test validates the telemetry pipeline:

```
Windows logon
  → 4624
  → privileged-logon assignment
  → 4672
  → Wazuh EventChannel ingestion
  → native rule 67028
```

The native rule's T1484 mapping is documented as a vendor/ruleset mapping. The observed event itself demonstrates privileged-logon telemetry and does not by itself establish domain policy modification.

## Result

**PASS.**
