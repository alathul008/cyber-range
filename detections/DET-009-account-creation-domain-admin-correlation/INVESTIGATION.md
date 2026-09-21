# DET-009 Investigation

## Evidence Chain

The controlled test created the domain account `DET009-TestUser` on DC-01 and then added it to Domain Admins.

The account SID was:

`S-1-5-21-2519611076-441997742-1464114610-1116`

### Event 4720

Wazuh received the account-creation event from DC-01.

- Rule: 100104
- Level: 10
- Target user: DET009-TestUser
- Target SID: S-1-5-21-2519611076-441997742-1464114610-1116
- Actor: CORP\\Administrator
- Event record ID: 19737
- System time: 2026-09-21T09:36:43.0615242Z

### Event 4728

Wazuh received the Domain Admins membership-change event.

- Rule: 60159
- Level: 12
- Target group: Domain Admins
- Member: DET009-TestUser
- Member SID: S-1-5-21-2519611076-441997742-1464114610-1116
- Actor: CORP\\Administrator
- Event record ID: 19766
- System time: 2026-09-21T09:37:41.5198365Z

## Correlation Analysis

The same SID appears in both events, providing the required relationship for a correlation use case.

The Wazuh ruleset was inspected for supported multi-event correlation primitives. Existing examples use `frequency`, `timeframe`, `if_matched_sid`, and `same_field`.

The relevant `same_field` examples use the same field path across matched events. This scenario requires comparing different field paths:

- `win.eventdata.targetSid`
- `win.eventdata.memberSid`

No validated cross-field rule-engine mechanism was found during this exercise.

## Decision

No custom Wazuh correlation rule was deployed.

This avoids claiming support for a rule-engine construction that was not demonstrated on the installed Wazuh version/ruleset.

The appropriate follow-on is a SIEM/query correlation implementation that can normalize and join the two event types on the account SID and a bounded time window.

## Cleanup

The temporary account was removed after evidence collection.

Verification:

`Get-ADUser "DET009-TestUser"`

returned an object-not-found result.

Wazuh syntax validation:

`sudo /var/ossec/bin/wazuh-analysisd -t`

returned no output/errors.

## Assessment

This exercise demonstrated a useful distinction between:

1. **Telemetry availability** — both Windows events were collected.
2. **Individual detection** — Wazuh detected both events.
3. **Correlation** — the relationship between the two events requires an additional correlation layer.

The gap is therefore documented as a detection-engineering finding rather than treated as a failed telemetry pipeline.
