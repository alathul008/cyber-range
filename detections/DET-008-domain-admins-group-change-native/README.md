# DET-008 — Domain Admins Group Change (Native Wazuh)

## Objective

Validate detection of a domain account being added to the Active Directory Domain Admins group and determine whether native Wazuh coverage is sufficient.

## Observed Telemetry

- Domain Controller: DC-01
- Wazuh Agent ID: 002
- Windows Event ID: 4728
- Member: DET008-TestUser
- Group: Domain Admins
- Actor: CORP\Administrator
- Native Wazuh rule: 60159
- Native alert level: 12
- Native ATT&CK metadata: T1484

## Controlled Test

A temporary domain account named DET008-TestUser was created and added to Domain Admins. Membership was verified before detection validation.

The Wazuh alert was confirmed in /var/ossec/logs/alerts/alerts.json for agent DC-01.

Relevant evidence included:

- Member SID: S-1-5-21-2519611076-441997742-1464114610-1115
- Group SID: S-1-5-21-2519611076-441997742-1464114610-512
- Event ID: 4728
- Rule ID: 60159
- Description: Domain Admins Group Changed
- Level: 12

## Detection Engineering Decision

No custom DET-008 rule was created.

Native Wazuh coverage already specifically detects the Domain Admins group change tested here. A duplicate custom rule would not add meaningful detection value.

## IOC / IOA / TTP

**IOC:** None established; this was a controlled benign test.

**IOA:** Addition of a domain account to Domain Admins.

**TTP:** Wazuh's native rule reports T1484 in its metadata. This report records that metadata without independently classifying the benign lab action as malicious.

## Investigation Considerations

For a real alert, investigate the actor, member identity, account creation history, authorization, related logons, process lineage, and subsequent privileged activity.

## Cleanup

The temporary account was removed from Domain Admins and deleted after evidence collection.

## Validation Result

**PASS — DC-01 telemetry, Event 4728, and native Wazuh rule 60159 were validated successfully.**

Validation date: 2026-09-21.
