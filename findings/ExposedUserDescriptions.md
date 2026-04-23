# Exposed Credentials in the AD User Description Details

| Field | Details |
|-------|---------|
| **Severity** | 🟠 High |
| **Technique** | LDAP User Enumeration / Credential Exposure |
| **Target** | `192.168.120.100` — `DC01-WINDOWSSERVER2019` |
| **Domain** | `lab.local` |
| **Prerequisite** | Assumed Initial Access (`lab.local\gsnake`) |

## Description

In the Active Directory, the User Description field is readable by any authenticated domain user and requires no elevated privileges to access.  Three accounts were found with credential information stored in plaintext in their User Description field.  This is easily discoverable with a single LDAP enumeration command.

## Evidence

### Evidence (A) : User Enumeration and Credential Discovery

Using credentials supplied from the Assumed Initial Access scenario, enumeration of users from LDAP revealed two accounts with exposed plaintext passwords, and a third with 'DefaultPassword' which did not authenticate.

    netexec smb 192.168.120.100 -u 'gsnake' -p 'iloveyou01!' --users

    SMB         192.168.120.100 445    DC01-WINDOWSSER  jodee.erin                    2026-03-21 19:13:38 0       User Password B:6_}lY$Hs+7
    SMB         192.168.120.100 445    DC01-WINDOWSSER  michaelina.petronia           2026-03-21 19:13:38 0       User Password 19t1>C+DJrDI
    SMB         192.168.120.100 445    DC01-WINDOWSSER  joceline.kaja                 <never>             0       New User ,DefaultPassword

### Evidence (B) : Credential Validation

Credentials for `jodee.erin` and `michaelina.petronia` were confirmed valid via authentication. `joceline.kaja` did not authenticate with 'DefaultPassword' or a blank password, but is a finding and should be noted.

    netexec smb 192.168.120.100 -u 'jodee.erin' -p 'B:6_}lY$Hs+7'
    
    SMB  192.168.120.100  445  DC01-WINDOWSSER  [*] Windows 10 / Server 2019 Build 17763 x64
     (name:DC01-WINDOWSSER) (domain:lab.local) (signing:True) (SMBv1:None) (Null Auth:True)

    SMB  192.168.120.100  445  DC01-WINDOWSSER  [+] lab.local\jodee.erin:B:6_}lY$Hs+7
---
    netexec smb 192.168.120.100 -u 'michaelina.petronia' -p '19t1>C+DJrDI'

    SMB  192.168.120.100  445  DC01-WINDOWSSER  [*] Windows 10 / Server 2019 Build 17763 x64
     (name:DC01-WINDOWSSER) (domain:lab.local) (signing:True) (SMBv1:None) (Null Auth:True)

    SMB  192.168.120.100  445  DC01-WINDOWSSER  [+] lab.local\michaelina.petronia:19t1>C+DJrDI
---
    netexec smb 192.168.120.100 -u 'joceline.kaja' -p 'DefaultPassword'

    SMB  192.168.120.100  445  DC01-WINDOWSSER  [-] lab.local\joceline.kaja:DefaultPassword

### Commands Used

    netexec smb 192.168.120.100 -u 'gsnake' -p 'iloveyou01!' --users
    netexec smb 192.168.120.100 -u 'jodee.erin' -p 'B:6_}lY$Hs+7'
    netexec smb 192.168.120.100 -u 'michaelina.petronia' -p '19t1>C+DJrDI'
    netexec smb 192.168.120.100 -u 'joceline.kaja' -p 'DefaultPassword'

## Impact

Because LDAP user enumeration can be done with any authenticated account on the DC, exposed secrets in "User Description" is a high impact, resulting in compromised secrets being exposed in the open.  3 Accounts were identified with information regarding their credentials in plaintext.  2 of those accounts had direct plaintext passwords in their User Description field which authenticated.  If using a "default password" is an existing provisioning standard, every new account is compromised before the user ever logs in.  Coupled with no account lockout threshold the default password can be sprayed to reveal other accounts that have never changed upon creation (read: [Password Spraying](PasswordSpraying)).

## Remediation

> Remediations are not listed in any specific order of severity unless otherwise listed.

1. **[IMPORTANT]** Never put secrets in the "User Description" field.

2. Replace any "default password" provisioning policies with a Randomized At First Login password policy.

3. Staff training and onboarding processes to eliminate this practice entirely.

4. Audit existing user profiles for exposed secrets, critical or vital information, in the "User Description" field.

---

## Detection

1. This finding generates no events during its exploitation, which makes detection difficult.

2. **Event ID 4661** will help flag bulk enumeration of user objects from non administrative accounts or anomalous sources.
