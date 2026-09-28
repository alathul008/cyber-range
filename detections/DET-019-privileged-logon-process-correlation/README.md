# DET-019 — Privileged Logon → Process Creation Correlation

**Status:** EXECUTED / TELEMETRY VALIDATED / DETECTION IMPLEMENTED / TUNING OBSERVED

## Objective

Investigate whether a privileged interactive logon is followed by process creation within the same Windows logon session, using stable session identifiers where the available telemetry supports correlation.

The controlled test extended the validated DET-018 chain:

```
4624 Successful Logon
        ↓
4672 Special Privileges
        ↓
4688 Process Creation
        ↓
Sysmon Event ID 1
        ↓
Wazuh
        ↓
Logon ID / process correlation
```

## Environment

- DC-01 — Windows Server 2025 Active Directory domain controller
- WAZUH-01 — Ubuntu/Wazuh security server
- DC-01 Wazuh agent: 002
- Sysmon installed on DC-01
- DC-01 Sysmon/Operational telemetry collected by Wazuh
- AD domain: `corp.home.arpa`
- Domain NetBIOS name: `CORP`
- Controlled account: `CORP\\admin`

## Test Execution

A fresh `CORP\\admin` interactive session was established on DC-01.

The first session was UAC-filtered:

- Logon Type: 2
- Logon ID: `0x131E2A`
- Elevated Token: No
- Integrity: Medium

A subsequent UAC elevation produced the privileged session used for the main correlation:

- Logon ID: `0xAF52CC`
- Elevated Token: Yes
- Linked Logon ID: `0xAF52DD`
- Workstation: DC-01

The linked-session relationship was independently confirmed from Security 4624 events.

## Validated Telemetry

### 4624 → 4672

The elevated session produced:

```
4624
  targetLogonId = 0xAF52CC
  logonType = 2
  elevatedToken = Yes

4672
  subjectLogonId = 0xAF52CC
```

This validates the privileged-session relationship using the Windows Logon ID.

### 4688 Process Creation

Wazuh preserved process-creation telemetry for the same privileged Logon ID:

```
4688
  subjectLogonId = 0xAF52CC
  subjectUserName = admin
  subjectDomainName = CORP
```

Observed processes included:

- `C:\\Windows\\System32\\conhost.exe`
- `C:\\Windows\\System32\\whoami.exe`

The `whoami.exe` events were associated with `subjectLogonId = 0xAF52CC`.

### Sysmon Event ID 1

A separate Sysmon Event ID 1 for `whoami.exe` was observed with:

```
User: CORP\\admin
LogonId: 0x2C54AA
IntegrityLevel: Medium
ParentImage: C:\\Windows\\System32\\cmd.exe
```

The corresponding Security 4688 event used:

```
subjectLogonId = 0x2C54AA
newProcessId = 0x1170
newProcessName = C:\\Windows\\System32\\whoami.exe
```

The Sysmon PID was `4464`, which equals Security 4688 PID `0x1170`.

This independently validates Security 4688 ↔ Sysmon Event 1 process correlation.

## Correlation Finding

The controlled test demonstrated two distinct but related correlation paths:

```
4624
  ↓ same LogonId
4672
  ↓ same privileged LogonId
4688
  ↓ process telemetry
Wazuh
```

For the privileged session, `0xAF52CC` was preserved by Wazuh across 4672 and 4688.

Separately, Sysmon Event 1 and Security 4688 correlated through the process identity and `LogonId = 0x2C54AA`.

### UAC/session boundary

The elevated 4624/4672 Logon ID `0xAF52CC` must not be assumed to equal every Sysmon process Logon ID. The observed Sysmon `whoami.exe` event used `0x2C54AA`, and no 4624 event with `0x2C54AA` was found in the searched Security events.

Therefore, a simplistic rule requiring:

```
4624.LogonId == Sysmon.Event1.LogonId
```

would not be sufficient for this UAC scenario.

## Wazuh Validation

Wazuh Indexer queries against `wazuh-alerts-4.x-2026.09.28` confirmed:

- 4624 native rule 60118: Windows Workstation Logon Success
- 4672 native rule 67028: Special privileges assigned to new logon
- 4688 native rule 67027: A process was created
- Wazuh preserved `targetLogonId` / `subjectLogonId` fields required for correlation.

The validated privileged correlation key was:

```
0xaf52cc
```

## Detection Status

**Telemetry validation: PASS**

**Correlation design: VALIDATED**

**Persistent custom detection: IMPLEMENTED**

Native Wazuh rule `100107` was deployed and validated after configuration testing.

Rule logic:

```xml
<rule id="100107" level="12" frequency="2" timeframe="60">
  <if_matched_sid>67028</if_matched_sid>
  <same_field>win.eventdata.subjectLogonId</same_field>
  <if_sid>67027</if_sid>
</rule>
```

The rule correlates a 4672 privileged-logon event with subsequent 4688 process-creation events sharing the same `subjectLogonId` within the 60-second correlation window.

Configuration validation with `wazuh-analysisd -t` passed, and the Wazuh manager restarted successfully with the rule loaded.

### Positive validation

A fresh controlled elevated `CORP\\admin` session generated three `100107` alerts using Logon ID `0xe25141`:

- Event Record ID `34958` — `conhost.exe`
- Event Record ID `34961` — `whoami.exe`
- Event Record ID `34963` — `whoami.exe`

The three alerts occurred within approximately 1.1 seconds and demonstrate that the native correlation path is functioning.

### Alert cardinality observation

The rule currently generates a correlation alert for each qualifying 4688 event that follows the matched 4672 condition within the correlation window. The controlled test therefore produced three alerts for the same privileged session.

This is documented as a tuning observation, not hidden as a successful single-alert-per-session metric.

### Negative validation

A subsequent ordinary `whoami` execution did not produce an additional `100107` alert.

### Current tuning status

**Detection logic: VALIDATED**

**Alert cardinality tuning: OPEN**

No claim is made that the rule currently provides one alert per privileged session.

## MITRE ATT&CK

**Status: TBD**

No ATT&CK technique is assigned solely from this benign validation process. The observed activity was administrative process execution used to validate telemetry.

## IOC / IOA / TTP

No malicious IOC is claimed.

Observed validation artifacts include:

- Account: `CORP\\admin`
- Logon IDs: `0xAF52CC`, `0xAF52DD`, `0x2C54AA`
- Process: `whoami.exe`
- Parent: `cmd.exe`

These are test artifacts, not malicious indicators.

## Detection Gap

The test identified a UAC/session-context boundary:

- Security 4624/4672 privileged session: `0xAF52CC`
- Sysmon/Security process context observed separately: `0x2C54AA`

Therefore, direct Security-to-Sysmon LogonId equality cannot be treated as universally reliable.

At the same time, Wazuh preserved the privileged Logon ID directly across 4672 and 4688, providing a viable correlation path for the observed elevated process events.

## Replay

The controlled positive and negative validation sequence was executed after rule deployment.

A formal repeatability/replay metric has not yet been established.

## Metrics

Observed validation result:

- Positive correlation: 3 `100107` alerts from 3 qualifying 4688 events in one controlled privileged session.
- Negative test: no additional `100107` alert from the subsequent ordinary `whoami` execution.
- Timeframe: 60 seconds.
- Rule frequency: 2.

These are validation observations, not production performance metrics.

## Evidence

Detailed evidence is recorded in:

`INVESTIGATION.md`

No screenshots or unobserved results are claimed.

## Status

**CURRENT / TELEMETRY VALIDATED / DETECTION IMPLEMENTED / TUNING OPEN**
