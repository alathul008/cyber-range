# Cyber Range

**Enterprise-Style Cybersecurity Detection, Investigation & Purple-Team Laboratory**

A practical, isolated cybersecurity range built with VMware Workstation Pro on Windows 11. The project is designed around measurable security workflows rather than a collection of disconnected tools.

> **Current status:** DET-001 → DET-021 completed. DET-021 is the latest completed detection-engineering objective; the next major capability is now selected from the established architecture and roadmap.

---

## Mission

Build a realistic, repeatable environment for developing practical skills across:

- Security Operations (SOC)
- Detection Engineering
- Threat Hunting
- MITRE ATT&CK
- Active Directory security
- Red Team / adversary simulation
- Purple Team validation
- DFIR and Incident Response
- Network Security
- Vulnerability Assessment
- Web Security
- Threat Intelligence
- Security automation
- Cloud and AI-assisted security workflows

The range follows an evidence-driven lifecycle:

```
Attack / Controlled Activity
        ↓
Telemetry
        ↓
Detection
        ↓
Triage
        ↓
Investigation
        ↓
MITRE ATT&CK / IOC / IOA / TTP
        ↓
Response
        ↓
Detection Gap
        ↓
Detection Improvement
        ↓
Replay
        ↓
Metrics / Report
```

---

## Current Architecture

The implemented core is built around four isolated security zones behind **FW-01 (pfSense)**:

| Zone | VMware Network | Subnet | Current Role |
|---|---|---|---|
| MGMT | VMnet2 | 10.10.10.0/24 | Management |
| ENTERPRISE | VMnet3 | 10.10.20.0/24 | AD and endpoints |
| ATTACK | VMnet4 | 10.10.30.0/24 | Attack/operator activity |
| SECURITY/SOC | VMnet5 | 10.10.40.0/24 | Security monitoring |

Upstream connectivity uses **VMnet8 NAT**. VMnet0 is the real-LAN bridged network and is outside the isolated lab core. VMnet1 is retained as a legacy host-only network.

### Current Assets

| Asset | Role | Address |
|---|---|---|
| FW-01 | pfSense firewall/router | Zone gateways |
| ARCH-01 | Arch Linux attack/operator box | 10.10.30.100 |
| DC-01 | Windows Server 2025 AD/DNS/GC | 10.10.20.10 |
| WIN-01 | Domain-joined Windows endpoint | 10.10.20.100 |
| WAZUH-01 | Ubuntu/Wazuh security server | 10.10.40.10 |

Domain:

```
corp.home.arpa
NetBIOS: CORP
```

### Monitoring Stack

The current security-monitoring platform is Wazuh:

- Wazuh Manager 4.14.7
- Wazuh Indexer / OpenSearch 2.19.5
- Filebeat 7.10.2
- Wazuh Dashboard
- Wazuh agents on DC-01 and WIN-01
- Sysmon on DC-01 and WIN-01

Validated Windows telemetry includes Security events such as 4624, 4625, 4672, 4698, 4720, 4728, 4732, 4740, 7045 and Sysmon Event IDs 1 and 11.

---

## Detection Engineering

The project does not treat an alert as proof of an attack chain.

Detection and correlation are deliberately treated as separate engineering problems.

### Completed

**DET-001 → DET-018 — COMPLETED**

The detection sequence covers areas including:

- PowerShell encoded-command detection
- CMD → PowerShell parent-child execution
- Account creation
- Scheduled task creation
- Windows service creation
- Failed logons
- Privileged group changes
- Domain Admins changes
- Multi-event account/privilege correlation exercises
- PowerShell process telemetry
- Privileged logon telemetry
- Account lockout
- Successful-logon investigation
- Successful logon → special-privilege telemetry

Where evidence supports it, detections are mapped to MITRE ATT&CK. Vendor/native ATT&CK metadata is kept distinct from what the observed evidence actually proves.

### Current

**DET-019 — Privileged Logon → Process Creation Correlation**

The intended investigation chain is:

```
4624 Successful Logon
        ↓
4672 Special Privileges
        ↓
Sysmon Event ID 1
        ↓
Process Creation
        ↓
Wazuh
        ↓
Logon-ID/session correlation
        ↓
Investigation
```

DET-019 has **not** been executed. Its ATT&CK mapping will be determined from observed behavior rather than pre-claimed.

---

## Validation Philosophy

A component is not considered complete merely because software is installed.

Major scenarios are validated through:

1. Objective
2. Controlled activity
3. Expected telemetry
4. Observed telemetry
5. Detection
6. SIEM query/rule
7. ATT&CK mapping
8. IOC/IOA/TTP analysis
9. Investigation
10. Evidence
11. Response
12. Detection gap
13. Detection improvement
14. Replay
15. Metrics
16. Documentation

Only work actually performed in the range is documented as completed.

---

## Operational Profiles

The host currently has **16 GB RAM**, while the architecture is designed for a **32-GB-class** system.

The range therefore uses selective VM activation through operational profiles:

- **SOC**
- **Red Team**
- **Network Security**
- **DFIR**
- **Purple Team**

Powering off a VM when it is not required does not remove it from the architecture.

---

## Repository Structure

```
cyber-range/
├── architecture/
│   ├── ARCHITECTURE.md
│   ├── topology.md
│   ├── network.md
│   ├── asset-inventory.md
│   ├── security-boundaries.md
│   ├── telemetry-architecture.md
│   ├── operational-profiles.md
│   ├── resource-plan.md
│   ├── target-architecture.md
│   └── adr/
│       ├── README.md
│       ├── ADR-001-isolated-four-zone-architecture.md
│       ├── ADR-002-32GB-target-profile-operation.md
│       ├── ADR-003-wazuh-central-monitoring.md
│       └── ADR-004-detection-vs-correlation.md
├── detections/
│   ├── DET-001-*/
│   ├── DET-002-*/
│   └── ...
└── docs/
    ├── DETECTION-ROADMAP.md
    └── PROJECT-STATE.md
```

The architecture documents contain the system-level design. Detection directories contain the evidence and engineering artifacts for individual scenarios. `PROJECT-STATE.md` is the cross-chat project handoff.

---

## Architecture Documentation

Start here:

- [Architecture Overview](architecture/ARCHITECTURE.md)
- [Topology](architecture/topology.md)
- [Network Specification](architecture/network.md)
- [Asset Inventory](architecture/asset-inventory.md)
- [Security Boundaries](architecture/security-boundaries.md)
- [Telemetry Architecture](architecture/telemetry-architecture.md)
- [Operational Profiles](architecture/operational-profiles.md)
- [Resource Plan](architecture/resource-plan.md)
- [Target Architecture](architecture/target-architecture.md)
- [Architecture Decision Records](architecture/adr/README.md)

---

## Detection Documentation

- [Detection Roadmap](docs/DETECTION-ROADMAP.md)
- [Project State](docs/PROJECT-STATE.md)

Individual DET artifacts live under `detections/`.

---

## Roadmap

### Current
- AD + endpoint telemetry
- Wazuh/Sysmon
- Detection engineering
- DET-001 → DET-021 completed
- DET-019 privileged-logon → process correlation validated
- DET-020 PowerShell process → network correlation validated
- DET-021 PowerShell download telemetry boundary documented

### Next capability layers

The target architecture may expand into:

- deeper event correlation
- threat hunting
- network telemetry
- Zeek / Suricata
- additional enterprise endpoints and servers
- vulnerable web applications
- DFIR workflows
- security automation / mini-SOAR
- malware-analysis workflows
- cloud-security scenarios
- AI-assisted SOC workflows

These are **planned capabilities**, not claims of current deployment.

---

## Safety and Isolation

The range is designed for authorized, controlled experimentation.

The adversarial/attack environment must remain isolated from the real LAN. The architecture uses FW-01 and dedicated VMware networks to establish controlled boundaries.

Network, firewall, host, storage, or VM changes should be treated as controlled infrastructure changes with rollback/snapshot planning where appropriate.

---

## Project Standards

This repository follows several evidence standards:

- Do not fabricate test results.
- Do not claim unvalidated capabilities.
- Do not confuse installed software with validated functionality.
- Do not treat vendor ATT&CK metadata as proof of observed behavior.
- Preserve negative/benign validation where performed.
- Keep current and target architecture separate.
- Never commit passwords, API keys, tokens, private keys, or personal information.

---

## Current Project State

| Item | Status |
|---|---|
| VMware core architecture | Implemented |
| pfSense segmentation | Implemented / validated selected paths |
| Active Directory | Implemented / validated |
| Windows endpoint telemetry | Implemented / validated |
| Wazuh | Implemented / validated |
| Sysmon | Implemented / validated |
| DET-001 → DET-018 | Completed |
| DET-019 | Current / not started |
| Target architecture | Documented |
| Architecture ADR layer | Established |
| DET-019 pre-attack snapshot | Not created |

---

## Portfolio Focus

This repository is intended to demonstrate **genuine hands-on security engineering**.

The emphasis is on:

**depth → evidence → repeatability → measurable validation → documentation**

rather than the number of tools installed.

