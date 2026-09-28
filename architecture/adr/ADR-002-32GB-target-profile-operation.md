# ADR-002 — 32-GB-Class Target with Profile-Based 16-GB Operation

**Status:** Accepted / Implemented  
**Decision:** Preserve the 32-GB-class target architecture while operating the current 16-GB host through selective VM activation and operational profiles.

## Context

The current Windows 11 host has 16 GB RAM, while the intended cyber range is designed for broader concurrent operation on a 32-GB-class system.

## Decision

Do not reduce the architecture to the current memory constraint.

Use operational profiles such as:

- SOC
- Red Team
- Network Security
- DFIR
- Purple Team

Power on only the assets required by the active profile.

## Rationale

This preserves the intended architecture without requiring every VM to run simultaneously on the current host.

## Consequences

**Benefits**
- Preserves target architecture
- Reduces unnecessary memory pressure
- Supports focused scenarios
- Avoids deleting architectural capabilities

**Trade-offs**
- Full-range simultaneous operation is limited on 16 GB
- Profile transitions require VM lifecycle management
- Heavy purple-team scenarios may require future RAM expansion

No host upgrade is implied by this ADR.
