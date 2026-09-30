# DET-020 — Investigation

## 1. Objective

Validate native Wazuh correlation between:

- PowerShell process creation
- PowerShell TCP network connection
- shared Sysmon ProcessGuid

The objective was to establish a repeatable process-to-network correlation primitive for later detection engineering.

## 2. Pre-change Safety

A rollback copy of the live Wazuh local rules file was created before modification:

~~~
sudo cp /var/ossec/etc/rules/local_rules.xml \
  /var/ossec/etc/rules/local_rules.xml.det020-prechange
~~~

Verification showed:

~~~
-rw-r----- 1 root root 3.4K Sep 30 06:13 local_rules.xml.det020-prechange
~~~

No VMware, host networking, firewall, partition, or boot configuration was changed.

## 3. Windows Telemetry Validation

### Event ID 1

Controlled PowerShell execution:

~~~
powershell.exe -NoProfile -Command "Invoke-WebRequest http://example.com -UseBasicParsing | Out-Null"
~~~

Observed Sysmon Event ID 1:

~~~
UtcTime: 2026-09-30 06:03:19.774
ProcessGuid: {939ab59d-a627-6abc-a101-000000001b00}
ProcessId: 1232
Image: C:\Windows\System32\WindowsPowerShell\v1.0\powershell.exe
User: CORP\Administrator
~~~

### Event ID 3

Observed Sysmon Event ID 3 for the same process:

~~~
UtcTime: 2026-09-30 06:03:24.625
ProcessGuid: {939ab59d-a627-6abc-a101-000000001b00}
ProcessId: 1232
Image: C:\Windows\System32\WindowsPowerShell\v1.0\powershell.exe
Protocol: tcp
Initiated: true
SourceIp: 10.10.20.10
DestinationIp: 104.20.23.154
DestinationPort: 80
~~~

The same ProcessGuid was present in both events.

Time difference:

~~~
4.851 seconds
~~~

## 4. Wazuh Telemetry Validation

Wazuh preserved the Event ID 3 field:

~~~
win.eventdata.processGuid
~~~

Example value:

~~~
{939ab59d-9dc9-6abc-0e00-000000001b00}
~~~

This established that ProcessGuid was available to the Wazuh rule engine.

## 5. Native Rule Inspection

The Sysmon Event ID 3 rules were inspected:

~~~
/var/ossec/ruleset/rules/0810-sysmon_id_3.xml
~~~

Relevant rule:

~~~xml
<rule id="92101" level="0">
  <if_group>sysmon_event3</if_group>
  <field name="win.eventdata.image" type="pcre2">(?i)\\powershell\.exe</field>
  <field name="win.eventdata.protocol">^tcp$</field>
  <description>Powershell process communicating over TCP</description>
</rule>
~~~

Because 92101 is level 0, it provides a correlation match without requiring a standalone alert.

The PowerShell Event ID 1 alert was observed as Wazuh rule 92027.

## 6. Correlation Rule

The following rule was added to local_rules.xml:

~~~xml
<rule id="100108" level="12">
  <if_matched_sid>92027</if_matched_sid>
  <same_field>win.eventdata.processGuid</same_field>
  <if_sid>92101</if_sid>
  <description>PowerShell process creation followed by network connection with same ProcessGuid.</description>
  <group>process_network_correlation,powershell,sysmon,windows,</group>
</rule>
~~~

The configuration was tested:

~~~
sudo /var/ossec/bin/wazuh-analysisd -t
~~~

Result: no error output.

The Wazuh manager was restarted and verified:

~~~
sudo systemctl is-active wazuh-manager
~~~

Result:

~~~
active
~~~

## 7. Positive Validation

The controlled PowerShell request was executed again after rule deployment.

The resulting Wazuh alert for rule 100108 was observed:

~~~
Timestamp: 2026-09-30T06:18:53.948+0000
Rule ID: 100108
Level: 12
Description: PowerShell process creation followed by network connection with same ProcessGuid.
~~~

The correlated Event ID 3 contained:

~~~
ProcessGuid: {939ab59d-a9c6-6abc-ac01-000000001b00}
ProcessId: 3080
Image: C:\Windows\System32\WindowsPowerShell\v1.0\powershell.exe
User: CORP\Administrator
DestinationIp: 172.66.147.243
DestinationPort: 80
Initiated: true
EventRecordID: 3831499
~~~

The Wazuh Threat Hunting interface showed exactly one result for:

~~~
agent.name:DC-01 AND rule.id:100108
~~~

during the displayed validation window.

## 8. Negative Validation

A separate HTTP request was generated with curl.exe rather than PowerShell.

Afterward:

~~~
sudo grep '"id":"100108"' /var/ossec/logs/alerts/alerts.json | wc -l
~~~

returned:

~~~
1
~~~

The existing positive alert remained the only 100108 result.

This provides a controlled negative result against a non-PowerShell network process.

## 9. Assessment

### PASS

The detection successfully correlated:

~~~
PowerShell Event 1
        ↓
same ProcessGuid
        ↓
PowerShell Event 3
        ↓
Wazuh 100108
~~~

The negative test did not create an additional 100108 alert.

### Important limitation

This validation proves a telemetry/correlation relationship. It does not prove that the observed PowerShell network activity was malicious.

No malicious IOC, threat actor, or ATT&CK technique is claimed from the benign test itself.

## 10. Metrics

| Metric | Observed result |
|---|---|
| Rule | 100108 |
| Positive correlation alerts | 1 |
| Negative additional alerts | 0 |
| Event 1 → Event 3 example delta | 4.851 s |
| Correlation field | win.eventdata.processGuid |
| Positive Event 3 Record ID | 3831499 |
| Wazuh agent | 002 / DC-01 |

These are controlled validation observations only.

## 11. Evidence Integrity

The documentation records only telemetry, commands, rule behavior, and results actually observed during the controlled lab validation.

No production false-positive rate, detection coverage percentage, or malicious activity is claimed.
