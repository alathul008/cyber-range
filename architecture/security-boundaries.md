# Cyber Range Security Boundaries

**Status:** Current architecture documentation  
**Phase:** Phase 2 — AD + Endpoint Telemetry + Detection Engineering  
**Basis:** Established topology, network specification, asset inventory, and validated segmentation tests

This document defines the security boundaries of the currently established Cyber Range. It documents the existing isolation model and validated traffic relationships. It does not authorize or perform infrastructure changes.

## 1. Boundary Model

The Cyber Range has four isolated lab security zones behind FW-01:

| Zone | Network | Trust / function boundary |
|---|---|---|
| MGMT | VMnet2 / 10.10.10.0/24 | Management plane |
| ENTERPRISE | VMnet3 / 10.10.20.0/24 | Identity and endpoint systems |
| ATTACK | VMnet4 / 10.10.30.0/24 | Authorized adversary/operator activity |
| SECURITY/SOC | VMnet5 / 10.10.40.0/24 | Security monitoring and investigation |

FW-01 is the routing and stateful firewall boundary between these zones.

VMnet0 is the real-LAN bridged path and is outside the isolated core. VMnet8 is the VMware NAT-side upstream for FW-01 WAN. VMnet1 is a retained legacy host-only network and is not part of the core four-zone security architecture.

## 2. Primary Isolation Boundary

The primary safety boundary is:

    REAL LAN
       |
     VMnet0
       |
    Windows Host
       |
    VMware Workstation
       |
      VMnet8
       |
    FW-01 WAN
       |
    +--+-----------------------------+
    |                                |
   FW-01                           Lab zones
    |                                |
    +-- MGMT                         |
    +-- ENTERPRISE                   |
    +-- ATTACK                        |
    +-- SECURITY/SOC                  |

Intentionally vulnerable or adversarial lab systems must remain on the isolated lab networks and must not be bridged directly to VMnet0.

The established design uses FW-01 as the control point for inter-zone routing and policy enforcement.

## 3. Zone Boundaries

### MGMT — 10.10.10.0/24

Purpose: management connectivity.

- Gateway: 10.10.10.1
- Host adapter: 10.10.10.254
- No claim is made that all future management workflows are implemented.

### ENTERPRISE — 10.10.20.0/24

Purpose: enterprise identity and endpoint systems.

Current assets:
- DC-01 — 10.10.20.10
- WIN-01 — 10.10.20.100

The enterprise zone contains the Active Directory domain controller and domain-joined endpoint.

### ATTACK — 10.10.30.0/24

Purpose: controlled adversary simulation and operator activity.

Current asset:
- ARCH-01 — 10.10.30.100

The attack zone is intentionally separated from enterprise and security infrastructure.

### SECURITY/SOC — 10.10.40.0/24

Purpose: monitoring, detection, alert search, and investigation.

Current asset:
- WAZUH-01 — 10.10.40.10

The security zone receives explicitly permitted telemetry from enterprise endpoints through FW-01.

## 4. Inter-Zone Policy Model

The established policy model is:

| Source | Destination | Established behavior |
|---|---|---|
| ATTACK | ENTERPRISE | BLOCK |
| ATTACK | SECURITY | BLOCK |
| ATTACK | MGMT | BLOCK |
| ENTERPRISE | ATTACK | BLOCK |
| ENTERPRISE | MGMT | BLOCK |
| ENTERPRISE | SECURITY | BLOCK by default, with explicit Wazuh exceptions |
| SECURITY | MGMT | ALLOW |
| SECURITY | ENTERPRISE | ALLOW |
| SECURITY | ATTACK | ALLOW |
| SECURITY | Internet | ALLOW |
| ATTACK | Internet | ALLOW |
| ENTERPRISE | Internet | ALLOW |

Firewall processing is stateful and rule order is significant. The documented policy is the established design; it is not a claim of exhaustive firewall assurance.

## 5. Explicit Telemetry Boundary Exceptions

Enterprise endpoints require access to WAZUH-01 for the current monitoring architecture.

Established exceptions:

| Source | Destination | Protocol / port | Purpose |
|---|---|---|---|
| WIN-01 | WAZUH-01 | TCP/1514 | Wazuh agent communication |
| WIN-01 | WAZUH-01 | TCP/1515 | Wazuh agent enrollment/communication |
| DC-01 | WAZUH-01 | TCP/1514 | Wazuh agent communication |
| DC-01 | WAZUH-01 | TCP/1515 | Wazuh agent enrollment/communication |

These exceptions are intentionally narrower than unrestricted ENTERPRISE → SECURITY access.

## 6. WAN Boundary

FW-01 WAN is connected to VMnet8:

- VMnet8: 192.168.150.0/24
- VMware NAT gateway: 192.168.150.2
- FW-01 WAN uses the NAT-side network.

The isolated lab zones are not directly attached to the real LAN. Internet access is routed through FW-01 and VMware NAT.

VMnet8 is an upstream transport network, not an internal enterprise trust zone.

## 7. Host Boundary

The Windows host has VMware host-side adapters on the lab host-only networks using .254:

- VMnet2 — 10.10.10.254
- VMnet3 — 10.10.20.254
- VMnet4 — 10.10.30.254
- VMnet5 — 10.10.40.254

These host-side adapters do not use default gateways or DNS for the Windows host.

This addressing convention was established after a previous collision where a host-side .1 address competed with the pfSense gateway .1 address. The current .254 convention prevents that specific gateway collision.

## 8. Validated Boundary Behavior

The following controlled behaviors have been validated:

### ATTACK → ENTERPRISE

ARCH-01 traffic toward the enterprise zone was blocked by FW-01 policy.

### ENTERPRISE → ATTACK

Enterprise traffic toward the attack zone was blocked by FW-01 policy.

### ATTACK → SECURITY

Attack-zone access toward WAZUH-01/security infrastructure was blocked by FW-01 policy.

### Enterprise → WAZUH-01

The explicit Wazuh exceptions permit required endpoint-to-manager communication. DC-01 and WIN-01 successfully communicated with WAZUH-01 through the established path.

### Internet

FW-01 WAN connectivity through VMnet8/VMware NAT was validated.

These are controlled validation results, not an exhaustive security assessment.

## 9. Boundary Failure Modes

The established architecture specifically guards against these known failure conditions:

1. **Direct bridging of attack targets to VMnet0**  
   Would bypass the intended isolated lab boundary.

2. **Host adapter using a pfSense gateway address**  
   Can create ARP/gateway conflicts. The established convention reserves .1 for FW-01 and .254 for host adapters.

3. **Unrestricted Enterprise → Security access**  
   Would weaken the intended monitoring-zone boundary. Current Wazuh connectivity is provided through explicit exceptions.

4. **Changing VMnet0/VMnet8 without review**  
   Could alter the relationship between the isolated range, real LAN, and Internet path.

5. **Treating segmentation tests as exhaustive firewall validation**  
   The current tests cover selected intended paths only.

## 10. Change-Control Requirements

Before modifying VMware networking, FW-01 interfaces, firewall policy, host networking, or other boundary controls:

1. Record the intended change.
2. Identify the affected security boundary.
3. Preserve rollback information.
4. Take a VMware snapshot when appropriate.
5. Apply the smallest required change.
6. Validate both the intended path and relevant blocked paths.
7. Record the result in the architecture or scenario documentation.

No boundary modification is required for DET-019 at this stage.

## 11. Current Boundary Summary

    VMnet0 / REAL LAN
            |
       [HOST BOUNDARY]
            |
         VMnet8
            |
        [FW-01 WAN]
            |
    +-------+--------+---------+---------+
    |                |         |         |
  MGMT          ENTERPRISE   ATTACK   SECURITY
10.10.10/24    10.10.20/24 10.10.30/24 10.10.40/24
                   |             |          |
              DC-01/WIN-01   ARCH-01    WAZUH-01
                   |
             explicit Wazuh
                exceptions

The security boundary model is therefore based on:

**Host isolation → FW-01 routing/firewall → zone separation → explicit service exceptions → validated telemetry path.**

## 12. State and Scope

**IMPLEMENTED:** The four-zone FW-01 boundary and established VMware network layout.

**VALIDATED:** Selected inter-zone blocks, Wazuh exceptions, and Internet path.

**NOT CLAIMED:** Exhaustive firewall testing, future network sensors, additional zones, or future security infrastructure.

No infrastructure change is implied by this document.
