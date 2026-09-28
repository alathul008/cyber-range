# Cyber Range Operational Profiles

**Status:** Current architecture documentation  
**Phase:** Phase 2 — AD + Endpoint Telemetry + Detection Engineering  
**Basis:** Established architecture, asset inventory, security boundaries, telemetry architecture, and current project state

This document defines how the existing Cyber Range is operated under the current 16-GB host constraint. Operational profiles describe which established assets are powered on and used for a specific objective. They do not imply that future capabilities are deployed.

## 1. Operating Model

The architecture is designed for a 32-GB-class system, while the current physical host has 16 GB RAM.

The operating model therefore uses **selective VM activation** rather than reducing the target architecture.

Key principle:

    Full architecture
          |
          v
    Select required profile
          |
          v
    Power on required VMs
          |
          v
    Execute / validate
          |
          v
    Power down non-required VMs

A VM being powered off does not mean its architectural role has been abandoned.

## 2. Profile Selection Rules

1. Start only the systems required for the current objective.
2. Keep FW-01 powered on whenever inter-zone routing or controlled Internet access is required.
3. Keep WAZUH-01 powered on when telemetry, detection, investigation, or SOC functions are required.
4. Keep DC-01 powered on when Active Directory, DNS, authentication, or domain-controller telemetry is required.
5. Keep WIN-01 powered on when endpoint telemetry or workstation scenarios are required.
6. Keep ARCH-01 powered on for controlled adversary/operator activity.
7. Do not power on future systems merely because they appear in the target architecture.
8. Avoid simultaneous operation of unnecessary resource-intensive workloads on the 16-GB host.

## 3. SOC Profile

**Objective:** Monitoring, detection, alert triage, investigation, and telemetry analysis.

### Required assets

| Asset | Required |
|---|---|
| FW-01 | YES |
| WAZUH-01 | YES |
| DC-01 | YES when DC telemetry is being monitored/generated |
| WIN-01 | YES when endpoint telemetry is being monitored/generated |
| ARCH-01 | NO unless generating controlled activity |

### Primary workflow

    DC-01 / WIN-01
          |
      Telemetry
          |
        FW-01
          |
      WAZUH-01
          |
    Alert / Search
          |
    Triage / Investigation

### Appropriate work

- Wazuh dashboard analysis
- Windows Security event investigation
- Sysmon investigation
- Detection validation
- Logon ID/SID pivots
- Alert triage
- Threat hunting against available telemetry

## 4. Red Team Profile

**Objective:** Controlled adversary simulation against authorized enterprise assets.

### Required assets

| Asset | Required |
|---|---|
| FW-01 | YES |
| ARCH-01 | YES |
| DC-01 | YES when it is the target |
| WIN-01 | YES when it is the target |
| WAZUH-01 | Recommended when telemetry must be observed |

### Primary workflow

    ARCH-01
       |
    Controlled activity
       |
     FW-01
       |
    Enterprise target
       |
    Telemetry
       |
    WAZUH-01

### Safety requirement

Attack activity remains inside the authorized lab. Intentionally vulnerable or adversarial systems must remain on the isolated lab networks and must not be bridged directly to VMnet0.

## 5. Network Security Profile

**Objective:** Validate routing, segmentation, firewall behavior, and security-zone connectivity.

### Required assets

| Asset | Required |
|---|---|
| FW-01 | YES |
| ARCH-01 | As test source |
| DC-01 | As enterprise test target/source |
| WIN-01 | As enterprise test target/source |
| WAZUH-01 | When validating security-zone services |

### Primary workflow

    Test source
        |
    FW-01 policy
        |
    Allowed / blocked path
        |
    Packet / connection evidence

### Validation principle

Test both:

- intended permitted paths
- intended blocked paths

A successful single-path test is not treated as exhaustive firewall assurance.

## 6. DFIR Profile

**Objective:** Evidence preservation, event reconstruction, investigation, and incident-response practice.

### Required assets

| Asset | Required |
|---|---|
| WAZUH-01 | YES |
| DC-01 | YES when evidence originates there |
| WIN-01 | YES when evidence originates there |
| FW-01 | YES when network-boundary evidence is relevant |
| ARCH-01 | Only when adversary-side evidence is relevant |

### Primary workflow

    Evidence source
          |
      Telemetry
          |
       WAZUH-01
          |
    Timeline / pivots
          |
    Investigation
          |
       Evidence
          |
      Findings

The current architecture provides endpoint and Wazuh evidence. It does not claim dedicated DFIR collection infrastructure.

## 7. Purple Team Profile

**Objective:** Execute controlled behavior, validate telemetry and detection, investigate results, improve detection, and replay.

### Required assets

| Asset | Required |
|---|---|
| FW-01 | YES |
| ARCH-01 | YES |
| DC-01 | YES |
| WIN-01 | As required by scenario |
| WAZUH-01 | YES |

### Core loop

    ATTACK
      ↓
    TELEMETRY
      ↓
    DETECTION
      ↓
    TRIAGE
      ↓
    INVESTIGATION
      ↓
    MITRE ATT&CK
      ↓
    DETECTION GAP
      ↓
    IMPROVEMENT
      ↓
    REPLAY
      ↓
    VALIDATION

This profile best represents the project's primary attack-to-detection engineering methodology.

## 8. Detection-Engineering Profile

**Objective:** Build and validate one detection without unnecessary infrastructure.

### Required assets

Usually:

- FW-01
- target telemetry source
- WAZUH-01

ARCH-01 is required only when the detection scenario uses it as the activity source.

### Workflow

1. Establish pre-test state.
2. Generate controlled activity.
3. Confirm expected telemetry.
4. Determine the observed fields.
5. Check native Wazuh coverage.
6. Add custom logic only where justified.
7. Validate alert generation.
8. Investigate the alert.
9. Record ATT&CK mapping from observed behavior.
10. Record gaps and false positives where actually observed.
11. Replay after improvement.

This preserves the project's distinction between telemetry, detection, and correlation.

## 9. Current DET-019 Profile

DET-019 is currently:

**Privileged Logon → Process Creation Correlation**

Status:

- NOT STARTED
- No pre-attack snapshot
- No DET-019 attack/test
- No DET-019 telemetry generated specifically for the scenario
- No ATT&CK technique pre-claimed

When DET-019 is eventually executed, its profile should include:

| Asset | Role |
|---|---|
| DC-01 | Privileged-logon and process telemetry source |
| WAZUH-01 | Ingestion, detection, search, investigation |
| ARCH-01 | Only if the actual approved test requires an operator-side source |
| WIN-01 | Not required unless the scenario is expanded to the endpoint |

No infrastructure change is currently required for DET-019.

## 10. Resource Strategy for 16 GB

The current host should prioritize the minimum active set required by the scenario.

A practical operating pattern is:

### Monitoring / investigation

    FW-01
    WAZUH-01
    DC-01
    WIN-01

### Attack exercise

    FW-01
    ARCH-01
    Required target
    WAZUH-01 if telemetry must be observed live

### Lightweight detection development

    WAZUH-01
    One relevant telemetry source
    FW-01 only if network routing is required

### Network validation

    FW-01
    Test source
    Test destination
    WAZUH-01 only if telemetry is part of the test

These are operating patterns, not permanent VM allocation changes.

## 11. Snapshot and Change Discipline

Before a major controlled scenario:

1. Identify the assets that will change.
2. Record the intended test state.
3. Take snapshots when appropriate.
4. Execute only the approved test.
5. Capture evidence.
6. Validate cleanup.
7. Verify the baseline is restored or document the permanent change.
8. Record the result in the scenario artifact.

Do not create snapshots as a substitute for evidence collection.

## 12. Profile Transition

When switching profiles:

1. Finish or pause the current scenario.
2. Preserve required evidence.
3. Confirm no test process is still running.
4. Shut down VMs that are no longer required.
5. Start only the next profile's required assets.
6. Validate network and telemetry prerequisites.
7. Continue the next task.

Do not change VMware networking merely to switch operational profiles.

## 13. Profile-to-Capability Matrix

| Capability | SOC | Red Team | Network Security | DFIR | Purple Team |
|---|---:|---:|---:|---:|---:|
| Wazuh monitoring | ✓ | optional | optional | ✓ | ✓ |
| AD telemetry | ✓ | target | optional | ✓ | ✓ |
| Sysmon telemetry | ✓ | target activity | optional | ✓ | ✓ |
| Controlled attack activity | optional | ✓ | optional | optional | ✓ |
| Firewall validation | optional | ✓ | ✓ | optional | ✓ |
| Detection engineering | ✓ | optional | optional | ✓ | ✓ |
| Investigation | ✓ | optional | optional | ✓ | ✓ |
| ATT&CK mapping | ✓ | ✓ | optional | ✓ | ✓ |
| Detection replay | ✓ | optional | optional | ✓ | ✓ |

The matrix describes the role of each profile, not a claim that every capability has been exercised in every profile.

## 14. Operational Safety Rules

1. Never bridge an adversarial or intentionally vulnerable lab system directly to VMnet0.
2. Do not alter VMnet0, VMnet8, host networking, or FW-01 policy without change control.
3. Preserve rollback information before major infrastructure changes.
4. Keep the current lab segmentation intact during scenario work.
5. Do not expose lab services to the real LAN.
6. Do not represent a powered-off asset as removed from the architecture.
7. Do not claim a profile validated a capability unless the capability was actually tested.

## 15. Current State

**Current phase:** Phase 2 — AD + Endpoint Telemetry + Detection Engineering

**Completed:** DET-001 → DET-018

**Current:** DET-019 — NOT STARTED

**Architecture documentation completed so far:**

- architecture/ARCHITECTURE.md
- architecture/topology.md
- architecture/network.md
- architecture/asset-inventory.md
- architecture/security-boundaries.md
- architecture/telemetry-architecture.md
- architecture/operational-profiles.md

**Next architecture tasks:** resource plan, target architecture, and ADR collection.

No infrastructure change is implied by this document.
