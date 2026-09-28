# ADR-004 — Detection and Correlation Are Separate Engineering Problems

**Status:** Accepted / Implemented  
**Decision:** Treat individual-event detection and multi-event correlation as separate engineering problems.

## Context

The project encountered scenarios where individual Wazuh detections existed but native cross-event correlation did not provide the required evidence relationship.

DET-009 documented this limitation, while DET-011 validated a read-only Indexer correlation prototype.

## Decision

A detection is not considered equivalent to correlation.

Correlation must demonstrate a defensible relationship using evidence such as:

- Logon ID
- account/member SID
- event timestamp
- event ID
- endpoint/agent identity

## Rationale

This prevents a single alert or vendor ATT&CK mapping from being mistaken for proof of a multi-event attack chain.

## Consequences

**Benefits**
- More rigorous detection engineering
- Better investigation pivots
- Explicit correlation validation
- Better distinction between telemetry, detection, and investigation

**Trade-offs**
- Some scenarios require additional engineering
- Correlation prototypes may need separate implementation and validation
- Detection documentation becomes more detailed

DET-019 must follow this principle.
