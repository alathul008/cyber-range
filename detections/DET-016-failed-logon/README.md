# DET-016 — Failed Logon Detection & Investigation

## Objective

Validate failed-logon telemetry end-to-end:

```
Controlled failed authentication
        ↓
Windows Event ID 4625
        ↓
Wazuh EventChannel ingestion
        ↓
Wazuh native rule 60122
        ↓
SOC investigation
```

The exercise focuses on extracting authentication context from a failed logon and distinguishing a local authentication failure from a remote-source event.

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

Native Wazuh rule **60105** handles Windows Event ID 4625 as a Windows Logon Failure.

Native child rule **60122** handles Event ID 4625 for unknown username or bad password:

- Level: 5
- Description: Logon Failure - Unknown user or bad password
- Category: authentication_failed
- Native MITRE mapping: T1531 — Account Access Removal

No duplicate custom 4625 rule was created.

## Controlled Test

A single intentionally incorrect password was supplied for:

```
CORP\DET008-TestUser
```

The controlled authentication attempt returned Windows error 1326:

```
RUNAS ERROR: Unable to run - cmd.exe
1326: The user name or password is incorrect.
```

The test was intentionally limited to one failed attempt so the permanent CORP lockout threshold of 5 would not be reached.

## Windows Event 4625

DC-01 generated Event ID 4625.

Validated evidence:

- Time: 2026-09-27 12:04:48
- Event ID: 4625
- Subject: CORP\admin
- Target account: CORP\DET008-TestUser
- Logon Type: 2 — Interactive
- Failure Reason: Unknown user name or bad password
- Status: 0xC000006D
- Sub Status: 0xC000006A
- Workstation: DC-01
- Source Network Address: ::1
- Caller Process: C:\Windows\System32\svchost.exe

The Windows event confirms a failed authentication request but does not provide a remote attacker IP. The source address was the IPv6 loopback address `::1`, and the event was generated on DC-01.

## Wazuh Validation

Wazuh ingested the same Event ID 4625 from DC-01 and fired native rule **60122**.

Validated evidence:

- Wazuh alert timestamp: 2026-09-27T06:34:49.500+0000
- Windows event system time: 2026-09-27T06:34:48.5059079Z
- Event Record ID: 30454
- Event ID: 4625
- Agent: DC-01 (002)
- Agent IP: 10.10.20.10
- Decoder: windows_eventchannel
- Wazuh rule: 60122
- Rule level: 5
- Target account: DET008-TestUser
- Failure status: 0xc000006d
- Failure substatus: 0xc000006a
- Logon Type: 2
- Workstation: DC-01
- Source address: ::1

## ATT&CK Mapping

The native Wazuh rule maps the alert to:

- T1531 — Account Access Removal

This is recorded as the native Wazuh ATT&CK mapping. The observed 4625 event itself establishes a failed authentication caused by an incorrect credential; it does not independently establish malicious account-access removal.

## Investigation Workflow

```
4625 alert
   ↓
Identify target account
   ↓
Inspect failure reason and status/substatus
   ↓
Inspect Logon Type
   ↓
Inspect workstation/source address
   ↓
Determine whether the event is local or remote
   ↓
Correlate surrounding authentication events
   ↓
Determine whether activity is benign, administrative, or suspicious
   ↓
Respond appropriately
```

## Investigation Findings

The validated event was a controlled local lab authentication failure.

Key pivots:

| Pivot | Observed value |
|---|---|
| Target | CORP\DET008-TestUser |
| Failure | Unknown username or bad password |
| Status | 0xC000006D |
| Substatus | 0xC000006A |
| Logon Type | 2 |
| Workstation | DC-01 |
| Source | ::1 |
| Process | svchost.exe |
| Event Record ID | 30454 |

Because the source was `::1` and the event was generated locally on DC-01, this test should **not** be documented as a remote brute-force or password-spray event.

## Relationship to DET-015

DET-015 validated the downstream account-lockout condition using Event ID 4740 and established the permanent CORP lockout baseline:

```
LockoutThreshold          = 5
LockoutDuration           = 10 minutes
LockoutObservationWindow  = 10 minutes
```

DET-016 validates the preceding failed-authentication telemetry without intentionally reaching that threshold.

## Limitations

- Event 4625 establishes a failed authentication but does not, by itself, establish malicious intent.
- The controlled test occurred locally on DC-01.
- The observed source address was `::1`, so this evidence cannot be used to attribute the failure to a remote source.
- Native Wazuh rule 60122 was used; no duplicate custom detection was necessary.
- The native ATT&CK mapping to T1531 is preserved as vendor rule metadata and is not treated as proof of T1531 behavior.

## Result

**PASS — Windows Event 4625, Wazuh ingestion, native rule 60122, authentication pivots, local-source interpretation, ATT&CK mapping, and SOC investigation workflow were validated.**
