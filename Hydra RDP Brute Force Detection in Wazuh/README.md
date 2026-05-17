# Hydra RDP Brute Force Detection in Wazuh

## Overview

This lab demonstrates the detection of a brute-force attack against a Windows machine using Hydra from a Kali Linux attacker system. The attack targeted the Remote Desktop Protocol (RDP) service, while Wazuh monitored and collected Windows Security Event Logs generated during failed authentication attempts.

The objective of this lab was to:

- Simulate a brute-force attack using Hydra
- Generate Windows authentication failure events
- Monitor the attack through Wazuh SIEM
- Analyze Windows Event ID 4625 logs
- Identify attacker information and authentication details

---

# Environment

| Component | Description |
|---|---|
| Attacker Machine | Kali Linux |
| Target Machine | Windows VM |
| SIEM Platform | Wazuh |
| Attack Tool | Hydra |
| Protocol | RDP |
| Windows Event ID | 4625 |

---

# Step 1 – Creating the Password List

The first step was creating a custom password wordlist that Hydra would use during the brute-force attack.

The following passwords were added into the `passwords.txt` file:

- Password123
- admin
- P@ssw0rd
- 1234567
- Welcome1
- Admin123
- qwerty
- passworddd

This list simulates commonly used weak passwords frequently targeted during brute-force attacks.

![IMG](assets/1.%20Slika.png)

---

# Step 2 – Creating the Username List

A custom username list was then created inside the `users.txt` file.

The following usernames were used:

- administrator
- admin
- backup
- sqladmin
- helpdesk
- user
- guest
- test

These usernames simulate common administrative and service accounts often targeted by attackers.

![IMG](assets/2.%20Slika.png)

---

# Step 3 – Launching the Hydra RDP Brute Force Attack

Hydra was executed against the Windows target machine over the RDP service.

## Command Used

```bash
hydra -L /tmp/users.txt -P /tmp/passwords.txt rdp://192.168.1.81 -t 4
```

## Command Explanation

| Parameter | Description |
|---|---|
| `-L` | Username wordlist |
| `-P` | Password wordlist |
| `rdp://` | Targeting the RDP protocol |
| `192.168.1.81` | Windows target IP address |
| `-t 4` | Four parallel threads |

Hydra attempted multiple username and password combinations against the RDP service.

The output also showed:

- Total login attempts
- Active tasks
- Connection failures
- Authentication failures

No valid credentials were discovered during the attack simulation.

![IMG](assets/3.%20Slika.png)

---

# Step 4 – Wazuh Detection

After the brute-force activity was performed, Wazuh successfully detected multiple failed Windows logon attempts.

The generated alert indicated:

- Multiple Windows logon failures
- Detection severity level 10
- Windows authentication monitoring activity

This confirms that Wazuh successfully ingested and analyzed Windows Security logs generated during the attack.

![IMG](assets/4.%20Slika.png)

---

# Step 5 – Windows Event Log Analysis

The Wazuh event details revealed critical forensic information related to the attack.

## Important Event Fields

| Field | Value | Description |
|---|---|---|
| Event ID | 4625 | Failed Windows logon |
| Authentication Package | NTLM | Authentication method |
| Source IP Address | 192.168.1.46 | Attacker machine IP |
| Workstation Name | kali | Attacker hostname |
| Target Username | administrator | Account targeted during attack |
| Logon Type | 3 | Network logon attempt |

The event logs confirmed that the authentication attempts originated from the Kali Linux machine.

The attack generated repeated failed authentication events against the Windows target system.

![IMG](assets/5.%20Slika.png)

---

# Analysis

This lab demonstrates how brute-force attacks generate valuable authentication telemetry inside Windows environments.

Key findings include:

- Hydra generated repeated authentication attempts against the RDP service
- Windows Security logs recorded failed authentication events
- Wazuh successfully detected suspicious authentication behavior
- The attacker IP address and workstation name were identified
- Event ID 4625 provided detailed forensic evidence

The lab also demonstrates how SIEM platforms can detect brute-force behavior even when the attack itself is unsuccessful.

---

# MITRE ATT&CK Mapping

| Tactic | Technique |
|---|---|
| Credential Access | T1110 – Brute Force |

---

# Conclusion

This lab successfully simulated a brute-force authentication attack using Hydra against a Windows RDP service.

Wazuh successfully collected and analyzed Windows Security Events related to the attack and generated alerts for repeated failed logon attempts.

The collected telemetry provided visibility into:

- Attacker IP address
- Authentication method
- Targeted usernames
- Logon types
- Failed authentication attempts

This type of detection is critical in real-world SOC environments for identifying credential attacks and unauthorized authentication attempts.
