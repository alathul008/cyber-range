# AD/Lateral-Movement Scenario — WinRM Evidence Record

## Status

**Evidence captured; scenario remains in Stage C planning/validation. No DET number assigned.**

This record documents the controlled WinRM activity performed on 2026-10-09 using ARCH-01 and WIN-01. It does not mark a new detection objective complete and does not replace DET-001 through DET-021.

## Scope and Objective

Validate whether a controlled remote Windows Remote Management (WinRM) session from the lab ATTACK zone to the lab ENTERPRISE endpoint produces observable network and Windows process telemetry in Wazuh.

- Operator: ARCH-01 — `10.10.30.100`
- Target: WIN-01 — `10.10.20.100`
- Domain: `corp.home.arpa` / `CORP`
- Transport: TCP destination port `5985` (WinRM HTTP)
- Monitoring: Wazuh on WAZUH-01; WIN-01 agent `001`

## Controlled Activity

A successful Evil-WinRM session was established from ARCH-01 to WIN-01 using `CORP\Administrator`. During the session, benign commands including `whoami` and `hostname` were executed. The session was then exited.

This was authorized activity in the isolated cyber range. The evidence below is from observed logs; it does not imply malicious intent.

## Observed Evidence

### 1. Sysmon network connection and Wazuh rule 92110

Wazuh generated native rule `92110` with description:

`Detected WinRM activity from 10.10.30.100 to 10.10.20.100`

Observed fields:

- Agent: `WIN-01` (agent ID `001`)
- Sysmon Event ID: `3`
- Event UTC time: `2026-10-09 07:00:15.404Z`
- Source: `10.10.30.100:34504`
- Destination: `10.10.20.100:5985`
- Protocol: TCP
- Initiated: false
- Process image: `System`
- Wazuh alert timestamp: `2026-10-09T07:00:37.098Z`
- Native ATT&CK metadata: `T1021.006 — Windows Remote Management` (Lateral Movement)

The rule establishes that the observed network connection matched Wazuh's WinRM detection. The technique mapping is native rule metadata; it is not, by itself, proof of malicious activity.

### 2. Windows Security privileged-logon telemetry

The test's observed Windows Security events included:

- Event ID `4672`: special privileges assigned to a new logon.
- Event ID `4688`: process creation.

The previously inspected event records showed the privileged account `CORP\Administrator` and Logon ID `0x4c69ba` for the relevant 4672/4688 sequence.

### 3. Custom rule 100107

Wazuh generated custom rule `100107`:

`Privileged logon followed by process creation on Windows endpoint.`

Two alerts were observed during the 07:00 UTC test:

- Alert timestamp: `2026-10-09T07:00:40.484Z`; Event ID 4688; process `whoami.exe`; Logon ID `0x4c69ba`; parent `wsmprovhost.exe`.
- Alert timestamp: `2026-10-09T07:00:40.505Z`; Event ID 4688; process `conhost.exe`; Logon ID `0x4c69ba`.

These records show that rule 100107 fired for process-creation events under the recorded Logon ID. The rule is process/correlation telemetry; the two alerts are not a single cross-event alert joining rule 92110 to rule 100107.

## ATT&CK Mapping

- **T1021.006 — Windows Remote Management:** present as native metadata on Wazuh rule 92110 for the observed connection to TCP 5985.
- No claim of malicious behavior is made. The session was intentionally generated as a controlled lab test.
- No additional ATT&CK technique is claimed from these records alone.

## Investigation Pivots

Use these observed pivots to reproduce the investigation:

1. Wazuh agent: `001` / `WIN-01`.
2. Source/destination: `10.10.30.100 → 10.10.20.100:5985`.
3. Sysmon Event ID 3 event time: `2026-10-09 07:00:15.404Z`.
4. Windows Security Logon ID: `0x4c69ba`.
5. Process chain: `wsmprovhost.exe → whoami.exe`, plus `conhost.exe`.
6. Wazuh rule IDs: `92110`, `100107`.

## Validation Assessment

| Check | Result | Evidence boundary |
|---|---|---|
| WinRM network telemetry | PASS | Sysmon Event ID 3 and rule 92110 observed |
| Windows process telemetry | PASS | Security Event ID 4688 observed |
| Privileged-logon/process rule | PASS for observed test | Rule 100107 fired on qualifying process events |
| Cross-rule end-to-end correlation (92110 + 100107) | NOT VALIDATED | The records are separate alerts; no single correlation joins them |
| Negative test for this scenario | NOT RECORDED HERE | Do not claim a negative validation from this test |
| Response/containment validation | NOT PERFORMED | No response metric is claimed |
| Replay metrics / false-positive rate | NOT MEASURED | No performance metric is claimed |

## Detection Gaps and Next Work

1. Review the relevant raw archived events alongside the alerts to preserve a reproducible timeline.
2. Investigate whether a useful correlation between WinRM network activity and the corresponding Windows logon/process events can be reliably established with available identifiers. Do not assume that timing alone proves a relationship.
3. Define a benign negative/control test and record its actual result before claiming specificity.
4. Decide on an alert unit and evaluate rule 100107 alert cardinality; multiple qualifying process events can produce multiple alerts.
5. Define response actions, detection improvement, replay criteria, and measurable outcomes before considering the scenario complete.

## Network Exception Note

During this test, temporary lab firewall exceptions were used to permit ARCH-01 to reach WIN-01 on TCP 5985. They are intentionally **not declared removed** in this record. Keep them scoped to the lab scenario and review them when the next scenario's required access is defined. VMnet0 remains the real-LAN bridged network and must not be used to expose vulnerable lab targets.

## Evidence Integrity

This record reflects the alert output inspected on 2026-10-09. It records only observed events and explicitly marks unperformed or unvalidated work. No new DET number is assigned.
