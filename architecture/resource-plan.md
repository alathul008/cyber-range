# Cyber Range Resource Plan

**Status:** Current architecture documentation  
**Phase:** Phase 2 — AD + Endpoint Telemetry + Detection Engineering  
**Basis:** Established asset inventory, operational profiles, current VM allocations, and the documented 32-GB-class target architecture

This document defines the resource model for operating the established Cyber Range on the current 16-GB host while preserving the 32-GB-class target architecture.

## 1. Resource Strategy

The project has two resource states:

- **Current execution platform:** 16 GB RAM
- **Target architecture:** 32 GB-class system

The 32-GB-class target is not being reduced to fit the current host.

Instead:

    32-GB-class architecture
              |
              v
       Operational profile
              |
              v
       Minimum required VMs
              |
              v
       Current 16-GB host

VMs that are not required for the active profile remain powered off.

## 2. Current Host Resources

| Resource | Established value |
|---|---|
| CPU | AMD Ryzen 7 7435HS |
| RAM | 16 GB |
| GPU | NVIDIA GeForce RTX 3050 Laptop GPU |
| Free storage previously recorded | ~262 GB |
| Hypervisor | VMware Workstation Pro |
| Host OS | Windows 11 |

The recorded free-storage value is historical project state and should not be treated as a current live measurement without rechecking the host.

## 3. Current VM Resource Allocations

| Asset | vCPU | RAM | Disk | Primary role |
|---|---:|---:|---:|---|
| FW-01 | 2 | 2 GB | 16 GB | Routing / firewall |
| ARCH-01 | Established VM | Established VM | Existing VM | Attack/operator |
| DC-01 | 2 | 3 GB | 40 GB | AD / DNS / GC |
| WIN-01 | Existing VM | Existing VM | Existing VM | Enterprise endpoint |
| WAZUH-01 | 4 | 6 GB | 50 GB | SOC / Wazuh |

Only resource values explicitly established in project state are recorded. ARCH-01 and WIN-01 are not assigned invented CPU/RAM/disk figures here.

## 4. Known Memory Baseline

The explicitly recorded allocations for:

- FW-01: 2 GB
- DC-01: 3 GB
- WAZUH-01: 6 GB

total approximately:

**11 GB RAM**

before accounting for ARCH-01, WIN-01, VMware overhead, Windows host requirements, and other host processes.

Therefore the current 16-GB system cannot safely treat the entire 32-GB-class architecture as simultaneously active.

This is expected and is the reason operational profiles exist.

## 5. Resource Priority

When RAM is constrained, prioritize according to the active objective.

### Priority 1 — Core control plane

**FW-01**

Required whenever the scenario depends on the established routed/segmented lab topology.

### Priority 2 — Security monitoring

**WAZUH-01**

Required for live telemetry, detection, alerting, and investigation.

### Priority 3 — Scenario dependency

Power on only the enterprise or attack systems actually required by the scenario:

- DC-01
- WIN-01
- ARCH-01

### Priority 4 — Optional / future systems

Future systems remain powered off unless and until they are actually implemented and required.

## 6. Profile Resource Model

### SOC

Minimum established set:

- FW-01
- WAZUH-01
- DC-01 when generating/monitoring DC telemetry
- WIN-01 when generating/monitoring endpoint telemetry

ARCH-01 is normally unnecessary.

### Red Team

Minimum established set:

- FW-01
- ARCH-01
- Required target: DC-01 or WIN-01
- WAZUH-01 when live telemetry/detection observation is required

### Network Security

Minimum established set:

- FW-01
- Required traffic source
- Required traffic destination

WAZUH-01 is added when the test includes telemetry or monitoring.

### DFIR

Minimum established set:

- WAZUH-01
- Relevant evidence source
- FW-01 when network-boundary evidence is relevant

### Purple Team

Expected set:

- FW-01
- ARCH-01
- DC-01
- WIN-01 when required
- WAZUH-01

This is the heaviest current profile and should not be assumed to be comfortable on 16 GB without managing workload and VM activation.

## 7. 16-GB Operating Rules

1. Do not run every VM simply because it exists.
2. Use one primary profile at a time unless a scenario explicitly requires overlap.
3. Shut down unused VMs cleanly after a scenario.
4. Avoid starting future infrastructure on the current host.
5. Keep adequate RAM available to Windows 11 and VMware itself.
6. If the host becomes memory-constrained, reduce active VM count before redesigning the architecture.
7. Do not change VM allocations solely to make the current host appear to support simultaneous full-range operation.
8. Do not confuse a powered-off VM with a removed capability.

## 8. CPU Strategy

The Ryzen 7 7435HS provides the host CPU resource for:

- Windows 11
- VMware Workstation Pro
- Active VMs
- Host background services

Current documented VM CPU allocations are preserved.

CPU contention should be handled operationally first:

1. Reduce unnecessary active VMs.
2. Avoid running multiple heavy workloads simultaneously.
3. Shut down unused services inside VMs where appropriate.
4. Only change VM CPU allocation after evidence shows a real bottleneck.

No CPU allocation change is currently required.

## 9. Storage Strategy

The established VMs include:

- FW-01 — 16 GB disk
- DC-01 — 40 GB disk
- WAZUH-01 — 50 GB disk

Wazuh storage requires particular attention because security telemetry grows continuously.

The project has already expanded WAZUH-01's root/LVM capacity during deployment. Storage monitoring should therefore be treated as part of normal SOC operations.

Do not claim a current free-space figure without a fresh host/VM measurement.

## 10. Snapshot Strategy

Snapshots consume storage and should be used deliberately.

Use snapshots before:

- major VM configuration changes
- firewall/network changes
- AD configuration changes
- Sysmon changes
- major detection test preparation

Do not create unnecessary snapshots for routine operations.

A snapshot is rollback protection; it is not a substitute for scenario evidence.

## 11. Resource Escalation Triggers

Consider increasing host resources only when measured constraints repeatedly prevent the required profile from operating.

### RAM escalation

A move toward the 32-GB target is justified operationally when:

- required profile VMs cannot remain active simultaneously,
- host memory pressure repeatedly affects VMware performance,
- SOC + enterprise + attack workloads cannot coexist for required purple-team validation,
- or resource management becomes the primary limitation rather than the security task itself.

### Storage escalation

Additional storage becomes relevant when:

- Wazuh/OpenSearch retention repeatedly approaches capacity,
- snapshots materially reduce available space,
- additional enterprise targets are implemented,
- or DFIR/evidence retention requires sustained local storage.

These are operational triggers, not current upgrade requirements.

## 12. Target 32-GB-Class Resource Model

The target architecture is intended to permit broader simultaneous operation than the current 16-GB host.

The exact future RAM allocation for additional VMs is intentionally not invented here because those systems have not yet been implemented.

The target model is:

    Windows 11 Host
          |
    VMware Workstation
          |
    +-----+----------------------+
    |                            |
 Current core              Future workloads
    |                            |
 FW-01                       Additional targets
 DC-01                       Network sensors
 WIN-01                      DFIR
 ARCH-01                     Web security
 WAZUH-01                    Automation / AI
    |
  32-GB-class host

The core architecture remains unchanged.

## 13. Resource Governance

Before adding a new VM:

1. Identify its architectural purpose.
2. Identify the operational profile that requires it.
3. Estimate RAM, CPU, disk, and telemetry impact.
4. Determine whether it can coexist with the current 16-GB host.
5. Decide whether it is a powered-off capability or an actively operated component.
6. Record the decision before implementation.

Avoid adding infrastructure solely to increase the tool count.

## 14. Current Resource Risks

### RAM

**Known constraint:** 16 GB host RAM.

**Impact:** Full simultaneous operation of the target architecture is not expected.

**Mitigation:** Operational profiles and selective VM power-on.

### Storage

**Known concern:** Wazuh/OpenSearch telemetry and snapshots consume storage.

**Impact:** Long-running SOC workloads can increase disk usage.

**Mitigation:** Monitor storage, manage retention deliberately, and avoid unnecessary snapshots.

### CPU

**Current status:** No documented CPU bottleneck requiring architectural change.

**Mitigation:** Profile-based VM activation.

## 15. Resource Decision Rules

The project should follow these rules:

- Preserve architecture before optimizing convenience.
- Measure before changing VM allocations.
- Prefer powering off unnecessary systems over deleting or redesigning them.
- Prefer workload consolidation over adding duplicate infrastructure.
- Treat 32 GB as the target capacity, not the current guaranteed capacity.
- Do not claim performance validation that has not been measured.
- Record significant resource decisions in an ADR once the ADR layer is established.

## 16. Current State

**Host:** Windows 11 / 16 GB RAM

**Target:** 32-GB-class cyber range

**Current profiles:** SOC, Red Team, Network Security, DFIR, Purple Team

**Current core assets:** FW-01, ARCH-01, DC-01, WIN-01, WAZUH-01

**Architecture impact:** No infrastructure change required.

**DET-019:** NOT STARTED

This resource plan is an operational constraint document. It does not authorize VM resizing, host upgrades, storage changes, or network changes.
