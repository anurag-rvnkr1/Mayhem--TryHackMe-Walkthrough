# Mayhem — Analyst Notes

## Investigation chain

```text
PCAP
 ↓
Wireshark triage
 ↓
HTTP stream
 ↓
install.ps1
 ↓
notepad.exe
 ↓
strings
 ↓
Havoc attribution
 ↓
DEMON_INIT
 ↓
AES-CTR
 ↓
header stripping
 ↓
CyberChef
 ↓
decrypted C2
 ↓
host artifacts
```

## Quick facts

| Item | Value |
|---|---|
| Suspected victim | `10.0.2.38` |
| Staging/C2 host | `10.0.2.37` |
| Staging port | `1337` |
| Transport | HTTP |
| Payload | `notepad.exe` |
| Launcher | `install.ps1` |
| Request header | 20 bytes |
| Response header | 12 bytes |
| AES key | 32 bytes |
| AES IV | 16 bytes |
| AES mode | CTR |
| Init command | `DEMON_INIT` / `99` |
| Important file | `C:\Users\paco\Desktop\Files\clients.csv` |

## Wireshark actions

```text
Follow → HTTP Stream
File → Export Objects → HTTP
Right-click File Data → Copy → … as Hex Stream
```

## Public-answer policy

Flags and direct answer strings are not stored in this repository. Use the packet references in the main documentation to reproduce them independently.
