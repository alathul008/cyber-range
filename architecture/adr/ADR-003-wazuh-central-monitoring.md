# ADR-003 — Wazuh as Current Central Monitoring Platform

**Status:** Accepted / Implemented  
**Decision:** Use Wazuh as the current central security-monitoring and detection platform.

## Context

The range requires centralized Windows telemetry, alerting, search, investigation, and detection engineering.

## Decision

Use WAZUH-01 with:

- Wazuh Manager 4.14.7
- Wazuh Indexer / OpenSearch 2.19.5
- Filebeat 7.10.2
- Wazuh Dashboard

DC-01 and WIN-01 use Wazuh agents.

## Rationale

Wazuh is already implemented and validated in the current range and provides the required current telemetry and detection workflow.

## Consequences

**Benefits**
- Centralized telemetry
- Native Windows detections
- Custom detection capability
- Search/investigation workflow
- Lower infrastructure complexity than deploying multiple SIEMs immediately

**Trade-offs**
- Current architecture depends on Wazuh for centralized monitoring
- Additional SIEM platforms remain future comparative capabilities
- Multi-event correlation requires explicit validation rather than assumption

This ADR does not claim that Wazuh is the only SIEM the target architecture may ever use.
