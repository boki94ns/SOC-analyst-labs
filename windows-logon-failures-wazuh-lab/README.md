# Windows Logon Failures Detection with Wazuh

## Project Overview

This lab demonstrates how failed Windows logon attempts can be generated in a controlled environment and detected using Wazuh SIEM.

The goal of this project is to document the full detection flow:

1. Generating failed logon attempts on a Windows endpoint
2. Recording the events in Windows Security logs
3. Collecting the logs with the Wazuh agent
4. Correlating the events in Wazuh
5. Triggering an alert for multiple failed Windows logon attempts
6. Mapping the activity to the MITRE ATT&CK Brute Force technique

> This lab was created for educational purposes in a local, controlled environment.

---

## Lab Environment

The lab environment consisted of:

- Windows endpoint with Wazuh agent installed
- Wazuh manager / server
- Wazuh dashboard
- PowerShell running as Administrator
- Windows Security Event Logs

---

## Scenario

In this scenario, multiple failed logon attempts were simulated on a Windows system by using PowerShell and the `runas` command.

Each attempt used a non-existing local username in the following format:

```text
hacker_1.
hacker_2.
hacker_3.
hacker_4.
hacker_5.
```

Because the accounts do not exist or the credentials are invalid, Windows generates failed logon events. These events are collected by Wazuh and correlated into an alert for multiple Windows logon failures.

---

## 1. Generating Failed Logon Attempts

PowerShell was opened as Administrator on the Windows endpoint. A loop was then executed to start the `cmd` process as several different invalid users.

The command uses `runas` to generate multiple failed authentication attempts within a short period of time.

```powershell
1..5 | ForEach-Object {
    runas /user:hacker_$_. cmd 2>&1
    Start-Sleep -Milliseconds 500
}
```

This simulates behavior similar to a brute-force attempt, without attacking any real system or network service.

![PowerShell command](assets/Slika%201.png)

---

## 2. Windows Returns Invalid Credential Errors

After the command is executed, Windows prompts for a password for each user.

Since the users `hacker_1.` through `hacker_5.` do not have valid credentials, each attempt fails with the following error:

```text
RUNAS ERROR: Unable to run - cmd
1326: The user name or password is incorrect.
```

This confirms that the logon attempts failed and that Windows generated the corresponding audit events.

![Failed runas attempts](assets/Slika%202.png)

---

## 3. Repeated Failed Logon Attempts

The output shows multiple failed logon attempts in sequence. Each attempt uses a different username, but all attempts are executed locally on the same Windows endpoint.

This pattern is important from a detection perspective, because multiple failed logons within a short time window may indicate credential guessing or brute-force behavior.

![Repeated failed logins](assets/Slika%203.png)

---

## 4. Wazuh Detects Multiple Windows Logon Failures

After Windows records the failed logon events, the Wazuh agent collects them from the Windows Security logs and forwards them to the Wazuh manager.

In the Wazuh dashboard, an alert is generated with the description:

```text
Multiple Windows logon failures.
```

The alert has a severity level of `10`, indicating a significant security event.

![Wazuh alert summary](assets/Slika%204.png)

---

## 5. Windows Security Event Details

The alert details show that the event originated from the Windows endpoint:

```text
agent.name: windows-meta
agent.ip: 192.168.1.81
data.win.system.eventID: 4625
data.win.system.channel: Security
```

Windows Event ID `4625` represents a failed logon attempt.

The event also shows `logonType: 2`, which indicates an interactive logon attempt. In this lab, the failed authentication attempts were generated locally through an interactive process using `runas`.

![Event details part 1](assets/Slika%205.png)

---

## 6. Target User, Status and SubStatus Analysis

This section of the event shows which username was targeted during the failed logon attempt.

Example from the log:

```text
data.win.eventdata.targetUserName: hacker_3.
data.win.eventdata.targetDomainName: DESKTOP-FQR1PJ8
data.win.eventdata.targetUserSid: S-1-0-0
data.win.eventdata.workstationName: DESKTOP-FQR1PJ8
```

The value `targetUserSid: S-1-0-0` indicates that the account was not successfully mapped to a valid SID. This is expected when the attempted username is invalid, does not exist, or cannot be resolved by the local system.

The event also contains important authentication failure fields:

```text
data.win.eventdata.status: 0xC000006D
data.win.eventdata.subStatus: 0xC0000064
```

The `Status` field provides the general reason for the failed logon. In this case, `0xC000006D` means that the logon attempt failed because the provided credentials were not valid.

The `SubStatus` field gives a more specific reason for the failure. In this lab, `0xC0000064` indicates that the attempted username does not exist.

Together, these values confirm that the failed logon was caused by an invalid or non-existing account rather than a valid user simply entering the wrong password.

![Target user details](assets/Slika%206.png)

---

## 7. Audit Failure and Provider Information

The event was recorded by the Windows Security Auditing provider:

```text
data.win.system.providerName: Microsoft-Windows-Security-Auditing
data.win.system.severityValue: AUDIT_FAILURE
data.win.system.eventID: 4625
```

This confirms that the event is an authentication audit failure.

![Audit failure details](assets/Slika%207.png)

---

## 8. Triggered Wazuh Rule

Wazuh correlated the failed logon events and triggered the following rule:

```text
rule.id: 60204
rule.description: Multiple Windows logon failures.
rule.level: 10
rule.frequency: 8
rule.groups: windows, windows_security, authentication_failures
```

This rule belongs to Windows security and authentication failure groups.

![Wazuh rule details](assets/Slika%208.png)

---

## 9. MITRE ATT&CK Mapping

The alert is mapped to the following MITRE ATT&CK tactic and technique:

```text
rule.mitre.tactic: Credential Access
rule.mitre.technique: Brute Force
rule.mitre.id: T1110
```

This means Wazuh classifies this behavior as credential access activity using the brute-force technique.

![MITRE mapping](assets/Slika%209.png)

---

## Detection Summary

| Field | Value |
|---|---|
| Platform | Windows |
| Data Source | Windows Security Event Logs |
| Event ID | 4625 |
| Logon Type | 2 - Interactive |
| Status | 0xC000006D |
| Status Meaning | Invalid credentials / failed logon |
| SubStatus | 0xC0000064 |
| SubStatus Meaning | Username does not exist |
| Detection Tool | Wazuh |
| Wazuh Rule ID | 60204 |
| Alert Description | Multiple Windows logon failures |
| Severity Level | 10 |
| MITRE Tactic | Credential Access |
| MITRE Technique | Brute Force |
| MITRE ID | T1110 |

---

## Conclusion

This lab demonstrates how a simple simulation of failed Windows logon attempts can be used to validate Wazuh log collection, event correlation, and alerting.

The lab confirms the following:

- Windows generates Security Event ID `4625` for failed logon attempts.
- The failed logon activity used Logon Type `2`, which represents an interactive logon attempt.
- The status code `0xC000006D` indicates that the logon attempt failed due to invalid credentials.
- The substatus code `0xC0000064` indicates that the attempted username does not exist.
- The Wazuh agent successfully collects Windows Security logs.
- Wazuh generates an alert for multiple failed Windows logon attempts.
- Wazuh rule `60204` is triggered.
- The alert is mapped to MITRE ATT&CK `T1110 - Brute Force`.

This project is useful for practicing Windows log analysis, SIEM alert investigation, and understanding basic detection logic for authentication failures.
