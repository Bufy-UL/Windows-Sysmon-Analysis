# Log Analysis and Process Detection with Windows Sysmon

## Objective
The objective of this project is to demonstrate hands-on Blue Team skills by monitoring, collecting, and analyzing Windows event logs using **Microsoft Sysmon**. Both common and potentially malicious commands were simulated from the terminal to document how a SOC Analyst can identify execution patterns, anomalies, and evasion techniques.

## Tools Used
* **Windows 10 / 11** (Virtualized environment in Oracle VirtualBox)
* **Microsoft Sysmon v14+** (System Monitor)
* **Windows PowerShell / CMD** (For command execution)
* **Windows Event Viewer** (For forensic log analysis)

---

## Steps Performed

### 1. Installation and Deployment
* Sysmon was configured and installed on the virtual machine using administrator privileges to capture process creation (`Event ID 1`) and network connections (`Event ID 3`).

### 2. Command Execution and Simulation
System and network reconnaissance commands were executed, simulating the early stages of an attack (Reconnaissance & Discovery) or the use of evasion techniques (Defacement/Obfuscation).

### 3. Evidence Analysis in Event Viewer
Events generated were isolated via the path `Applications and Services Logs -> Microsoft -> Windows -> Sysmon -> Operational`.

---

## Forensic Event Analysis (Findings)

Below are the key findings mapped from Event Viewer:

### Case 1: Execution of Reconnaissance Commands (`whoami`)
* **Description:** Typical command used by attackers to identify the current user's privileges after compromising a machine.
* **Log Analysis (Event ID 1):** 
  * `Image`: Shows the real path of the legitimate binary executed (`C:\Windows\System32\whoami.exe`).
  * `ParentImage`: Indicates it was launched from `powershell.exe`. This allows tracing the execution chain.

![Whoami Reconnaissance](Screenshots/03-whoami.png)

---

### Case 2: Evasion Simulation - Encoded PowerShell Command (`-enc`)
* **Critical Analysis:** This is the most important finding for a SOC profile. Attackers often use the `-enc` or `-ExecutionPolicy Bypass` parameter in PowerShell to evade system restriction policies and hide malicious scripts in Base64.
* **Log Analysis (Event ID 1):**
  * `CommandLine`: Captures the full command, including the obfuscated string (`-enc ZWNobyBoZWxsbw==`). As analysts, this instantly alerts us to decode the content (which in this case translates to a simple `echo hello`).
  * `IntegrityLevel`: Shows `High`, indicating the process ran with elevated privileges.

![Encoded PowerShell Evasion](Screenshots/02-powershell-enc.png)

---

### Case 3: Network Monitoring via Ping to Google
* **Description:** External connectivity check.
* **Log Analysis (Event ID 1):**
  * `CommandLine`: Logs `PING.EXE google.com`. In a real environment, this helps us detect if malware is attempting to check whether the compromised machine has internet access (Beaconing or Connection Check).

![Ping Network Log](Screenshots/05-google.png)

---

## Conclusion
This project demonstrates how **Sysmon** drastically expands native Windows auditing capabilities. While standard Windows logs may overlook specific command arguments, Sysmon accurately records the full command line (`CommandLine`), binary hashes for reputation checks (MD5/SHA256), and the parent process (`ParentImage`). 

Understanding these traces is essential for a **Junior SOC Analyst** when creating threat detection rules and investigating incidents in corporate environments.
