# Cyber Range — Project State

## Current Phase
**Phase 2 — AD + Endpoint Telemetry + Detection Engineering**

## Current Objective
Stage C sequencing is now documented after DET-021. The next capability is an AD/lateral-movement detection and investigation scenario using the existing architecture; no DET-022 has been created.

## Current DET
**DET-021 — PowerShell Download Chain Telemetry Boundary**

**Status: COMPLETED / TELEMETRY BOUNDARY DOCUMENTED**

DET-021 controlled benign PowerShell HTTP download testing is complete. Event 1 → Event 3 correlation was validated through ProcessGuid and existing Wazuh rule 100108. The downloaded file was confirmed on disk, but Sysmon Event 11 for that exact file and ProcessGuid was not observed. No new three-stage Wazuh rule was created.

Conceptual chain:

```
4624 Successful Logon
        ↓
4672 Special Privileges
        ↓
Sysmon Event ID 1
        ↓
Process execution
        ↓
Wazuh
        ↓
Session correlation
        ↓
Investigation
```

ATT&CK mapping must be determined from actual observed behavior. No technique is pre-claimed.

## Completed DETs
**DET-001 → DET-021: COMPLETED**

## Documentation Audit
Live GitHub repository state was revalidated on 2026-10-05.

| DET | README | Status |
|---|---|---|
| DET-001 | Present | COMPLETED |
| DET-002 | Present | COMPLETED |
| DET-003 | Present | COMPLETED |
| DET-004 | Present | COMPLETED |
| DET-005 | Present | COMPLETED |
| DET-006 | Present | COMPLETED |
| DET-007 | Present | COMPLETED |
| DET-008 | Present | COMPLETED |
| DET-009 | Present | COMPLETED |
| DET-010 | Present | COMPLETED |
| DET-011 | Present | COMPLETED |
| DET-012 | Present | COMPLETED |
| DET-013 | Present | COMPLETED |
| DET-014 | Present | COMPLETED |
| DET-015 | Present | COMPLETED |
| DET-016 | Present | COMPLETED |
| DET-017 | Present | COMPLETED |
| DET-018 | Present | COMPLETED |
| DET-019 | Present in live history | COMPLETED |
| DET-020 | Present in live history | COMPLETED |
| DET-021 | Present in live history | COMPLETED |

**Audit result: no individual README is missing for DET-001 through DET-018 in the current GitHub tree.**

The previously suspected DET-004 through DET-008 gap is therefore already resolved in the live repository. Their existing READMEs were inspected and were not recreated.

## Infrastructure State
- Host: Windows 11
- CPU: AMD Ryzen 7 7435HS
- RAM: 16 GB currently
- GPU: NVIDIA GeForce RTX 3050 Laptop GPU
- VMware Workstation Pro
- ARCH-01: primary attack/operator VM
- WIN-01: enterprise Windows endpoint
- DC-01: Windows Server 2025 domain controller
- WAZUH-01: Ubuntu/Wazuh security server

Target architecture remains 32-GB-class; current memory constraints are handled through VM operational profiles.

## Network State
- VMnet0: Bridged to real LAN through TP-Link Wireless USB Adapter.
- VMnet1: legacy host-only 192.168.195.0/24.
- VMnet8: NAT 192.168.150.0/24.
- VMnet2: MGMT 10.10.10.0/24.
- VMnet3: ENTERPRISE 10.10.20.0/24.
- VMnet4: ATTACK 10.10.30.0/24.
- VMnet5: SECURITY/SOC 10.10.40.0/24.
- pfSense FW-01 uses .1 on lab segments.
- Host virtual adapters use .254 on lab host-only segments.
- ARCH-01: 10.10.30.100.
- DC-01: 10.10.20.10.
- WIN-01: 10.10.20.100.
- WAZUH-01: 10.10.40.10.

No DET-019 network change is required at this stage.

## AD State
- Domain: `corp.home.arpa`
- NetBIOS: `CORP`
- DC: DC-01
- DNS and Global Catalog enabled on DC-01.
- Windows Server 2025 forest/domain functional level.
- WIN-01 is domain joined.
- Existing users include alice, bob, carol, and david.
- `CORP\admin` is an existing domain administrative account.
- Permanent domain lockout baseline: threshold 5, duration 10 minutes, observation window 10 minutes.

## Wazuh State
- Wazuh validated at version 4.14.7 in project evidence.
- WIN-01: agent 001.
- DC-01: agent 002.
- Wazuh indexer/OpenSearch telemetry is operational.
- Windows Security and Sysmon telemetry from DC-01 are retained/queryable.
- Native Wazuh rules are preferred when they already provide the required coverage.
- DET-011 includes a validated read-only Python Indexer correlation prototype; it is not persistent native Wazuh correlation.

## Sysmon State
- Sysmon is installed on WIN-01 and DC-01.
- DC-01 Sysmon/Operational telemetry is explicitly collected by Wazuh.
- Sysmon Event ID 1 process creation was validated.
- Sysmon Event ID 11 file creation was validated during DET-012.

## Telemetry State
Validated telemetry includes:
- Security 4624 — successful logon
- Security 4625 — failed logon
- Security 4672 — special privileges assigned
- Security 4698 — scheduled task created
- Security 4720 — account created
- Security 4728 — Domain Admins membership change
- Security 4732 — local Administrators membership change
- Security 4740 — account lockout
- System 7045 — service installed
- Sysmon Event 1 — process creation
- Sysmon Event 11 — file creation
- Sysmon Event 3 — network connection

DET-018 validated 4624 → 4672 correlation through Logon ID `0x2c5499`.
DET-019 validated 4672 → 4688 correlation through subjectLogonId using Wazuh rule 100107.
DET-020 validated PowerShell Event 1 → Event 3 correlation through ProcessGuid using Wazuh rule 100108.

## Detection-Engineering State
- DET-001 → DET-021 completed.
- Native Wazuh coverage is used where sufficient.
- Custom rules are used where contextual value was demonstrated.
- Detection and correlation are treated as separate engineering problems.
- Windows Logon ID and account/member SID are used as investigation pivots where supported.
- Native/vendor ATT&CK metadata is distinguished from what observed evidence proves.
- Controlled benign tests and negative validation are recorded where actually performed.

## DET-021 State
- PowerShell Event 1: validated.
- Sysmon Event 3: validated.
- Wazuh Event 3 ingestion: validated.
- Existing rule 100108 fired.
- Downloaded file: confirmed on disk.
- Sysmon Event 11 for the downloaded file: not observed.
- Three-stage Event 1 → Event 3 → Event 11 correlation: not validated.
- New Wazuh rule: not created.

## Known Gaps
- DET-019 alert-cardinality tuning remains open; the validated rule can generate multiple alerts for multiple qualifying 4688 events in one privileged session.
- DET-020 has not undergone long-term replay/coverage measurement or further contextual tuning.
- No malicious ATT&CK claim is made from the benign DET-020 validation.

## Last Verified GitHub Commit
**`0c173247689c55a6fad20050a490e99093f511e3` — `docs: update project state through DET-021`**

Recent architecture documentation commits:
- `467f96067ffa5bbbf98b868a8d8d7094d47e1d54` — `docs: add cyber range network specification`
- `3bbabb6843c480168a1b95c18d1628bb97a081ef` — `docs: clarify cyber range topology boundaries`
- `e1756cb917e525d88beaff1bd9a0c6a4d4951235` — `architecture: add master architecture`
- `40a0417cb9f274bb7c0f44b5f69cf8758b8cdc59` — `architecture: add cyber range asset inventory`
- `0749f2a56a126dc7f2fea2eacb80082017c4182b` — `architecture: add cyber range resource plan`
- `7372bc81f2044e863fe45d94013aaf368a0f4c5d` — `architecture: add target architecture`
- `296d891302f7ca3b905d822208de814089ce9586` — `architecture: add ADR-001`
- `cef3be4b6a9a48ac2efe0acab007f0cb713f9e7f` — `architecture: add ADR-002`
- `cc61678c447ae1de74cb2581ee08b5cc1e2a8f45` — `architecture: add ADR-003`
- `5a680871e3925bcb11ca73b3405f43499fb86922` — `architecture: add ADR-004`

## Current Blockers
None blocking the next planning gate.

Known engineering gaps remain non-blocking: DET-019 alert-cardinality tuning and DET-020 long-term replay/coverage measurement.

## Next Task
Review the documented controlled WinRM evidence in `docs/AD-LATERAL-MOVEMENT-WINRM-INVESTIGATION.md`. Continue the Stage C AD/lateral-movement scenario by validating the event timeline, defining a benign negative/control test, and specifying investigation and replay criteria. Do not assign a DET number until the scenario scope and validation plan are explicitly approved.

## AD/Lateral-Movement WinRM Evidence — 2026-10-09
- Evidence record: `docs/AD-LATERAL-MOVEMENT-WINRM-INVESTIGATION.md`.
- Controlled Evil-WinRM session from ARCH-01 (`10.10.30.100`) to WIN-01 (`10.10.20.100`) over TCP 5985 using `CORP\Administrator`.
- Wazuh native rule 92110 fired on WIN-01 Sysmon Event ID 3 for source `10.10.30.100:34504` to destination `10.10.20.100:5985`; event time `2026-10-09 07:00:15.404Z`.
- Wazuh custom rule 100107 fired on Windows Security Event ID 4688 records for `whoami.exe` and `conhost.exe`, with Logon ID `0x4c69ba` and parent/process activity including `wsmprovhost.exe`.
- Native T1021.006 metadata on rule 92110 is recorded as rule metadata, not proof of malicious intent.
- The test validates separate network and process detections. A single cross-rule correlation joining 92110 and 100107 has **not** been validated.
- Negative/control test, response validation, replay metrics, and false-positive rate remain unrecorded/unmeasured.
- Temporary WinRM firewall exceptions remain in place unless separately verified and changed; do not claim cleanup occurred.
- No DET number assigned. DET-001 → DET-021 remain completed. Stage C continues; Stage D network visibility is not started.

## Snapshot State
Earlier controlled scenarios have documented pre-attack snapshots, including DET-010 and DET-013.

For DET-019:
- Detection implementation: **VALIDATED**
- Positive validation: **COMPLETED**
- Negative validation: **COMPLETED**
- Alert-cardinality tuning: **OPEN / NON-BLOCKING**
- Existing project evidence is the source of truth; no new snapshot is required merely to reopen the completed detection.

## Documentation State
- `docs/DETECTION-ROADMAP.md`: present and canonical.
- `docs/PROJECT-STATE.md`: this permanent cross-chat handoff.
- `architecture/ARCHITECTURE.md`: present and canonical high-level architecture.
- `architecture/topology.md`: present and validated against the established architecture.
- `architecture/network.md`: present and validated against the established network model.
- `architecture/ARCHITECTURE.md`: master architecture.
- `architecture/topology.md`: logical/virtual topology.
- `architecture/network.md`: detailed network specification.
- `architecture/target-architecture.md`: target 32-GB-class architecture and capability expansion.
- `architecture/adr/`: architecture decision records.
- DET-001 → DET-021 detection/investigation documentation: **PRESENT in live GitHub history**.
- DET-019 implementation evidence: **PRESENT in live GitHub history**.
- DET-020 investigation evidence: **PRESENT in live GitHub history**.
- DET-021 README and investigation evidence: **PRESENT in live GitHub history**.

## Cross-Chat Recovery Procedure
At the start of every new project chat:

1. Read `docs/PROJECT-STATE.md`.
2. Read `docs/DETECTION-ROADMAP.md`.
3. Inspect latest GitHub commits.
4. Inspect the current DET README.
5. Reconcile project history/context with GitHub.
6. Determine the actual current task.
7. Never infer state solely from the latest commit.
8. Never assume a missing README means the detection was not performed.
9. Never restart the project.
10. Never invent a missing detection or evidence.

## Documentation Integrity Rule
Only actual project evidence may be recorded as completed work.

If exact historical evidence cannot be recovered, write:

> Exact evidence not recovered from available project history.

Never invent commands, timestamps, screenshots, metrics, findings, telemetry, or ATT&CK coverage.
