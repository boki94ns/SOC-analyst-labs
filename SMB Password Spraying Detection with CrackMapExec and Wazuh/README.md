# SMB Password Spraying Detection with CrackMapExec and Wazuh

## Overview

This lab demonstrates how SMB authentication attempts against a Windows machine can be detected and analyzed using Wazuh SIEM.

A Kali Linux machine was used to simulate password spraying activity against a Windows 11 virtual machine over the SMB protocol (port 445). The generated authentication failures were successfully collected by the Windows Security Event Log and forwarded to Wazuh for analysis.

The objective of this lab was to:

- Simulate SMB authentication attacks
- Generate Windows failed logon events
- Analyze Event ID 4625
- Observe Wazuh alert correlation
- Investigate authentication metadata
- Review MITRE ATT&CK mappings

---

# Environment

| Component | Description |
|---|---|
| Attacker Machine | Kali Linux |
| Target Machine | Windows 11 |
| SIEM Platform | Wazuh |
| Attack Tool | CrackMapExec |
| Protocol | SMB |
| Port | 445 |

---

# Attack Simulation

The following CrackMapExec command was executed from the Kali Linux machine:

```bash
crackmapexec smb 192.168.1.81 -u /tmp/users.txt -p 'Password123!'
```

The command attempted SMB authentication using multiple usernames with the same password.

Parameters explanation:

| Parameter | Description |
|---|---|
| smb | SMB protocol module |
| 192.168.1.81 | Target Windows machine |
| -u | Username list |
| -p | Password used for authentication attempts |

---

# Attack Results

The SMB authentication attempts generated multiple failed login responses.

Observed output included:

```txt
STATUS_LOGON_FAILURE
```

This indicates that the provided credentials were invalid or the targeted accounts did not exist.

---

## Attack Execution Screenshot

![IMG](assets/1.%20Slika.png)

---

# Wazuh Detection

After the attack execution, Wazuh successfully detected multiple failed Windows authentication attempts.

The following alerts were generated:

- Logon failure - Unknown user or bad password
- Multiple Windows logon failures

These alerts indicate suspicious authentication activity against the Windows endpoint.

---

## Wazuh Alert Overview

![IMG](assets/2.%20Slika.png)

---

# Windows Event Analysis

The generated alert corresponds to:

```txt
Windows Event ID 4625
```

Event ID 4625 represents:

```txt
An account failed to log on
```

This is one of the most important Windows Security events related to failed authentication attempts.

---

# Source IP Address

The event details show the originating IP address of the attacker machine:

```txt
192.168.1.46
```

This allows analysts to identify the source system responsible for the authentication attempts.

---

# Authentication Package

The authentication package detected was:

```txt
NTLM
```

NTLM is commonly used during SMB authentication and is frequently observed during password spraying and brute-force activity.

---

# Logon Type Analysis

The event contained:

```txt
Logon Type: 3
```

Logon Type 3 indicates:

```txt
Network Logon
```

This confirms that the authentication attempt occurred remotely over the network using SMB.

---

## Event Details Screenshot

![IMG](assets/3.%20Slika.png)

---

# Status and SubStatus Analysis

The Windows Security Event contained the following status codes:

## Status

```txt
0xc000006d
```

Meaning:

```txt
Bad username or authentication information
```

---

## SubStatus

```txt
0xc0000064
```

Meaning:

```txt
User account does not exist
```

This indicates that the attack attempted authentication against invalid or non-existing accounts.

---

## Authentication Failure Details

![IMG](assets/4.%20Slika.png)

---

# Windows Security Provider

The event provider responsible for generating the event was:

```txt
Microsoft-Windows-Security-Auditing
```

Severity:

```txt
AUDIT_FAILURE
```

This confirms that the Windows Security subsystem successfully recorded the failed authentication attempts.

---

## Security Provider Screenshot

![IMG](assets/5.%20Slika.png)

---

# Wazuh Rule Analysis

Wazuh generated Rule ID:

```txt
60122
```

Description:

```txt
Logon failure - Unknown user or bad password
```

Wazuh also grouped the repeated events into a higher-level detection for multiple authentication failures.

---

# MITRE ATT&CK Mapping

Wazuh automatically mapped the activity to MITRE ATT&CK techniques:

| Technique ID | Description |
|---|---|
| T1078 | Valid Accounts |
| T1531 | Account Access Removal |

Associated tactics included:

- Initial Access
- Persistence
- Privilege Escalation
- Defense Evasion

---

## Wazuh Rule and MITRE Mapping

![IMG](assets/6.%20Slika.png)

---

# Detection Summary

| Detection Element | Value |
|---|---|
| Event ID | 4625 |
| Protocol | SMB |
| Authentication Type | NTLM |
| Logon Type | 3 |
| Source IP | 192.168.1.46 |
| Detection Platform | Wazuh |
| Alert Type | Failed Authentication |
| Wazuh Rule | 60122 |

---

# Conclusion

This lab successfully demonstrated how SMB password spraying activity can be detected using Wazuh SIEM and Windows Security logs.

The attack generated multiple failed authentication attempts against a Windows 11 endpoint, which resulted in Windows Event ID 4625 entries being forwarded to Wazuh.

The analysis showed:

- Failed SMB authentication attempts
- Source IP attribution
- NTLM authentication usage
- Network logon activity
- Wazuh alert generation
- MITRE ATT&CK correlation
- Aggregated authentication failure detection

This type of activity is commonly associated with password spraying and brute-force attacks and represents an important detection scenario for SOC analysts and blue team environments.

# Indicators Observed

| Indicator | Value |
|---|---|
| Source IP | 192.168.1.46 |
| Destination IP | 192.168.1.81 |
| Target Protocol | SMB |
| Destination Port | 445 |
| Authentication Package | NTLM |
| Event ID | 4625 |
| Logon Type | 3 |
| Wazuh Rule ID | 60122 |

---

# Security Impact

Repeated failed SMB authentication attempts may indicate:

- Password spraying
- Brute-force activity
- Unauthorized access attempts
- Credential attacks
- Lateral movement attempts

If successful, this type of activity could allow attackers to gain unauthorized access to internal systems.

---

# Recommendations

To reduce the risk of SMB password spraying attacks:

- Enforce strong password policies
- Implement account lockout policies
- Disable unused accounts
- Restrict SMB exposure
- Monitor failed authentication attempts
- Enable MFA where possible
- Use SIEM alert correlation for repeated failures
