
# DET-005 — Windows Service Created with CMD Execution under LocalSystem

## Objective
Detect creation of a Windows service whose service image path executes cmd.exe and whose service account is LocalSystem.

## Lab Environment
- Endpoint: WIN-01
- Endpoint IP: 10.10.20.100
- SIEM/Manager: WAZUH-01
- Windows Event ID: 7045 — A service was installed in the system
- Native Wazuh rule: 61138 — New Windows Service Created
- Custom Wazuh rule: 100106
- MITRE ATT&CK: T1543.003 — Create or Modify System Process: Windows Service
- MITRE ATT&CK: T1059.003 — Command and Scripting Interpreter: Windows Command Shell

## Raw Telemetry
The controlled service creation generated Windows System Event ID 7045 from Service Control Manager.

Observed values:
- Service Name: DET005 Test Service
- Service File Name: C:\Windows\System32\cmd.exe /c exit 0
- Service Type: user mode service
- Service Start Type: demand start
- Service Account: LocalSystem

Wazuh normalized the event into win.eventdata.serviceName, win.eventdata.imagePath, win.eventdata.serviceType, win.eventdata.startType, and win.eventdata.accountName.

## Native Detection
Wazuh native rule 61138 detected Event 7045 as "New Windows Service Created" and mapped it to T1543.003.

The custom rule adds context rather than duplicating the native detection.

## Custom Detection
Rule ID: 100106
Level: 12
Parent rule: 61138

Rule logic:
- win.eventdata.imagePath contains cmd.exe
- win.eventdata.accountName equals LocalSystem

Description: Windows service created with CMD execution under LocalSystem.

MITRE:
- T1543.003
- T1059.003

The rule was syntax-validated before the Wazuh manager restart.

## Positive Validation
A controlled service was created with:
    sc.exe create DET005-TestService binPath= "C:\Windows\System32\cmd.exe /c exit 0" start= demand DisplayName= "DET005 Test Service"

Wazuh Threat Hunting produced one custom alert:
- Rule ID: 100106
- Level: 12
- Description: Windows service created with CMD execution under LocalSystem.
- Event ID: 7045
- Service: DET005 Test Service
- Image path: C:\Windows\System32\cmd.exe /c exit 0
- Account: LocalSystem
- MITRE: T1543.003 and T1059.003

Result: PASS.

## Negative Validation
A second service was created using PowerShell instead of CMD:
    sc.exe create DET005-FP-TestService binPath= "C:\Windows\System32\WindowsPowerShell\v1.0\powershell.exe -NoProfile -Command exit 0" start= demand DisplayName= "DET005 FP Test Service"

The Wazuh query agent.name:WIN-01 AND rule.id:100106 continued to show only the earlier positive 100106 alert.

No new 100106 alert was observed for the PowerShell-based service.

Result: PASS.

The underlying Event 7045/native rule 61138 may still detect the PowerShell service; that is expected.

## IOC / IOA / TTP
### IOC
No malicious IOC was established. Both services were controlled lab artifacts.

### IOA
A Windows service was created with a command-shell executable and LocalSystem service account.

### TTP
- T1543.003 — Windows Service
- T1059.003 — Windows Command Shell

## Investigation Assessment
Service creation can be legitimate. The detection should therefore be treated as a behavioral signal requiring context.

Useful investigation questions:
- Who created the service?
- Is the software or service authorized?
- What binary and arguments are configured?
- Which service account is used?
- Is the binary located in a normal software path?
- Was the service created shortly before suspicious process execution?
- Does the service start automatically?
- Does the resulting process make network connections?
- Is there related process-creation telemetry?

## False Positives
Potential legitimate causes include software installation, software updates, endpoint management agents, security products, IT administration, and enterprise deployment tooling.

The LocalSystem condition increases context but does not establish malicious intent.

## Detection Limitation
The custom rule specifically detects cmd.exe in the service image path together with the LocalSystem account.

It does not cover PowerShell services, services executing other LOLBins, services running under other accounts, malicious service modification after initial creation, or command-line forms not matched by the rule.

Future improvements can correlate Event 7045 with process creation, binary reputation/hash, file location, network activity, and subsequent service execution.

## Cleanup
Both controlled services were removed.

Final verification for DET005-FP-TestService returned Windows error 1060, confirming that the service no longer existed.

The original DET005-TestService had already been deleted before the negative test.

## Validation Result
PASS — Windows Event 7045 was generated and ingested by Wazuh; native rule 61138 detected the event; custom rule 100106 fired for CMD + LocalSystem; the PowerShell negative test did not trigger 100106; and the temporary services were removed.

Validation date: 2026-09-21.
