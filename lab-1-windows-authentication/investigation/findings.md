# Investigation Findings

## Executive Summary

The investigation demonstrated that Windows Security Event Logs, when collected in Splunk and analyzed using source, target, frequency, and time-based fields, can identify several authentication attack patterns.

The lab successfully demonstrated:

- Failed authentication detection using Event ID 4625
- Repeated authentication analysis
- Threshold-based brute-force detection
- Password-spraying behavior across multiple accounts
- Successful authentication using Event ID 4624
- Correlation of repeated failures followed by successful authentication
- Interpretation of authentication telemetry based on observed event fields

Account lockout behavior was not experimentally validated because no Event ID 4740 was generated.

---

## Finding 1 — Failed Authentication

Event ID 4625 successfully captured the failed authentication activity generated from Kali Linux.

The observed event contained:

- Target account: `labuser01`
- Source IP: `192.168.56.102`
- Workstation: `kali`
- Logon Type: `3`
- Authentication package: `NTLM`
- Status: `0xc000006d`
- SubStatus: `0xc000006a`

The combination of source, target, authentication method, and failure status provided sufficient context for initial L1 triage.

---

## Finding 2 — RDP/NLA Authentication Was Recorded as Logon Type 3

An important observation from the lab was that the RDP authentication activity generated during testing was recorded as Logon Type 3 rather than Logon Type 10.

This demonstrates that an analyst should not classify authentication activity solely from the application or protocol being used.

The actual Windows telemetry should be validated and interpreted using the complete event context.

---

## Finding 3 — Repeated Authentication Against One Account

Five failed authentication events were observed against `labuser01` from `192.168.56.102` with Logon Type 3 within approximately one minute.

This behavior was used to validate a threshold-based brute-force detection.

The detection identified the activity when the number of failures reached the configured threshold of five events within a five-minute time bucket.

---

## Finding 4 — Password-Spraying Behavior

The password-spray investigation identified:

- Source: `192.168.56.102`
- Distinct target accounts: `4`
- Total failed authentications: `9`
- Targeted accounts: `labuser01`–`labuser04`
- First observed: `09/25/2026 10:14:03.736`
- Last observed: `09/25/2026 10:16:56.437`

The activity demonstrated the characteristic distribution of authentication failures across multiple accounts from the same source.

This is significant because a detection that only counts failures against an individual account could fail to identify distributed authentication attempts.

---

## Finding 5 — Failed Authentication Followed by Successful Authentication

A correlation search identified three failed authentication events followed by a successful authentication for the same account and source:

```text
10:59:48       4625
10:59:58       4625
11:00:07.879   4625
11:00:12.546   4624
```

All four events involved:

```text
Target: labuser01
Source: 192.168.56.102
Logon Type: 3
```

The successful authentication occurred approximately five seconds after the final observed failure.

This demonstrates why successful authentication should be considered when investigating repeated authentication failures.

---

## Finding 6 — Account Lockout Was Not Validated

Event ID 4740 was considered as part of the authentication investigation model, but no controlled lockout event was generated.

The lab therefore does not make any experimental claim regarding account-lockout behavior.

In a production SOC, a 4740 event could be correlated with preceding 4625 events to investigate the source, target account, timing, and potential cause of the lockout.

---

## False-Positive Considerations

The observed authentication events were generated intentionally as part of a controlled lab.

In a production environment, similar telemetry could have legitimate explanations, including:

- User password mistakes
- Stale credentials
- Legitimate administrative activity
- Automated services
- Misconfigured applications

Event ID 4625 should therefore be treated as an investigation signal rather than proof of malicious activity.

---

## Detection Limitations

The implemented detections have several limitations.

### Fixed Thresholds

A threshold-based brute-force detection may miss low-and-slow attacks that remain below the configured threshold.

### Distributed Authentication

Password spraying can evade per-account thresholds by distributing attempts across multiple accounts.

### Event Context

Authentication events alone may not provide sufficient information to determine what happened after successful authentication.

Additional telemetry such as Sysmon, EDR, PowerShell logging, or network telemetry may be required.

### Logon Type Assumptions

The lab demonstrated that assumptions about Logon Type can be incorrect.

The observed RDP/NLA authentication telemetry was recorded as Logon Type 3.

---

## Analyst Takeaways

The main investigation lessons from this lab were:

1. **Start with the authentication event and establish the source and target.**
2. **Do not treat every 4625 as malicious.**
3. **Look for repetition and temporal patterns.**
4. **Compare one-account behavior with multi-account behavior.**
5. **Use distinct-account analysis to identify potential password spraying.**
6. **Investigate successful authentication following repeated failures.**
7. **Validate Logon Type against the actual telemetry.**
8. **Correlate authentication events with additional telemetry when investigating possible compromise.**
9. **Document untested scenarios rather than presenting them as validated detections.**
