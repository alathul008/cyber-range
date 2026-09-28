# Cyber Range — Project State

## Current Phase
**Phase 2 — AD + Endpoint Telemetry + Detection Engineering**

## Current Objective
Complete permanent documentation/state continuity so cross-chat recovery is deterministic.

## Current DET
**DET-019 — Privileged Logon → Process Creation Correlation**

**Status: CURRENT / NOT STARTED**

No DET-019 attack or test has been executed.

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
**DET-001 → DET-018: COMPLETED**

## Documentation Audit
Live GitHub repository tree was audited on 2026-09-28.

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

DET-018 validated 4624 → 4672 correlation through Logon ID `0x2c5499`.

## Detection-Engineering State
- DET-001 → DET-018 completed.
- Native Wazuh coverage is used where sufficient.
- Custom rules are used where contextual value was demonstrated.
- Detection and correlation are treated as separate engineering problems.
- Windows Logon ID and account/member SID are used as investigation pivots where supported.
- Native/vendor ATT&CK metadata is distinguished from what observed evidence proves.
- Controlled benign tests and negative validation are recorded where actually performed.

## Known Gaps
- DET-019 has not been executed.
- DET-019 persistent documentation has not yet been created.
- DET-019 pre-attack snapshot has not yet been created.
- DET-019 privileged-logon → process-creation session correlation has not yet been validated.
- DET-019 ATT&CK mapping remains intentionally undetermined.

## Last Verified GitHub Commit
**`187c831ed615a4c9d216bf2205a8b1692be87919` — `docs: mark DET-019 current and not started`**

The immediately preceding canonical roadmap commit was:
`0509102e462a7b90b5817c8b8b3f7d40eaed64d7` — `docs: add canonical detection roadmap`.

## Current Blockers
1. DET-019 persistent documentation.
2. DET-019 pre-attack snapshot.
3. These gates must be completed before any DET-019 attack/test execution.

## Next Task
Prepare the persistent DET-019 README and pre-attack snapshot plan.

Do not execute DET-019 yet.

## Snapshot State
Earlier controlled scenarios have documented pre-attack snapshots, including DET-010 and DET-013.

For DET-019:
- Pre-attack snapshot: **NOT CREATED**
- Attack/test: **NOT STARTED**

## Documentation State
- `docs/DETECTION-ROADMAP.md`: present and canonical.
- `docs/PROJECT-STATE.md`: this permanent cross-chat handoff.
- DET-001 → DET-018 individual README artifacts: **PRESENT**.
- DET-019 README: **NOT CREATED**.
- DET-019 investigation evidence: **NOT CREATED**.

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
