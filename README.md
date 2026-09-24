# 🌪️ Mayhem — TryHackMe Walkthrough

<div align="center">

# Professional Blue Team • Network Forensics • Havoc C2 Investigation

*A complete DFIR-style investigation of the **TryHackMe Mayhem** room using Wireshark, malware analysis, PowerShell investigation, HTTP object extraction, Havoc Command & Control protocol analysis, AES-CTR decryption, and CyberChef.*

![License](https://img.shields.io/badge/License-MIT-2563EB?style=for-the-badge)
![Platform](https://img.shields.io/badge/TryHackMe-Blue_Team-0EA5E9?style=for-the-badge)
![Difficulty](https://img.shields.io/badge/Difficulty-Medium-059669?style=for-the-badge)
![Category](https://img.shields.io/badge/Category-Network_Forensics-7C3AED?style=for-the-badge)
![Status](https://img.shields.io/badge/Status-Completed-16A34A?style=for-the-badge)

**Digital Forensics • Incident Response • Threat Hunting • Malware Traffic Analysis**

</div>

---

## 📖 Overview

**Mayhem** is a Blue Team focused **TryHackMe** room that simulates the investigation of a compromised Windows workstation communicating with a malicious **Havoc C2 Teamserver**.

Instead of exploiting a target machine, this room challenges the analyst to reconstruct the entire intrusion from captured network traffic. The investigation begins with a **packet capture (PCAP)** and progresses through PowerShell analysis, malware extraction, protocol reverse engineering, encrypted command-and-control traffic decryption, persistence discovery, and evidence validation.

This repository documents the investigation as a **real-world SOC / DFIR case study**, emphasizing methodology, evidence correlation, and defensive understanding rather than simply revealing challenge answers.

> **Public Repository Policy:** All challenge flags, credentials, passwords, and answer strings have been intentionally **redacted** to preserve the educational integrity of the room.

---

# 🎯 Objectives

* Investigate a suspicious packet capture using **Wireshark**.
* Recover attacker-delivered payloads from HTTP traffic.
* Analyze malicious PowerShell download behavior.
* Extract and inspect a disguised executable.
* Identify **Havoc C2** communications.
* Recover AES session parameters from Havoc initialization traffic.
* Decrypt encrypted C2 commands and responses.
* Reconstruct attacker actions and persistence mechanisms.
* Produce a professional DFIR investigation report.

---

# 🛡️ Skills Demonstrated

| Blue Team Skill             | Demonstrated |
| --------------------------- | ------------ |
| PCAP Analysis               | ✅            |
| Wireshark Investigation     | ✅            |
| HTTP Stream Analysis        | ✅            |
| HTTP Object Extraction      | ✅            |
| PowerShell Malware Analysis | ✅            |
| Malware Triage              | ✅            |
| Strings Analysis            | ✅            |
| VirusTotal Intelligence     | ✅            |
| Havoc C2 Traffic Analysis   | ✅            |
| AES-CTR Decryption          | ✅            |
| CyberChef Workflow          | ✅            |
| Windows Artifact Analysis   | ✅            |
| Threat Hunting              | ✅            |
| Digital Forensics Reporting | ✅            |

---

# 🧰 Technologies & Tools Used

| Tool                   | Purpose                      |
| ---------------------- | ---------------------------- |
| **Wireshark**          | Packet capture investigation |
| **CyberChef**          | AES-CTR traffic decryption   |
| **VirusTotal**         | Malware intelligence lookup  |
| **strings**            | Static binary analysis       |
| **PowerShell**         | Script analysis              |
| **Windows Networking** | Artifact interpretation      |
| **Markdown**           | Technical documentation      |
| **GitHub Pages**       | Portfolio presentation       |

---

# 🧠 Investigation Workflow

```text
                  Network Packet Capture (.pcap)

                           │
                           ▼

                Wireshark Network Investigation

                           │
                           ▼

                 HTTP Stream Reconstruction

                           │
                           ▼

            install.ps1 PowerShell Downloader Analysis

                           │
                           ▼

             HTTP Object Extraction (notepad.exe)

                           │
                           ▼

              Static Malware Triage (strings)

                           │
                           ▼

          Havoc C2 Attribution via Threat Intelligence

                           │
                           ▼

         DEMON_INIT Packet Identification (Command 99)

                           │
                           ▼

          AES Key + IV Recovery from Initialization

                           │
                           ▼

           AES-CTR Decryption with CyberChef

                           │
                           ▼

       Decrypted Commands, Responses & Host Artifacts

                           │
                           ▼

         Incident Timeline Reconstruction & Findings
```

---

# 🔍 Attack Chain Summary

| Stage                      | Description                                                          |
| -------------------------- | -------------------------------------------------------------------- |
| **Initial Access**         | Windows host downloads `install.ps1`.                                |
| **Execution**              | PowerShell retrieves and executes `notepad.exe`.                     |
| **Payload Delivery**       | Executable transferred over HTTP port **1337**.                      |
| **Malware Identification** | Binary linked to **Havoc C2**.                                       |
| **Command & Control**      | HTTP beaconing begins after execution.                               |
| **Encryption**             | Havoc uses **AES-CTR** encrypted payloads.                           |
| **Persistence**            | Attacker creates an additional Windows account.                      |
| **Discovery**              | Host identity, IPv6 configuration, SID, and sensitive file accessed. |

---

# 📂 Repository Structure

```text
Mayhem--TryHackMe-Walkthrough
│
├── README.md
├── LICENSE
├── SECURITY.md
├── CONTRIBUTING.md
│
├── Documentation/
│   ├── Documentation.md
│   └── Documentation.docx
│
├── Resources/
│   └── notes.md
│
├── Screenshots/
│   ├── 01_room_banner.png
│   ├── 02_pcap_overview.png
│   ├── 03_http_install_ps1.png
│   ├── 04_http_object_export.png
│   ├── 05_strings_analysis.png
│   ├── 06_virustotal_analysis.png
│   ├── 07_havoc_init_packet.png
│   ├── 08_aes_key_iv.png
│   ├── 09_cyberchef_decryption.png
│   ├── 10_decrypted_metadata.png
│   ├── 11_sid_artifact.png
│   ├── 12_ipv6_artifact.png
│   ├── 13_flag_artifact.png
│   ├── 14_persistence_artifact.png
│   ├── 15_clients_csv_artifact.png
│   └── 16_answer_validation.png
│
└── docs/
    ├── index.md
    └── assets/
        ├── css/
        └── images/
```

---

# 📸 Evidence Gallery

This walkthrough includes a curated collection of **16 numbered investigation screenshots**, documenting each major forensic milestone.

| Investigation Evidence       | Included |
| ---------------------------- | -------- |
| Room Banner                  | ✅        |
| Wireshark PCAP Overview      | ✅        |
| PowerShell HTTP Stream       | ✅        |
| HTTP Object Export           | ✅        |
| Static Strings Analysis      | ✅        |
| VirusTotal Malware Detection | ✅        |
| Havoc Initialization Packet  | ✅        |
| AES Key & IV Extraction      | ✅        |
| CyberChef AES Decryption     | ✅        |
| Metadata Decryption          | ✅        |
| Windows SID Artifact         | ✅        |
| IPv6 Artifact                | ✅        |
| Attacker Command Artifact    | ✅        |
| Persistence Evidence         | ✅        |
| Sensitive File Discovery     | ✅        |
| Final Validation Evidence    | ✅        |

Every image is referenced throughout the documentation and GitHub Pages portfolio.

---

# 🕵️ Investigation Highlights

## Phase 1 — PCAP Triage

* Network communication reconstruction.
* Identification of suspicious hosts.
* HTTP conversation analysis.

## Phase 2 — PowerShell Analysis

* Recovery of `install.ps1`.
* Identification of malicious download behavior.
* Process execution chain reconstruction.

## Phase 3 — Malware Extraction

* Exporting HTTP objects.
* Recovering `notepad.exe`.
* Static binary investigation.

## Phase 4 — Havoc C2 Analysis

* Havoc Teamserver identification.
* DEMON_INIT packet recognition.
* Protocol structure investigation.

## Phase 5 — Cryptographic Analysis

* AES Key extraction.
* IV extraction.
* CTR mode decryption workflow.

## Phase 6 — Traffic Decryption

* CyberChef workflow.
* Command reconstruction.
* Response reconstruction.

## Phase 7 — Artifact Recovery

* Windows SID.
* IPv6 configuration.
* Persistence account.
* Sensitive file discovery.

---

# 🔐 MITRE ATT&CK Mapping

| ATT&CK Technique | Description                            |
| ---------------- | -------------------------------------- |
| T1059.001        | PowerShell Execution                   |
| T1105            | Ingress Tool Transfer                  |
| T1036            | Masquerading                           |
| T1071.001        | Application Layer Protocol (HTTP)      |
| T1136            | Create Account                         |
| T1016            | System Network Configuration Discovery |
| T1083            | File and Directory Discovery           |

---

# 📑 Documentation Included

| File                 | Description                          |
| -------------------- | ------------------------------------ |
| `README.md`          | Project landing page.                |
| `Documentation.md`   | Complete investigation report.       |
| `Documentation.docx` | Printable professional report.       |
| `Resources/notes.md` | Analyst notes & quick references.    |
| `docs/index.md`      | Premium GitHub Pages portfolio page. |

---

# 🚨 Key DFIR Takeaways

### Network Forensics

* HTTP object reconstruction.
* Protocol-aware packet filtering.
* Session timeline analysis.

### Malware Analysis

* Static binary triage.
* PowerShell staging detection.
* Threat intelligence correlation.

### Threat Hunting

* Havoc C2 behavioral indicators.
* Beaconing traffic identification.
* Windows persistence detection.

### Incident Response

* Evidence preservation.
* Artifact correlation.
* Investigation reporting.

---

# 🎓 Learning Outcomes

This room teaches practical techniques used by:

* SOC Analysts
* DFIR Analysts
* Threat Hunters
* Malware Analysts
* Incident Responders
* Blue Team Engineers

The investigation emphasizes **evidence-driven reasoning** rather than exploitation.

---

# 🌐 GitHub Pages Portfolio

A premium documentation website accompanies this repository through **GitHub Pages**.

Features include:

* Cyber-themed landing page.
* Investigation timeline.
* Evidence gallery.
* MITRE ATT&CK mapping.
* Interactive navigation.
* Responsive layout.
* Custom SCSS styling.

---

# ⚠️ Public Write-up Policy

This repository intentionally **does not disclose**:

* 🚫 Challenge flags.
* 🚫 Passwords.
* 🚫 Persistence credentials.
* 🚫 Direct answer strings.
* 🚫 Plagiarism-friendly solutions.

All sensitive values appear as:

```text
[REDACTED]
```

The documentation explains **how** evidence was recovered without publishing the final challenge answers.

---

# 📚 References

* TryHackMe — **Mayhem**
* Wireshark Documentation
* VirusTotal Malware Intelligence
* CyberChef Documentation
* Havoc C2 Research
* MITRE ATT&CK Framework

---

# 👨‍💻 Author

## Anurag R

**Cybersecurity Enthusiast • Blue Team • SOC • DFIR • Threat Hunting**

Passionate about building professional cybersecurity portfolios through practical labs, CTF investigations, malware analysis, Active Directory security, and incident response documentation.

---

<div align="center">

### ⭐ If this repository helped you learn Network Forensics or Havoc C2 analysis, consider giving it a Star.

**Built for educational purposes, authorized security training, and cybersecurity portfolio development.**

</div>
