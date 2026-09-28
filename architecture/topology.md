# Cyber Range Topology

**Status:** Current architecture documentation  
**Phase:** Phase 2 — AD + Endpoint Telemetry + Detection Engineering  
**Current position:** DET-001 → DET-018 completed; DET-019 not started

This document describes the established logical and virtual topology of the Cyber Range. It documents relationships between the Windows host, VMware virtual networks, pfSense, enterprise systems, attack systems, and security infrastructure.

It does not modify or prescribe infrastructure changes.

## 1. Topology at a Glance

    Windows 11 Host
    VMware Workstation Pro
             |
       +-----+-------------------------------+
       +-------------+-------------+-------------+
                     |             |             |
                   VMnet0        VMnet8        VMnet1
                   Bridged        NAT       Legacy host-only
                  Real LAN    192.168.150.0/24 192.168.195.0/24
                     |             |
                     |             v
                     |           FW-01
                     |          pfSense
       |                    |
       |        +-----------+-----------+-----------+
       |        |           |           |           |
       |      VMnet2      VMnet3       VMnet4      VMnet5
       |       MGMT     ENTERPRISE     ATTACK    SECURITY/SOC
       |    10.10.10/24 10.10.20/24 10.10.30/24 10.10.40/24
       |        |           |           |           |
       |        |       +---+---+       |           |
       |        |       |       |       |           |
       |        |    DC-01   WIN-01  ARCH-01     WAZUH-01
       |        |    .20.10   .20.100   .30.100    .40.10
       |        |
       |    Host adapter
       |       .10.254

VMnet0 represents the real LAN/bridged path and is not an enterprise lab segment.

The four isolated lab segments (MGMT, ENTERPRISE, ATTACK, and SECURITY/SOC) are behind FW-01 on VMnet2–VMnet5. VMnet0 and VMnet1 are separate VMware networks; VMnet8 is the independent NAT-side upstream used by FW-01 WAN.

VMnet8 provides the established NAT-side WAN path for FW-01.

VMnet1 is retained as a legacy host-only network and is not part of the core four-zone security architecture.

## 2. Core Security Zones

| Zone | VMware Network | Subnet | Primary Purpose | Established Systems |
|---|---|---|---|---|
| MGMT | VMnet2 | 10.10.10.0/24 | Management | Host-side management connectivity |
| ENTERPRISE | VMnet3 | 10.10.20.0/24 | Enterprise identity/endpoints | DC-01, WIN-01 |
| ATTACK | VMnet4 | 10.10.30.0/24 | Attack/operator activity | ARCH-01 |
| SECURITY/SOC | VMnet5 | 10.10.40.0/24 | Monitoring/security services | WAZUH-01 |

The zones are logically separated by FW-01.

## 3. FW-01 Placement

FW-01 is the central routing and security boundary for the isolated lab zones.

| Interface Role | VMware Network | Address |
|---|---|---|
| WAN | VMnet8 | NAT-side interface |
| LAN/MGMT | VMnet2 | 10.10.10.1 |
| ENTERPRISE | VMnet3 | 10.10.20.1 |
| ATTACK | VMnet4 | 10.10.30.1 |
| SECURITY | VMnet5 | 10.10.40.1 |

The established design uses .1 as the pfSense gateway on the four lab segments.

The Windows host-side virtual adapters use .254 on the lab host-only networks.

## 4. Enterprise Zone

    ENTERPRISE
    10.10.20.0/24
           |
     +-----+-----+
     |           |
   DC-01       WIN-01
   10.10.20.10 10.10.20.100
     |           |
   AD/DNS     Domain Endpoint
   Global GC     Sysmon
                 Wazuh Agent

### DC-01

- Windows Server 2025
- Active Directory Domain Controller
- DNS
- Global Catalog
- Domain: corp.home.arpa
- NetBIOS: CORP
- IP: 10.10.20.10
- Wazuh agent: 002
- Sysmon installed
- Sysmon/Operational explicitly collected by Wazuh

### WIN-01

- Windows enterprise endpoint
- IP: 10.10.20.100
- Domain joined to corp.home.arpa
- Wazuh agent: 001
- Sysmon installed

## 5. Attack Zone

    ATTACK
    10.10.30.0/24
           |
        ARCH-01
      10.10.30.100
           |
    Controlled activity
           |
         FW-01
           |
       Lab targets

ARCH-01 is the primary operator/attack environment.

The attack zone is intentionally separated from the enterprise and security zones. Controlled activity must remain inside the authorized lab environment.

## 6. Security / SOC Zone

    SECURITY / SOC
     10.10.40.0/24
            |
        WAZUH-01
       10.10.40.10
            |
       +----+----+
       |         |
     Wazuh    OpenSearch
    Manager
                 |
          Alert / Search
                 |
          Investigation

WAZUH-01 is the current security-monitoring platform.

Validated telemetry sources include Windows Security events, Sysmon telemetry, Wazuh agent events, Wazuh native detection rules, and project-specific custom rules where validated.

## 7. Management Zone

    MGMT
    10.10.10.0/24
           |
         FW-01
           |
    Windows host adapter
       10.10.10.254

The MGMT zone exists as a separate lab network.

The Windows host-side virtual adapter convention is .254.

No claim is made here that every future management workflow has been implemented.

## 8. WAN / NAT Path

    Host environment
          |
        VMnet8
    192.168.150.0/24
          |
       FW-01 WAN
          |
    NAT gateway
      192.168.150.2
          |
       Internet

VMnet8 is the VMware NAT network used by FW-01's WAN interface.

The core MGMT, ENTERPRISE, ATTACK, and SECURITY/SOC networks remain separate from this NAT segment.

## 9. Host-Only Network Details

### VMnet2 — MGMT

- Network: 10.10.10.0/24
- pfSense gateway: 10.10.10.1
- Host adapter: 10.10.10.254
- DHCP convention: .100–.199 where applicable

### VMnet3 — ENTERPRISE

- Network: 10.10.20.0/24
- pfSense gateway: 10.10.20.1
- Host adapter: 10.10.20.254
- DC-01: 10.10.20.10
- WIN-01: 10.10.20.100
- DHCP convention: .100–.199 where applicable

### VMnet4 — ATTACK

- Network: 10.10.30.0/24
- pfSense gateway: 10.10.30.1
- Host adapter: 10.10.30.254
- ARCH-01: 10.10.30.100
- DHCP convention: .100–.199 where applicable

### VMnet5 — SECURITY/SOC

- Network: 10.10.40.0/24
- pfSense gateway: 10.10.40.1
- Host adapter: 10.10.40.254
- WAZUH-01: 10.10.40.10
- DHCP convention: .100–.199 where applicable

## 10. Core Traffic Relationships

### ATTACK → ENTERPRISE

Controlled attack activity originates from ARCH-01 and targets authorized enterprise systems.

FW-01 enforces the established ATTACK isolation policy.

### ENTERPRISE → SECURITY

Enterprise agents require explicitly permitted connectivity to WAZUH-01 for Wazuh agent communication.

Established Wazuh agent traffic includes TCP 1514 and TCP 1515 to WAZUH-01 where required by the current design.

### SECURITY → ENTERPRISE

The security zone requires access to enterprise systems for monitoring, investigation, and security operations according to the established firewall design.

### SECURITY → ATTACK

The security zone can reach the attack zone where required for security operations and controlled validation.

### ENTERPRISE ↛ ATTACK

Unnecessary enterprise-to-attack communication is restricted by the established firewall policy.

### ATTACK ↛ SECURITY

Unnecessary attack-to-security access is restricted by the established firewall policy.

The exact firewall rule set remains defined in the established FW-01 configuration; this topology document does not replace that configuration.

## 11. Telemetry Flow

    DC-01 / WIN-01
           |
     +-----+-----+
     |           |
 Windows       Sysmon
 Security
     |           |
     +-----+-----+
           |
      Wazuh Agent
           |
           v
       WAZUH-01
           |
      +----+----+
      |         |
 Wazuh Manager OpenSearch
      |         |
      +----+----+
           |
     Detection /
     Investigation

For DC-01, the validated Sysmon path specifically includes:

    Microsoft-Windows-Sysmon/Operational
                     |
                     v
                Wazuh Agent
                     |
                     v
                 WAZUH-01

Sysmon Event ID 1 process creation has been validated through this path.

## 12. Detection / Investigation Relationship

    ARCH-01 / controlled activity
                |
                v
         Enterprise target
                |
                v
      Windows / Sysmon telemetry
                |
                v
           Wazuh ingestion
                |
                v
             Detection
                |
                v
          Investigation
                |
                v
        ATT&CK / IOC / IOA / TTP
                |
                v
        Detection improvement
                |
                v
              Replay

This relationship is the core reason the ATTACK, ENTERPRISE, and SECURITY/SOC zones are separate.

## 13. Current Asset Map

| Asset | Zone | IP | Primary Role | State |
|---|---|---:|---|---|
| FW-01 | Multi-zone | .1 per lab zone | Firewall/router | IMPLEMENTED |
| ARCH-01 | ATTACK | 10.10.30.100 | Attack/operator | IMPLEMENTED |
| DC-01 | ENTERPRISE | 10.10.20.10 | AD/DNS/GC | IMPLEMENTED / VALIDATED |
| WIN-01 | ENTERPRISE | 10.10.20.100 | Windows endpoint | IMPLEMENTED / VALIDATED |
| WAZUH-01 | SECURITY/SOC | 10.10.40.10 | SIEM/security monitoring | IMPLEMENTED / VALIDATED |

Future assets are intentionally not included in the current asset map as deployed systems.

## 14. Safety Boundary

The cyber range is an authorized isolated environment.

The architecture's safety model depends on:

1. Keeping enterprise, attack, and security lab networks behind FW-01.
2. Preventing intentionally vulnerable systems from being exposed directly to the real LAN.
3. Preserving the distinction between VMnet0/real-LAN connectivity and the isolated lab zones.
4. Validating firewall behavior before introducing additional attack scenarios.
5. Taking snapshots before major controlled changes where appropriate.

This document does not authorize changes to the host network, VMware networking, or pfSense configuration.

## 15. Current vs Future Topology

### Current

    Windows 11
        |
    VMware Workstation
        |
      FW-01
        |
    +---+-----------+-----------+
    |               |           |
  ATTACK         ENTERPRISE   SECURITY
    |               |           |
 ARCH-01        DC-01/WIN-01  WAZUH-01

### Target evolution

The topology may eventually expand with additional enterprise endpoints, Linux workloads, vulnerable web applications, network telemetry, DFIR workflows, alternate SIEM platforms, cloud-security environments, malware-analysis infrastructure, automation, and AI-assisted SOC capabilities.

Those systems are not deployed by this document.

## 16. Relationship to Other Architecture Documents

This document defines the topology and relationships.

Detailed information will be maintained separately:

- architecture/ARCHITECTURE.md — master architecture
- architecture/network.md — detailed network specification
- architecture/asset-inventory.md — complete asset lifecycle inventory
- architecture/security-boundaries.md — detailed trust boundaries and permitted flows
- architecture/telemetry-architecture.md — detailed telemetry pipeline
- architecture/operational-profiles.md — profile-specific VM/resource operation
- architecture/resource-plan.md — 16-GB vs 32-GB resource planning
- architecture/target-architecture.md — future-state architecture

These are separate future documentation tasks and are not implied to already exist.

## 17. Topology Integrity Rules

1. This document describes the established topology; it does not change it.
2. Current systems must not be confused with future systems.
3. An implemented network is not automatically a fully validated security boundary.
4. Future assets must not be presented as deployed.
5. Any future VMware or pfSense change requires explicit validation and rollback planning.
6. The real LAN/VMnet0 must remain distinct from the isolated lab zones.
7. The topology must remain consistent with architecture/ARCHITECTURE.md and docs/PROJECT-STATE.md.
