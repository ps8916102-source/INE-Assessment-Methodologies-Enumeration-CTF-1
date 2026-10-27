# INE Assessment Methodologies: Enumeration CTF 1

## Overview

This repository documents my walkthrough of the **Assessment Methodologies: Enumeration CTF 1** lab on INE. The lab focuses on enumerating network services, discovering exposed resources, identifying weak credentials, and inspecting service banners.

The walkthrough covers four completed flags, with sensitive flag values and passwords redacted.

## Lab Objectives

- Identify accessible network services.
- Discover SMB/Samba shares that permit anonymous access.
- Enumerate Samba users and investigate weak credentials.
- Discover an FTP service running on a non-standard port.
- Inspect FTP and SSH banners for useful information.
- Practice a structured enumeration methodology.

## Environment

| Item | Details |
|---|---|
| Platform | INE |
| Lab | Assessment Methodologies: Enumeration CTF 1 |
| Target hostname | `target.ine.local` |
| Target IP | `192.170.178.3` |

## Tools Used

- Nmap
- smbclient
- enum4linux
- Metasploit Framework
- Hydra
- FTP client
- SSH client
- Bash scripting

## Key Findings

1. An SMB share named `pubfiles` allowed anonymous access.
2. Samba enumeration revealed user accounts and a private share containing a flag file.
3. An FTP service was discovered on TCP port `5554`, and its banner provided useful enumeration clues.
4. An SSH login warning banner contained information relevant to the final flag.

## Repository Contents

- **`README.md`** — Lab overview, objectives, tools, and key findings.
- **`lab-notes.md`** — Detailed commands, explanations, and the step-by-step methodology.

## Disclaimer

This repository is intended for educational purposes and documents activity in an authorized training environment. Flag values and passwords are intentionally excluded. Always obtain permission before testing any system and review INE's current rules before publishing lab solutions.
