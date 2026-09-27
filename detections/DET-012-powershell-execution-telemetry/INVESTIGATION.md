# DET-012 — Investigation Evidence

## Investigation Objective

Determine what occurred when the controlled PowerShell test executed on DC-01 and identify related telemetry using the PowerShell process identity.

## Initial Process

Controlled command:

```powershell
powershell.exe -NoProfile -Command "Write-Output 'DET012-PowerShell-Test'"
```

Sysmon Event ID 1 identified:

- User: `CORP\admin`
- Process ID: `6424`
- ProcessGuid: `{939ab59d-7551-6ab3-7002-000000001500}`
- Image: `C:\Windows\System32\WindowsPowerShell\v1.0\powershell.exe`
- ParentImage: `C:\Windows\System32\WindowsPowerShell\v1.0\powershell.exe`
- IntegrityLevel: `High`

Wazuh generated Rule 92027:

- Level: 4
- Description: `Powershell process spawned powershell instance`
- ATT&CK: T1059.001

## Investigation Pivot

The ProcessGuid was used to search related Wazuh telemetry:

`{939ab59d-7551-6ab3-7002-000000001500}`

The pivot returned two relevant events:

1. Sysmon Event ID 1 — PowerShell process creation.
2. Sysmon Event ID 11 — file creation by the same PowerShell process.

This established a direct process-to-file telemetry relationship.

## Related File Creation

Sysmon Event ID 11:

```
UtcTime: 2026-09-23 06:44:34.204
ProcessGuid: {939ab59d-7551-6ab3-7002-000000001500}
ProcessId: 6424
Image: C:\WINDOWS\System32\WindowsPowerShell\v1.0\powershell.exe
TargetFilename: C:\Users\admin\AppData\Local\Temp\__PSScriptPolicyTest_xmtobrr0.rs4.ps1
User: CORP\admin
```

Wazuh generated Rule 92213 for this Event ID 11.

## Rule 92213 Assessment

The installed rule matches executable/script extensions created under user AppData Local Temp paths.

The observed `.ps1` policy-test file therefore matched the rule.

The test was intentionally benign. The rule match is consequently treated as an observed false-positive/tuning candidate rather than a confirmed malicious event.

No modification was made to Rule 92213.

## Existing Custom Detection Assessment

The existing custom rules were checked after the DET-012 execution:

- 100101 — no alert observed.
- 100102 — no alert observed.
- 100103 — no alert observed.

This is consistent with the actual command:

- no `-EncodedCommand` / `-enc`
- parent process was PowerShell, not CMD

## SOC Investigation Conclusion

The controlled activity can be reconstructed from telemetry as:

`PowerShell process creation → PowerShell child relationship → temporary .ps1 creation`

The ProcessGuid provided a useful pivot from the initial process event to related file telemetry.

The exercise also demonstrates that a high-severity native alert requires investigation and context before classification.

## Detection Engineering Follow-up

The existing PowerShell rules should remain unchanged from this exercise.

The observed PowerShell → PowerShell execution is recorded as a coverage boundary. A future detection should be created only after defining a specific suspicious behavior and validating its telemetry against both positive and benign cases.
