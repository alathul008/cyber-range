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

The 10-minute value is a prototype design parameter. It has now been validated against both the controlled DET-010 sequence and a fresh DET-011 replay, but has not yet been tuned against a larger benign-event dataset.

## Data Source

The prototype queries the Wazuh Indexer alert index.

For the fresh replay, the target index was:

`wazuh-alerts-4.x-2026.09.23`

The live Indexer API was verified on WAZUH-01 at `10.10.40.10:9200`.

Wazuh Manager and Wazuh Indexer were both confirmed active before validation.

## Deduplication

The Indexer query can return duplicate documents for controlled events.

The prototype therefore normalizes events by:

- event type
- correlation SID
- timestamp

This prevents duplicate Indexer documents from producing multiple apparent correlations from the same underlying Windows events.

The retained DET-010 validation returned:

- Indexer documents: **4**
- Unique 4720 events: **1**
- Unique 4728 events: **1**
- Correlation matches: **1**

## Validation — DET-010 Evidence

The prototype was first executed against retained DET-010 telemetry.

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

## Fresh Replay — DET-011 Validation

A new controlled replay was executed on **2026-09-23** using a new temporary account.

### Attack chain

1. Created `det011.test` on DC-01.
2. Added `det011.test` to `Domain Admins`.
3. Confirmed the corresponding Windows Security events.
4. Confirmed Wazuh alert ingestion in the 2026-09-23 alert index.
5. Executed the Indexer-side correlation prototype.
6. Confirmed a single correlation match.
7. Removed the account from Domain Admins and deleted the account.

### Observed Windows telemetry

| Event | Evidence |
|---|---|
| 4720 | Account created; target SID ended in `-1118` |
| 4722 | Account enabled; target SID ended in `-1118` |
| 4738 | Account-change telemetry observed for SID `...-1118` |
| 4728 | `det011.test` added to `Domain Admins`; member SID ended in `-1118` |
| 4728 Windows Record ID | `25453` |

The Windows Security log independently confirmed Event 4728 before correlation testing.

### Wazuh telemetry

The fresh 4720 alert was observed in:

`wazuh-alerts-4.x-2026.09.23`

with Wazuh rule **100104**.

The fresh 4728 alert was observed with Wazuh rule **60159**, level **12**, description **Domain Admins Group Changed**. The event contained:

- `memberSid = S-1-5-21-2519611076-441997742-1464114610-1118`
- `memberName = CN=DET011-TestUser,CN=Users,DC=corp,DC=home,DC=arpa`
- `targetUserName = Domain Admins`
- Windows event record ID `25453`

### Correlation result

The prototype was retargeted to the 2026-09-23 alert index and executed against the fresh telemetry.

Observed result:

| Field | Observed value |
|---|---|
| Account | `det011.test` |
| Account SID | `S-1-5-21-2519611076-441997742-1464114610-1118` |
| 4720 timestamp | `2026-09-23T04:55:37.878Z` |
| 4728 timestamp | `2026-09-23T04:56:04.544Z` |
| Time delta | **26.666 seconds** |
| Privileged group | `Domain Admins` |
| Unique 4720 events | **1** |
| Unique 4728 events | **1** |
| Correlation matches | **1** |
| Result | **Account creation followed by Domain Admins membership detected** |

The temporary account was subsequently removed from Domain Admins and deleted. Verification after cleanup confirmed that `det011.test` no longer existed.

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

The fresh replay independently validated that correlation against new telemetry.

## ATT&CK Context

The underlying exercise involves account creation and privileged-group membership modification.

The native Wazuh mappings observed during validation were:

- Event 4720 / rule 100104: Wazuh mapped the event to **T1136.001 / Local Account**.
- Event 4728 / rule 60159: Wazuh mapped the event to **T1484 / Domain Policy Modification**.

The 4720 native description/mapping was observed to label the DC-01 domain-account creation event as a local account event. That observation is retained rather than silently corrected.

## Current Status

**Fresh replay validated.**

The correlation logic is validated against both retained DET-010 evidence and a new DET-011 replay.

This detection currently exists as a query/prototype rather than a continuously running production-style correlation service.

## Next Engineering Steps

1. Test benign account/group activity to measure false-positive behavior.
2. Formalize the deduplication strategy using Windows event identity where available.
3. Decide where persistent correlation should execute.
4. Add an alert/output mechanism only after replay validation.
5. Document detection performance and limitations from actual measurements.
