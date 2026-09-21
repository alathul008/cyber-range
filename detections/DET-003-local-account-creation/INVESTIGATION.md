# DET-003 Investigation

## 1. Objective

Validate detection of a controlled local Windows account creation on WIN-01.

## 2. Test Action

A temporary local account named `DET003-TestUser2` was created on WIN-01 using the Windows `net user` command.

This was an authorized lab activity.

No credentials are recorded in this report.

## 3. Expected Telemetry

Expected Windows Security telemetry:

- Event ID 4720 — A user account was created.
- TargetUserName identifying the created account.
- TargetDomainName identifying the account scope.
- SubjectUserName identifying the account that performed the creation.

## 4. Observed Telemetry

Wazuh received Event ID 4720 from WIN-01.

Observed fields:

| Field | Observed value |
|---|---|
| Agent | WIN-01 |
| Agent IP | 10.10.20.100 |
| Event ID | 4720 |
| TargetUserName | DET003-TestUser2 |
| TargetDomainName | DESKTOP-8PB40LJ |
| SubjectUserName | Administrator |
| SubjectDomainName | CORP |

## 5. Detection

Custom Wazuh rule:

- Rule ID: 100104
- Level: 10
- Parent rule: 60109
- MITRE: T1136.001
- Description: Local user account created on Windows endpoint.

The alert appeared in Wazuh Threat Hunting after the controlled test.

## 6. IOC / IOA / TTP

### IOC

No malicious IOC was identified.

### IOA

Local Windows account creation.

### TTP

T1136.001 — Create Account: Local Account.

The behavior maps to the Persistence tactic.

## 7. Investigation Assessment

The activity was expected because it was intentionally generated as a controlled detection test.

The detection should therefore be treated as a security-relevant signal rather than proof of malicious activity.

Useful investigation context includes:

- Who created the account.
- Whether the account creation was authorized.
- Whether the account receives administrative or privileged-group membership.
- Whether the account subsequently authenticates.
- Whether the account is used for persistence or other suspicious activity.

## 8. Detection Gap

The current rule uses Event ID 4720 and the Wazuh parent rule 60109.

The test confirms the rule fires correctly for the observed local-account creation event.

The test does not prove that the rule can distinguish every local-account creation from every possible account-creation scenario. Additional context should be tested before treating the rule as a high-confidence detection.

## 9. Future Improvement

Potential next-stage enrichment:

1. Identify local versus domain account scope.
2. Correlate account creation with privileged-group membership.
3. Correlate the new account with subsequent logon events.
4. Add creator/user context.
5. Measure false positives from legitimate administrative activity.

## 10. Validation Result

**PASS — controlled Event ID 4720 generated and custom Wazuh rule 100104 fired successfully.**
