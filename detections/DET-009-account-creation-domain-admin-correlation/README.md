# DET-009 — Account Creation to Domain Admins Correlation

## Objective

Validate whether an account created in Active Directory and subsequently added to the Domain Admins group can be correlated across the two Windows Security events.

## Scenario

A temporary domain account was created on DC-01 and then added to Domain Admins.

### Test account

- Account: DET009-TestUser
- Agent: DC-01
- Account SID: S-1-5-21-2519611076-441997742-1464114610-1116

### Events observed

1. **Event ID 4720 — A user account was created**
   - Wazuh rule: 100104
   - Description: Local user account created on Windows endpoint.
   - Actor: CORP\\Administrator
   - Target: DET009-TestUser
   - Event record ID: 19737
   - Timestamp: 2026-09-21T09:36:43.0615242Z

2. **Event ID 4728 — A member was added to a security-enabled global group**
   - Wazuh rule: 60159
   - Description: Domain Admins Group Changed
   - Actor: CORP\\Administrator
   - Target group: Domain Admins
   - Member: DET009-TestUser
   - Member SID: S-1-5-21-2519611076-441997742-1464114610-1116
   - Event record ID: 19766
   - Timestamp: 2026-09-21T09:37:41.5198365Z

The two events occurred approximately 59 seconds apart and shared the same account SID.

## Detection Result

Wazuh successfully detected both individual events.

The investigation then evaluated the Wazuh rule-engine correlation primitives available in the installed ruleset. The validated examples of `same_field` correlate repeated events using the same parsed field name, such as `win.eventdata.targetUserName` or `win.eventdata.ipAddress`.

For this scenario, the relevant identifiers are exposed under different fields:

- Event 4720: `win.eventdata.targetSid`
- Event 4728: `win.eventdata.memberSid`

No supported cross-field correlation pattern was demonstrated in the installed ruleset during this exercise.

Therefore, **no custom Wazuh correlation rule was deployed**.

## Detection Gap

The individual telemetry exists and is detected, but the current Wazuh rule-engine approach validated in this lab does not provide a demonstrated direct correlation from:

`4720.targetSid` → `4728.memberSid`

This is recorded as a detection-engineering gap rather than being hidden with an unverified rule.

A future SIEM/query correlation layer can address this by joining the two events on the account SID within an appropriate time window.

## ATT&CK Context

The underlying behaviors are associated with account creation and privileged-group modification. The individual Wazuh detections provide the telemetry and native ATT&CK metadata used during the exercise.

This exercise focuses on correlation capability rather than asserting that the controlled test itself was malicious.

## Cleanup

After evidence collection:

- DET009-TestUser was removed from the lab.
- `Get-ADUser "DET009-TestUser"` returned an object-not-found result.
- Wazuh configuration validation returned no errors with `wazuh-analysisd -t`.

## Validation

- [x] Account creation generated Event 4720
- [x] Domain Admins membership generated Event 4728
- [x] Same account SID linked the events
- [x] Both events reached Wazuh
- [x] Correlation capability investigated
- [x] Unsupported correlation rule avoided
- [x] Test account cleaned up
- [x] Wazuh configuration validated

## Status

**Complete — correlation gap documented.**
