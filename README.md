# Active Directory / RBAC Home Lab

A hands-on build of an Active Directory environment implementing role-based
access control (RBAC), designed to practice the identity and access
management fundamentals used in real enterprise environments.

## Overview

This lab simulates the IT environment of a fictional company,
**ElysiumCorporation.Local**, with department-based organizational units,
security groups, NTFS/share permissions, and hardened domain policies.

**Environment:** Windows Server 2025, Active Directory Domain Services

## What Was Built

- **Domain Controller** (`DC01`) promoted and configured for
  `ElysiumCorporation.Local`
- **6 Organizational Units**, one per department: Warehouse Ops,
  Procurement, Fleet Management, Finance, Customer Service, IT Department
- **6 users and 6 security groups**, mapped to departments
- **NTFS/share permissions** applied and verified on department folders
  under `C:\CompanyShares`, following least-privilege design
- **Group Policy (Default Domain Policy)**:
  - Password Policy: 24-password history, 90-day max age, 1-day min age,
    12-character minimum length, complexity enabled, reversible encryption
    disabled
  - Account Lockout Policy: threshold 10 attempts, 1-minute lockout
    duration, 1-minute reset counter, Administrator account lockout enabled
- **Access matrix** documenting group-to-folder permissions (see
  `docs/RBAC_Access_Matrix.xlsx`)

## Design Decisions

**Least-privilege folder access.** Each department security group has
access to its own share only; every other department folder is set to
No Access. `IT-Admins` is the sole exception, with Full Control across
all shares, reflecting its administrative function.

**Finance restricted to Read-Only.** Given the sensitivity of financial
data, the Finance group was scoped to Read-Only rather than Modify, even
though every other department group has Modify on its own share.

**Account lockout set loose intentionally.** The lockout threshold/duration
were set permissively (10 attempts, 1-minute duration) rather than to a
hardened baseline (e.g. 5 attempts / 30 minutes). This was a deliberate
choice to avoid accidental admin lockouts while iterating on the lab, not
an oversight. In a production environment this would be tightened to
align with standard hardening baselines (e.g. CIS benchmarks).

## Repo Structure

```
ad-rbac-lab/
├── README.md
├── docs/
│   └── RBAC_Access_Matrix.xlsx   # Group-to-folder permission matrix
├── screenshots/                  # Build evidence (GPO settings, permissions, AD structure)
└── scripts/                      # Any PowerShell/automation scripts used in the build
```

## Next Steps

Planned follow-on labs on the same Domain Controller: DNS, DHCP, LDAP, and
Kerberos configuration and troubleshooting exercises.

## Protocol Labs

Hands-on exploration of the protocols underpinning this AD environment:

- [LDAP](protocol-labs/ldap/README.md) — directory structure and queries via `ldp.exe` and PowerShell
