## Domain Controller / Active Directory attacks.
I'm using my home lab VLAN to host an intentionally vulnerable Windows Server 2019 Active Directory & Domain Controller for practice attacks.  Built for educational purposes in: Networking, Setting up an AD / DC, preparation for professional penetration testing.  Attacks documented in this series are from an "Assumed Breach" initial foothold where low-level credentials have been supplied by the party requiring the testing.

| # | Technique | Status | Writeup |
|---|-----------|--------|---------|
| 1 | Kerberoasting | ✅ Complete | [findings/Kerberoasting.md](findings/Kerberoasting.md) |
| 2 | Golden Ticket | ✅ Complete | [findings/GoldenTicket.md](findings/GoldenTicket.md) |
| 3 | Pass the Hash | ✅ Complete | [findings/PassTheHash.md](findings/PassTheHash.md) |
| 4 | DCSync (secretsdump.py) | ✅ Complete | [findings/DCSync.md](findings/DCSync.md) |
| 5 | Password Spraying | ✅ Complete | [findings/PasswordSpraying.md](findings/PasswordSpraying.md) |
| 6 | LDAP Exposed User Descriptions | ✅ Complete | [findings/ExposedUserDescriptions.md](findings/ExposedUserDescriptions.md) |
| 7 | AS-REP Roasting | ⬜ Pending | — |
| 8 | Abusing ACLs/ACEs | ⬜ Pending | — |
| 9 | Silver Ticket | ⬜ Pending | — |
| 10 | Pass-the-Ticket | ⬜ Pending | — |
| 11 | Abuse DnsAdmins | ⬜ Pending | — |
| 12 | NTLM Relay (ntlmrelayx.py) | ⬜ Pending | — |
| 13 | Responder — NetNTLMv2 Capture | ⬜ Pending | — |
| 14 | ADCS — Certify.exe / Certify.py | ⬜ Pending | — |
| 15 | AV/AMSI Bypass — Certify | ⬜ Pending | — |
| 16 | AV/AMSI Bypass — BloodHound | ⬜ Pending | — |
| 17 | AV/AMSI Bypass — Rubeus | ⬜ Pending | — |
| 18 | SMB Signing Disabled | ⬜ Pending | — |
| 19 | AWS C2 + SSH SOCKS Proxy | ⚠️ Partial | — |

## Lab Environment

| Host | IP | OS | Role |
|------|----|----|------|
| Kali (Attack Machine) | 10.0.8.2 (VPN) | Kali Linux Rolling | Bare metal attack machine |
| DC01-WINDOWSSERVER2019 | 192.168.120.100 | Windows Server 2019 | Active Directory / Domain Controller |
| Workstation | 192.168.110.100 | Windows 10 Pro | Domain-joined workstation |
| Ubuntu (DMZ) | 192.168.100.101 | Ubuntu Server | JuiceShop (:80), DVWA (:8081), bWAPP (:8082) |

**Domain:** `lab.local`  
**Hypervisor:** VMware on dedicated Laptop.  
**Router:** pfSense (VPN access via OpenVPN)  
**Vulnerable AD:** [safebuffer/vulnerable-AD](https://github.com/safebuffer/vulnerable-AD) + [David Prowe's BadBlood](https://github.com/davidprowe/BadBlood)

---

## Initial Recon

```bash
netexec smb 192.168.120.100
```

```
SMB  192.168.120.100  445  DC01-WINDOWSSER  [*] Windows 10 / Server 2019 Build 17763 x64
     (name:DC01-WINDOWSSER) (domain:lab.local) (signing:True) (SMBv1:None) (Null Auth:True)
```

**Key observations:**
- SMB Signing: **True** — Direct NTLM relay to this host not possible.
- Null Auth: **True** — Misconfiguration, allows unauthenticated enumeration.
- SMBv1: **None** — Legacy attack surface not present.

---

> ⚠️ **This lab is for documentation practices, educational purposes, and all of my own equipment/property.**
