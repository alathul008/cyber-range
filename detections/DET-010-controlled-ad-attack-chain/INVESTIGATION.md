# DET-010 Investigation

## Evidence Chain

The controlled test created the domain account `det010.test` on DC-01 and then added it to Domain Admins.

Observed target SID:

`S-1-5-21-2519611076-441997742-1464114610-1117`

### Event 4720 — Account creation

Wazuh received the Windows Security account-creation event.

- Agent: DC-01 / 002
- Target: `CORP\\det010.test`
- Target SID: `S-1-5-21-2519611076-441997742-1464114610-1117`
- Actor shown by event: `CORP\\Administrator`
- Wazuh rule: 100104
- Rule level: 10
- Event system time shown in Wazuh: `2026-09-22T09:52:23.4961222Z`
- Wazuh timestamp: approximately `2026-09-22 15:22:39.945`

The raw event identified Windows Security Event ID 4720.

### Event 4728 — Domain Admins membership

Wazuh received the Windows Security group-membership event.

- Target group: `Domain Admins`
- Member: `DET010-TestUser`
- Member SID: `S-1-5-21-2519611076-441997742-1464114610-1117`
- Actor shown by event: `CORP\\Administrator`
- Wazuh rule: 60159
- Rule level: 12
- Event system time shown in Wazuh: `2026-09-22T09:53:34.7337748Z`
- Wazuh timestamp: approximately `2026-09-22 15:23:36.813`

The raw event message stated that a member was added to a security-enabled global group.

## Correlation Analysis

The same test account and SID were visible across the account-creation and Domain Admins membership events.

The investigation therefore established the relationship:

`4720.targetSid = 4728.memberSid`

Wazuh displayed the individual events, but the `det010.test` search did not produce a dedicated automatic DET-009 correlation alert.

No new correlation rule was deployed.

This is consistent with the existing DET-009 finding that the installed Wazuh rule-engine approach had not demonstrated a supported direct cross-field join between `win.eventdata.targetSid` and `win.eventdata.memberSid`.

## Detection Assessment

### Confirmed

- Windows generated the expected account-creation telemetry.
- Wazuh ingested the account-creation event.
- Wazuh generated rule 100104.
- Windows generated the expected Domain Admins membership telemetry.
- Wazuh ingested the membership event.
- Wazuh generated rule 60159.
- Analysts could correlate the events by the same account/SID.

### Not demonstrated

- A single automatic Wazuh correlation alert joining Event 4720 and Event 4728.

## Cleanup

The test account was first removed from Domain Admins.

Verification returned no matching Domain Admins membership.

The account was then deleted.

`Get-ADUser -Identity "det010.test"`

returned:

`Cannot find an object with identity: 'det010.test'`

## Assessment

DET-010 validates the complete telemetry and individual-detection path for a controlled AD privilege-escalation chain.

The exercise also reproduced the correlation limitation documented in DET-009 using a fresh test account and fresh evidence.

The appropriate next engineering step is a dedicated SIEM/query correlation layer that can normalize and join account-creation and privileged-group events on the account SID within a bounded time window.

No unsupported Wazuh correlation syntax was introduced.
