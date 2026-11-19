# ADR-005 — Expand Cyber Range into an Offensive + Defensive Security Research Platform

- **Status:** ACCEPTED
- **Decision date:** 2026-11-02
- **Decision scope:** Project mission, capability roadmap, and documentation
- **Implementation status:** Scope approved; infrastructure changes not authorized by this ADR

## Context

The Cyber Range has an established isolated four-zone VMware architecture, FW-01 (pfSense), ARCH-01, DC-01, WIN-01, WAZUH-01, OpenSearch, and a documented detection-engineering history through DET-021.

The project began with a strong SOC, detection-engineering, investigation, and purple-team emphasis. The owner has now approved expanding the project mission to include deeper offensive security, exploit development, and independent vulnerability/security research while retaining the existing defensive capabilities.

The expansion must not invalidate completed work, silently alter the deployed architecture, or turn planned capabilities into claims of implementation.

## Decision

The Cyber Range is formally broadened into an **Offensive + Defensive Security Research Cyber Range** with three integrated capability tracks:

1. **Offensive security and red-team operations**
   - Authorized adversary simulation
   - Active Directory and enterprise attack-path analysis
   - Web and application security testing
   - Vulnerability assessment and controlled exploitation
   - Adversary emulation and operational reporting

2. **Exploit development and low-level security**
   - C and systems programming
   - Operating-system and process internals
   - Assembly, debugging, and reverse engineering
   - Memory-safety vulnerability analysis
   - Fuzzing, crash triage, and safe proof-of-concept development

3. **Defensive security and security research**
   - SOC operations, telemetry, and incident investigation
   - Detection engineering and threat hunting
   - Network security and packet/protocol analysis
   - DFIR, incident response, and evidence handling
   - Purple-team validation, replay, and measurement
   - Security tooling, automation, and original technical research

These tracks are integrated through an evidence-driven workflow:

```
Understand → Hypothesize → Reproduce in an authorized lab
     → Collect evidence → Analyze impact
     → Detect / investigate → Improve
     → Replay → Document and publish responsibly
```

## Architectural constraints

- The existing four-zone architecture and asset roles remain authoritative.
- Existing VMs, pfSense rules, VMware networks, routes, permissions, and detection rules are not changed by this scope decision.
- The 16-GB host constraint and 32-GB-class target design remain in force; use existing operational profiles and selective VM activation.
- New VMs, network zones, sensors, targets, and tools are proposed only when a defined experiment requires them and resource/isolation implications are reviewed.
- Intentionally vulnerable targets and adversary tooling remain isolated from VMnet0 / the real LAN.
- Offensive testing is limited to systems owned by the project owner or covered by explicit authorization.
- Current implementation, validated capability, planned capability, and research aspiration must be labelled separately.

## Project sequencing

The existing Stage C task remains the immediate project task: complete the evidence-led AD/WinRM investigation and its validation gate. The Alice Logon ID `0x6e737d` / Wazuh rule `100107` mismatch remains unresolved until supported by evidence. Do not assign DET-022 until its scope and acceptance criteria are explicitly approved.

After the current Stage C gate, capabilities may be developed in dependency order:

1. Foundations: Linux, Windows, networking, programming, and operating-system concepts.
2. Offensive practice: AD, web, host, and network security in controlled targets.
3. Defensive depth: telemetry, detection, hunting, investigation, and replay.
4. Low-level research: C, assembly, debugging, reverse engineering, and fuzzing.
5. Integrated research: original tools, reproducible findings, responsible disclosure, and technical reports.

The sequence is a learning and capability roadmap, not authorization to deploy every technology or run every scenario immediately.

## Alternatives considered

- **Keep the project SOC-only:** rejected because the owner explicitly approved a broader offensive and research mission.
- **Replace the existing lab with a new red-team lab:** rejected because it would discard valuable validated infrastructure and evidence.
- **Deploy all suggested tools and VMs immediately:** rejected because it creates unnecessary resource and isolation risk without a defined requirement.

## Consequences

### Positive
- One coherent platform supports offensive, defensive, and research development.
- Existing detection and investigation work becomes a foundation for purple-team validation.
- New capabilities can be added incrementally with measurable outcomes.
- The repository can demonstrate both practical operations and deeper engineering/research.

### Costs and risks
- Broader scope increases learning and documentation demands.
- Exploit research and vulnerable targets require strict isolation and snapshot/change discipline.
- The 16-GB host requires selective operation; concurrent heavy workloads may not be practical.
- Research findings must be reproduced and responsibly disclosed; no discovery or proficiency outcome is guaranteed.

## Approval and change control

The project owner explicitly approved the scope expansion in conversation on 2026-11-02.

This approval covers the project mission and roadmap only. It does **not** authorize infrastructure, firewall, routing, VMware, VM, account/permission, endpoint configuration, or detection-rule changes. Each material implementation change requires a separate explanation, impact assessment, rollback plan, and explicit approval.

## Acceptance criteria for this decision

- [x] Owner approval recorded.
- [x] Existing architecture and completed DET history explicitly preserved.
- [x] Offensive, exploit-development, and defensive/research tracks defined.
- [x] Immediate Stage C task and DET-022 approval gate preserved.
- [x] Infrastructure changes explicitly excluded from this approval.
- [ ] Roadmap capabilities implemented and individually validated — future work, not completed by this ADR.
