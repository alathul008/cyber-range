# Offensive + Defensive Security Research Roadmap

**Status:** Approved project direction; capabilities are planned unless separately marked implemented and validated.  
**Authority:** ADR-005.  
**Immediate execution priority:** Existing Stage C AD/WinRM evidence investigation remains active.

## 1. Mission

Develop the existing Cyber Range into an integrated environment for:

- Red-team operations and authorized adversary emulation
- Vulnerability assessment and web/application security
- Exploit development and low-level systems security
- Reverse engineering and fuzzing
- SOC operations, detection engineering, and threat hunting
- Network security, packet analysis, and protocol research
- DFIR, incident response, and purple-team validation
- Original security tooling and reproducible technical research

The objective is demonstrable technical capability: explain mechanisms, reproduce results, preserve evidence, assess impact, and communicate findings responsibly. No particular level of expertise or career outcome is guaranteed by a timeline.

## 2. Three integrated tracks

### Track A — Offensive security / red team

**Foundations**
- TCP/IP, DNS, routing, HTTP/TLS, and network services
- Linux and Windows administration and internals
- Python, Bash, and PowerShell
- Identity, authentication, authorization, and access control

**Applied practice**
- Active Directory security and attack-path analysis
- Web application testing using intentionally vulnerable applications
- Linux and Windows privilege-boundary analysis
- Network service assessment
- Authorized adversary emulation and attack-chain reporting

**Expected artifacts**
- Reproducible lab notes
- Attack-path diagrams
- Safe proof-of-concept scripts
- Evidence-backed penetration-test reports
- ATT&CK mapping with explicit evidence caveats

### Track B — Exploit development / low-level security

**Foundations**
- C programming, pointers, memory layout, data structures
- CPU architecture, registers, calling conventions, and x86-64 assembly
- Processes, virtual memory, system calls, and executable formats
- Debugging and compiler/toolchain fundamentals

**Applied research**
- GDB and debugger-assisted program analysis
- Ghidra-based reverse engineering
- Memory-safety and input-validation flaw analysis
- Fuzzing, crash triage, and root-cause analysis
- Mitigation analysis and safe, controlled proof-of-concept development

**Expected artifacts**
- Small C programs and test harnesses
- Debugging/reverse-engineering notes
- Reproducible crash cases
- Root-cause write-ups
- Fuzzing harnesses and minimized test cases

All exploit work must be confined to owned or explicitly authorized targets. Avoid weaponizing findings against third-party systems.

### Track C — Defensive security / security research

**Foundations**
- Windows Security auditing and Sysmon
- Linux logs and host telemetry
- Wazuh/OpenSearch search and correlation
- Network packet and protocol analysis

**Applied practice**
- Detection engineering and alert triage
- Threat hunting and incident timeline reconstruction
- Zeek/Suricata evaluation when Stage D is reached
- DFIR and evidence preservation
- Purple-team replay and detection-quality measurement
- Auditable enrichment and response automation

**Expected artifacts**
- Detection specifications and test cases
- Positive and negative validation evidence
- Queries, scripts, and reproducible investigations
- Incident reports and timeline analyses
- Network analysis reports and detection-gap assessments

## 3. Integrated learning sequence

This sequence is a dependency guide. It does not replace the active project task or imply that all phases must be completed on fixed dates.

| Stage | Focus | Exit evidence |
|---|---|---|
| 0 | Current project continuity | Stage C WinRM evidence gate resolved or explicitly bounded; project state current |
| 1 | Linux, Windows, networking, Python, C foundations | Independently explain and reproduce core system/network behavior |
| 2 | AD, web, host, and network security | Evidence-backed authorized assessments in controlled targets |
| 3 | Red-team and purple-team operations | End-to-end controlled simulation with telemetry and investigation |
| 4 | C, assembly, debugging, reverse engineering | Root-cause analysis of a controlled vulnerable program |
| 5 | Fuzzing and vulnerability research | Reproducible crash triage and technical write-up |
| 6 | Integrated independent research | Original tool or finding with reproducibility and responsible handling |

## 4. Daily and weekly practice model

A sustainable baseline is four focused hours per study day, with optional extension when energy and workload allow.

| Block | Time | Method |
|---|---:|---|
| Concept study | 60 min | Read authoritative documentation and explain concepts in your own words |
| Hands-on lab | 90 min | Reproduce the concept in an isolated environment |
| Independent problem solving | 60 min | Attempt without a walkthrough; document hypotheses and tests |
| Programming/tooling | 30 min | Build or improve a small utility |
| Research log | 30 min | Record evidence, errors, conclusions, and next question |

Use up to two additional hours for deeper research or revision only when sustainable. Keep at least one recovery day each week.

Suggested weekly rhythm:
- Monday: concepts
- Tuesday: implementation
- Wednesday: independent challenge
- Thursday: programming/tooling
- Friday: controlled experiment
- Saturday: review, documentation, and assessment
- Sunday: rest

Consistency is measured by completed focused sessions and reproducible outputs, not by streaks or hours alone.

## 5. First 30-day foundation block

This is a proposed learning block, separate from and subordinate to the immediate Stage C project task.

| Week | Focus | Deliverable |
|---|---|---|
| 1 | Linux processes, permissions, storage, and networking | Annotated system inventory and process/network explanation |
| 2 | TCP/IP and packet analysis | Small TCP client/server plus Wireshark packet walkthrough |
| 3 | Python and C fundamentals | Two small programs with tests and source comments |
| 4 | Windows process and authentication telemetry | Evidence-led explanation of process creation and logon events |

Do not claim completion until the deliverables are produced and reviewed.

## 6. Resource catalogue

Introduce resources only when their subject is active; do not attempt every platform simultaneously.

- PortSwigger Web Security Academy — https://portswigger.net/web-security
- Hack The Box Academy — https://academy.hackthebox.com/
- pwn.college — https://pwn.college/
- OverTheWire — https://overthewire.org/wargames/
- OpenSecurityTraining2 — https://ost2.fyi/
- MITRE ATT&CK — https://attack.mitre.org/
- GDB — https://www.sourceware.org/gdb/
- Ghidra — https://ghidra-sre.org/
- AFL++ — https://aflplus.plus/

Resource use is not itself evidence of proficiency. Record the specific concepts and practical outcomes achieved.

## 7. Cyber Range integration

The existing architecture remains the primary environment:

- ARCH-01: operator workstation, programming, analysis, and authorized testing.
- FW-01: existing segmentation and controlled routing boundary.
- DC-01 and WIN-01: enterprise identity and endpoint scenarios.
- WAZUH-01/OpenSearch: telemetry, detection, search, and investigation.

Potential future targets or capabilities include disposable vulnerable Linux systems, intentionally vulnerable web applications, reverse-engineering samples, fuzzing targets, and network sensors. Each remains planned until approved, deployed, and validated.

## 8. Safety, evidence, and change control

1. Test only owned systems or systems with explicit authorization.
2. Keep vulnerable targets and adversary activity isolated from VMnet0 / the real LAN.
3. Preserve the existing four-zone topology.
4. Do not modify VMware networks, pfSense rules, routing, host networking, permissions, VMs, or detection rules without a separate approval.
5. Before an infrastructure change, document purpose, scope, expected impact, resource use, rollback/snapshot plan, and verification.
6. Separate current, validated, planned, and aspirational capabilities.
7. Preserve raw evidence and record timestamps, identifiers, commands, versions, and query scope where available.
8. Distinguish observed behavior from assumptions and vendor ATT&CK metadata.
9. Document negative/control tests and limitations.
10. Handle potential vulnerabilities through responsible disclosure.

## 9. Progress measurement

Review progress monthly using evidence, not self-comparison.

- Can the concept be explained without notes?
- Can the experiment be reproduced from a clean starting state?
- Can the learner solve a related problem without a walkthrough?
- Is the code understood and tested?
- Are findings supported by raw evidence?
- Are limitations and false positives documented?
- Is the output useful to another technical reader?

A missed day is not a failure. Resume with the next planned session; do not compensate with unsustainable hours.

## 10. Immediate next action

Continue the already-approved Stage C investigation:

1. Reconcile Alice's WinRM timeline using Logon ID `0x6e737d`.
2. Determine whether a matching 4672 prerequisite exists for Wazuh rule 100107.
3. Explain the rule non-match from actual event fields and active configuration.
4. Define the benign control/negative test and acceptance criteria.
5. Seek separate approval before changing any rule, permission, firewall, or network setting.

Do not create DET-022 until scenario scope and acceptance criteria are explicitly approved.
