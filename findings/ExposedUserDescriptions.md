# Exposed Credentials in the AD User Description Details

| Field | Details |
|-------|---------|
| **Severity** | 🟠 High |
| **Technique** | LDAP User Enumeration / Credential Exposure |
| **Target** | `192.168.120.100` — `DC01-WINDOWSSERVER2019` |
| **Domain** | `lab.local` |
| **Prerequisite** | Known Breach Credentials (`lab.local\gsnake`) |

## Description

Simply checking the "User Description" field of LDAP enumerated accounts.

## Evidence

### Evidence (A) : User Enumeration and Credential Discovery

Using credentials supplied from the Known Breach scenario (this can be performed by any authenticated account on the domain), enumeration of users from LDAP revealed two accounts with exposed plaintext passwords, and a third with 'DefaultPassword' which did not authenticate, but is dangerous if its cracked and the default password is used for all new accounts, allowing all new accounts to be open to exposure. Using a "default password" also opens all new accounts created for new users to guess the passwords for any new accounts created as well, coupled with no lockout procedure (read: [Password Spraying](PasswordSpraying)) this is dangerous.

    netexec smb 192.168.120.100 -u 'gsnake' -p 'iloveyou01!' --users

    SMB         192.168.120.100 445    DC01-WINDOWSSER  jodee.erin                    2026-03-21 19:13:38 0       User Password B:6_}lY$Hs+7
    SMB         192.168.120.100 445    DC01-WINDOWSSER  michaelina.petronia           2026-03-21 19:13:38 0       User Password 19t1>C+DJrDI
    SMB         192.168.120.100 445    DC01-WINDOWSSER  joceline.kaja                 <never>             0       New User ,DefaultPassword

### Evidence (B) : Credential Validation

Credentials for `jodee.erin` and `michaelina.petronia` were confirmed valid via authentication. `joceline.kaja` did not authenticate with 'DefaultPassword' or a blank password, but is a finding and should be noted.

### Commands Used

    netexec smb 192.168.120.100 -u 'gsnake' -p 'iloveyou01!' --users

## Impact

Because LDAP user enumeration can be done with any authenticated account on the DC, exposed secrets in "User Description" is a high impact, resulting in compromised secrets being exposed in the open.

## Remediation

> Remediations are not listed in any specific order of severity unless otherwise listed.

1. **[IMPORTANT]** Never put secrets in the "User Description" field.
2. Audit existing user profiles for exposed secrets, critical or vital information, in the "User Description" field.
3. Replace any "default password" provisioning policies with a Randomized At First Login password policy.
