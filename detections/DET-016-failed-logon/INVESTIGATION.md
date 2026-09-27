# DET-016 — Investigation Evidence

## Evidence Summary

A controlled failed authentication was generated against `CORP\DET008-TestUser` using an intentionally incorrect password.

Windows returned:

```
RUNAS ERROR: Unable to run - cmd.exe
1326: The user name or password is incorrect.
```

The test was limited to one failed attempt to avoid reaching the permanent domain lockout threshold of 5.

## Windows Evidence

Event ID **4625** was validated on DC-01.

Observed values:

| Field | Value |
|---|---|
| TimeCreated | 2026-09-27 12:04:48 |
| Event ID | 4625 |
| Subject account | CORP\admin |
| Target account | CORP\DET008-TestUser |
| Logon Type | 2 |
| Failure Reason | Unknown user name or bad password |
| Status | 0xC000006D |
| Sub Status | 0xC000006A |
| Workstation | DC-01 |
| Source Network Address | ::1 |
| Caller Process | C:\Windows\System32\svchost.exe |
| Logon Process | seclogo |
| Authentication Package | Negotiate |

## Wazuh Evidence

The corresponding alert was located in:

```
/var/ossec/logs/alerts/alerts.json
```

Validated alert:

| Field | Value |
|---|---|
| Wazuh timestamp | 2026-09-27T06:34:49.500+0000 |
| Windows system time | 2026-09-27T06:34:48.5059079Z |
| Event Record ID | 30454 |
| Event ID | 4625 |
| Agent | DC-01 |
| Agent ID | 002 |
| Agent IP | 10.10.20.10 |
| Decoder | windows_eventchannel |
| Rule | 60122 |
| Rule level | 5 |
| Description | Logon Failure - Unknown user or bad password |
| Target | DET008-TestUser |
| Status | 0xc000006d |
| Substatus | 0xc000006a |
| Logon Type | 2 |
| Workstation | DC-01 |
| Source | ::1 |

## Investigation Assessment

The event represents a controlled local authentication failure.

The source address `::1` is the IPv6 loopback address, and the Windows event identifies DC-01 as the workstation. Therefore, this evidence does not establish a remote attacker source.

The authentication failure is consistent with the intentionally supplied incorrect password.

## ATT&CK Interpretation

Wazuh rule 60122 carries the native mapping:

- T1531 — Account Access Removal
- Tactic: Impact

This mapping is recorded as native Wazuh metadata. The observed Event ID 4625 itself demonstrates a failed authentication with an incorrect credential; it does not independently prove malicious account-access removal.

## SOC Investigation Pivots

For a real failed-logon alert, an analyst should pivot on:

1. Target username and SID.
2. Failure status and substatus.
3. Logon type.
4. Workstation name.
5. Source network address.
6. Authentication package and logon process.
7. Nearby 4625 events for the same account.
8. Subsequent 4740 lockout events.
9. Successful 4624 events surrounding the failure.
10. Related process and endpoint telemetry where available.

## Evidence Chain

```
Intentional bad password
        ↓
Windows error 1326
        ↓
Security Event 4625
        ↓
EventRecordID 30454
        ↓
Wazuh EventChannel decoder
        ↓
Rule 60122
        ↓
SOC investigation
        ↓
Local-source determination
```

## Result

**PASS — The failed authentication was observed at the Windows layer, ingested by Wazuh, matched by native rule 60122, and investigated using the available authentication context.**
