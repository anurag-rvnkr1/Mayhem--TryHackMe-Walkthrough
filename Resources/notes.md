# 📘 Resources / notes.md

> **Mayhem — TryHackMe Blue Team Investigation Notes**
>
> **Room:** Mayhem
>
> **Category:** Network Forensics • Digital Forensics • Malware Traffic Analysis • Havoc C2
>
> **Difficulty:** Medium
>
> **Platform:** TryHackMe
>
> **Author:** Anurag R

---

# Overview

This file contains concise investigation notes and analyst references collected while solving the **TryHackMe Mayhem** room. These notes are intended for **SOC analysts, DFIR practitioners, threat hunters, and cybersecurity students** who want a quick technical reference without revealing the room's challenge flags.

Unlike the main documentation, this file focuses on packet-level observations, protocol behavior, malware indicators, Wireshark filters, cryptographic workflow, and investigative methodology.

> **Public Repository Notice:** All flags, passwords, credentials, and direct challenge answers are intentionally **redacted**.

---

# Investigation Summary

| Item                  | Observation                                                |
| --------------------- | ---------------------------------------------------------- |
| Investigation Type    | Blue Team Network Forensics                                |
| Evidence Source       | Packet Capture (`traffic.pcapng`)                          |
| Victim Host           | `10.0.2.38`                                                |
| Suspicious Server     | `10.0.2.37`                                                |
| Initial Transfer Port | `1337/TCP`                                                 |
| Payload               | `install.ps1` → `notepad.exe`                              |
| Malware Family        | Havoc C2                                                   |
| C2 Transport          | HTTP                                                       |
| Encryption            | AES-CTR                                                    |
| Primary Goal          | Recover attacker actions through encrypted network traffic |

---

# Incident Timeline (Quick View)

```text
PCAP Collected
      │
      ▼
HTTP Request for install.ps1
      │
      ▼
PowerShell downloads notepad.exe
      │
      ▼
Malicious executable launched
      │
      ▼
HTTP Beaconing Begins
      │
      ▼
Havoc DEMON_INIT
      │
      ▼
AES Key + IV Recovered
      │
      ▼
Encrypted Traffic Decrypted
      │
      ▼
Persistence + Discovery + Sensitive File Access
```

---

# Evidence Chain

## Stage 1 — Packet Capture Analysis

### Initial Findings

* Ignore initial FIN packets.
* Focus on new TCP session between:

  * `10.0.2.38`
  * `10.0.2.37`
* Observe HTTP communication after TCP three-way handshake.

### Indicators

| Indicator     | Description                                   |
| ------------- | --------------------------------------------- |
| TCP Handshake | New HTTP communication begins.                |
| Port 1337     | Python HTTP server used for staging payloads. |
| HTTP GET      | Downloads `install.ps1`.                      |

---

## Stage 2 — PowerShell Downloader

### Observed Behaviour

The recovered PowerShell script performs:

1. Download executable.
2. Save inside user Downloads folder.
3. Execute payload.

### Analyst Observation

This resembles a common malware staging technique:

* Living-off-the-land.
* User writable directory.
* Immediate execution.

### Detection Opportunities

* PowerShell downloading executables.
* WebClient object creation.
* Invoke-WebRequest abuse.

---

## Stage 3 — HTTP Object Extraction

### Wireshark Path

```text
File
 └── Export Objects
      └── HTTP
```

### Exported Objects

| Object      | Purpose               |
| ----------- | --------------------- |
| install.ps1 | Initial launcher.     |
| notepad.exe | Malicious executable. |

### Best Practice

Always export the binary instead of executing it.

---

## Stage 4 — Static Malware Triage

### `strings` Investigation

Useful command:

```bash
strings notepad.exe
```

### Interesting Findings

* Suspicious executable references.
* Havoc Demon artifacts.
* Internal Windows API references.

### Malware Characteristics

* Masquerades as Notepad.
* References Havoc components.
* Designed for post-exploitation.

---

## Stage 5 — Threat Intelligence

### VirusTotal Observations

Malware classified as:

* Trojan
* Backdoor
* Havoc
* Demon Agent

### Analyst Conclusion

Multiple vendors associate the sample with **Havoc C2** infrastructure.

### IOC Type

| IOC             | Value         |
| --------------- | ------------- |
| Executable Name | `notepad.exe` |
| Framework       | Havoc         |
| Communication   | HTTP          |

---

# Havoc C2 Notes

## What is Havoc?

Havoc is a modern **Command and Control framework** used for:

* Red Team Operations.
* Adversary Simulation.
* Post-Exploitation.
* Beacon Management.

### Architecture

```text
Operator
    │
    ▼
Teamserver
    │
HTTP Listener
    │
    ▼
Demon Agent
    │
Victim Host
```

---

## Havoc Session Initialization

The first HTTP POST contains the registration packet.

### Important Command

| Command | Meaning    |
| ------- | ---------- |
| `99`    | DEMON_INIT |

### DEMON_INIT Contains

* Agent ID
* Magic Bytes
* AES Session Key
* AES Initialization Vector

This packet is the foundation for decrypting later traffic.

---

# Havoc Packet Structure

## Request Header

| Offset | Field        |
| ------ | ------------ |
| 0–3    | Payload Size |
| 4–7    | Magic Bytes  |
| 8–11   | Agent ID     |
| 12–15  | Command ID   |
| 16–19  | Memory ID    |

### Header Length

```text
20 Bytes
```

---

## Response Header

Response packets contain:

```text
12 Byte Header
Encrypted Payload
```

### Important Difference

| Traffic       | Remove Before Decryption |
| ------------- | ------------------------ |
| POST Request  | First 20 Bytes           |
| HTTP Response | First 12 Bytes           |

---

# AES-CTR Decryption Notes

## Encryption Mode

```text
AES
Mode : CTR
```

### Key Information

| Item    | Size     |
| ------- | -------- |
| AES Key | 32 Bytes |
| AES IV  | 16 Bytes |

---

## CyberChef Recipe

```text
AES Decrypt
Mode : CTR

Input : Hex

Output : Raw
```

### Required Inputs

* AES Key
* AES IV
* Ciphertext
* CTR Mode

---

## Wireshark Extraction Method

Right click:

```text
File Data
    │
    ▼
Copy
    ▼
...as Hex Stream
```

Paste directly into CyberChef.

---

## Header Removal Rule

### POST Requests

```text
20 Bytes
=
40 Hex Characters
```

Remove before decrypting.

### HTTP Responses

```text
12 Bytes
=
24 Hex Characters
```

Remove before decrypting.

---

# Useful Wireshark Filters

## HTTP Only

```wireshark
http
```

---

## Traffic Between Two Hosts

```wireshark
ip.addr==10.0.2.38 && ip.addr==10.0.2.37
```

---

## Port 1337 Traffic

```wireshark
tcp.port==1337
```

---

## Hide Empty Havoc Check-ins

```wireshark
tcp.port != 1337 &&
http &&
!(http.file_data == 00:00:00:10:de:ad:be:ef:0e:9f:b7:d8:00:00:00:01:00:00:00:00) &&
!(http.file_data == 0a:00:00:00:00:00:00:00:00:00:00:00)
```

Purpose:

* Removes empty beacon traffic.
* Keeps meaningful encrypted commands.

---

# Interesting Artifacts

## Windows User SID

Recovered from decrypted response.

**Status**

```text
[REDACTED]
```

---

## IPv6 Configuration

Recovered through decrypted `ipconfig`.

**Status**

```text
[REDACTED]
```

---

## Persistence Account

Recovered from attacker command execution.

**Status**

```text
Username : [REDACTED]
Password : [REDACTED]
```

---

## Sensitive File Discovery

Recovered from decrypted file search command.

```text
C:\Users\paco\Desktop\Files\clients.csv
```

---

## Final Challenge Artifacts

Recovered from decrypted HTTP responses.

**Status**

```text
[REDACTED]
```

---

# Network Indicators of Compromise

## Network IOCs

| IOC              | Observation   |
| ---------------- | ------------- |
| Source Host      | `10.0.2.38`   |
| Destination Host | `10.0.2.37`   |
| Transport        | HTTP          |
| Staging Port     | TCP 1337      |
| Beacon Port      | HTTP/80       |
| Payload Transfer | `install.ps1` |
| Malware Delivery | `notepad.exe` |

---

## Host IOCs

| IOC                 | Description             |
| ------------------- | ----------------------- |
| `install.ps1`       | Downloader script       |
| `notepad.exe`       | Masquerading executable |
| `demon.x64.exe`     | Havoc Demon artifact    |
| Downloads Directory | Payload execution path  |
| New Windows Account | Persistence mechanism   |

---

# Malware Behaviour Summary

## Initial Access

* HTTP download.
* PowerShell execution.

## Execution

* Downloads executable.
* Launches immediately.

## Command & Control

* Registers with Havoc Teamserver.
* Sends encrypted HTTP POST requests.

## Persistence

* Creates Windows account.

## Discovery

* SID enumeration.
* Network enumeration.
* Sensitive file discovery.

---

# MITRE ATT&CK Mapping

| Technique ID | Technique                       |
| ------------ | ------------------------------- |
| T1059.001    | PowerShell                      |
| T1105        | Ingress Tool Transfer           |
| T1036        | Masquerading                    |
| T1071.001    | HTTP Command & Control          |
| T1136        | Create Account                  |
| T1016        | Network Configuration Discovery |
| T1083        | File Discovery                  |

---

# Detection Opportunities

## Sigma Ideas

* PowerShell downloading executable from HTTP.
* PowerShell invoking `Start-Process`.
* Executable written to Downloads folder.
* Outbound HTTP to uncommon internal port.
* New local user creation.
* HTTP beacon interval detection.

---

## Sysmon Events Worth Monitoring

| Event       | Description           |
| ----------- | --------------------- |
| Event ID 1  | Process Creation      |
| Event ID 3  | Network Connection    |
| Event ID 7  | Image Loaded          |
| Event ID 11 | File Creation         |
| Event ID 13 | Registry Modification |
| Event ID 22 | DNS Query             |

---

## Windows Event IDs

| Event ID | Meaning                |
| -------- | ---------------------- |
| 4624     | Successful Logon       |
| 4625     | Failed Logon           |
| 4688     | Process Creation       |
| 4720     | User Account Created   |
| 4722     | Account Enabled        |
| 4732     | Group Membership Added |

---

# Analyst Cheat Sheet

## Quick Workflow

```text
Open PCAP
    ↓
Find install.ps1
    ↓
Follow HTTP Stream
    ↓
Export HTTP Objects
    ↓
Analyse notepad.exe
    ↓
Identify Havoc
    ↓
Locate DEMON_INIT
    ↓
Extract AES Key
    ↓
Extract IV
    ↓
Copy Hex Stream
    ↓
Remove Header Bytes
    ↓
CyberChef AES CTR
    ↓
Recover Artifacts
```

---

## Investigation Checklist

* [x] Identify victim host.
* [x] Identify attacker infrastructure.
* [x] Recover PowerShell downloader.
* [x] Export malicious executable.
* [x] Perform static analysis.
* [x] Identify Havoc framework.
* [x] Recover AES cryptographic material.
* [x] Decrypt encrypted HTTP payloads.
* [x] Recover Windows artifacts.
* [x] Document findings professionally.

---

# Lessons Learned

### Blue Team Perspective

* PCAPs preserve the complete intrusion narrative.
* HTTP object extraction is invaluable during malware investigations.
* Malware frameworks often reveal recognizable protocol structures.
* Understanding protocol headers is essential before attempting decryption.
* External intelligence should validate evidence—not replace packet analysis.

### DFIR Perspective

* Build a timeline before answering questions.
* Correlate packets, processes, files, and decrypted output.
* Preserve evidence integrity throughout the investigation.
* Document every finding with packet references and analyst reasoning.

---

# References

### Official Room

* **TryHackMe — Mayhem**

### Research References

* Havoc C2 Research (Zscaler)
* Immersive Labs — HavocC2-Forensics
* Wireshark Documentation
* CyberChef Documentation
* MITRE ATT&CK Framework

---

# Repository Notes

This repository is intended for:

* 📘 Educational use.
* 🛡️ Blue Team training.
* 🔬 Digital Forensics practice.
* 🎯 SOC analyst portfolio projects.
* 🧑‍💻 Threat hunting demonstrations.

The accompanying `Documentation.md` provides the complete investigation report, while `docs/index.md` contains a premium GitHub Pages portfolio version with evidence galleries and interactive documentation.

> **Flags, passwords, credentials, and direct challenge answers remain intentionally redacted throughout this repository.**
