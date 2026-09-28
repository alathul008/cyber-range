# Cyber Range Telemetry Architecture

**Status:** Current architecture documentation  
**Phase:** Phase 2 — AD + Endpoint Telemetry + Detection Engineering  
**Basis:** Established architecture, topology, network specification, asset inventory, security boundaries, and validated telemetry evidence

This document describes the telemetry architecture currently established in the Cyber Range. It separates telemetry sources, transport, ingestion, processing, storage/search, detection, and investigation. It does not claim capabilities that have not been validated.

## 1. Telemetry Mission

The telemetry architecture exists to support the project workflow:

    Controlled Activity
           ↓
    Endpoint / AD Telemetry
           ↓
      Wazuh Ingestion
           ↓
    Detection / Alerting
           ↓
    Search / Investigation
           ↓
    Evidence / Correlation
           ↓
    Detection Improvement
           ↓
       Replay / Validation

Telemetry is treated as security evidence, not merely log collection.

## 2. Current Telemetry Sources

| Source | Asset | Telemetry | State |
|---|---|---|---|
| Windows Security | DC-01 | Authentication, account, group, task, lockout and related security events | VALIDATED |
| Windows Security | WIN-01 | Windows security telemetry | VALIDATED |
| Sysmon | DC-01 | Process creation, file creation and other configured Sysmon events | VALIDATED |
| Sysmon | WIN-01 | Broad endpoint telemetry | VALIDATED |
| Wazuh agent | DC-01 | Agent transport and endpoint collection | VALIDATED |
| Wazuh agent | WIN-01 | Agent transport and endpoint collection | VALIDATED |

Validated Windows Security events currently include:

- 4624 — successful logon
- 4625 — failed logon
- 4672 — special privileges assigned
- 4698 — scheduled task creation
- 4720 — account creation
- 4728 — Domain Admins membership change
- 4732 — local Administrators membership change
- 4740 — account lockout
- 7045 — service installation

Validated Sysmon telemetry includes:

- Event ID 1 — process creation
- Event ID 11 — file creation

DC-01 Sysmon/Operational is explicitly configured for Wazuh collection.

## 3. Endpoint Collection Layer

### DC-01

DC-01 has:

- Windows Security event collection
- Sysmon installed
- Sysmon/Operational collection through Wazuh
- Wazuh agent 002

The DC-01 telemetry path is validated end-to-end into WAZUH-01.

### WIN-01

WIN-01 has:

- Windows Security event collection
- Sysmon installed
- Wazuh agent 001

WIN-01 participates in the established Wazuh monitoring architecture.

## 4. Wazuh Agent Transport

The current telemetry transport is:

    DC-01 / WIN-01
           |
      Wazuh Agent
           |
       ENTERPRISE
           |
         FW-01
           |
       SECURITY/SOC
           |
       WAZUH-01

The established firewall exceptions permit DC-01 and WIN-01 to reach WAZUH-01 on:

- TCP/1514
- TCP/1515

These are explicit Enterprise → Security exceptions rather than unrestricted Enterprise → Security access.

## 5. WAZUH-01 Processing Layer

WAZUH-01 is the current central security-monitoring platform.

Recorded platform components:

| Component | Current state |
|---|---|
| Wazuh Manager | IMPLEMENTED / VALIDATED |
| Wazuh Indexer | IMPLEMENTED / VALIDATED |
| OpenSearch | IMPLEMENTED / VALIDATED |
| Filebeat | IMPLEMENTED / VALIDATED |
| Wazuh Dashboard | IMPLEMENTED / VALIDATED |

Recorded versions:

- Wazuh: 4.14.7
- OpenSearch/Wazuh Indexer: 2.19.5
- Filebeat: 7.10.2
- Ubuntu Server: 24.04.5 LTS AMD64

WAZUH-01 address: 10.10.40.10

## 6. Ingestion and Normalization

The established ingestion model is:

    Windows Event Log / Sysmon
              ↓
          Wazuh Agent
              ↓
        Wazuh Manager
              ↓
        Decoder / Rule
              ↓
            Alert
              ↓
      Indexer / OpenSearch
              ↓
     Dashboard / Investigation

Validated DC-01 Sysmon alerts show Wazuh parsing Windows Event Channel data and exposing fields including:

- provider
- event ID
- channel
- computer
- image
- command line
- parent image
- parent command line
- user
- integrity level
- SHA256

The exact field structure should be treated as evidence-backed and queried from observed Wazuh documents rather than assumed from generic Sysmon schemas.

## 7. Detection Layer

The detection layer contains two established categories:

### Native Wazuh detections

Native Wazuh rules are preferred when they already provide the required coverage.

Validated examples include native detection for:

- successful/failed logons
- special privileges
- scheduled task creation
- local Administrators membership changes
- Domain Admins membership changes
- service installation
- account lockout

### Project-specific detections

Custom rules are used where additional contextual value was demonstrated.

The project currently documents DET-001 through DET-018 as completed.

The detection sequence includes custom rules, native Wazuh coverage, investigation pivots, and correlation exercises.

## 8. Investigation and Correlation Layer

Detection and correlation are treated as separate engineering problems.

Established investigation pivots include:

- Windows Logon ID
- account SID
- member SID
- event ID
- event record ID
- timestamp
- endpoint/agent identity

DET-018 validated:

    Event 4624
          ↓
    Logon ID 0x2c5499
          ↓
    Event 4672
          ↓
    Same privileged session

DET-011 also contains a validated read-only Python Indexer correlation prototype. It is not persistent native Wazuh correlation.

DET-009 demonstrated an account-creation → Domain Admin membership correlation exercise using a shared SID and timestamps. The project explicitly documented the distinction between detection and unsupported native cross-event correlation rather than deploying an unverified rule.

## 9. Evidence Model

The telemetry architecture supports evidence collection at several levels:

| Evidence layer | Examples |
|---|---|
| Raw Windows telemetry | Security/System events |
| Raw Sysmon telemetry | Sysmon/Operational events |
| Wazuh parsed data | Decoded event fields |
| Wazuh alerts | Native/custom rule matches |
| Search results | OpenSearch/Indexer queries |
| Correlation evidence | Shared Logon ID/SID/time relationships |
| Investigation artifacts | Detection READMEs and investigation records |

The project documentation rule is evidence-bound: completed work must be supported by actual project evidence.

## 10. Telemetry-to-MITRE Workflow

MITRE ATT&CK is applied after observing the behavior represented by telemetry.

The established workflow is:

    Activity
       ↓
    Telemetry
       ↓
    Detection
       ↓
    Investigation
       ↓
    Observed behavior
       ↓
    ATT&CK technique mapping

The project does not pre-claim an ATT&CK technique for a scenario when the actual observed behavior has not yet been established.

This is particularly important for DET-019, whose ATT&CK mapping remains intentionally undetermined until its test is executed.

## 11. Telemetry and Network Boundaries

Telemetry crosses the established security boundary only through explicit permitted paths.

    ENTERPRISE
    DC-01 / WIN-01
          |
      Wazuh Agent
          |
        FW-01
          |
    SECURITY/SOC
       WAZUH-01

The attack zone does not have unrestricted access to WAZUH-01.

This preserves the separation between adversary simulation and security infrastructure while allowing enterprise telemetry to reach the SOC.

## 12. Current Telemetry Validation

Validated capabilities include:

- Wazuh manager operation
- Wazuh indexer/OpenSearch operation
- Wazuh dashboard operation
- WIN-01 agent connectivity
- DC-01 agent connectivity
- Windows Security telemetry ingestion
- DC-01 Sysmon telemetry ingestion
- Sysmon Event ID 1 process creation visibility
- Sysmon Event ID 11 file creation visibility
- Wazuh native detection
- Project-specific Wazuh detection rules
- OpenSearch/Indexer investigation
- Logon ID/SID-based correlation exercises

DC-01 Sysmon telemetry was explicitly validated after adding the Microsoft-Windows-Sysmon/Operational event channel to the Wazuh agent configuration.

## 13. Current Telemetry Baseline

The project has a documented DC-01 telemetry baseline under:

detections/telemetry-baseline-dc01/README.md

The baseline records actual observed Sysmon activity and explicitly distinguishes noisy baseline telemetry from intentional attack activity.

The baseline is evidence for telemetry availability, not a claim that all telemetry is already tuned for production-quality detection.

## 14. Known Telemetry Gaps

The current architecture does not claim:

- Zeek telemetry
- Suricata telemetry
- Splunk telemetry
- Additional network sensors
- Dedicated DFIR collection infrastructure
- Cloud telemetry
- Dedicated malware-analysis telemetry
- Persistent native Wazuh correlation for every multi-event scenario
- Exhaustive event-channel coverage beyond what has been validated

These are future capabilities or engineering opportunities, not current telemetry assets.

## 15. Current vs Target Telemetry Architecture

### Current

    DC-01 ──┐
             ├─ Wazuh Agents ── FW-01 ── WAZUH-01
    WIN-01 ─┘                         |
                                      +─ Manager
                                      +─ Indexer/OpenSearch
                                      +─ Dashboard
                                      |
                                Detection / Investigation

### Target

The longer-term architecture may add:

- network telemetry
- additional endpoints
- additional enterprise servers
- alternate SIEM/detection platforms
- DFIR collection
- cloud telemetry
- malware-analysis telemetry
- automation and response telemetry
- AI-assisted SOC workflows

These remain target capabilities and are not represented as deployed.

## 16. Telemetry Integrity Rules

1. Do not claim telemetry that has not been validated.
2. Distinguish source telemetry from Wazuh alerts.
3. Distinguish detection from correlation.
4. Preserve observed field names when writing queries or detections.
5. Use stable investigation pivots where supported.
6. Map ATT&CK from observed behavior rather than assumption.
7. Record negative validation and false positives when actually observed.
8. Do not treat a broad telemetry baseline as a tuned production detection configuration.
9. Do not represent future sensors or SIEMs as deployed.
10. Preserve evidence required to reproduce each detection.

## 17. Current Project Position

**Phase:** Phase 2 — AD + Endpoint Telemetry + Detection Engineering

**Completed:** DET-001 → DET-018

**Current:** DET-019 — Privileged Logon → Process Creation Correlation

**DET-019 status:** NOT STARTED

No DET-019 telemetry has been generated specifically for the scenario.

No DET-019 ATT&CK mapping has been claimed.

No DET-019 persistent detection or correlation rule has been deployed.

No DET-019 pre-attack snapshot has been created.

No infrastructure change is implied by this document.
