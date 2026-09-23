# DET-011 — Account Creation to Domain Admins Correlation

## Objective

Promote the correlation gap identified in DET-009 and reproduced in DET-010 into a repeatable Indexer-side correlation prototype.

The detection identifies a sequence where:

`Windows Event 4720 account creation → Windows Event 4728 Domain Admins membership`

The events are correlated using the account SID and a bounded time window.

## Detection Logic

### Event 1 — Account creation

Windows Security Event ID **4720**:

- Correlation field: `data.win.eventdata.targetSid`
- Account field: `data.win.eventdata.targetUserName`
- DC: `DC-01`

### Event 2 — Domain Admins membership

Windows Security Event ID **4728**:

- Correlation field: `data.win.eventdata.memberSid`
- Group field: `data.win.eventdata.targetUserName`
- DC: `DC-01`

### Correlation condition

A match is produced when:

`4720.targetSid == 4728.memberSid`

and:

- Event 4728 occurs after Event 4720.
- The elapsed time is no greater than **600 seconds (10 minutes)**.
- Both events originate from DC-01.

The 10-minute value is a prototype design parameter. It has been validated against the controlled DET-010 sequence but has not yet been tuned against a larger benign-event dataset.

## Data Source

The prototype queries the Wazuh Indexer alert index:

`wazuh-alerts-4.x-2026.09.22`

The live Indexer API was verified on WAZUH-01 at `10.10.40.10:9200`.

Wazuh Manager and Wazuh Indexer were both confirmed active before validation.

## Deduplication

The Indexer query returned duplicate documents for the controlled 4720 and 4728 events.

The prototype therefore normalizes events by:

- event type
- correlation SID
- timestamp

This prevented duplicate Indexer documents from producing four apparent correlations from the two underlying Windows events.

Observed validation:

- Indexer documents returned: **4**
- Unique 4720 events: **1**
- Unique 4728 events: **1**
- Correlation matches: **1**

## Validation — DET-010 Evidence

The prototype was executed against the retained DET-010 telemetry.

Observed values:

| Field | Observed value |
|---|---|
| Account | `det010.test` |
| Account SID | `S-1-5-21-2519611076-441997742-1464114610-1117` |
| Event 4720 | `2026-09-22T09:52:39.945Z` |
| Event 4728 | `2026-09-22T09:53:36.813Z` |
| Time delta | **56.868 seconds** |
| Privileged group | `Domain Admins` |
| Correlation result | **1 match** |

The test account was subsequently removed from Domain Admins and deleted as documented in DET-010.

## Detection Engineering Progression

`DET-009`

Individual detections worked, but a supported direct Wazuh rule-engine cross-field join was not demonstrated.

↓

`DET-010`

The complete attack chain was reproduced with fresh telemetry and the correlation gap was confirmed.

↓

`DET-011`

A read-only Indexer/query correlation layer successfully joined:

`4720.targetSid → 4728.memberSid`

within a bounded time window.

## ATT&CK Context

The underlying exercise involves account creation and privileged-group membership modification.

The native Wazuh mappings observed during DET-010 were:

- Event 4720 / rule 100104: Wazuh mapped the event to **T1136.001 / Local Account**.
- Event 4728 / rule 60159: Wazuh mapped the event to **T1484 / Domain Policy Modification**.

The 4720 native description/mapping was observed to label the DC-01 domain-account creation event as a local account event. That observation is retained rather than silently corrected.

## Current Status

**Prototype validated.**

This detection currently exists as a query/prototype rather than a continuously running production-style correlation service.

## Next Engineering Steps

1. Validate the prototype against a fresh replay rather than only retained DET-010 evidence.
2. Test benign account/group activity to measure false-positive behavior.
3. Formalize the deduplication strategy using Windows event identity where available.
4. Decide where persistent correlation should execute.
5. Add an alert/output mechanism only after replay validation.
6. Document detection performance and limitations from actual measurements.
