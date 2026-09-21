# DET-003 — Local Account Creation

## Objective

Detect creation of a local Windows user account on WIN-01 and map the behavior to MITRE ATT&CK.

## Lab Environment

- Endpoint: WIN-01
- Endpoint IP: 10.10.20.100
- SIEM/Manager: WAZUH-01
- Windows Event ID: 4720
- Built-in Wazuh parent rule: 60109
- Custom Wazuh rule: 100104
- MITRE ATT&CK: T1136.001 — Create Account: Local Account

## Controlled Test

A temporary local account was created on WIN-01 using the Windows `net user` command.

The test generated Windows Security Event ID 4720.

Observed target:

- Target account: `DET003-TestUser2`
- Target domain: `DESKTOP-8PB40LJ`
- Creator domain: `CORP`
- Creator user: `Administrator`

No password or secret is documented in this repository.

## Detection Logic

The custom Wazuh rule chains from built-in rule 60109 and requires Windows Event ID 4720:

```xml
<rule id="100104" level="10">
  <if_sid>60109</if_sid>
  <field name="win.system.eventID">4720</field>
  <description>Local user account created on Windows endpoint.</description>
  <mitre>
    <id>T1136.001</id>
  </mitre>
  <group>account_creation,windows_security,persistence,</group>
</rule>
```

## Validation

The controlled replay produced:

- Rule ID: 100104
- Level: 10
- Description: Local user account created on Windows endpoint.
- MITRE technique: T1136.001
- Windows Event ID: 4720

The alert was visible in Wazuh Threat Hunting.

## IOC / IOA / TTP

### IOC

No malicious IOC was established. The created account was a controlled lab test account.

### IOA

Creation of a local Windows account.

### TTP

- MITRE ATT&CK T1136.001 — Create Account: Local Account
- Tactic: Persistence

## Detection Limitation

The current rule identifies Event ID 4720 through Wazuh rule 60109. The current validation proves the rule fires on a local-account creation event, but does not establish that every possible Event ID 4720 is local-account creation.

Future improvement should add contextual conditions such as target domain, creator identity, privileged-group membership, and subsequent authentication activity.

## Evidence

Validation performed on 2026-09-21.

The Wazuh event showed:

- Event ID 4720
- TargetUserName: `DET003-TestUser2`
- TargetDomainName: `DESKTOP-8PB40LJ`
- SubjectUserName: `Administrator`
- SubjectDomainName: `CORP`

## Status

**Validated in the lab.**
