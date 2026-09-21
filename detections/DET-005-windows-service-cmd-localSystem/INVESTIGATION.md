
# DET-005 Investigation

## 1. Objective
Validate detection of a controlled Windows service creation where the service executes cmd.exe under the LocalSystem account.

## 2. Test Action
A temporary service named DET005-TestService was created on WIN-01:
    sc.exe create DET005-TestService binPath= "C:\Windows\System32\cmd.exe /c exit 0" start= demand DisplayName= "DET005 Test Service"

This was an authorized lab activity.

## 3. Expected Telemetry
Expected Windows telemetry:
- System Event ID 7045
- Service Control Manager provider
- Service name
- Service file/image path
- Service account

## 4. Observed Telemetry
Windows generated Event ID 7045.

Observed values:
| Field | Value |
|---|---|
| Event ID | 7045 |
| Provider | Service Control Manager |
| Service Name | DET005 Test Service |
| Service File Name | C:\Windows\System32\cmd.exe /c exit 0 |
| Service Type | user mode service |
| Start Type | demand start |
| Service Account | LocalSystem |

Wazuh normalized the event and exposed win.eventdata.imagePath and win.eventdata.accountName.

## 5. Native Detection
Wazuh rule 61138 detected the event as "New Windows Service Created" and mapped it to T1543.003.

## 6. Custom Detection
- Rule ID: 100106
- Level: 12
- Parent rule: 61138
- Description: Windows service created with CMD execution under LocalSystem.
- MITRE: T1543.003 and T1059.003

## 7. Positive Validation
Threat Hunting showed one 100106 alert containing:
- Event ID 7045
- Service DET005 Test Service
- Image path C:\Windows\System32\cmd.exe /c exit 0
- Account LocalSystem
- Rule level 12
- MITRE T1543.003
- MITRE T1059.003

Result: PASS.

## 8. Negative Validation
A second service used PowerShell instead of CMD:
    sc.exe create DET005-FP-TestService binPath= "C:\Windows\System32\WindowsPowerShell\v1.0\powershell.exe -NoProfile -Command exit 0" start= demand DisplayName= "DET005 FP Test Service"

The query agent.name:WIN-01 AND rule.id:100106 continued to show only the earlier positive detection.

No new 100106 alert was observed for the PowerShell service.

Result: PASS.

## 9. IOC / IOA / TTP
### IOC
No malicious IOC was established.

### IOA
Windows service creation with cmd.exe execution under LocalSystem.

### TTP
- T1543.003 — Windows Service
- T1059.003 — Windows Command Shell

## 10. Investigation Assessment
The test was intentionally generated and is not evidence of compromise.

For a real alert, investigate:
- Creator identity
- Service name
- Binary path and arguments
- Service account
- Start type
- Binary hash and location
- Related process creation
- Network connections
- Timing relative to other suspicious activity

## 11. Detection Gap
The rule is intentionally narrow. It does not detect PowerShell-based services or every possible service persistence technique.

It also cannot determine whether a detected service is authorized.

## 12. Future Improvement
Potential enrichment:
1. Correlate 7045 with process creation.
2. Add suspicious binary/path logic.
3. Add hash/reputation context.
4. Correlate with network activity.
5. Detect service modifications in addition to creation.
6. Measure legitimate service-creation false positives.

## 13. Cleanup
The temporary services were deleted.

Final verification of DET005-FP-TestService returned Windows error 1060, confirming that the service was no longer installed.

The original positive-test service had already been removed before the negative test.

## 14. Validation Result
PASS — 7045 telemetry, native Wazuh detection, custom 100106 detection, positive validation, negative validation, and cleanup were all completed.

Validation date: 2026-09-21.
