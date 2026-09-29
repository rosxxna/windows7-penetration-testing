# EternalBlue Home Lab Penetration Test

Documentation of a controlled penetration test against Windows 7 using MS17-010 (EternalBlue) in a home lab environment.

## Overview

This lab demonstrates the complete exploitation chain:
1. **Reconnaissance** — nmap service enumeration and OS detection
2. **Vulnerability Identification** — SMB signing disabled, guest access enabled, unpatched MS17-010
3. **Exploitation** — Metasploit EternalBlue module execution
4. **Post-Exploitation** — Meterpreter shell, Kiwi credential dump, system info harvest
5. **Remediation** — Patch paths and hardening steps

## Lab Environment

| Component | Details |
|---|---|
| Attacker | Kali Linux (VirtualBox) |
| Target | Windows 7 Home Basic, Build 7600 (VirtualBox) |
| Network | VirtualBox host-only network (isolated) |
| Virtualization | Oracle VirtualBox |

## Tools Used

- **nmap** — port scanning, service enumeration, OS detection
- **Metasploit Framework** — MS17-010 EternalBlue exploit module
- **Meterpreter** — post-exploitation shell
- **Kiwi** — credential harvesting from memory

## Key Findings

- **CVE-2017-0143** — SMB Remote Code Execution (EternalBlue)
- **Root Cause** — Unpatched Windows 7, SMB signing not enforced, guest access allowed
- **Impact** — Unauthenticated remote code execution with SYSTEM privileges
- **Remediation** — Apply MS17-010 patches (KB4012212 / KB4012215), enforce SMB signing, disable guest SMB

## Detailed Writeup

See [eternalblue-lab-writeup.md](eternalblue-lab-writeup.md) for step-by-step exploitation documentation.

## Lab Safety

This is a **controlled, isolated home lab** for educational purposes:
- Both VMs owned and operated by the lab owner
- Host-only network with no internet exposure
- No client systems, no real data, no production environment
- Standard penetration testing skillbuilding

## What You'll Learn

- Vulnerability reconnaissance and enumeration
- Exploit module selection and configuration
- Payload generation and delivery
- Post-exploitation data harvesting
- Remediation and hardening strategies

## Replication (High Level)

1. Spin up two VirtualBox VMs on host-only network
2. Configure Windows 7 target (no updates, build 7600)
3. From Kali, run nmap against target
4. Open msfconsole, search for eternalblue
5. Set RHOSTS, set LHOST/LPORT, run
6. Interact with resulting meterpreter session
7. Load kiwi, dump credentials

**Note:** This exploits a 2017 vulnerability on EOL software. Do not use against systems you don't own.

## Files

- `README.md` — This file
- `eternalblue-lab-writeup.md` — Detailed exploitation writeup
- `screenshots/` — Supporting screenshots and nmap output

---

**Portfolio Context:** This lab demonstrates hands-on penetration testing skills relevant to SOC analyst, security engineer, and junior penetration testing roles. It shows understanding of vulnerability research, exploitation frameworks, and post-compromise analysis.
