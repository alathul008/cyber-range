# Cyber Range Network Specification

**Status:** Current network architecture documentation  
**Phase:** Phase 2 — AD + Endpoint Telemetry + Detection Engineering  
**Current position:** DET-001 → DET-018 completed; DET-019 not started

This document records the established VMware virtual networks, pfSense interfaces, addressing conventions, DHCP design, firewall segmentation model, and validated network behavior.

It documents the existing design. It does not authorize or perform network changes.

## 1. Network Model

The lab uses VMware Workstation virtual networks with FW-01/pfSense as the routing and security boundary for the four isolated lab segments.

| Network | Type | Subnet | Role | State |
|---|---|---|---|---|
| VMnet0 | Bridged | Real LAN | Host/LAN connectivity | EXISTING / NOT PART OF LAB CORE |
| VMnet1 | Host-only | 192.168.195.0/24 | Legacy private network | EXISTING / LEGACY |
| VMnet2 | Host-only | 10.10.10.0/24 | MGMT | IMPLEMENTED |
| VMnet3 | Host-only | 10.10.20.0/24 | ENTERPRISE | IMPLEMENTED |
| VMnet4 | Host-only | 10.10.30.0/24 | ATTACK | IMPLEMENTED |
| VMnet5 | Host-only | 10.10.40.0/24 | SECURITY/SOC | IMPLEMENTED |
| VMnet8 | NAT | 192.168.150.0/24 | FW-01 WAN/upstream | EXISTING / IMPLEMENTED |

VMnet0, VMnet1, and VMnet8 are deliberately preserved and are not to be conflated with the four isolated lab security zones.

## 2. Addressing Convention

For the four lab segments:

- pfSense gateway: .1
- Windows host-side VMware adapter: .254
- DHCP pool convention: .100–.199
- Static infrastructure addresses are assigned outside the DHCP pool where practical.

Current established addresses:

| Asset | Network | Address | Assignment |
|---|---|---:|---|
| FW-01 LAN/MGMT | VMnet2 | 10.10.10.1 | Static |
| Host adapter | VMnet2 | 10.10.10.254 | Host |
| FW-01 ENTERPRISE | VMnet3 | 10.10.20.1 | Static |
| DC-01 | VMnet3 | 10.10.20.10 | Static |
| WIN-01 | VMnet3 | 10.10.20.100 | Established endpoint address |
| Host adapter | VMnet3 | 10.10.20.254 | Host |
| FW-01 ATTACK | VMnet4 | 10.10.30.1 | Static |
| ARCH-01 | VMnet4 | 10.10.30.100 | Established endpoint address |
| Host adapter | VMnet4 | 10.10.30.254 | Host |
| FW-01 SECURITY | VMnet5 | 10.10.40.1 | Static |
| WAZUH-01 | VMnet5 | 10.10.40.10 | Static |
| Host adapter | VMnet5 | 10.10.40.254 | Host |

The established VMnet8 NAT gateway is 192.168.150.2. FW-01 WAN uses the VMnet8 NAT network.

## 3. VMware Network Construction

### VMnet0 — Bridged

- Bridges to the real LAN through the established TP-Link Wireless USB Adapter.
- Preserved as existing host/LAN connectivity.
- Not used as the enterprise, attack, or security lab segment.
- Intentionally not modified as part of the lab build.

### VMnet1 — Legacy Host-only

- 192.168.195.0/24
- DHCP enabled in the existing configuration.
- Retained for compatibility with existing workloads.
- Not part of the core four-zone architecture.

### VMnet2 — MGMT

- Host-only.
- 10.10.10.0/24.
- VMware DHCP disabled.
- Host adapter: 10.10.10.254.
- FW-01 gateway: 10.10.10.1.
- pfSense DHCP pool: .100–.199.

### VMnet3 — ENTERPRISE

- Host-only.
- 10.10.20.0/24.
- VMware DHCP disabled.
- Host adapter: 10.10.20.254.
- FW-01 gateway: 10.10.20.1.
- pfSense DHCP pool: .100–.199.
- DC-01: 10.10.20.10.
- WIN-01: 10.10.20.100.

### VMnet4 — ATTACK

- Host-only.
- 10.10.30.0/24.
- VMware DHCP disabled.
- Host adapter: 10.10.30.254.
- FW-01 gateway: 10.10.30.1.
- pfSense DHCP pool: .100–.199.
- ARCH-01: 10.10.30.100.

### VMnet5 — SECURITY/SOC

- Host-only.
- 10.10.40.0/24.
- VMware DHCP disabled.
- Host adapter: 10.10.40.254.
- FW-01 gateway: 10.10.40.1.
- pfSense DHCP pool: .100–.199.
- WAZUH-01: 10.10.40.10.

### VMnet8 — NAT

- NAT network.
- 192.168.150.0/24.
- VMware NAT gateway: 192.168.150.2.
- Used as FW-01 WAN/upstream.
- Not part of the isolated internal security zones.

## 4. FW-01 Interface Mapping

FW-01 has five NICs:

| pfSense Interface | Role | VMware Network | IP |
|---|---|---|---|
| em0 | WAN | VMnet8 | NAT/upstream address |
| em1 | LAN/MGMT | VMnet2 | 10.10.10.1/24 |
| em2 | ENTERPRISE | VMnet3 | 10.10.20.1/24 |
| em3 | ATTACK | VMnet4 | 10.10.30.1/24 |
| em4 | SECURITY | VMnet5 | 10.10.40.1/24 |

IPv6 is disabled/not used on the lab interfaces.

## 5. DHCP and DNS Model

pfSense provides DHCP on the lab segments with the established .100–.199 ranges.

The DHCP design provides:

- Interface gateway = pfSense interface address.
- DNS = established interface/DHCP DNS configuration.
- Domain = home.arpa at the pfSense DHCP layer.

Active Directory uses the separate namespace:

- AD DNS domain: corp.home.arpa
- DNS server: DC-01 / 10.10.20.10

This distinction is intentional: home.arpa is the pfSense/DHCP-side local domain convention, while corp.home.arpa is the Active Directory namespace.

WIN-01 was configured to use DC-01 for AD DNS.

## 6. Security Segmentation Model

FW-01 is stateful and evaluates firewall rules in order.

The established model is:

| Source | Destination | Policy |
|---|---|---|
| ATTACK | ENTERPRISE | BLOCK |
| ATTACK | SECURITY | BLOCK |
| ATTACK | MGMT | BLOCK |
| ATTACK | Internet | ALLOW |
| ENTERPRISE | ATTACK | BLOCK |
| ENTERPRISE | MGMT | BLOCK |
| ENTERPRISE | SECURITY | BLOCK by default |
| ENTERPRISE | WAZUH-01:1514 | ALLOW |
| ENTERPRISE | WAZUH-01:1515 | ALLOW |
| SECURITY | MGMT | ALLOW |
| SECURITY | ENTERPRISE | ALLOW |
| SECURITY | ATTACK | ALLOW |
| SECURITY | Internet | ALLOW |

The Wazuh-specific ENTERPRISE → SECURITY exceptions are placed above the general enterprise-to-security block.

WAN filtering retains the established private/bogon blocking model.

## 7. Wazuh Agent Connectivity

The established enterprise-to-security exceptions support Wazuh agent communication:

- WIN-01 10.10.20.100 → WAZUH-01 10.10.40.10 TCP 1514
- WIN-01 → WAZUH-01 TCP 1515
- DC-01 10.10.20.10 → WAZUH-01 10.10.40.10 TCP 1514
- DC-01 → WAZUH-01 TCP 1515

Connectivity was validated during the Wazuh deployment.

## 8. Validated Segmentation Behavior

The network architecture has been validated through controlled tests.

### ATTACK → ENTERPRISE

Cross-zone traffic from ARCH-01 toward the enterprise zone was blocked according to the established firewall policy.

### ENTERPRISE → ATTACK

Enterprise-to-attack traffic was blocked according to the established firewall policy.

### ATTACK → SECURITY

Attack-to-security access was blocked according to the established policy.

### Wazuh Agent Path

Enterprise endpoints successfully communicate with WAZUH-01 through the explicit Wazuh firewall exceptions.

### Internet Path

FW-01 WAN connectivity through VMnet8/NAT was validated.

These tests establish the intended segmentation behavior; they do not constitute a claim of exhaustive firewall assurance.

## 9. Host Adapter Collision Prevention

A previous network issue was caused by the Windows host-side VMnet adapter using the same .1 address intended for pfSense.

The established remediation is:

- pfSense gateways use .1.
- Windows host-side VMware adapters use .254.
- Host-only adapters do not provide default gateways or DNS for the Windows host.

This prevents the Windows host adapter from competing with FW-01 for the lab gateway address.

## 10. Routing Model

The logical routing path is:

    ARCH-01
       |
    VMnet4
       |
    FW-01 / 10.10.30.1
       |
    routing/firewall policy
       |
    destination zone

For enterprise telemetry:

    DC-01 / WIN-01
       |
    VMnet3
       |
    FW-01 / 10.10.20.1
       |
    explicit Wazuh rule
       |
    VMnet5
       |
    WAZUH-01 / 10.10.40.10

For Internet access:

    Lab VM
       |
    FW-01
       |
    VMnet8
       |
    VMware NAT
       |
    192.168.150.2
       |
    Internet

## 11. Safety Constraints

1. Do not bridge intentionally vulnerable systems directly to VMnet0.
2. Do not expose attack targets directly to the real LAN.
3. Do not change VMnet0, VMnet1, or VMnet8 while validating an unrelated detection unless required and explicitly approved.
4. Do not reuse .1 on host-side VMnet adapters.
5. Snapshot before major network changes.
6. Validate routing and firewall behavior after any approved network modification.
7. Preserve rollback information for every significant network change.

## 12. Current vs Future Network Architecture

### Current

The current network is the five-NIC FW-01 design:

VMnet8/WAN → FW-01 → VMnet2/MGMT, VMnet3/ENTERPRISE, VMnet4/ATTACK, VMnet5/SECURITY

### Future

The architecture may eventually introduce additional networks or sensors for:

- expanded enterprise workloads
- vulnerable web applications
- network detection
- DFIR
- malware analysis
- cloud-security exercises
- additional SOC services

Those are target capabilities and are not currently deployed.

## 13. Relationship to Other Documents

- architecture/ARCHITECTURE.md — high-level architecture
- architecture/topology.md — logical/virtual topology
- architecture/network.md — this detailed network specification
- docs/PROJECT-STATE.md — current project state
- docs/DETECTION-ROADMAP.md — detection sequence

Future detailed documents will cover assets, security boundaries, telemetry architecture, operational profiles, resource planning, target architecture, and ADRs.

## Network Integrity Rules

1. Documentation must reflect the established network, not invent changes.
2. VMnet0 must remain distinct from the isolated lab.
3. VMnet8 is an upstream NAT network, not an internal lab security zone.
4. .1 belongs to pfSense gateways on the lab segments; .254 is reserved for Windows host-side adapters.
5. Enterprise-to-security exceptions must remain explicit for Wazuh traffic.
6. Implemented segmentation must not be described as exhaustive security validation.
