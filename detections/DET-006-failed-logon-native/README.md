# DET-006 — Windows Failed Logon (Native Wazuh Detection)

## Objective

Validate Windows failed-logon telemetry on WIN-01 and determine whether Wazuh already provides native detection before creating a custom rule.

## Lab

- Endpoint: WIN-01
- IP: 10.10.20.100
- Windows Security Event ID: 4625
- Wazuh agent: 001
- Wazuh native rule: 60122
- Native alert level: 5

## Test

A single controlled local authentication attempt was made with:

`runas /user:WIN-01\david cmd.exe`

The authentication failed because Windows reported that the domain was unavailable.

The test was intentionally limited to one failed attempt.

## Windows telemetry

Event ID 4625 was confirmed in the Windows Security log.

Important fields:

- Target account: david
- Target domain: WIN-01
- Logon type: 2
- Status: 0xC000005E
- Sub-status: 0x0
- Failure reason: %%2304
- Process: C:\Windows\System32\svchost.exe
- Source address: ::1
- Workstation: DESKTOP-8PB40LJ

The event therefore represents a failed authentication attempt, but this specific test does **not** establish a bad-password condition.

## Wazuh validation

Wazuh ingested Event ID 4625 and generated its existing native rule:

- Rule ID: 60122
- Description: Logon Failure - Unknown user or bad password
- Level: 5
- Groups: windows, windows_security, authentication_failed

No custom rule was created because native Wazuh coverage already exists.

## Detection decision

**Custom rule: Not required for this test case.**

Creating another rule that simply matches Event ID 4625 would duplicate existing coverage.

Future detection engineering should focus on higher-confidence context such as:

- repeated failures against one account
- failures against multiple accounts
- unusual source hosts or addresses
- privileged account targeting
- suspicious logon types
- successful authentication following repeated failures
- correlation across multiple endpoints

## Validation status

- Windows Event 4625: PASS
- Wazuh ingestion: PASS
- Native Wazuh rule 60122: PASS
- Custom rule: NOT CREATED — redundant
- Controlled test: PASS
- Cleanup required: NONE
