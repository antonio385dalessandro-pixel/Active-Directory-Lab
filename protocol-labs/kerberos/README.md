# Kerberos

Part of the [Active Directory / RBAC Home Lab](../../README.md) — a set of
protocol-level ramp labs exploring the mechanisms underneath the AD/RBAC
build.

## What Kerberos Is

Kerberos is the authentication protocol Active Directory uses for logins.
Instead of sending a password across the network every time a resource is
accessed, a client authenticates once and receives a **ticket**, which is
then presented to prove identity to other services.

Two ticket types matter:
- **TGT (Ticket-Granting Ticket)** — issued at logon, proves "this is who
  I am" to the domain
- **Service ticket** — requested using the TGT, scoped to a specific
  service (e.g. file sharing, a specific host)

## What Was Done

**1. Captured an active ticket set with `klist`** immediately after login
on DC01, showing:
- A TGT issued by `krbtgt/ELYSIUMCORPORATION.LOCAL`, flagged
  `forwardable renewable initial pre_authent`, using
  `AES-256-CTS-HMAC-SHA1-96` encryption
- A service ticket for `host/dc01.elysiumcorporation.local`, generated
  automatically for local machine services
- Both tickets carry a ~10-hour lifetime and a 7-day renew window,
  matching the Kerberos Policy values set earlier in the domain GPO
  (Maximum lifetime for user ticket: 10 hours; renewal: 7 days)

**2. Purged and re-verified the ticket cache** with `klist purge`,
confirming the cache dropped to zero cached tickets.

**3. Tested ticket reissuance by accessing a network share**
(`\\DC01\Finance`) and re-running `klist`.

## Finding: Loopback Access Does Not Force a New Kerberos Exchange

Accessing a share hosted on DC01 *from DC01 itself* did not produce a new
Kerberos ticket exchange after the cache was purged — the ticket cache
remained at zero. This is expected Windows loopback behavior: when a
machine accesses its own resources, Windows can rely on the local
security token already established at logon rather than performing a
full network authentication round-trip via Kerberos.

This is a useful distinction — Kerberos is specifically a **network**
authentication protocol. Its ticket exchange is only meaningfully
observable between two separate machines (a client requesting access to
a resource hosted elsewhere), not when a machine accesses itself.

## Scope Note

A full cross-machine Kerberos exchange (domain-joining a second VM and
observing ticket issuance from that client) was planned but not
completed in this pass — it was set aside due to host VM resource
constraints, not a technical blocker. The ticket structure, lifetime
policy alignment, and loopback behavior captured above still demonstrate
the core mechanics of how Kerberos underpins AD authentication.

## Screenshots

See `screenshots/` in this folder for the `klist` output before and
after purge, and the loopback share access test.
