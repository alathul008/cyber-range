# DET-011 Correlation Method

## Purpose

This document records the implemented correlation logic used to validate the DET-011 detection against the Wazuh Indexer.

## Query Scope

The prototype queries DC-01 alerts for:

- Event ID 4720
- Event ID 4728

Only the required fields are retrieved:

- `timestamp`
- `agent.name`
- `rule.id`
- `data.win.system.eventID`
- `data.win.eventdata.targetSid`
- `data.win.eventdata.memberSid`
- `data.win.eventdata.targetUserName`
- `data.win.eventdata.memberName`

## Normalization

Event 4720 contributes:

`targetSid`

Event 4728 contributes:

`memberSid`

The prototype normalizes both into a common correlation key:

`SID`

The account creation event is represented as:

`SID = targetSid`

The privileged-group event is represented as:

`SID = memberSid`

## Temporal Correlation

For every matching SID:

`0 <= timestamp(4728) - timestamp(4720) <= 600 seconds`

This ensures the privileged-group modification occurs after the account creation and within the configured prototype window.

## Duplicate Handling

The Wazuh Indexer returned duplicate documents for the controlled DET-010 events.

The prototype deduplicates observations using the combination of:

`event type + SID + timestamp`

This produced one logical 4720 event and one logical 4728 event from the four returned DET-010 Indexer documents.

A future implementation should prefer a stronger Windows event identity, such as the Windows event record identifier, when that field is available in the retained alert document.

## Controlled Validation — DET-010

The validated DET-010 pair was:

`4720.targetSid = S-1-5-21-2519611076-441997742-1464114610-1117`

and:

`4728.memberSid = S-1-5-21-2519611076-441997742-1464114610-1117`

Observed timestamps:

- 4720: `2026-09-22T09:52:39.945Z`
- 4728: `2026-09-22T09:53:36.813Z`

Elapsed time:

`56.868 seconds`

Result:

**Correlation match detected.**

## Fresh Replay Validation — DET-011

A new controlled replay was performed on **2026-09-23** using temporary account `det011.test`.

Observed correlation inputs:

- 4720 target SID: `S-1-5-21-2519611076-441997742-1464114610-1118`
- 4728 member SID: `S-1-5-21-2519611076-441997742-1464114610-1118`
- 4720 timestamp: `2026-09-23T04:55:37.878Z`
- 4728 timestamp: `2026-09-23T04:56:04.544Z`
- Windows 4728 Record ID: `25453`
- Privileged group: `Domain Admins`
- Time delta: **26.666 seconds**

Prototype output:

- Unique 4720 events: **1**
- Unique 4728 events: **1**
- Correlation matches: **1**

Result:

**Fresh correlation match detected.**

The temporary account was removed from Domain Admins and deleted after validation.

## Security and Operational Constraints

The prototype must not contain:

- Wazuh passwords
- API credentials
- tokens
- private keys

Credentials are supplied interactively during validation and are not stored in this repository artifact.

The prototype is read-only with respect to Wazuh telemetry and does not modify Wazuh Manager rules or Indexer configuration.

## Limitations

- The 10-minute window has been validated against two controlled attack sequences, but not against a representative benign dataset.
- Duplicate handling is based on the currently observed alert schema.
- The prototype is not yet a continuously running alerting service.
- A persistent production implementation requires additional validation, error handling, credential handling, scheduling, and alert-output design.

## Validation Result

**PASS — the Indexer-side correlation logic successfully detected both controlled account-creation → Domain-Admins sequences, including a fresh replay using new telemetry.**
