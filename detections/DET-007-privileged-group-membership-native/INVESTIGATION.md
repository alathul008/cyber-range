# DET-007 Investigation

## 1. Objective

Validate telemetry and native detection for adding a temporary local account to the Windows Administrators group.

## 2. Test Procedure

Created the temporary account:

```cmd
net user DET007-TestUser "Lab-Test-2026!" /add
```

Added it to Administrators:

```cmd
net localgroup Administrators DET007-TestUser /add
```

## 3. Expected Telemetry

Windows Security Event ID 4732.

## 4. Observed Telemetry

The newest 4732 event showed:

| Field | Value |
|---|---|
| Event ID | 4732 |
| Subject Account | Administrator |
| Subject Domain | CORP |
| Group Name | Administrators |
| Group Domain | Builtin |
| Group SID | S-1-5-32-544 |
| Member SID | S-1-5-21-...-1004 |
| Member Account Name | - |

The member name was not resolved in the event, so the SID was retained as the evidence identifier.

## 5. Native Wazuh Detection

Wazuh produced:

- Rule ID: 60154
- Level: 12
- Description: Administrators Group Changed
- Event ID: 4732
- ATT&CK metadata: T1484

This provides sufficiently specific coverage for the tested condition.

## 6. Detection Engineering Decision

No custom rule was deployed.

A duplicate custom rule for Event 4732 targeting Builtin\Administrators would not add meaningful detection value in this exercise.

## 7. IOC / IOA / TTP

### IOC

No malicious IOC was established. The account and group change were controlled lab artifacts.

### IOA

Addition of an account/member to the local Administrators group.

### TTP

The native Wazuh rule reports T1484 in its metadata. This report records that metadata without treating the controlled lab event itself as proof of malicious activity.

## 8. Analyst Assessment

The activity was intentionally generated to validate privilege-change telemetry.

In a real investigation, determine whether the group modification was authorized and correlate the actor, member SID/account, process lineage, account creation, and subsequent privileged activity.

## 9. Detection Gap

The native rule provides the base alert, but additional correlation could improve triage:

- Correlate account creation with subsequent Administrators membership.
- Resolve member SID to account identity.
- Detect unusual or newly created accounts added to privileged groups.
- Correlate with process creation and logon events.
- Track privileged-group changes across endpoints.

These are future improvements, not findings from this test.

## 10. Cleanup

The temporary account was deleted successfully.

Verification returned:

`The user name could not be found.`

## 11. Validation Result

**PASS — Windows 4732 telemetry and Wazuh native rule 60154 were validated, and the temporary privileged test account was removed.**

Validation date: 2026-09-21.
