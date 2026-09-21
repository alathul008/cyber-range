# DET-008 Investigation

## Objective

Validate Active Directory telemetry and native Wazuh detection for adding a temporary domain account to Domain Admins.

## Telemetry Prerequisite

DC-01 was enrolled as Wazuh agent 002. TCP 1514 and 1515 connectivity to WAZUH-01 was validated, and DC-01 Security events were confirmed in Wazuh before the test.

## Test

Created and verified temporary account:

```text
DET008-TestUser
SID: S-1-5-21-2519611076-441997742-1464114610-1115
```

The account was added to Domain Admins.

## Observed Event

Windows Security Event ID 4728: a member was added to a security-enabled global group.

Relevant evidence:

| Field | Value |
|---|---|
| Event ID | 4728 |
| Actor | CORP\Administrator |
| Member | DET008-TestUser |
| Member SID | S-1-5-21-2519611076-441997742-1464114610-1115 |
| Group | Domain Admins |
| Group SID | S-1-5-21-2519611076-441997742-1464114610-512 |
| Group Domain | CORP |

## Wazuh Detection

Native Wazuh rule 60159 fired:

- Description: Domain Admins Group Changed
- Level: 12
- Native ATT&CK metadata: T1484

## Detection Engineering Decision

No custom rule was created because the native rule already provides specific coverage for the tested condition.

## IOC / IOA / TTP

**IOC:** None established; controlled lab artifact.

**IOA:** Addition of a domain account to Domain Admins.

**TTP:** Native Wazuh metadata reports T1484. This is recorded as rule metadata, not as an independent malicious classification.

## Investigation

A real-world alert would require correlation of actor identity, member identity, account creation, authorization/change-management context, authentication events, source process lineage, and subsequent privileged operations.

## Cleanup

The test account was removed from Domain Admins and deleted after evidence collection.

## Validation Result

**PASS — AD group-change telemetry and native Wazuh detection were validated successfully.**

Validation date: 2026-09-21.
