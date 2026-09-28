# DET-019 Investigation — Privileged Logon → Process Creation Correlation

## Scope

Controlled telemetry-validation exercise on DC-01 using the existing `CORP\\admin` account.

No malicious payload was executed. The process activity was benign and used only to validate the detection/correlation path.

## 1. Privileged Session Establishment

A fresh interactive `CORP\\admin` session was established on DC-01.

### Initial UAC-filtered session

Windows Security 4624:

- Logon Type: 2
- Logon ID: `0x131E2A`
- Elevated Token: No
- Workstation: DC-01
- Source Network Address: `::1`
- Process: `C:\\Windows\\System32\\svchost.exe`

The corresponding `whoami /groups` output showed:

- `CORP\\admin`
- `BUILTIN\\Administrators` — Group used for deny only
- Medium Mandatory Level

This session was not used as the privileged correlation anchor.

## 2. Elevated Session

A UAC elevation was then performed from the `CORP\\admin` context.

The resulting session showed:

- Account: `CORP\\admin`
- `BUILTIN\\Administrators` enabled
- High Mandatory Level

Windows Security 4624 established:

```
Logon Type:       2
New Logon ID:     0xAF52CC
Elevated Token:   Yes
Linked Logon ID:  0xAF52DD
Workstation:      DC-01
```

The paired filtered session showed:

```
New Logon ID:     0xAF52DD
Elevated Token:   No
Linked Logon ID:  0xAF52CC
```

This confirms the UAC-linked relationship between `0xAF52CC` and `0xAF52DD`.

## 3. 4672 Correlation

Windows Security Event 4672 was found with:

```
Subject User:     admin
Subject Domain:   CORP
Subject Logon ID: 0xAF52CC
```

Therefore:

```
4624 (0xAF52CC)
       ↓
4672 (0xAF52CC)
```

The same relationship was confirmed in Wazuh.

Wazuh rule:

- Rule ID: `67028`
- Description: `Special privileges assigned to new logon.`
- Event ID: `4672`
- Subject Logon ID: `0xaf52cc`

## 4. Process Creation

Wazuh rule `67027` provided Security Event 4688 process telemetry.

Wazuh returned three process-creation events associated with the privileged session identifier `0xaf52cc`:

| UTC timestamp | Process | New Process ID |
|---|---|---|
| 2026-09-28 05:49:46.800 | `conhost.exe` | `0xce8` |
| 2026-09-28 05:49:54.946 | `whoami.exe` | `0x1a50` |
| 2026-09-28 05:49:55.007 | `whoami.exe` | `0x16b0` |

Each event contained:

```
subjectLogonId = 0xaf52cc
subjectUserName = admin
subjectDomainName = CORP
```

This establishes a Wazuh-visible:

```
4624 → 4672 → 4688
```

relationship using the privileged Logon ID.

## 5. Sysmon Event ID 1

A benign `whoami.exe` execution was also captured directly from Sysmon.

Observed Sysmon Event ID 1:

```
UtcTime:          2026-09-28 05:52:10.004
ProcessGuid:      {939ab59d-008a-6aba-3302-000000001800}
ProcessId:        4464
Image:            C:\\Windows\\System32\\whoami.exe
CommandLine:      whoami
User:             CORP\\admin
LogonGuid:        {939ab59d-dfb3-6ab9-aa54-2c0000000000}
LogonId:          0x2C54AA
TerminalSessionId: 1
IntegrityLevel:   Medium
ParentProcessId:  2308
ParentImage:      C:\\Windows\\System32\\cmd.exe
ParentUser:       CORP\\admin
```

The matching Windows Security 4688 event identified the same process:

```
New Process ID:       0x1170
New Process Name:     C:\\Windows\\System32\\whoami.exe
Creator Process ID:   0x904
Creator Process Name: C:\\Windows\\System32\\cmd.exe
Creator Logon ID:     0x2C54AA
Token Elevation Type: TokenElevationTypeLimited (3)
```

PID conversion:

```
0x1170 = 4464
```

Therefore Security 4688 and Sysmon Event 1 independently describe the same `whoami.exe` process.

## 6. UAC / Logon-ID Boundary

The elevated Security 4624/4672 session used:

```
0xAF52CC
```

The Sysmon `whoami.exe` process used:

```
0x2C54AA
```

A search of recent Security 4624 events for `0x2C54AA` returned no 4624 event.

A broader Security search for `0x2C54AA` returned the process lifecycle events:

- 4688
- 4689

The 4688 event identified `whoami.exe` and `TokenElevationTypeLimited (3)`.

This demonstrates that a process can expose a Logon ID that is not directly represented by the privileged 4624/4672 Logon ID selected for the UAC elevation.

The test therefore does **not** support a universal rule of:

```
Security 4624 LogonId == Sysmon Event 1 LogonId
```

for UAC-filtered/elevated interactive sessions.

## 7. Wazuh Evidence

Wazuh Indexer:

```
wazuh-alerts-4.x-2026.09.28
```

Agent:

```
DC-01
```

Validated native rules:

| Event | Wazuh Rule | Description |
|---|---:|---|
| 4624 | 60118 | Windows Workstation Logon Success |
| 4672 | 67028 | Special privileges assigned to new logon |
| 4688 | 67027 | A process was created |

Wazuh preserved the privileged correlation identifier:

```
0xaf52cc
```

across the 4672 and 4688 events observed during the validation.

## 8. Correlation Assessment

### Validated

```
4624
  targetLogonId = 0xAF52CC
       ↓
4672
  subjectLogonId = 0xAF52CC
       ↓
4688
  subjectLogonId = 0xAF52CC
```

This is a viable Wazuh-side correlation path for the observed privileged process events.

### Also validated

```
Security 4688
  PID 0x1170 / LogonId 0x2C54AA
       ↕
Sysmon Event 1
  PID 4464 / LogonId 0x2C54AA
```

### Boundary

The Sysmon process-context Logon ID cannot automatically be substituted for the elevated 4624/4672 Logon ID in this UAC scenario.

## 9. Detection Status

**Telemetry validation: PASS**

**Correlation path: VALIDATED**

**Persistent custom detection: NOT IMPLEMENTED**

No custom Wazuh correlation rule was deployed during this test.

The next engineering phase is to evaluate implementation options using the observed fields, beginning with native Wazuh capabilities before introducing custom correlation logic.

## 10. ATT&CK / IOC / TTP

No malicious ATT&CK technique, IOC, IOA, or TTP is claimed from this benign telemetry-validation exercise.

The process execution was intentional and controlled.

## 11. Evidence Integrity

All findings above are based on telemetry actually observed during the controlled DET-019 exercise.

No detection performance metrics, false-positive rates, or replay results are claimed yet.
