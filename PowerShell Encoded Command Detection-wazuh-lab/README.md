# Lab 01 - PowerShell Encoded Command Detection

## Overview

This lab demonstrates the detection of suspicious PowerShell activity using a Base64-encoded command executed on a Windows endpoint. The activity was monitored using Sysmon and successfully detected by Wazuh SIEM.

The objective of this lab was to simulate a commonly abused attacker technique involving the `-EncodedCommand` PowerShell parameter, which is frequently used by malware and threat actors to obfuscate malicious commands and evade detection.

The generated alert was automatically mapped to the MITRE ATT&CK framework under:

- Technique: `T1059.001 - PowerShell`
- Tactic: `Execution`

---

## Lab Environment

- Windows 11 Virtual Machine
- Wazuh SIEM
- Sysmon
- PowerShell
- MITRE ATT&CK Mapping
- Wazuh Agent

---

## PowerShell Encoded Command Execution

The following PowerShell command was executed in order to simulate suspicious encoded PowerShell activity.

```powershell
powershell.exe -EncodedCommand VwByAGkAdABlAC0ASABvAHMAdAAgACIASABlAGwAbABvACI=
```

Although the command generated a parsing error inside PowerShell, Sysmon and Wazuh still successfully detected the use of the `-EncodedCommand` parameter and generated a corresponding security alert.

<img src="assets/1. Slika.png" width="1000">

---

## Wazuh Alert Detection

After executing the encoded PowerShell command, Wazuh generated a security alert related to suspicious Base64-encoded PowerShell activity.

The alert was mapped to:

- MITRE ATT&CK Technique: `T1059.001`
- Tactic: `Execution`

The generated alert description was:

> `Powershell.exe spawned a powershell process which executed a base64 encoded command`

<img src="assets/2. Slika.png" width="1000">

---

## Command Line Analysis

The expanded Wazuh alert details clearly show the full command line used during execution.

The analysis confirms:

- Usage of `powershell.exe`
- Usage of the `-EncodedCommand` parameter
- Presence of the encoded Base64 payload
- PowerShell process creation activity

This type of behavior is frequently associated with:

- Malware execution
- Obfuscated PowerShell attacks
- Malicious scripting activity
- Initial access techniques
- Defense evasion activity

<img src="assets/3. Slika.png" width="1000">

---

## Process and Parent Process Analysis

The Sysmon telemetry also provided detailed process creation information including:

- Parent process
- Parent command line
- Process GUID
- Process ID
- User context
- Integrity level

The event confirms that the PowerShell process spawned another PowerShell process using the encoded command parameter.

<img src="assets/4. Slika.png" width="1000">

---

## Full Sysmon Telemetry

The full Sysmon event log contains detailed telemetry regarding the PowerShell execution event.

Important information captured includes:

- Event ID 1 (Process Creation)
- PowerShell image path
- Encoded command line
- Parent process information
- File hashes
- User account context
- Sysmon operational channel

This demonstrates how Sysmon provides rich endpoint visibility for SIEM analysis.

<img src="assets/5. Slika.png" width="1000">

---

## MITRE ATT&CK Mapping and Rule Information

Wazuh successfully mapped the detected activity to the MITRE ATT&CK framework.

### Detection Details

- Rule ID: `92057`
- Rule Level: `12`
- Technique: `T1059.001 - PowerShell`
- Tactic: `Execution`

The alert belongs to the following rule groups:

- `sysmon`
- `sysmon_eid1_detections`
- `windows`

<img src="assets/6. Slika.png" width="1000">

---

## Conclusion

This lab successfully demonstrated how Wazuh and Sysmon can detect suspicious PowerShell execution involving Base64-encoded commands.

Even though the PowerShell command itself generated a parsing error, the telemetry generated during process creation was still sufficient for Sysmon and Wazuh to identify and alert on the suspicious activity.

Key takeaways from this lab include:

- Detection of encoded PowerShell execution
- Sysmon Event ID 1 process creation monitoring
- MITRE ATT&CK mapping
- Visibility into parent-child process relationships
- Command-line analysis capabilities within Wazuh

This type of detection is highly relevant in modern SOC environments because PowerShell encoded commands are commonly used by attackers for:

- Malware delivery
- Defense evasion
- Obfuscation
- Remote execution
- Post-exploitation activity
