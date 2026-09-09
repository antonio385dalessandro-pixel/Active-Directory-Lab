# LDAP (Lightweight Directory Access Protocol)

Part of the [Active Directory / RBAC Home Lab](../../README.md) — a set of
protocol-level ramp labs exploring the mechanisms underneath the AD/RBAC
build, aimed at building help-desk-to-IAM fundamentals.

## What LDAP Is

LDAP is the protocol Active Directory uses under the hood to store and
query every object in the directory — every user, group, and
organizational unit built in the main lab is really an LDAP object
underneath the GUI.

## What Was Done

**1. Explored the raw directory via `ldp.exe`** (built into Windows Server)
- Connected to `localhost:389` and reviewed the RootDSE, confirming
  `rootDomainNamingContext: DC=ElysiumCorporation,DC=Local`
- Bound as the domain Administrator via Kerberos/Negotiate
- Browsed the full directory tree, confirming all six OUs exist as raw
  LDAP containers
- Drilled into an individual user object (`Keycee Filippina`, IT
  Department) and reviewed raw attributes: `distinguishedName`,
  `sAMAccountName`, `userPrincipalName`, `userAccountControl`, and
  critically `memberOf: CN=IT Admins,OU=IT Department,...` — confirming
  her group membership directly at the protocol level
- Also observed domain-wide password/lockout policy values
  (`lockoutDuration`, `lockoutThreshold`, `maxPwdAge`, `minPwdLength`)
  stored as raw LDAP attributes on the domain object — the same GPO
  settings configured earlier in the main lab, visible here as directory
  data rather than GUI settings

**2. Queried the same data via PowerShell** (practical, scriptable
equivalent of the same LDAP queries)

```powershell
# Users and group membership within a specific OU
Get-ADUser -Filter * -SearchBase "OU=IT Department,DC=ElysiumCorporation,DC=Local" `
    -Properties MemberOf | Select-Object Name, SamAccountName, MemberOf

# Every user in the domain, with group membership
Get-ADUser -Filter * -Properties MemberOf | Select-Object Name, SamAccountName, MemberOf
```

Ran the domain-wide query and confirmed all six department users resolve
to the correct security group, matching the access matrix exactly:

| User               | Group                  |
|--------------------|-------------------------|
| Giulia Colli        | Warehouse-Staff         |
| Tyra Soeum           | Procurement-Staff       |
| Prue Waldon          | Fleet Management-Staff  |
| Sharon Win           | Finance-ReadOnly        |
| Stephanie Barlow    | Customer Service-Staff  |
| Keycee Filippina     | IT Admins                |

## Why This Matters

This confirms the RBAC design isn't just a GUI configuration that looks
right — it holds up at the protocol/data level, which is what actually
gets queried during authentication and access decisions. Being able to
query this directly (rather than only through the GUI) is also the
starting point for real audit and automation work — e.g. `Get-ADGroupMember`
to review who holds privileged access, which is a common IAM/security
review task.

## Screenshots

See `screenshots/` in this folder for the `ldp.exe` bind/browse sequence
and the PowerShell query output.
