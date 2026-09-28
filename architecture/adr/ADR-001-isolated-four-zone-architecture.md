# ADR-001 — Isolated Four-Zone VMware Architecture

**Status:** Accepted / Implemented  
**Decision:** Use FW-01/pfSense as the central routing and security boundary for four isolated lab zones.

## Context

The range requires separation between management, enterprise systems, adversary simulation, and security infrastructure while retaining controlled upstream connectivity.

## Decision

Use:

- VMnet2 — MGMT — 10.10.10.0/24
- VMnet3 — ENTERPRISE — 10.10.20.0/24
- VMnet4 — ATTACK — 10.10.30.0/24
- VMnet5 — SECURITY/SOC — 10.10.40.0/24

FW-01 provides the zone gateways and stateful policy boundary.

VMnet8 provides the FW-01 WAN/upstream NAT path. VMnet0 remains the real-LAN bridged network and is outside the isolated core.

## Rationale

This creates an explicit security boundary and allows controlled telemetry paths without directly bridging adversarial systems to the real LAN.

## Consequences

**Benefits**
- Clear trust boundaries
- Controlled inter-zone routing
- Explicit telemetry exceptions
- Repeatable network-security validation

**Trade-offs**
- Additional routing/firewall complexity
- More VMware virtual networks
- Requires careful change control

## Validation

Project evidence validates selected blocked inter-zone paths, permitted Wazuh traffic, and the Internet path.

This ADR does not claim exhaustive firewall assurance.
