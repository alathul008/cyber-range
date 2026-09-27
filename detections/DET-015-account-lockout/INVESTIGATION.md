# DET-015 — Investigation Evidence

## Evidence Chain

```
5 controlled authentication failures
        ↓
DET008-TestUser locked
        ↓
Windows Event 4740
        ↓
Wazuh rule 60115
        ↓
Level 9 authentication-failure alert
```

## Windows Evidence

Account baseline before test:

```
DET008-TestUser
Enabled    : True
LockedOut  : False
```

After five incorrect authentication attempts:

```
DET008-TestUser
Enabled    : True
LockedOut  : True
```

Windows Security Event 4740:

```
Time: 2026-09-27 11:48:59
Event ID: 4740
Account Name: DET008-TestUser
Target SID: S-1-5-21-2519611076-441997742-1464114610-1115
Caller Computer Name: DC-01
```

## Wazuh Evidence

Validated alert:

```
timestamp: 2026-09-27T06:19:48.091+0000
rule.id: 60115
rule.level: 9
rule.description: User account locked out (multiple login errors)
agent.id: 002
agent.name: DC-01
win.system.eventID: 4740
win.system.eventRecordID: 30239
targetUserName: DET008-TestUser
targetSid: S-1-5-21-2519611076-441997742-1464114610-1115
subjectUserName: DC-01$
subjectUserSid: S-1-5-18
```

Native ATT&CK mappings:

```
T1110 — Brute Force
T1531 — Account Access Removal
```

## Commands Used

Windows:

```powershell
auditpol /get /subcategory:"Account Lockout"

Get-ADDefaultDomainPasswordPolicy |
Select-Object LockoutThreshold,LockoutDuration,LockoutObservationWindow

Get-ADUser -Identity "DET008-TestUser" -Properties LockedOut,LockoutTime |
Select-Object SamAccountName,Enabled,LockedOut,LockoutTime

Get-WinEvent -FilterHashtable @{
    LogName='Security'
    Id=4740
} -MaxEvents 10 |
Select-Object TimeCreated, Id, Message |
Format-List

Unlock-ADAccount -Identity "DET008-TestUser"
```

Wazuh:

```bash
sudo grep '"eventID":"4740"' /var/ossec/logs/alerts/alerts.json | tail -5
```

## Permanent Baseline Change

The domain initially had:

```
LockoutThreshold = 0
```

For the cyber range architecture, the domain baseline was intentionally changed to:

```
LockoutThreshold          = 5
LockoutDuration           = 10 minutes
LockoutObservationWindow = 10 minutes
```

This change is permanent for the current lab baseline and must be considered by future authentication scenarios.

## Cleanup Verification

After validation:

```
DET008-TestUser
Enabled    : True
LockedOut  : False
```

The test account was successfully unlocked.

## Result

**PASS.**
