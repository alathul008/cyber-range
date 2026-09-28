# DET-019 — Privileged Logon → Process Creation Correlation

**Status:** EXECUTED / TELEMETRY VALIDATED / DETECTION NOT YET IMPLEMENTED

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

**Persistent custom detection: NOT YET IMPLEMENTED**

The next engineering step is to determine whether the validated `4624 → 4672 → 4688` relationship can be implemented reliably using native Wazuh capabilities or requires a custom correlation mechanism.

No unsupported Wazuh correlation syntax is claimed.

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

Not yet performed.

## Metrics

Not yet available.

## Evidence

Detailed evidence is recorded in:

`INVESTIGATION.md`

No screenshots or unobserved results are claimed.

## Status

**CURRENT / TELEMETRY VALIDATED / DETECTION IMPLEMENTATION PENDING**
