# DET-015 — Account Lockout Detection & Investigation

## Objective

Validate Active Directory account-lockout telemetry end-to-end:

```
Authentication failures
        ↓
AD account lockout
        ↓
Windows Event ID 4740
        ↓
Wazuh native rule 60115
        ↓
SOC investigation
```

The exercise also establishes account lockout as part of the permanent CORP domain security baseline.

## Environment

| Component | Value |
|---|---|
| Domain Controller | DC-01 |
| DC IP | 10.10.20.10 |
| Domain | corp.home.arpa |
| Test account | DET008-TestUser |
| Wazuh agent | 002 / DC-01 |
| Wazuh manager | wazuh-01 |

## Existing Wazuh Coverage

Native Wazuh rule **60115** handles Event ID 4740:

- Level: 9
- Description: User account locked out (multiple login errors)
- Category: authentication_failures
- MITRE T1110 — Brute Force
- MITRE T1531 — Account Access Removal

No duplicate custom 4740 rule was created.

## Windows Audit Prerequisite

Before testing, DC-01 was verified with:

```powershell
auditpol /get /subcategory:"Account Lockout"
```

Result:

```
Account Lockout    Failure
```

No audit-policy change was required.

## Baseline Before Test

The dedicated lab account was verified:

```powershell
Get-ADUser -Identity "DET008-TestUser" -Properties LockedOut,LockoutTime |
Select-Object SamAccountName,Enabled,LockedOut,LockoutTime
```

Result:

- Enabled: True
- LockedOut: False

The domain lockout policy was initially found to have LockoutThreshold 0, meaning account lockout was disabled.

For this cyber range, account lockout was deliberately enabled as a permanent AD security baseline:

| Setting | Value |
|---|---|
| LockoutThreshold | 5 |
| LockoutDuration | 10 minutes |
| LockoutObservationWindow | 10 minutes |

This is an intentional architecture decision, not a temporary test-only configuration.

## Controlled Test

Five controlled authentication failures were generated against:

```
CORP\DET008-TestUser
```

The failed authentication attempts returned Windows error 1326:

```
The user name or password is incorrect.
```

After the threshold was reached, the account was confirmed locked:

```
DET008-TestUser
Enabled    : True
LockedOut  : True
```

## Windows Event 4740

DC-01 generated Event ID 4740:

- Time: 2026-09-27 11:48:59
- Account: DET008-TestUser
- SID: S-1-5-21-2519611076-441997742-1464114610-1115
- Caller Computer Name: DC-01

The event confirmed:

```
A user account was locked out.
```

The Caller Computer Name field provides an investigation pivot for determining which system Windows associated with the lockout.

## Wazuh Validation

Wazuh ingested the DC-01 Event ID 4740 and fired native rule **60115**.

Validated evidence:

- Wazuh alert timestamp: 2026-09-27T06:19:48.091+0000
- Windows event system time: 2026-09-27T06:18:59.2428907Z
- Event Record ID: 30239
- Event ID: 4740
- Agent: DC-01 (002)
- Target account: DET008-TestUser
- Target SID: S-1-5-21-2519611076-441997742-1464114610-1115
- Caller: DC-01
- Wazuh rule: 60115
- Rule level: 9

## ATT&CK Mapping

The native Wazuh rule maps the event to:

- T1110 — Brute Force
- T1531 — Account Access Removal

These mappings are recorded as the native rule's ATT&CK mappings. Event 4740 itself establishes an account-lockout condition; the surrounding investigation determines why the lockout occurred.

## Investigation Workflow

```
4740 alert
   ↓
Identify target account
   ↓
Identify target SID
   ↓
Inspect Caller Computer Name
   ↓
Review preceding authentication failures
   ↓
Determine whether activity is benign, administrative, or suspicious
   ↓
Respond appropriately
```

## Cleanup

After telemetry validation, the dedicated test account was unlocked:

```powershell
Unlock-ADAccount -Identity "DET008-TestUser"
```

Final state:

- Enabled: True
- LockedOut: False

The account remains available as a controlled lab test identity.

## Permanent AD Security Baseline

The CORP domain now intentionally retains:

```
LockoutThreshold          = 5
LockoutDuration           = 10 minutes
LockoutObservationWindow  = 10 minutes
```

Future authentication, password-spray, brute-force, and SOC exercises must account for this baseline because repeated failures can legitimately trigger account lockout.

## Limitations

- Event 4740 identifies the account that was locked and provides the Caller Computer Name reported by Windows; it does not, by itself, prove malicious intent.
- The controlled test was performed locally on DC-01, so the observed caller was DC-01.
- Native Wazuh rule 60115 was used; no custom duplicate detection was necessary.
- The lockout policy affects the entire corp.home.arpa domain and is therefore now part of the documented lab baseline.

## Result

**PASS — AD account lockout, Windows Event 4740, Wazuh rule 60115, ATT&CK mapping, investigation pivots, cleanup, and permanent domain lockout baseline were validated.**
