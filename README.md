# Kenobi — TryHackMe

A walkthrough and practical penetration-testing exercise based on the **Kenobi** room from TryHackMe.

## 📌 Overview

This lab focuses on Linux enumeration, service discovery, exploitation, SSH key access, and privilege escalation.

During the assessment, multiple network services were identified, including **FTP, SSH, HTTP, SMB, and NFS**. Further enumeration revealed misconfigurations that allowed access to sensitive information and ultimately provided an initial foothold on the target system.

After gaining access as the `kenobi` user, local enumeration was performed to identify potential privilege-escalation vectors. A vulnerable **SUID binary** was discovered and abused to obtain root-level access.

## 🔍 Key Areas Covered

* Network and service enumeration
* FTP enumeration and ProFTPD exploitation
* SMB share enumeration
* NFS enumeration
* SSH private-key retrieval
* Initial access as `kenobi`
* Linux privilege escalation
* SUID binary analysis
* PATH hijacking
* Root access

## 🛠️ Tools Used

* Nmap
* Metasploit Framework
* smbclient
* Netcat
* NFS utilities
* LinPEAS
* Linux command-line utilities

## 🎯 Learning Objectives

This room provides practical experience with:

* Identifying exposed services
* Finding vulnerable service configurations
* Enumerating network shares
* Understanding Linux file permissions
* Analyzing SUID binaries
* Exploiting insecure command execution
* Performing Linux privilege escalation

## 📄 Detailed Walkthrough

The complete step-by-step exploitation process, commands, outputs, and screenshots are documented in the accompanying PDF report.

> **Note:** This README intentionally provides only a high-level overview. Refer to the PDF for the complete technical walkthrough.

## 🏁 Outcome

The assessment resulted in successful compromise of the target, including:

* **User access:** `kenobi`
* **Root access:** Obtained through local privilege escalation
* **User flag:** Captured
* **Root flag:** Captured

## ⚠️ Disclaimer

This walkthrough is intended for **educational purposes** and was performed against an authorized TryHackMe lab environment. Do not use these techniques against systems without explicit permission.
