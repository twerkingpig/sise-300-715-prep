# 07 — Network Access Device Administration (TACACS+)

> ~5% of the SISE exam. Light topic but easy points if you know the moving parts.

This is about administrators logging into network devices (switches, routers, firewalls) — *who* can SSH to your switch, and what commands they're allowed to run. The mechanism is TACACS+ (Cisco's proprietary protocol), and the configuration in ISE happens in a separate "Work Center."

---

## What the exam tests on this topic

1. **TACACS+ vs RADIUS** — the differences, when to use which.
2. **Device Administration license** — separate from the rest of ISE; required for TACACS+.
3. **TACACS+ Work Center** — the dedicated UI for device admin policy.
4. **Command Sets** — which commands a logged-in admin can/can't run.
5. **Shell Profiles** — what privilege level and other shell attributes the admin gets.
6. **TACACS+ policy sets** — separate from RADIUS policy sets.
7. **Device Admin authn vs authz** — same split as RADIUS, but applied to admin sessions.

---

## Key concepts in plain English

### TACACS+ vs RADIUS

| | TACACS+ | RADIUS |
|---|---|---|
| **Owner** | Cisco proprietary | IETF standard |
| **Transport** | TCP/49 | UDP/1812-1813 |
| **Encryption** | Entire payload encrypted | Only password encrypted |
| **AAA separation** | Authn / Authz / Accounting are separate exchanges | Authn and Authz combined in one exchange |
| **Use case** | Device administration (per-command authorization) | Network access (802.1X, MAB) |
| **Granularity** | Per-command authorization possible | Session-level only |

For *device admin* (who can log into a switch and what commands they can run), TACACS+ is the right protocol. For *network access* (endpoints connecting to the LAN), RADIUS is the right protocol.

### Device Administration license

TACACS+ is gated by a separate license SKU. **Even if you have full Premier ISE licensing for everything else, TACACS+ won't work without the Device Admin license.** Common mistake.

Per Cisco licensing model, this is: *Device Administration license, per node where TACACS+ runs*.

### TACACS+ Work Center

ISE has dedicated "Work Centers" for major feature areas. The TACACS+ work center lives at **Work Centers → Device Administration**. Inside it:

- TACACS+ Servers (the PSNs running the TACACS+ service)
- Network Devices (switches, routers — same NAD list, but with TACACS+ shared secrets configured)
- Policy Elements: Command Sets, Shell Profiles
- Policy Sets: Device Admin Policy Sets (separate from RADIUS Policy Sets)

Don't confuse the RADIUS policy area with the Device Admin policy area. They're separate.

### Command Sets

Define *which commands an admin can run*. Example:

```
Command Set: NetOps-RO  (read-only)
  Permit:
    show *
    ping *
  Deny:
    *  (everything else)
```

```
Command Set: NetOps-Full
  Permit:
    *
```

Command Sets are matched per-command after a successful login. Each command the admin types gets evaluated against the assigned Command Set.

### Shell Profiles

Define *what privilege level and shell attributes the admin gets*. Example:

```
Shell Profile: NetOps-Priv15
  Default Privilege: 15
  Maximum Privilege: 15
  Auto-Command: terminal length 0
```

Shell Profile is applied at login. Together, Shell Profile + Command Set define the admin's experience.

### Device Admin Policy Sets

Just like RADIUS Policy Sets, but for TACACS+. Authentication policy → Authorization policy → return Shell Profile + Command Set. The structure is parallel; the contents are different.

---

## Common exam traps

1. **TACACS+ on UDP is wrong.** TCP/49. Don't confuse with RADIUS UDP ports.

2. **Device Admin license is separate.** Easy to miss, and it can't be a sub-license of an existing tier — it's its own SKU.

3. **Per-command authorization is a distinct TACACS+ exchange.** Each command the admin types triggers a TACACS+ Authorization-Request to ISE. ISE checks against Command Set and returns Permit/Deny. RADIUS *cannot* do per-command authz.

4. **Shell Profile + Command Set are returned separately.** You configure them as separate Policy Elements; an authz rule attaches both.

5. **NAD configuration:** the switch needs `aaa authentication login default group <tacacs-group>`, `aaa authorization commands 15 default group <tacacs-group>`, `aaa accounting commands 15 default start-stop group <tacacs-group>`. Each line is a different AAA scope.

---

## Next

- [`lab.md`](lab.md), [`self-test.md`](self-test.md), [`notes.md`](notes.md)
