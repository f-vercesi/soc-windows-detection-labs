\# SOC Windows Detection Labs



A hands-on SOC portfolio project focused on Windows security monitoring, detection engineering, log analysis, and incident investigation.



The project consists of five progressive labs designed to demonstrate practical SOC Analyst skills using realistic attack scenarios and security telemetry.



\## Labs



| Lab |                       Topic                        |               Primary Telemetry                 |

|-----|----------------------------------------------------|-------------------------------------------------|

|  1  | Windows Authentication Attack Analysis             | Windows Security Logs, Splunk                   |  

|  2  | Suspicious Process Execution and PowerShell Abuse  | Sysmon                                          |

|  3  | Windows Persistence and Local Privilege Indicators | Windows Security Logs, Sysmon                   |

|  4  | Credential Access and Lateral Movement Correlation | Windows Security Logs, Sysmon, Active Directory |

|  5  | Advanced Windows Detection and Investigation       | Multiple telemetry sources                      |


## Current Progress



\*\*Lab 1 — Windows Authentication Attack Analysis\*\*



Completed and documented.



The remaining labs are planned as part of the project's progressive detection and investigation roadmap. 

\## Lab 1 — Windows Authentication Attack Analysis



The first lab focuses on detecting and investigating authentication attacks against a Windows workstation.



The lab covers:



\- Failed authentication

\- Repeated authentication failures

\- Brute-force behavior

\- Password spraying

\- Successful authentication

\- Account lockout detection concepts

\- Authentication timeline reconstruction

\- Splunk detection and investigation

\- False-positive analysis

\- SOC L1 investigation methodology

\- MITRE ATT\&CK mapping



\## Technologies



\- Windows 10 Pro

\- Kali Linux

\- Splunk Enterprise

\- Splunk Universal Forwarder

\- Sysmon

\- Windows Security Event Logs

\- VirtualBox



\## Investigation Approach



The labs follow a practical SOC investigation workflow:



\*\*Alert → Triage → Validation → Evidence Collection → Correlation → Timeline → Scope → Risk Assessment → Disposition → Escalation → Documentation\*\*

