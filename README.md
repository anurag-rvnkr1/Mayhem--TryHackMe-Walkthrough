# Mayhem — TryHackMe Walkthrough

![Mayhem](docs/assets/01_cover_mayhem.png)

> **Blue-team network forensics · Havoc C2 · AES-CTR traffic reconstruction**

A portfolio-grade investigation of the **TryHackMe Mayhem** room, documenting the forensic path from raw PCAP traffic through PowerShell staging, payload extraction, Havoc C2 attribution, session decryption, and host-artifact recovery.

## Investigation flow

```text
PCAP
 → HTTP staging
 → install.ps1
 → notepad.exe
 → static triage
 → Havoc C2
 → DEMON_INIT
 → AES-CTR
 → decrypted C2
 → host artifacts
```

## Repository contents

```text
Mayhem--TryHackMe-Walkthrough/
├── Documentation/
│   ├── Documentation.md
│   └── Documentation.docx
├── Resources/
│   └── notes.md
├── docs/
│   ├── index.md
│   └── assets/
└── README.md
```

## Skills demonstrated

`Wireshark` · `PCAP Analysis` · `Network Forensics` · `Digital Forensics` · `HTTP Object Extraction` · `PowerShell Analysis` · `PE Triage` · `Havoc C2` · `AES-CTR` · `CyberChef`

## Public-writeup policy

Challenge flags, direct answer strings, and account credentials are intentionally **redacted**. The methodology and evidence chain remain documented so the room can be reproduced without publishing an answer key.

## GitHub Pages

The polished portfolio presentation is available from `docs/index.md`.

> For authorized labs and CTF environments only.
