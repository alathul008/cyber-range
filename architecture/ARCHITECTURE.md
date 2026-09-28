# Cyber Range Architecture

**Architecture status:** Current project architecture documented from established project state
**Current phase:** Phase 2 — AD + Endpoint Telemetry + Detection Engineering
**Current position:** DET-001 → DET-018 COMPLETED; DET-019 CURRENT / NOT STARTED

This is the high-level architecture reference for the Cyber Range. It separates the architecture currently implemented and/or validated from the longer-term 32-GB-class target and future capabilities.

## 1. Mission and Purpose

The Cyber Range is an isolated, enterprise-style security laboratory built on Windows 11 and VMware Workstation Pro.

Its purpose is to provide a repeatable environment for practical cybersecurity work centered on Active Directory and Windows enterprise telemetry, SOC/SIEM workflows, detection engineering, threat hunting and investigation, MITRE ATT&CK-based analysis, purple-team validation, and incident-response/DFIR-oriented workflows.

The project is not intended to be a collection of disconnected security tools. Detection, telemetry, evidence, correlation, validation, and repeatability are architectural concerns.

## 2. Architecture Principles

1. Isolation first.
2. Evidence over assumption.
3. Implemented is not automatically validated.
4. Detection before tool accumulation.
5. Detection and correlation are separate engineering problems.
6. Use stable investigation pivots such as Windows Logon ID and account/member SID where supported.
7. Prefer validated native Wazuh coverage before adding custom logic.
8. Use controlled tests, snapshots, evidence, investigation, and replay for validation.
9. Design for a 32-GB-class system while operating the current 16-GB host through profiles.
10. Never represent planned or future capabilities as deployed.

## 3. Current Implemented Architecture

| Asset | Role | State |
|---|---|---|
| Windows 11 host | Physical virtualization host | IMPLEMENTED |
| VMware Workstation Pro | Hypervisor | IMPLEMENTED |
| FW-01 | pfSense firewall/router | IMPLEMENTED in established architecture |
| ARCH-01 | Arch Linux attack/operator environment | IMPLEMENTED |
| DC-01 | Windows Server 2025 AD domain controller and DNS | IMPLEMENTED / VALIDATED |
| WIN-01 | Windows enterprise endpoint | IMPLEMENTED / VALIDATED |
| WAZUH-01 | Ubuntu/Wazuh security server | IMPLEMENTED / VALIDATED |

Established addresses: ARCH-01 — 10.10.30.100; DC-01 — 10.10.20.10; WIN-01 — 10.10.20.100; WAZUH-01 — 10.10.40.10.

The current architecture does not claim deployment of additional future endpoints, web targets, alternate SIEMs, network sensors, dedicated DFIR infrastructure, or cloud environments.

## 4. Current Validated Capabilities

### Active Directory
- Domain: corp.home.arpa
- NetBIOS: CORP
- DC-01 is the domain controller.
- DNS and Global Catalog are enabled.
- Windows Server 2025 forest/domain functional level.
- WIN-01 is domain joined.
- Existing users include alice, bob, carol, and david.
- CORP\admin exists as a domain administrative account.
- Permanent domain lockout baseline: threshold 5, duration 10 minutes, observation window 10 minutes.

### Wazuh
- Wazuh 4.14.7 validated in project evidence.
- WIN-01 agent 001.
- DC-01 agent 002.
- Wazuh indexer/OpenSearch operational.
- Windows Security and Sysmon telemetry from DC-01 retained/queryable.

### Sysmon
- Sysmon installed on WIN-01 and DC-01.
- DC-01 Sysmon/Operational telemetry explicitly collected by Wazuh.
- Sysmon Event ID 1 process creation validated.
- Sysmon Event ID 11 file creation validated during DET-012.

### Validated security telemetry
- Security 4624 — successful logon
- Security 4625 — failed logon
- Security 4672 — special privileges assigned
- Security 4698 — scheduled task creation
- Security 4720 — account creation
- Security 4728 — Domain Admins membership change
- Security 4732 — local Administrators membership change
- Security 4740 — account lockout
- System 7045 — service installation
- Sysmon Event ID 1 — process creation
- Sysmon Event ID 11 — file creation

DET-018 validated 4624 → 4672 correlation using Logon ID 0x2c5499.

## 5. VMware Topology

    Windows 11 HOST
    VMware Workstation Pro
             |
       +-----+-------------------------------+
       |             |                       |
     VMnet0        VMnet8                  VMnet1
     Bridged        NAT                 Host-only
     Real LAN    192.168.150.0/24      192.168.195.0/24
       |
       | WAN path
       v
     FW-01
    pfSense
       |
    +--+---------+---------+---------+
    |            |         |         |
  VMnet2       VMnet3    VMnet4    VMnet5
   MGMT       ENTERPRISE ATTACK   SECURITY/SOC
 10.10.10/24 10.10.20/24 10.10.30/24 10.10.40/24

VMnet1 is retained as a legacy host-only network.

## 6. Network Zones and Subnets

| Zone | VMware network | Subnet | Established purpose |
|---|---|---|---|
| Host/LAN | VMnet0 | Bridged | Host/LAN connectivity |
| Legacy host-only | VMnet1 | 192.168.195.0/24 | Existing legacy private network |
| WAN/NAT | VMnet8 | 192.168.150.0/24 | NAT path / pfSense WAN |
| MGMT | VMnet2 | 10.10.10.0/24 | Management |
| ENTERPRISE | VMnet3 | 10.10.20.0/24 | Enterprise systems |
| ATTACK | VMnet4 | 10.10.30.0/24 | Attack/operator systems |
| SECURITY/SOC | VMnet5 | 10.10.40.0/24 | Security monitoring |

Established convention: pfSense uses .1 on lab segments; Windows host virtual adapters use .254; DHCP ranges were established as .100-.199 where applicable.

## 7. VM / Asset Roles

### ARCH-01
Arch Linux primary attack/operator environment in ATTACK; 10.10.30.100.

### DC-01
Windows Server 2025 AD domain controller, DNS, and Global Catalog in ENTERPRISE; 10.10.20.10; Wazuh agent 002; Sysmon telemetry.

### WIN-01
Windows enterprise endpoint, domain joined to corp.home.arpa, in ENTERPRISE; 10.10.20.100; Wazuh agent 001; Sysmon installed.

### WAZUH-01
Ubuntu Server Wazuh security server in SECURITY/SOC; 10.10.40.10.

### FW-01
pfSense firewall/router forming the established routing and security boundary between lab zones.

## 8. pfSense Placement and Interfaces

| Interface role | VMware network | Address |
|---|---|---|
| WAN | VMnet8 | NAT-side interface |
| LAN/MGMT | VMnet2 | 10.10.10.1 |
| ENTERPRISE | VMnet3 | 10.10.20.1 |
| ATTACK | VMnet4 | 10.10.30.1 |
| SECURITY | VMnet5 | 10.10.40.1 |

Established policy model: ATTACK is isolated from enterprise/security management paths except explicitly required services; ENTERPRISE is restricted from unnecessary ATTACK/SECURITY access; SECURITY/SOC has the required lab visibility; Wazuh agent traffic to WAZUH-01 is explicitly permitted by the established design.

This document does not modify FW-01.

## 9. AD Architecture

    CORP
    └── corp.home.arpa
        └── DC-01
            ├── Active Directory
            ├── DNS
            └── Global Catalog

Established AD organization includes User Accounts, Computer Accounts, Admins, and Security areas.

Established security groups include CORP-IT-Admins, CORP-SOC-Analysts, CORP-Helpdesk, CORP-Server-Admins, and CORP-Workstation-Admins.

## 10. Wazuh / Sysmon Telemetry Architecture

    DC-01                         WIN-01
    Security + Sysmon             Security + Sysmon
          |                             |
          +------- Wazuh Agents --------+
                       |
                       v
                   WAZUH-01
                       |
              +--------+--------+
              |                 |
          Wazuh Manager   Indexer/OpenSearch
              |                 |
              +--------+--------+
                       |
                       v
              Detection / Investigation

DC-01 Sysmon/Operational telemetry is explicitly collected by Wazuh.

DET-011 includes a validated read-only Python Indexer correlation prototype; it is not persistent native Wazuh correlation.

## 11. Detection-Engineering Architecture

    Controlled Activity
           ↓
    Endpoint / AD Telemetry
           ↓
    Wazuh Ingestion
           ↓
    Native or Custom Detection
           ↓
    Investigation / Correlation
           ↓
    Evidence
           ↓
    Detection Gap
           ↓
    Improvement
           ↓
    Replay / Validation

Completed sequence: DET-001 → DET-018.

Established patterns include native Wazuh coverage where sufficient, custom contextual rules where demonstrated, Windows Logon ID/SID pivots, and explicit separation of vendor ATT&CK metadata from what evidence proves.

## 12. Security Boundaries

    REAL LAN
       |
     VMnet0
       |
      FW-01
       |
       +-- MGMT
       +-- ENTERPRISE
       +-- ATTACK
       +-- SECURITY/SOC

The boundaries are intended to prevent adversarial lab activity from reaching the real network, isolate attack/operator systems from enterprise systems, restrict unnecessary enterprise-to-security/attack access, and provide the security zone with required visibility.

No network modification is performed by this document.

## 13. Resource Model

| Resource | Current |
|---|---|
| CPU | AMD Ryzen 7 7435HS |
| RAM | 16 GB |
| GPU | NVIDIA GeForce RTX 3050 Laptop GPU |
| Free storage previously recorded | ~262 GB |
| Hypervisor | VMware Workstation Pro |

The architecture is designed as a 32-GB-class range while the physical host currently has 16 GB. Operational profiles allow VMs to be powered on/off according to the current workload without abandoning their architectural role.

## 14. Operational Profiles

### SOC
WAZUH-01 with required DC-01/WIN-01 systems for monitoring, detection, and investigation.

### Red Team
ARCH-01 with required enterprise target systems for controlled attack activity.

### Network Security
FW-01 plus required security/enterprise/attack systems for network-security testing and telemetry.

### DFIR
WAZUH-01 plus relevant evidence-producing systems for investigation and incident-response workflows.

### Purple Team
Combines controlled adversary activity, telemetry generation, detection validation, investigation, replay, and detection improvement.

These profiles describe operation of the established architecture and do not imply future capabilities are deployed.

## 15. Current vs Target Architecture

### Current

    Windows 11 Host
           |
    VMware Workstation Pro
           |
          FW-01
           |
    +------+---------+----------+
    |                |          |
  ATTACK          ENTERPRISE  SECURITY
    |                |          |
 ARCH-01          DC-01      WAZUH-01
                     |
                   WIN-01

This represents the current established architecture.

### Target 32-GB-class architecture

The target expands capability while preserving the current core. Longer-term additions may include additional enterprise workloads/endpoints, web-security targets, network telemetry, alternate SIEM/detection platforms, dedicated DFIR workflows, cloud-security scenarios, malware-analysis capability, automation, and AI-assisted security workflows.

These are not represented as deployed by this document.

## 16. State Vocabulary

**IMPLEMENTED** — component or architectural element exists in the established lab.

**VALIDATED** — project evidence demonstrates the stated capability or behavior.

**PLANNED** — intended future addition with an identified architectural purpose.

**FUTURE** — longer-term project capability not currently being implemented.

**DEFERRED** — deliberately postponed because of sequencing, resources, prerequisites, or another project decision.

IMPLEMENTED does not automatically mean VALIDATED. PLANNED/FUTURE does not mean deployed.

## 17. Current Project Position

**Phase:** Phase 2 — AD + Endpoint Telemetry + Detection Engineering

**Completed:** DET-001 → DET-018 COMPLETED

**Current:** DET-019 CURRENT / NOT STARTED

DET-019 is Privileged Logon → Process Creation Correlation.

Conceptual chain:

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

DET-019 has not been executed. Its ATT&CK mapping must be determined from actual observed behavior. Persistent documentation and a pre-attack snapshot are required before execution.

## 18. Documentation Architecture

### architecture/
System-level architecture: master architecture, topology, network design, asset inventory, security boundaries, telemetry architecture, operational profiles, resource planning, target architecture, and architecture decisions.

### detections/
Detection-engineering artifacts. DET-001 through DET-018 are currently documented here.

### docs/
Project continuity and sequencing:
- DETECTION-ROADMAP.md
- PROJECT-STATE.md

PROJECT-STATE.md is the permanent cross-chat handoff. DETECTION-ROADMAP.md is the canonical detection sequence.

## Architecture Integrity Rules

1. Never claim a future component is deployed without evidence.
2. Never infer implementation from the long-term vision.
3. Keep implementation state separate from validation state.
4. Keep current architecture separate from target architecture.
5. Record significant architecture decisions as ADRs when the ADR layer is established.
6. Architecture documentation must not imply infrastructure changes.
7. Preserve evidence-backed project terminology and state.

## Current Architecture Boundary

This document is the high-level architecture reference. Detailed topology, network specification, asset inventory, security-boundary specification, telemetry architecture, operational profiles, and resource plan are now established in architecture/topology.md, architecture/network.md, architecture/asset-inventory.md, architecture/security-boundaries.md, architecture/telemetry-architecture.md, architecture/operational-profiles.md, and architecture/resource-plan.md. Target architecture and ADR collection remain separate documentation tasks.

No infrastructure change is implied by this document.