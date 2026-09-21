# DET-006 Investigation

## Case

Controlled failed authentication test on WIN-01.

## Timeline

1. A single `runas` attempt was made against the local account `WIN-01\david`.
2. Windows generated Security Event ID 4625.
3. The event was confirmed locally with `Get-WinEvent`.
4. Wazuh ingested the event.
5. Wazuh generated native rule 60122.
6. The Wazuh Dashboard event was inspected and the normalized fields were verified.

## Evidence

### Windows Event 4625

Observed values:

| Field | Value |
|---|---|
| Event ID | 4625 |
| Account | david |
| Account Domain | WIN-01 |
| Logon Type | 2 |
| Status | 0xC000005E |
| Sub-status | 0x0 |
| Failure Reason | %%2304 |
| Process | C:\Windows\System32\svchost.exe |
| Source Address | ::1 |
| Workstation | DESKTOP-8PB40LJ |

### Wazuh

| Field | Value |
|---|---|
| Agent | WIN-01 |
| Agent IP | 10.10.20.100 |
| Rule ID | 60122 |
| Level | 5 |
| Description | Logon Failure - Unknown user or bad password |
| Event ID | 4625 |

The Wazuh event also exposed the normalized fields for authentication package, logon process, target account, target domain, status, process ID, process name, workstation name, and source address.

## IOC / IOA / TTP assessment

### IOC

No malicious IOC was established.

### IOA

A failed authentication attempt is an authentication-related indicator, but this single controlled event does not establish malicious activity.

### TTP

No ATT&CK technique is assigned by this lab based solely on this test.

Wazuh's native rule 60122 reports an ATT&CK mapping of T1531 (Account Access Removal). That is the native rule's metadata and is not treated as a conclusion about this controlled event.

## Analyst assessment

The event confirms that the endpoint and Wazuh pipeline can capture failed-logon telemetry.

The specific failure returned status 0xC000005E and Windows reported that the domain was unavailable during the attempted authentication. Therefore this test should not be described as a confirmed bad-password attempt.

## Detection engineering decision

No custom DET-006 rule was created.

Reason: Wazuh already detects Event ID 4625 with rule 60122. A custom rule matching only Event ID 4625 would provide duplicate coverage without increasing detection value.

## Detection gap / future improvement

A more useful authentication detection should add behavioral context rather than duplicate the base event.

Potential future correlation:

- multiple 4625 events from one source
- multiple targeted accounts
- privileged account failures
- 4625 followed by 4624
- abnormal logon type
- unusual source address
- distributed failures across endpoints

These are future detection-engineering scenarios, not findings from this test.

## Result

DET-006 validation complete with native Wazuh coverage.

No custom rule was deployed.
