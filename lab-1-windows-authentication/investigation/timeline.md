# Investigation Timeline

## Overview

This timeline reconstructs the authentication activity observed during Lab 1 using Windows Security Event Logs collected in Splunk.

The investigation focused on authentication failures, repeated authentication attempts, password-spraying behavior, and subsequent successful authentication.

---

## Baseline

Before intentional authentication activity was generated, a baseline search was performed against Windows Security Event ID 4624.

The observed baseline result was:

| Event ID | Target Account | Logon Type | Count |
|---|---|---:|---:|
| 4624 | SYSTEM | 5 | 1 |

This established that successful authentication telemetry was present before the attack simulations and that the observed baseline event represented a service logon rather than an interactive user authentication.

---

## Failed Authentication

A failed RDP authentication attempt was generated from Kali Linux against `WIN10-WS01`.

| Field | Value |
|---|---|
| Source IP | `192.168.56.102` |
| Target Account | `labuser01` |
| Event ID | `4625` |
| Logon Type | `3` |
| Workstation | `kali` |
| Authentication Package | `NTLM` |
| Status | `0xc000006d` |
| SubStatus | `0xc000006a` |

The collected telemetry demonstrated that the RDP/NLA authentication activity observed during testing was recorded as Logon Type 3.

---

## Repeated Authentication Failures

Multiple authentication failures were subsequently generated against `labuser01`.

The repeated-authentication investigation identified:

| Source IP | Target Account | Logon Type | Failures |
|---|---|---:|---:|
| `192.168.56.102` | `labuser01` | 3 | 5 |

The first and last observed timestamps covered approximately one minute.

This activity was used to validate a threshold-based brute-force detection.

---

## Brute-Force Detection

The five-minute detection search identified the repeated authentication activity:

| Source IP | Target Account | Logon Type | Failures |
|---|---|---:|---:|
| `192.168.56.102` | `labuser01` | 3 | 5 |

The detection threshold was five or more failed authentication events within a five-minute time bucket.

---

## Password-Spraying Activity

Authentication failures were then distributed across multiple target accounts.

The Splunk aggregation identified:

| Source IP | Distinct Accounts | Total Failures | Targeted Accounts |
|---|---:|---:|---|
| `192.168.56.102` | 4 | 9 | `labuser01`–`labuser04` |

Observed time range:

| First Seen | Last Seen |
|---|---|
| `09/25/2026 10:14:03.736` | `09/25/2026 10:16:56.437` |

The activity therefore represented nine failed authentication events against four distinct accounts from the same source over approximately three minutes.

---

## Failed-to-Successful Authentication

A later correlation search for `labuser01` from `192.168.56.102` showed the following sequence:

| Time | Event ID | Account | Source | Logon Type |
|---|---:|---|---|---:|
| 10:59:48 | 4625 | `labuser01` | `192.168.56.102` | 3 |
| 10:59:58 | 4625 | `labuser01` | `192.168.56.102` | 3 |
| 11:00:07.879 | 4625 | `labuser01` | `192.168.56.102` | 3 |
| 11:00:12.546 | 4624 | `labuser01` | `192.168.56.102` | 3 |

The successful authentication occurred approximately five seconds after the final observed failure.

This provided evidence of repeated authentication failures followed by a successful authentication for the same account and source.

---

## Account Lockout

Event ID 4740 was investigated as a relevant authentication detection concept.

No controlled account-lockout event was generated during the lab.

Therefore, no account-lockout event is included in the observed timeline.

---

## Timeline Summary

The investigation demonstrated the following progression:

```text
Baseline authentication
        ↓
Single failed authentication
        ↓
Repeated failures against one account
        ↓
Brute-force detection
        ↓
Failures distributed across multiple accounts
        ↓
Password-spray detection
        ↓
Repeated failures
        ↓
Successful authentication
```
