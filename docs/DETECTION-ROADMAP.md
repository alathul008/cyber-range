# Detection Roadmap

Canonical recovery of the Cyber Range detection-engineering sequence.

> Scope: DET-001 through DET-018 completed work recovered from project conversation/context and available artifacts. Where the exact historical detail is not fully recoverable, it is explicitly marked rather than inferred.

## Phase 2 — AD + Endpoint Telemetry + Detection Engineering

| DET | Title | Status | Recovered evidence / lesson |
|---|---|---|---|
| DET-001 | PowerShell encoded command | COMPLETED | Custom Wazuh rule 100101; T1059.001. Actual rule/telemetry validated in project history. |
| DET-002 | CMD → PowerShell parent-child execution | COMPLETED | Custom Wazuh rule 100102; T1059.001/T1059.003. |
| DET-003 | Local account creation | COMPLETED | Custom Wazuh rule 100104; Windows account-management telemetry; T1136.001. |
| DET-004 | Scheduled task creation | COMPLETED | Scheduled-task creation telemetry; T1053.005. |
| DET-005 | Windows service creation / LocalSystem execution | COMPLETED | Service-creation telemetry and custom rule coverage; T1543.003. |
| DET-006 | Failed logon | COMPLETED | Native Wazuh authentication-failure coverage; later expanded by DET-016. |
| DET-007 | Local privileged group membership | COMPLETED | Native Windows/Wazuh privileged-group telemetry. |
| DET-008 | Domain Admins group change | COMPLETED | Native Wazuh rule 60159; Event 4728; T1484 native mapping. |
| DET-009 | Account creation → Domain Admin privilege correlation exercise | COMPLETED | Deliberately documented Wazuh correlation-engineering limitation. Detection ≠ correlation. |
| DET-010 | Account creation → Domain Admins | COMPLETED | Controlled det010.test. Events 4720 + 4728 observed and manually correlated by SID within 600s. |
| DET-011 | Account creation → Domain Admin correlation + benign negative | COMPLETED | det011.test correlated successfully; benign det011.fp produced no correlation. Read-only Python Indexer correlation prototype validated. |
| DET-012 | PowerShell process telemetry | COMPLETED | Sysmon Event 1 + Wazuh rule 92027. Sysmon Event 11 also observed for PowerShell temp script. Existing custom PowerShell rules remained silent, establishing a coverage boundary for PowerShell → PowerShell. |
| DET-013 | Scheduled task → cmd.exe execution | COMPLETED | Targeted audit enabled. Event 4698 + Wazuh custom rule 100105 validated; T1053.005/T1059.003. |
| DET-014 | Privileged logon telemetry | COMPLETED | Event 4624 + 4672 correlated by Logon ID. Native Wazuh rules 60106/67028. Documented caveat: 4672 does not itself prove Domain Policy Modification. |
| DET-015 | Account lockout | COMPLETED | Event 4740 + Wazuh rule 60115. Domain lockout baseline permanently set to threshold 5, duration 10m, observation 10m. |
| DET-016 | Failed-logon investigation | COMPLETED | Event 4625 + Wazuh rule 60122. Controlled local bad-password test; documented ::1/local-source limitation. |
| DET-017 | Successful-logon investigation | COMPLETED | Event 4624 + Wazuh rule 60118. Controlled local interactive logon; pivoted by Logon ID. Documented ::1/local-source caveat. |
| DET-018 | Successful logon → special-privilege telemetry | COMPLETED | Event 4624 and 4672 correlated by Logon ID 0x2c5499. Windows 4672 Record ID 33066 and Wazuh rule 67028 validated. Native T1484 mapping is metadata, not proof of Domain Policy Modification. |
| DET-019 | Privileged Logon → Process Creation Correlation | COMPLETED | Native Wazuh rule 100107 validated with 4672 → 4688 correlation by subjectLogonId. Positive and negative validation completed; alert-cardinality tuning remains open. |
| DET-020 | PowerShell Process → Network Connection Correlation | COMPLETED | Native Wazuh correlation rule 100108 validated using 92027 → 92101 with same win.eventdata.processGuid. Positive and negative validation completed. |
| DET-021 | PowerShell Download Chain Telemetry Boundary | COMPLETED | Controlled PowerShell HTTP download validated Event 1 → Event 3 correlation and Wazuh 100108 ingestion. Downloaded file was confirmed on disk, but Event 11 for that exact file and same ProcessGuid was not observed; no three-stage rule created. |

## DET-010 Evidence

- Snapshot: PHASE2-DET010-PREATTACK-DC01
- Snapshot: PHASE2-DET010-PREATTACK-WIN01
- Test account: det010.test
- 4720 observed, followed by 4728 Domain Admins membership.
- Correlation was initially manual by target/member SID because native Wazuh direct cross-field correlation was unavailable.
- Account was cleaned up after the test.

## DET-011 Evidence

- Test account: det011.test
- SID: S-1-5-21-2519611076-441997742-1464114610-1118
- 4720: 2026-09-23T04:55:37.878Z
- 4728: 2026-09-23T04:56:04.544Z
- Delta: 26.666 seconds
- Windows Record ID: 25453
- Correlation result: 1
- Benign negative: det011.fp, SID ...-1120, correlation result 0
- Python prototype: /root/det010-correlation.py
- Prototype is read-only Indexer correlation, not persistent native Wazuh correlation.

## DET-012 Evidence

- Sysmon Event 1 validated.
- PID 6424.
- Process: powershell.exe
- User: CORP\admin
- Integrity: High
- ProcessGuid: {939ab59d-7551-6ab3-7002-000000001500}
- Parent PID: 6868
- Wazuh rule 92027, level 4.
- Sysmon Event 11 matched the same ProcessGuid and recorded a temporary PowerShell policy-test file under the user's Temp path.
- Wazuh rule 92213, level 15.
- Existing custom rules 100101–100103 did not fire for this PowerShell → PowerShell behavior.

## DET-013 Evidence

- Snapshot: PHASE2-DET013-PREATTACK-DC01
- Targeted audit: Other Object Access Events / Success.
- Event 4698 observed for DET013-TestTask.
- Wazuh rule 100105, level 12.
- ATT&CK: T1053.005 and T1059.003.
- Event time: 2026-09-27T04:10:18.9210189Z.
- Wazuh alert timestamp: 2026-09-27T04:11:04.966+0000.
- Task content contained cmd.exe and the controlled echo command.

## DET-014 Evidence

- Event 4624 and matching 4672 shared Logon ID 0x3F868B.
- Account: CORP\admin.
- Logon Type: 2.
- Elevated Token: Yes.
- Wazuh rule 67028 generated for Event 4672.
- Rule 67028 is located in /var/ossec/ruleset/rules/0955-WEF-baseline_rules.xml.
- Native rule 67028 maps 4672 to T1484; this mapping is treated as vendor metadata, not proof of Domain Policy Modification.

## DET-015 Evidence

- Test account: DET008-TestUser.
- Five controlled incorrect-password attempts caused lockout.
- Windows Event 4740 observed.
- Wazuh rule 60115, level 9.
- Account was unlocked and verified after testing.
- Permanent domain baseline: LockoutThreshold 5; LockoutDuration 10 minutes; LockoutObservationWindow 10 minutes.

## DET-016 Evidence

- One controlled incorrect-password attempt against DET008-TestUser.
- Windows Event 4625.
- Status 0xC000006D; SubStatus 0xC000006A.
- Logon Type 2.
- Source ::1; Workstation DC-01.
- Wazuh rule 60122, level 5.
- Investigation explicitly does not claim remote brute force.

## DET-017 Evidence

- Controlled local interactive logon using CORP\admin.
- Windows Event 4624.
- Logon Type 2.
- Logon ID 0x2C5499.
- Source ::1; Workstation DC-01.
- Wazuh rule 60118.
- Wazuh pivot by targetLogonId=0x2c5499 returned the matching event.

## DET-018 Evidence

- Controlled local interactive logon using CORP\admin.
- Windows Event 4624 and Event 4672 share Logon ID 0x2c5499.
- Event 4672: 2026-09-28 09:02:03; Record ID 33066.
- Wazuh index: wazuh-alerts-4.x-2026.09.28.
- Wazuh rule 67028.
- Wazuh retained the 4672 event and preserved the same Logon ID.
- Source ::1 indicates local controlled activity on DC-01.
- Finding is privileged-logon telemetry; no maliciousness or remote-access claim.

## DET-020 Evidence

- Wazuh custom rule: 100108, level 12.
- Parent/process rule: 92027.
- Network base rule: 92101, level 0.
- Correlation field: win.eventdata.processGuid.
- Positive validation: one 100108 alert after controlled PowerShell HTTP activity.
- Positive Event 3 Record ID: 3831499.
- Negative validation: curl.exe produced no additional 100108 alert; total remained 1.
- Example Sysmon Event 1 → Event 3 delta: 4.851 seconds.
- Rollback backup: /var/ossec/etc/rules/local_rules.xml.det020-prechange.

## Detection Engineering Lessons

1. Detection ≠ correlation. DET-009 remains the baseline lesson.
2. Prefer validated native Wazuh telemetry before adding custom rules.
3. Test actual telemetry instead of assuming an ATT&CK mapping proves coverage.
4. Correlate Windows security events using stable identifiers such as Logon ID or SID where appropriate.
5. Document vendor/native ATT&CK mappings separately from what the evidence actually demonstrates.
6. Preserve negative/benign tests where performed.
7. Do not claim remote activity when controlled tests use ::1/local sources.
8. Treat permanent baseline changes separately from temporary test configuration.

## Post-DET-021 Sequencing Decision

DET-001 through DET-021 are complete. The roadmap does not explicitly name the next scenario after DET-021. The target architecture places the project in **Stage C — Detection Depth**, with **Stage D — Network Visibility** subsequent to Stage C.

Therefore, the sequencing decision is to remain in Stage C and next evaluate/build an **AD/lateral-movement detection and investigation scenario** using the existing DC-01, WIN-01, ARCH-01, and Wazuh foundation. This is intentionally not assigned a DET number yet.

Rationale: AD/lateral movement is directly aligned with the existing enterprise identity/endpoint foundation and Stage C detection-depth objective, while network visibility (Zeek/Suricata) is explicitly Stage D. No VMware, pfSense, Zeek, or Suricata change is authorized by this decision.

## Current State

- DET-001 → DET-020: COMPLETED
- DET-019: COMPLETED / DETECTION IMPLEMENTED / TUNING OPEN
- DET-020: COMPLETED / DETECTION IMPLEMENTED / POSITIVE + NEGATIVE VALIDATED
- DET-021: COMPLETED / TELEMETRY INVESTIGATION / BOUNDARY DOCUMENTED
- Current phase: Phase 2 — AD + Endpoint Telemetry + Detection Engineering
- DET-020 established ProcessGuid-based PowerShell process → network correlation in Wazuh.
- DET-021 validated the PowerShell → network leg and documented the unverified downloaded-file Event 11 boundary.
- No malicious ATT&CK claim is made from the benign validation activity.

## Recovery Rule

This roadmap records only details supported by available project evidence. Missing historical details are not replaced with generic scenarios.
