\# Lab 1 — Windows Authentication Attack Analysis



\## Objective



Detect and investigate authentication attacks against a Windows workstation using Windows Security Event Logs and Splunk.



The lab focuses on identifying and investigating:



\- Failed authentication

\- Repeated authentication attempts

\- Brute-force behavior

\- Password spraying

\- Successful authentication

\- Authentication timelines

\- Authentication-related detection logic

\- False positives and detection limitations



\---



\## Scenario



A Windows workstation is monitored for suspicious authentication activity originating from a separate Kali Linux system.



The investigation focuses on determining:



\- Which account was targeted?

\- Which source generated the authentication attempts?

\- How many failures occurred?

\- Were multiple accounts targeted?

\- Did authentication eventually succeed?

\- What authentication information was recorded by Windows?

\- What was the timeline of activity?

\- Could the activity have a legitimate explanation?



\---



\## Architecture



The lab uses an isolated VirtualBox host-only network.



\### Systems



| System | Role | IP Address |

|---|---|---|

| Windows 11 Host | Splunk Enterprise | `192.168.56.1` |

| WIN10-WS01 | Authentication target | `192.168.56.101` |

| Kali Linux | Attack simulation source | `192.168.56.102` |



\### Authentication and Telemetry Flow



```text

Kali Linux

192.168.56.102

&#x20;     |

&#x20;     | RDP / TCP 3389

&#x20;     v

WIN10-WS01

192.168.56.101

&#x20;     |

&#x20;     | Windows Security Logs

&#x20;     | Sysmon

&#x20;     v

Splunk Universal Forwarder

&#x20;     |

&#x20;     | TCP 9997

&#x20;     v

Splunk Enterprise

192.168.56.1

&#x20;     |

&#x20;     v

index=lab1\_windows

```



\---



\## Telemetry



\### Windows Security Event Log



The primary authentication telemetry comes from the Windows Security event log.



Relevant events investigated in this lab include:



| Event ID |     Description    |                Lab Usage                 |

|----------|--------------------|------------------------------------------|

|   4624   | Successful logon   | Investigated successful authentication   |

|   4625   | Failed logon       | Primary authentication failure telemetry |

|   4740   | Account locked out | Investigated as a detection concept only |



\### Logon Types



Relevant Windows Logon Types include:



| Logon Type |           Description              |

|------------|------------------------------------|

|      3     | Network                            |

|     10     | RemoteInteractive / Remote Desktop |

|      5     | Service                            |



The lab did \*\*not\*\* assume that the use of RDP would necessarily produce Logon Type 10.



During the actual RDP authentication testing, Windows recorded the failed authentication events as \*\*Logon Type 3\*\*.



This was an important observation because the authentication was performed through RDP using Network Level Authentication (NLA). Therefore, the Logon Type must be interpreted together with the other event fields rather than being inferred solely from the application or protocol involved.



\### Sysmon



Sysmon was configured and successfully ingested into Splunk.



Sysmon Event ID 1 (Process Create) was verified as available for additional investigation context.



\---



\## Splunk



Lab telemetry was collected in:



```text

index=lab1\_windows

```



Primary sourcetypes:



```text

XmlWinEventLog:Security

XmlWinEventLog:Microsoft-Windows-Sysmon/Operational

```



The Splunk Universal Forwarder collected the Windows Security and Sysmon logs from `WIN10-WS01` and forwarded them to Splunk Enterprise on the Windows host.



\---



\# Attack Scenarios



\## 1. Baseline Authentication Activity



Before generating intentional authentication failures, a baseline was established.



The observed baseline contained a successful Event ID 4624 associated with:



```text

TargetUserName: SYSTEM

LogonType: 5

```



This demonstrated an important investigation principle:



> Event ID 4624 does not necessarily represent an interactive human login.



The observed Logon Type 5 represented service authentication activity.



No baseline RDP authentication activity was identified.



\---



\## 2. Single Failed Authentication



A failed RDP authentication attempt was generated from Kali Linux against:



```text

Target: labuser01

Source: 192.168.56.102

```



Windows generated Event ID 4625.



The observed event contained:



```text

EventID:                    4625

TargetUserName:             labuser01

LogonType:                  3

LogonProcessName:           NtLmSsp

AuthenticationPackageName:  NTLM

WorkstationName:            kali

IpAddress:                  192.168.56.102

Status:                     0xc000006d

SubStatus:                  0xc000006a

```



The event represented an authentication failure caused by an incorrect password.



\### Status Codes



The observed values were:



\- `0xc000006d` — general logon failure

\- `0xc000006a` — incorrect password



The important analyst fields were the target account, source IP, Logon Type, authentication package, workstation, and failure status.



\---



\## 3. Repeated Authentication Failures



Multiple failed authentication attempts were generated against `labuser01`.



The resulting telemetry demonstrated a pattern in which:



```text

One source

&#x20;     |

&#x20;     v

One target account

&#x20;     |

&#x20;     v

Multiple authentication failures

```



This represents the behavioral pattern that a detection could use to identify potential brute-force activity.



The lab then used Splunk aggregation and threshold-based searches to identify repeated failures.



\---



\## 4. Password Spraying



Authentication attempts were distributed across multiple accounts from the same source.



The observed accounts were:



```text

192.168.56.102

&#x20;     |

&#x20;     +── labuser01

&#x20;     +── labuser02

&#x20;     +── labuser03

&#x20;     +── labuser04

```



Each account received a small number of failed authentication attempts.



This demonstrated the behavioral distinction between repeated attacks against one account and distributed authentication attempts against multiple accounts.



\### Behavioral distinction



\*\*Brute force:\*\*



```text

One source

→ One account

→ Many attempts

```



\*\*Password spraying:\*\*



```text

One source

→ Many accounts

→ Few attempts per account

```



This distinction is important because a detection based only on repeated failures against an individual account may miss password-spraying activity.



\---



\## 5. Successful Authentication



Successful authentication telemetry was generated for `labuser01`.



The observed Event ID 4624 included:



```text

TargetUserName:   labuser01

LogonType:        3

IpAddress:        192.168.56.102

WorkstationName:  kali

```



This demonstrated that the same authentication telemetry can contain both failed and successful authentication events and that successful authentication should be considered during investigation.



The lab also investigated correlation between Event IDs 4625 and 4624 using source IP and target account.



The repository does not claim that a specific individual failed attempt was definitively followed by a specific successful attempt unless supported by the recorded timeline.



\---



\## 6. Account Lockout



Event ID 4740 was investigated as a relevant Windows authentication detection concept.



However, \*\*no controlled account-lockout event was generated during this lab\*\*.



Therefore:



\- No 4740 event is presented as experimental evidence.

\- No account-lockout scenario is claimed as completed.

\- 4740 is documented only as a relevant detection and investigation concept.



In a real SOC investigation, a 4740 event could be correlated with preceding 4625 events to investigate the source, target account, timing, and potential cause of the lockout.



\---



\# Detection



The detection logic focuses on behavioral patterns rather than treating every authentication failure as malicious.



\## Failed Authentication



Event ID 4625 was used as the primary telemetry source for authentication failures.



Important fields included:



\- `TargetUserName`

\- `IpAddress`

\- `WorkstationName`

\- `LogonType`

\- `AuthenticationPackageName`

\- `Status`

\- `SubStatus`

\- `\_time`



\---



\## Brute-Force Detection



The brute-force detection looks for multiple failed authentications associated with the same target account and source.



Conceptually:



```text

One source

\+

One target account

\+

Multiple failures

\+

Relevant time period

```



A threshold-based SPL search was used to identify groups with five or more failures.



\---



\## Password-Spray Detection



The password-spray detection examines the number of distinct accounts targeted by a source.



Conceptually:



```text

One source

\+

Multiple target accounts

\+

Authentication failures distributed across accounts

```



The lab used the distinct target-account count to identify this pattern.



This approach addresses a limitation of per-account thresholds: an attacker can distribute attempts across multiple accounts while keeping each individual account below a brute-force threshold.



\---



\# Investigation Workflow



The investigation follows a practical SOC L1 workflow:



```text

Alert

&#x20; ↓

Triage

&#x20; ↓

Validation

&#x20; ↓

Evidence Collection

&#x20; ↓

Correlation

&#x20; ↓

Timeline Reconstruction

&#x20; ↓

Scope Assessment

&#x20; ↓

Risk Assessment

&#x20; ↓

Disposition

&#x20; ↓

Escalation / Response

&#x20; ↓

Documentation

```



\### Triage



The analyst examines:



\- Source IP

\- Target account

\- Event ID

\- Logon Type

\- Authentication package

\- Failure status

\- Number of attempts

\- Time range



\### Validation



The analyst verifies that:



\- The events are present in the Windows Security log.

\- Splunk extraction is correct.

\- Source and target information match the observed activity.

\- The authentication result is understood correctly.

\- The activity is not immediately explained by legitimate behavior.



\### Correlation



Authentication events can be correlated using:



\- Source IP

\- Target account

\- Timestamp

\- Event ID

\- Logon Type

\- Workstation name

\- Authentication result



The investigation should also consider successful authentication following failed attempts.



\### Scope



The analyst determines whether the activity involves:



\- One account

\- Multiple accounts

\- One source

\- Multiple sources

\- A short burst

\- A prolonged low-volume pattern

\- Successful authentication

\- Account lockout



\---



\# False Positives



Authentication failures are not inherently malicious.



Potential benign explanations include:



\- User-entered password mistakes

\- Stale credentials

\- Legitimate administrative activity

\- Automated services or scheduled processes

\- Misconfigured applications



An L1 analyst should therefore investigate the surrounding context rather than escalate every individual Event ID 4625.



Important questions include:



1\. Who was targeted?

2\. Where did the request originate?

3\. How many failures occurred?

4\. Were multiple accounts targeted?

5\. Did authentication eventually succeed?

6\. Is the source expected?

7\. Is there related activity in other telemetry?



\---



\# Detection Limitations



\## Threshold-Based Detection



An attacker can remain below a fixed threshold by slowing authentication attempts.



\## Password Spraying



Per-account thresholds may fail to detect password spraying because each account may receive only a small number of attempts.



Distinct-account analysis can help identify this behavior.



\## Authentication Failures Are Not Automatically Malicious



Event ID 4625 indicates an authentication failure, not malicious intent.



Context is required.



\## Logon Type Interpretation



Logon Type should not be interpreted in isolation.



This lab specifically demonstrated that the RDP/NLA authentication activity observed during testing generated \*\*Logon Type 3\*\*, despite RDP commonly being associated with Logon Type 10 for RemoteInteractive sessions.



\## Limited Authentication Context



Authentication events provide important information but may not explain activity following authentication.



Additional telemetry such as Sysmon, EDR, PowerShell logging, and network telemetry may be required for deeper investigation.



\## Account Lockout



Account lockout behavior was not experimentally validated because no Event ID 4740 was generated during the lab.



\---



\# MITRE ATT\&CK



Relevant MITRE ATT\&CK techniques will be mapped here based on the documented attack behaviors.



Final technique IDs and descriptions will be verified against the current MITRE ATT\&CK framework before publication.



\---



\# Evidence



Evidence is organized under:



```text

evidence/

```



The repository prioritizes analyst-facing evidence such as:



\- Splunk detection results

\- Event IDs

\- Source IPs

\- Target accounts

\- Logon Types

\- Authentication results

\- Timestamps

\- Detection output



Raw XML is included only where it provides useful validation or demonstrates an important underlying event field.



\---



\# Findings



The final findings will document:



\- Authentication patterns observed

\- Source and target relationships

\- Repeated authentication behavior

\- Password-spraying behavior

\- Successful authentication telemetry

\- Detection results

\- False-positive considerations

\- Detection limitations



\---



\# Conclusion



This lab demonstrates a practical SOC workflow for detecting and investigating Windows authentication activity using Windows Security logs and Splunk.



The investigation progressed from baseline authentication telemetry to individual authentication failures, repeated authentication behavior, password-spraying analysis, successful authentication telemetry, and behavioral detection logic.



The lab also demonstrated the importance of validating assumptions against actual telemetry. In particular, the RDP/NLA authentication activity observed during testing was recorded as Logon Type 3 rather than Logon Type 10.



\---



\# Lessons Learned



\- Event IDs should be interpreted in context rather than isolation.

\- Event ID 4625 provides valuable failed-authentication telemetry.

\- Event ID 4624 provides successful-authentication telemetry.

\- Source IP and target-account distribution are critical investigation dimensions.

\- Brute-force and password-spraying behavior require different detection approaches.

\- Per-account thresholds can miss distributed password-spraying activity.

\- Logon Type should be validated against actual telemetry rather than assumed from the protocol.

\- Raw event data is useful for validation, while extracted fields are more efficient for routine investigation.

\- Additional telemetry is required to investigate activity occurring after authentication.

\- Detection limitations and untested scenarios should be documented explicitly rather than presented as completed tests.

