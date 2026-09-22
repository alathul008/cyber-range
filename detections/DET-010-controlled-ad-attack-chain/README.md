# DET-010 — Controlled AD Account-to-Domain-Admin Attack Chain

## Objective

Validate an end-to-end Active Directory attack chain in the isolated cyber range:

`account creation → account enablement → Domain Admins membership → Wazuh detection → investigation → correlation assessment → cleanup`

The exercise was performed against DC-01 and used a temporary test account.

## Environment

- Domain: `corp.home.arpa`
- Domain controller: DC-01
- DC-01 IP: `10.10.20.10`
- Wazuh agent: DC-01 / agent 002
- Wazuh manager: WAZUH-01 / `10.10.40.10`
- Test account: `det010.test`

## Pre-attack validation

- DC-01 snapshot: `PHASE2-DET010-PREATTACK-DC01`
- WIN-01 snapshot: `PHASE2-DET010-PREATTACK-WIN01`
- DC-01 Sysmon telemetry had already been validated end-to-end.
- `CORP\\admin` was confirmed as a member of the domain `Administrators` group.
- The attack was performed only inside the isolated lab.

## Attack actions

### 1. Create test domain account

The account was created with:

`New-ADUser -Name "DET010-TestUser" -SamAccountName "det010.test" ...`

Verification confirmed:

- Name: `DET010-TestUser`
- SamAccountName: `det010.test`
- Enabled: `True`

### 2. Add test account to Domain Admins

The account was added with:

`Add-ADGroupMember -Identity "Domain Admins" -Members "det010.test"`

Verification confirmed that `det010.test` appeared in the Domain Admins membership.

## Telemetry and detection

### Account creation

Windows Security Event ID **4720** was received from DC-01.

Observed Wazuh data:

- Agent: DC-01 / 002
- Event ID: 4720
- Channel: Security
- Provider: Microsoft-Windows-Security-Auditing
- Target account: `CORP\\det010.test`
- Target SID: `S-1-5-21-2519611076-441997742-1464114610-1117`
- Observed subject: `CORP\\Administrator`
- Wazuh rule: **100104**
- Rule level: **10**
- Wazuh description: `Local user account created on Windows endpoint.`
- Wazuh ATT&CK metadata: T1136.001 / Local Account

**Detection-engineering observation:** the underlying Windows event was a domain account creation event on DC-01, while the native Wazuh rule description/ATT&CK mapping described it as a local account. This should be treated as a labeling/mapping observation, not silently corrected.

### Domain Admins membership

Windows Security Event ID **4728** was received from DC-01.

Observed Wazuh data:

- Agent: DC-01 / 002
- Event ID: 4728
- Channel: Security
- Target group: `Domain Admins`
- Member: `DET010-TestUser`
- Member SID: `S-1-5-21-2519611076-441997742-1464114610-1117`
- Observed subject: `CORP\\Administrator`
- Wazuh rule: **60159**
- Rule level: **12**
- Wazuh description: `Domain Admins Group Changed`
- Wazuh ATT&CK metadata: T1484 / Domain Policy Modification

The Wazuh event was observed at approximately **2026-09-22 15:23:36.813**.

## Investigation result

Searching Wazuh for `det010.test` exposed the account across the individual Windows events.

The observed chain included:

- 4720 — account created
- 4722 — account enabled
- 4728 — added to Domain Admins
- 4738 — account changed

The same target account/SID linked the relevant events.

## Correlation result

The exercise did **not** produce a dedicated automatic DET-009 correlation alert.

Individual detection worked:

- Account creation: detected
- Domain Admins modification: detected

Manual investigation successfully correlated the two events using the same test account/SID.

This confirms the distinction documented in DET-009:

**telemetry availability → individual detection → correlation**

The first two stages worked during this exercise. The final relationship was established during investigation rather than by a demonstrated automatic cross-event Wazuh correlation rule.

No new correlation rule was deployed during DET-010.

## Cleanup

After evidence collection:

1. `det010.test` was removed from Domain Admins.
2. Verification showed no Domain Admins membership for the account.
3. `Get-ADUser "det010.test"` subsequently returned an object-not-found result.
4. The temporary account was therefore fully removed.

## ATT&CK context

The exercise generated telemetry associated with:

- Account creation
- Account manipulation
- Privileged-group membership modification

The Wazuh native mappings observed during the exercise were recorded as evidence rather than replaced with assumed mappings.

## Validation

- [x] Pre-attack snapshots confirmed
- [x] Temporary domain account created
- [x] Account creation telemetry reached Wazuh
- [x] Event 4720 observed
- [x] Account enabled telemetry observed
- [x] Account added to Domain Admins
- [x] Event 4728 observed
- [x] DET-008/native Domain Admins detection observed
- [x] Same account linked across events during investigation
- [x] Automatic DET-009 correlation alert not observed
- [x] Privilege removed
- [x] Temporary account deleted
- [x] Cleanup verified

## Status

**Complete — controlled attack chain validated; correlation gap confirmed by live evidence.**
