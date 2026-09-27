# SOC Windows Detection Labs


A hands-on Security Operations Center (SOC) portfolio project focused on Windows detection engineering, log analysis, investigation, and incident triage using realistic attack scenarios.


## Objective


The project consists of five progressive Windows security labs designed to demonstrate practical SOC Analyst L1/L2 skills.


The labs cover:


- Windows authentication monitoring

- Authentication attack detection

- Suspicious process execution

- Persistence detection

- Credential access and lateral movement

- Security event analysis

- Splunk detection and investigation

- MITRE ATT&CK mapping

- Incident investigation and documentation


## Labs


| Lab | Topic | Primary Telemetry |
|---|---|---|
| 1 | Windows Authentication Attack Analysis | Windows Security Logs, Splunk |
| 2 | Suspicious Process Execution and PowerShell Abuse | Sysmon |
| 3 | Windows Persistence and Local Privilege Indicators | Windows Security Logs, Sysmon |
| 4 | Credential Access and Lateral Movement Correlation | Windows Security Logs, Sysmon, Active Directory |
| 5 | Advanced Windows Detection and Investigation | Multiple telemetry sources |


## Current Progress


Lab 1 — Windows Authentication Attack Analysis


Completed and documented.


The remaining labs are planned as part of the project's progressive detection and investigation roadmap.


## Lab 1 — Windows Authentication Attack Analysis


The first lab focuses on detecting and investigating authentication attacks against a Windows workstation.


Scenarios include:


- Failed authentication

- Repeated authentication failures

- Brute-force behavior

- Password spraying

- Successful authentication

- Account lockout detection concepts

- Authentication timeline reconstruction


### Technologies


- Windows 10 Pro

- Kali Linux

- Splunk Enterprise

- Splunk Universal Forwarder

- Sysmon

- Windows Security Event Logs

- VirtualBox


### Skills Demonstrated


- Windows event analysis

- Splunk SPL

- Authentication investigation

- Detection logic

- Attack pattern identification

- Timeline reconstruction

- False-positive analysis

- SOC L1 triage methodology

- MITRE ATT&CK mapping

- Security documentation


---


## Project Philosophy


The labs emphasize the analyst's workflow:


**Alert → Triage → Validation → Evidence Collection → Correlation → Timeline → Scope → Risk Assessment → Disposition → Escalation → Documentation**


The goal is not simply to generate security events, but to demonstrate how those events can be investigated and turned into actionable security findings.
