# Cyber Range Asset Inventory

**Status:** Current architecture documentation  
**Phase:** Phase 2 — AD + Endpoint Telemetry + Detection Engineering  
**Inventory basis:** Established project state and validated architecture documentation

This document is the authoritative inventory of assets currently established in the Cyber Range. It distinguishes deployed assets from future capabilities and does not imply that planned systems are currently deployed.

## 1. Inventory Rules

1. **IMPLEMENTED** means the asset exists in the established lab architecture.
2. **VALIDATED** means project evidence demonstrates the stated capability or behavior.
3. Planned or future systems are not listed as deployed assets.
4. An asset can be IMPLEMENTED without every capability on that asset being independently validated.
5. IP addresses recorded here reflect the established lab addressing model.
6. This document does not prescribe infrastructure changes.

## 2. Physical / Virtualization Layer

| Asset | Type | Role | State |
|---|---|---|---|
| Windows 11 Host | Physical host | VMware virtualization host | IMPLEMENTED |
| VMware Workstation Pro | Hypervisor | Runs the cyber-range VMs | IMPLEMENTED |

Recorded host hardware:
- CPU: AMD Ryzen 7 7435HS
- RAM: 16 GB currently
- GPU: NVIDIA GeForce RTX 3050 Laptop GPU
- Previously recorded free storage: approximately 262 GB

The architecture remains designed for a 32-GB-class host. The current 16-GB constraint is handled through operational profiles and selective VM power-on rather than removing architectural roles.

## 3. Core Cyber-Range Assets

| Asset | Role | Zone | Address | State |
|---|---|---|---|---|
| FW-01 | pfSense firewall/router and zone boundary | Multi-zone | .1 per lab segment | IMPLEMENTED |
| ARCH-01 | Primary attack/operator environment | ATTACK | 10.10.30.100 | IMPLEMENTED |
| DC-01 | Active Directory domain controller, DNS, Global Catalog | ENTERPRISE | 10.10.20.10 | IMPLEMENTED / VALIDATED |
| WIN-01 | Domain-joined Windows enterprise endpoint | ENTERPRISE | 10.10.20.100 | IMPLEMENTED / VALIDATED |
| WAZUH-01 | Security monitoring / Wazuh server | SECURITY/SOC | 10.10.40.10 | IMPLEMENTED / VALIDATED |

## 4. FW-01 — Firewall / Router

**Role:** Central routing and security boundary for the isolated lab zones.

**Platform:** pfSense CE 2.9.0.

**Recorded VM allocation:**
- CPU: 2 vCPU
- RAM: 2 GB
- Disk: 16 GB SCSI
- NICs: 5

**Interfaces:**

| Interface | VMware network | Address / purpose |
|---|---|---|
| WAN | VMnet8 | 192.168.150.0/24 NAT-side network |
| LAN/MGMT | VMnet2 | 10.10.10.1 |
| ENTERPRISE | VMnet3 | 10.10.20.1 |
| ATTACK | VMnet4 | 10.10.30.1 |
| SECURITY | VMnet5 | 10.10.40.1 |

FW-01 provides the established segmentation boundary between the lab zones. The WAN path uses VMware NAT through VMnet8.

## 5. ARCH-01 — Attack / Operator

**Role:** Primary authorized attack and operator environment.

- OS: Arch Linux
- Zone: ATTACK
- IP: 10.10.30.100
- Primary function: controlled adversary simulation, security testing, and operator activity
- State: IMPLEMENTED

ARCH-01 is intentionally separated from the ENTERPRISE and SECURITY/SOC zones by FW-01 policy.

## 6. DC-01 — Domain Controller

**Role:** Enterprise identity, authentication, DNS, and Global Catalog.

**Recorded VM allocation:**
- OS: Windows Server 2025 Standard Evaluation, Desktop Experience
- CPU: 2 vCPU
- RAM: 3 GB
- Disk: 40 GB
- Zone: ENTERPRISE
- IP: 10.10.20.10

**Directory services:**
- AD domain: corp.home.arpa
- NetBIOS: CORP
- DNS: enabled
- Global Catalog: enabled
- Forest functional level: Windows Server 2025
- Domain functional level: Windows Server 2025

**Security telemetry:**
- Wazuh agent: 002
- Sysmon: installed
- Sysmon/Operational channel: explicitly collected by Wazuh
- Validated process creation telemetry: Sysmon Event ID 1
- Validated file creation telemetry: Sysmon Event ID 11
- Windows Security telemetry includes validated 4624, 4625, 4672, 4698, 4720, 4728, 4732, and 4740 events
- Windows System 7045 telemetry has also been validated in the detection sequence

**State:** IMPLEMENTED / VALIDATED.

## 7. WIN-01 — Enterprise Endpoint

**Role:** Domain-joined Windows endpoint used as an enterprise workstation and detection target.

Recorded project state:
- OS: Windows enterprise endpoint
- Zone: ENTERPRISE
- IP: 10.10.20.100
- Domain: corp.home.arpa
- Wazuh agent: 001
- Sysmon: installed
- State: IMPLEMENTED / VALIDATED

WIN-01 participates in the established enterprise telemetry architecture and is available for controlled endpoint scenarios.

## 8. WAZUH-01 — Security Monitoring Server

**Role:** Central Wazuh security-monitoring platform.

**Recorded VM allocation:**
- OS: Ubuntu Server 24.04.5 LTS AMD64
- CPU: 4 vCPU
- RAM: 6 GB
- Disk: 50 GB
- Zone: SECURITY/SOC
- IP: 10.10.40.10

**Software:**
- Wazuh: 4.14.7
- Wazuh indexer: OpenSearch 2.19.5
- Filebeat: 7.10.2
- Wazuh dashboard: HTTPS on 10.10.40.10

**Validated functions:**
- Wazuh manager operational
- OpenSearch/indexer operational
- Windows Security telemetry ingestion
- DC-01 Sysmon telemetry ingestion
- Native Wazuh detection rules
- Project-specific custom detection rules
- Alert search and investigation
- Read-only Indexer correlation prototype used during DET-011

**State:** IMPLEMENTED / VALIDATED.

## 9. VMware Network Assets

| VMware network | Type | Subnet / network | Established role |
|---|---|---|---|
| VMnet0 | Bridged | Real LAN path | Host/LAN connectivity; outside the isolated core |
| VMnet1 | Host-only | 192.168.195.0/24 | Legacy private network |
| VMnet2 | Host-only | 10.10.10.0/24 | MGMT |
| VMnet3 | Host-only | 10.10.20.0/24 | ENTERPRISE |
| VMnet4 | Host-only | 10.10.30.0/24 | ATTACK |
| VMnet5 | Host-only | 10.10.40.0/24 | SECURITY/SOC |
| VMnet8 | NAT | 192.168.150.0/24 | FW-01 WAN / upstream NAT |

Established host-side adapter convention:
- VMnet2: 10.10.10.254
- VMnet3: 10.10.20.254
- VMnet4: 10.10.30.254
- VMnet5: 10.10.40.254

The .254 convention prevents the Windows host-side adapters from competing with FW-01's .1 gateway addresses.

## 10. Identity / Directory Assets

The established AD environment contains:

**Domain**
- corp.home.arpa
- NetBIOS: CORP

**Organizational areas**
- User Accounts
  - IT
  - Finance
  - HR
  - Employees
- Computer Accounts
  - Workstations
  - Servers
- Admins
- Security
- Built-in Domain Controllers remains the domain-controller container.

**Established security groups**
- CORP-IT-Admins
- CORP-SOC-Analysts
- CORP-Helpdesk
- CORP-Server-Admins
- CORP-Workstation-Admins

**Established users referenced in project state**
- alice
- bob
- carol
- david
- CORP\\admin
- Built-in domain Administrator

User/group membership should not be inferred beyond the documented project state.

## 11. Telemetry Relationships

Current endpoint telemetry path:

    DC-01 / WIN-01
          |
      Wazuh Agent
          |
          v
       FW-01
          |
       SECURITY
          |
          v
      WAZUH-01
          |
      +---+---+
      |       |
    Manager  OpenSearch
                |
             Dashboard
                |
        Detection / Investigation

Established Wazuh firewall exceptions permit DC-01 and WIN-01 to reach WAZUH-01 on TCP 1514 and TCP 1515 as required by the current design.

## 12. Current Asset State Summary

### IMPLEMENTED
- Windows 11 host
- VMware Workstation Pro
- FW-01
- ARCH-01
- DC-01
- WIN-01
- WAZUH-01
- VMnet0–VMnet5 and VMnet8 as documented

### VALIDATED
- FW-01 lab routing and segmentation behavior
- DC-01 AD/DNS/Global Catalog operation
- WIN-01 domain membership
- Wazuh manager/indexer operation
- DC-01/WIN-01 Wazuh agent connectivity
- DC-01 Sysmon telemetry ingestion
- Windows Security telemetry ingestion
- Detection sequence DET-001 through DET-018

### NOT DEPLOYED / FUTURE
The following remain target capabilities rather than current assets:
- Additional Windows endpoints
- Additional Linux servers
- Vulnerable web applications
- Splunk or other alternate SIEM platforms
- Suricata
- Zeek
- Dedicated DFIR infrastructure
- Cloud-security infrastructure
- Dedicated malware-analysis infrastructure
- Automation / mini-SOAR infrastructure
- AI-assisted SOC infrastructure

These capabilities remain part of the long-term architecture but are not represented as deployed.

## 13. Inventory Integrity

This inventory is evidence-bound. It does not claim capabilities merely because they appear in the long-term project vision.

Where an asset is marked **IMPLEMENTED / VALIDATED**, implementation and validation are intentionally kept as separate states.

No infrastructure change is implied by this document.
