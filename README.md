## Domain Controller / Active Directory attacks.
I'm using my home lab VLAN to host an intentinoally vulnerable Windows Server 2019 Active Directory & Domain Controller for practice attacks.  Built for educational purposes in: Networking, Setting up an AD / DC, preparation for professional penetration testing.

| # | Technique | Status | Writeup |
|---|-----------|--------|---------|
| 1 | Kerberoasting | ✅ Complete | [findings/Kerberoasting.md](findings/Kerberoasting.md) |
| 2 | Golden Ticket | ✅ Complete | [findings/GoldenTicket.md](findings/GoldenTicket.md) |
| 3 | Pass the Hash | ✅ Complete | [findings/PassTheHash.md](findings/PassTheHash.md) |
| 4 | DCSync (secretsdump.py) | ✅ Complete  | [findings/DCSync.md](findings/DCSync.md) |
| 5 | AV/AMSI Bypass — Certify | ⬜ Pending | — |
| 6 | AV/AMSI Bypass — BloodHound | ⬜ Pending | — |
| 7 | AV/AMSI Bypass — Rubeus | ⬜ Pending | — |
| 8 | Responder — NetNTLMv2 Capture | ⬜ Pending | — |
| 9 | NTLM Relay (ntlmrelayx.py) | ⬜ Pending | — |
| 10 | Password Spraying | ⬜ Pending | — |
| 11 | ADCS — Certify.exe / Certify.py | ⬜ Pending | — |
| 12 | Vulnerable-AD attack list | ⬜ Pending | [Reference](https://github.com/safebuffer/vulnerable-AD?tab=readme-ov-file) |
| 13 | AWS C2 + SSH SOCKS proxy | ⬜ Pending | — |

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
**Vulnerable AD:** [safebuffer/vulnerable-AD](https://github.com/safebuffer/vulnerable-AD) + BadBlood

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
- SMB Signing: **True** — direct NTLM relay to this host not possible
- Null Auth: **True** — misconfiguration, allows unauthenticated enumeration
- SMBv1: **None** — legacy attack surface not present

---

> ⚠️ **This lab is for educational purposes only. All testing is performed against intentionally vulnerable systems in an isolated private network.**
