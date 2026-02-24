# Windows Event Logs & Finding Evil – DFIR Case Study

## Introduction

This portfolio documents my hands-on investigation through the Hack The Box (HTB) Academy module, **"Windows Event Logs & Finding Evil."**  

The objective of this project was to simulate the role of a Digital Forensics and Incident Response (DFIR) analyst using advanced Windows logging mechanisms to uncover malicious activity.

Across three sections, I leveraged:

- Native Windows Security Event Logs  
- Sysmon configuration analysis  
- Event Tracing for Windows (ETW)  
- Forensic tooling such as Seatbelt  
- PowerShell-based log querying  

This report outlines investigative steps, findings, and technical insights gained during the analysis.

---

# Section 1: Windows Event Logs & Finding Evil – Part 1

This section focused on foundational Windows Event Log analysis, including Event Viewer filtering, XPath queries, and interpretation of security Event IDs.

---

## Screenshot 1.1 – Launching Windows Event Viewer

Initial access to Windows Event Viewer via the Start Menu.  
This represents the first step in Windows forensic log analysis.

---

## Screenshot 1.2 – Security Log Interface

Displays the Event Viewer interface with the **Security** log selected.  
Demonstrates navigation and filtering options used in forensic investigations.

---

## Screenshot 1.3 – Basic Event ID Filtering

Filtered the Security log for:

- **Event ID 4624** – Successful Logon

This isolates authentication activity for analysis.

---

## Screenshot 1.4 – Event 4624 Detailed Analysis

Detailed breakdown of a successful logon event:

- **LogonType: 5** (Service logon)  
- **SubjectLogonId: 0x3e7** (SYSTEM context)  
- Authentication package details  

This demonstrates how logon events provide insight into service activity and privilege context.

---

## Screenshot 1.5 – Advanced XPath Query Construction

Created a manual XPath query to filter:

- **Event ID 4907** – Audit policy changes  
- Specific `SubjectLogonId` values  

This enabled precision filtering beyond standard GUI options.

---

## Screenshot 1.6 – Filter Results with Process Details

Event 4907 analysis revealed:

- **ProcessName:** `TiWorker.exe` (Windows servicing stack)  
- **ObjectName:** WPF-related assembly  

This demonstrates tracking file or policy modifications tied to specific processes.

---

## Screenshot 1.7 – Complex XPath Filter Configuration

Constructed a combined XPath filter targeting:

- Specific `ProcessName`
- Specific `ObjectName`

This approach reduces noise and isolates actionable artifacts.

---

## Screenshot 1.8 – Filter Summary Statistics

Security log filtering results:

- **9,127 total events**
- Reduced to **1 relevant event**

Keyword: **Audit Success**

This highlights the importance of narrowing large datasets into investigative artifacts.

---

## Part 1 Summary

Key skills developed:

- Understanding event structure (`System` vs `EventData`)
- Targeted filtering using XPath
- Authentication artifact analysis (4624)
- Audit policy modification tracking (4907)

This section reinforced that Windows Security logs provide a structured chronological record critical for incident reconstruction.

---

# Section 2: Windows Event Logs & Finding Evil – Part 2

This section focused on Sysmon configuration analysis and privilege inspection using forensic tooling.

---

## Screenshot 2.1 – Sysmon Configuration Loading

Loaded a custom configuration file:

- `sysmonconfig-export.xml`
- Schema version 4.50
- Sysmon schema 4.83

Confirms successful configuration updates.

---

## Screenshot 2.2 – Sysmon Configuration Analysis (Part 1)

Review of XML configuration:

- **Event ID 6 – Driver Load**
  - Excludes Microsoft-signed drivers
- **Event ID 7 – DLL Image Load**
  - Disabled by default due to performance considerations

---

## Screenshot 2.3 – Sysmon Configuration Analysis (Part 2)

Configuration references:

- **MITRE ATT&CK T1014 – Rootkit**
- **T1073 – DLL Side-Loading**

Logs non-Microsoft driver loads while excluding trusted vendors.

Demonstrates mapping telemetry to adversary techniques.

---

## Screenshot 2.4 – Seatbelt Token Privilege Analysis (Initial Execution)

Executed Seatbelt to analyze `TokenPrivileges`.

Key finding:

- **SeDebugPrivilege – ENABLED**

This privilege allows debugging other processes, including SYSTEM-level processes.

---

## Screenshot 2.5 – Seatbelt Privilege Confirmation

Confirmed enabled privileges:

- `SeDebugPrivilege`
- `SeImpersonatePrivilege`
- `SeCreateGlobalPrivilege`

These privileges represent potential privilege escalation pathways.

---

## Part 2 Summary

Major concepts learned:

- Sysmon as enhanced telemetry over native logs
- Configuration tradeoffs (visibility vs performance)
- Mapping logs to MITRE ATT&CK
- Privilege escalation indicators via token analysis

Understanding process privileges is essential for identifying post-exploitation behavior.

---

# Section 3: Windows Event Logs & Finding Evil – Part 3

Focused on advanced telemetry through ETW and lateral movement detection.

---

## Screenshot 3.1 – SilkETW Collection

Used SilkETW to capture runtime telemetry from:

- `Microsoft-Windows-DotNETRuntime` provider
- Keyword mask `0x2038`
- Output in JSON format

Captured **593 events**

Demonstrates collection of in-memory telemetry not always written to disk.

---

## Screenshot 3.2 – PowerShell Lateral Movement Detection

Used `Get-WinEvent` to analyze attack samples.

Identified:

- **Event ID 5142 – Network share added**
- Share Name: `\\PRINT`
- Share Path: `C:\Windows\System32`

This pattern aligns with **PrintNightmare-style exploitation**.

---

## Screenshot 3.3 – Code Analysis for Forensic Context

Reviewed code structure to understand:

- API interactions
- Artifact generation patterns
- Log creation behavior

Understanding underlying execution helps correlate telemetry artifacts.

---

## Part 3 Summary

Key insights:

- ETW operates at kernel level
- Some sessions exist only in memory
- Multi-source telemetry correlation is essential
- Lateral movement artifacts require contextual interpretation

This section emphasized modern DFIR requires cross-log correlation.

---

# Final Reflection

This module significantly strengthened my ability to perform structured forensic analysis within Windows environments.

Progression included:

- Basic event navigation
- Advanced XPath filtering
- Sysmon configuration analysis
- Privilege escalation detection
- ETW runtime telemetry collection
- Multi-source log correlation

Analyzing Event IDs:

- **4624** – Logon activity  
- **4672** – Special privileges assigned  
- **5142** – Network share creation  

Reinforced how authentication, privileges, and share modifications indicate attack progression.

Most importantly, this experience developed an investigative mindset:

- Reduce noise  
- Validate anomalies  
- Correlate telemetry  
- Reconstruct timelines  

These competencies directly align with professional Digital Forensics and Incident Response roles.
