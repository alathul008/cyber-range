# DET-007 — Privileged Local Group Membership Change (Native Wazuh)

## Objective

Validate detection of a local account being added to the Windows Administrators group on WIN-01 and determine whether native Wazuh coverage is sufficient.

## Lab

- Endpoint: WIN-01
- IP: 10.10.20.100
- Windows Event ID: 4732 — A member was added to a security-enabled local group
- Native Wazuh rule: 60154 — Administrators Group Changed
- Native alert level: 12
- Native ATT&CK metadata: T1484

## Controlled Test

A temporary local account was created:

```cmd
net user DET007-TestUser "Lab-Test-2026!" /add
```

The account was then added to the local Administrators group:

```cmd
net localgroup Administrators DET007-TestUser /add
```

Windows generated Event ID 4732.

## Raw Telemetry

The observed 4732 event contained:

- Subject: CORP\Administrator
- Group: Builtin\Administrators
- Group SID: S-1-5-32-544
- Member SID: S-1-5-21-...-1004
- Member Account Name: -

Windows did not resolve the member name in the event; the member SID was the reliable identifier in this event.

## Wazuh Validation

Wazuh ingested Event 4732 and generated native rule 60154:

- Rule ID: 60154
- Description: Administrators Group Changed
- Level: 12
- Group: Builtin\Administrators
- ATT&CK metadata: T1484

## Detection Engineering Decision

No custom DET-007 rule was created.

The native rule already provides specific detection for the security-relevant condition being tested. A custom rule matching the same 4732 + Administrators condition would duplicate existing coverage without adding detection value.

## Investigation Considerations

For a real alert, investigate:

- Who performed the group change.
- Which member SID was added.
- Whether the member is a local or domain account.
- Whether the Administrators group change was authorized.
- Account creation and group-change timing.
- Related process creation telemetry.
- Subsequent privileged activity.

## Cleanup

The temporary test account was deleted:

```cmd
net user DET007-TestUser /delete
```

Verification:

```text
The user name could not be found.
```

The test account therefore no longer exists.

## Validation Result

**PASS — Event 4732 was generated, Wazuh ingested it, native rule 60154 detected the Administrators group change at level 12, no redundant custom rule was deployed, and the temporary account was removed.**

Validation date: 2026-09-21.
