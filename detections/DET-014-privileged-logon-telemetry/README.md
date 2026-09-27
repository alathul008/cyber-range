# DET-014 — Privileged Logon Telemetry (4624 → 4672)

## Objective

Validate Windows privileged-logon telemetry on DC-01 and confirm that an elevated CORP\admin logon produces Event ID 4672 and is ingested by Wazuh through the native rule 67028.

## Environment

| Component | Value |
|---|---|
| Domain Controller | DC-01 |
| DC IP | 10.10.20.10 |
| Domain | corp.home.arpa |
| Test account | CORP\admin |
| Wazuh agent | 002 / DC-01 |
| Wazuh manager | wazuh-01 |
| Windows event | 4672 |
| Native Wazuh rule | 67028 |
| Rule level | 3 |

## Prerequisites

Windows auditing was already configured for:

- Logon/Logoff → Special Logon: Success

Verified on DC-01 with:

```powershell
auditpol /get /subcategory:"Special Logon"
```

Result:

```
Special Logon    Success
```

No audit-policy change was required for this validation.

## Test

A new elevated PowerShell session was opened using CORP\admin.

Identity and token state were verified with:

```powershell
whoami
whoami /groups
```

The session showed CORP\admin, BUILTIN\Administrators, and High Mandatory Level.

## Windows Telemetry

A successful logon (Event ID 4624) was identified for CORP\admin:

- Time: 2026-09-27 11:04:25
- Logon Type: 2 (Interactive)
- Elevated Token: Yes
- Logon ID: 0x3F868B
- Account SID: S-1-5-21-2519611076-441997742-1464114610-1000

The corresponding Event ID 4672 was correlated by the same Logon ID:

- Time: 2026-09-27 11:04:25
- Account: CORP\admin
- Logon ID: 0x3F868B
- Privileges included SeDebugPrivilege, SeBackupPrivilege, SeRestorePrivilege, SeTakeOwnershipPrivilege, SeImpersonatePrivilege, and others.

This establishes the Windows-side 4624 → 4672 correlation.

## Wazuh Validation

Wazuh received the Event ID 4672 from DC-01 and fired native rule 67028.

Validated alert evidence:

- Wazuh alert timestamp: 2026-09-27T05:34:36.249+0000
- Windows event time: 2026-09-27T05:34:25.4059063Z
- Event Record ID: 29662
- Event ID: 4672
- Agent: DC-01 (002)
- Subject: CORP\admin
- Subject Logon ID: 0x3f868b
- Rule: 67028
- Rule level: 3
- Description: Special privileges assigned to new logon.

## Native Rule

Rule 67028 is defined in `0955-WEF-baseline_rules.xml`:

```xml
<rule id="67028" level="3">
  <if_sid>60103</if_sid>
  <field name="win.system.eventID">^4672$</field>
  <field name="win.eventdata.subjectUserSid" negate="yes">^S-1-5-18$</field>
  <description>Special privileges assigned to new logon.</description>
  <mitre>
    <id>T1484</id>
  </mitre>
</rule>
```

The rule excludes LocalSystem (S-1-5-18).

## 4624 Baseline

Wazuh native rule 60106 provides the successful-logon baseline:

- Rule: 60106
- Level: 3
- Description: Windows Logon Success
- Event: 4624
- MITRE: T1078 (Valid Accounts)

Detail rules further differentiate logon types and service accounts.

No duplicate custom 4624 or 4672 rule was created.

## Investigation Value

The validated investigation pivot is:

```
4624 successful logon
  → identify Logon ID
  → correlate 4672 on same Logon ID
  → inspect privileged token and assigned privileges
  → investigate surrounding activity
```

This provides a useful privileged-logon telemetry pivot for future detection engineering and threat-hunting exercises.

## ATT&CK Note

The native Wazuh rule maps Event 4672 to T1484 (Domain Policy Modification). This validation demonstrates privileged-logon telemetry, not independent evidence that a domain policy was modified. The ATT&CK mapping is therefore recorded as the native rule's mapping rather than as a conclusion that domain policy modification occurred.

## Limitations

- Rule 67028 is level 3 and is native Wazuh coverage.
- This exercise validates telemetry and investigation workflow; it does not create a persistent custom correlation rule.
- Event 4672 can also be generated for LocalSystem and other privileged contexts; the native rule excludes S-1-5-18.
- No claim is made that every privileged logon will produce identical telemetry across all Windows logon contexts.

## Result

**PASS — Windows 4624 → 4672 privileged-logon telemetry was validated on DC-01 and confirmed in Wazuh.**
