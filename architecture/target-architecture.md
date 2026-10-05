# Cyber Range Target Architecture

**Status:** Target architecture  
**Current phase:** Phase 2 — AD + Endpoint Telemetry + Detection Engineering  
**Design basis:** Current validated architecture plus explicitly planned project capabilities

This document defines the longer-term architecture without representing planned systems as deployed.

## 1. Target Mission

Preserve the current isolated enterprise/security core while expanding it into a measurable cyber range supporting:

- Red Team
- Blue Team / SOC
- Detection Engineering
- Threat Hunting
- MITRE ATT&CK validation
- Purple Team replay
- DFIR / Incident Response
- Network Security
- Web Security
- Vulnerability Assessment
- Threat Intelligence
- Malware Analysis
- Automation
- Cloud Security
- AI-assisted security workflows

The target architecture grows by capability, not by tool count.

## 2. Architectural Foundation

The current core remains authoritative:

    Windows 11 Host
           |
    VMware Workstation Pro
           |
         FW-01
           |
    +------+------+------+------+
    |             |      |      |
   MGMT       ENTERPRISE ATTACK SECURITY
                  |       |       |
              DC-01     ARCH-01 WAZUH-01
              WIN-01

The four-zone model remains the security foundation.

## 3. Target Capability Layers

### Layer 1 — Control and Segmentation

Current:
- FW-01
- VMnet2–VMnet5
- VMnet8 upstream

Target principle:
- preserve explicit trust boundaries
- add zones only when a demonstrated security or operational requirement exists

### Layer 2 — Enterprise Identity and Endpoints

Current:
- DC-01
- WIN-01

Target additions may include:
- additional Windows endpoints
- additional Windows servers
- additional Linux enterprise servers
- role-specific identities and privilege tiers

These additions are future assets until implemented.

### Layer 3 — Security Monitoring

Current:
- WAZUH-01
- Wazuh Manager
- OpenSearch/Indexer
- Dashboard

Target additions may include:
- additional telemetry sources
- network telemetry
- alternate SIEM/detection platform for comparative exercises
- longer-retention evidence workflows

### Layer 4 — Network Security

Target capability:
- network metadata and packet-oriented visibility
- IDS/NSM exercises
- protocol analysis
- detection validation against network behavior

Potential technologies include Zeek and Suricata, but neither is currently deployed.

### Layer 5 — Adversary Simulation

Current:
- ARCH-01

Target capability:
- multiple attack paths
- credential abuse scenarios
- lateral movement scenarios
- persistence scenarios
- web/application attack scenarios
- controlled replay of ATT&CK behaviors

Every scenario remains isolated and evidence-driven.

### Layer 6 — DFIR / Incident Response

Target capability:
- endpoint evidence acquisition
- timeline construction
- artifact analysis
- incident containment exercises
- evidence preservation
- post-incident detection improvement

Dedicated DFIR infrastructure is not currently deployed.

### Layer 7 — Automation

Target capability:
- Python/API workflows
- enrichment
- alert triage helpers
- repeatable investigation
- controlled response actions
- mini-SOAR workflows

Automation must remain auditable and should not replace evidence collection.

### Layer 8 — Cloud / AI Security

Target capability:
- cloud identity and logging scenarios
- cloud detection engineering
- AI-assisted SOC analysis
- controlled AI enrichment and investigation workflows

These are future capabilities and are not currently deployed.

## 4. Target Security-Zone Model

The current four zones remain the minimum foundation:

| Zone | Current role | Target role |
|---|---|---|
| MGMT | Management | Controlled administration |
| ENTERPRISE | AD/endpoints | Enterprise workload simulation |
| ATTACK | Operator | Adversary simulation |
| SECURITY/SOC | Wazuh | Monitoring, detection, investigation |

Additional zones should be introduced only when they provide a concrete isolation boundary, such as:

- dedicated network-security sensor segment
- dedicated DFIR/evidence segment
- application/web-security segment
- malware-analysis segment

No additional zone is required by the current DET-019 scenario.

## 5. Target Telemetry Architecture

    Enterprise / Attack Activity
              |
       Endpoint telemetry
       Network telemetry
       Identity telemetry
              |
              v
        Security boundary
              |
              v
       Central monitoring
              |
       +------+------+
       |             |
   Detection      Search
       |             |
       +------+------+
              |
       Investigation
              |
       Evidence / IOC
              |
        ATT&CK / TTP
              |
       Detection gap
              |
         Improvement
              |
           Replay

The target architecture retains the current evidence-first workflow.

## 6. Target Detection Engineering Model

Every major scenario should eventually produce:

1. Objective
2. Attack/activity
3. Expected telemetry
4. Observed telemetry
5. Detection
6. SIEM/query/rule
7. ATT&CK mapping
8. IOC/IOA/TTP analysis
9. Investigation
10. Evidence
11. Response
12. Detection gap
13. Improved detection
14. Replay
15. Metrics
16. Incident report

A future capability is not considered complete merely because its software is installed.

## 7. Resource Model

The target is designed for a **32-GB-class host**.

The current 16-GB host remains operational through profiles and selective VM activation.

Future resource decisions must consider:

- RAM
- CPU
- storage
- telemetry retention
- snapshot growth
- concurrent profile requirements

No future VM receives an invented resource allocation in this document.

## 8. Expansion Order

The target should expand in dependency order:

### Stage A — Current core
- FW-01
- ARCH-01
- DC-01
- WIN-01
- WAZUH-01

### Stage B — Architecture hardening
- target architecture documentation
- ADRs
- resource governance
- operational profiles

### Stage C — Detection depth
- DET-019 onward
- correlation
- threat hunting
- detection tuning
- replay and metrics
- **Sequencing decision after DET-021:** deepen the existing AD/endpoint detection model with an AD/lateral-movement scenario before advancing to Stage D network visibility.

### Stage D — Network visibility
- network telemetry
- IDS/NSM
- packet investigation

### Stage E — Additional enterprise/application targets
- additional endpoints
- Linux servers
- vulnerable web applications

### Stage F — DFIR / automation
- evidence workflows
- automation
- mini-SOAR

### Stage G — Advanced domains
- malware analysis
- cloud security
- AI-assisted SOC workflows

The exact order after Stage C may change based on measured prerequisites and project goals. The current post-DET-021 sequencing decision is explicitly documented above; it does not authorize implementation of a new DET number yet.

## 9. Target Architecture Integrity

1. Current and target states must remain separate.
2. Future systems are not recorded as deployed until implemented and validated.
3. Additional infrastructure requires an architectural purpose.
4. New zones require a security-boundary justification.
5. New telemetry must have a detection or investigation purpose.
6. Resource impact must be considered before implementation.
7. Major decisions should be recorded as ADRs.
8. Every completed scenario must retain reproducible evidence.
9. ATT&CK mapping must reflect observed behavior.
10. The target architecture must not weaken the existing isolation model.

## 10. Current Position

Current implemented core:

- FW-01
- ARCH-01
- DC-01
- WIN-01
- WAZUH-01

Current completed detections:

- DET-001 → DET-018

Current detection:

- DET-019 — Privileged Logon → Process Creation Correlation
- Status: NOT STARTED

The target architecture does not require any infrastructure change before DET-019.
