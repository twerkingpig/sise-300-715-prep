# 08 — TrustSec (SGT)

> Tested as part of Policy Enforcement (split across the blueprint, but conceptually distinct enough to deserve its own section).

TrustSec is Cisco's tag-based segmentation. Instead of enforcing access by VLAN or IP, you tag the *traffic* with a Security Group Tag (SGT) when it enters the network, then enforce based on tag-vs-tag at any point in the fabric.

---

## What the exam tests on this topic

1. **What an SGT is** — and why it's better than VLAN/ACL-based segmentation in some scenarios.
2. **How SGTs are assigned** — typically via ISE authz profile during dot1x/MAB.
3. **SXP (SGT Exchange Protocol)** — how SGTs propagate to devices that don't natively process them.
4. **SGACLs** — how SGTs are enforced.
5. **Environment Data** — what NADs download from ISE to implement TrustSec.
6. **Inline tagging vs SXP** — two ways SGT info travels.

---

## Key concepts in plain English

### Why SGT vs VLAN+ACL

| Approach | How segmentation works | Limitations |
|---|---|---|
| VLAN + ACL | Endpoint goes into VLAN → ACLs at distribution enforce | VLAN is L2, tied to physical/logical topology; ACLs grow with N×N policy |
| SGT-based | Endpoint gets tagged with an SGT (e.g., "Employee", "Contractor") → SGACLs enforce based on tags anywhere in the fabric | Requires NAD support and an environment data exchange |

The big win: **policy follows the user, not the IP/VLAN.** Move an endpoint, the SGT moves with the auth.

### How SGTs get assigned

Authorization profiles in ISE can include an SGT alongside (or instead of) a VLAN. Example:

```
Authorization Profile: EMPLOYEE-FULL-ACCESS
  Access Type: ACCESS_ACCEPT
  VLAN: EMPLOYEES (100)
  SGT: 10  (Employee)
```

When the switch authorizes the endpoint, it tags the endpoint's session with SGT 10. Future traffic from the endpoint gets tagged at L2 (inline tagging) or referenced via SXP (mapped IP-to-SGT propagation).

### SXP — SGT Exchange Protocol

Some devices can *enforce* SGTs but can't *natively process* them at L2 (e.g., older firewalls, non-TrustSec-capable equipment). SXP is a TCP-based protocol that lets a TrustSec-aware device tell another device "IP 10.10.30.42 has SGT 10."

This way, an ASA firewall that doesn't speak inline TrustSec can still enforce SGT-based policy by getting IP-to-SGT mappings from the access switch.

### SGACL

An SGACL is an ACL whose match conditions are SGTs:

```
SGACL: Permit Employees to Servers
  Source SGT: 10 (Employee)
  Destination SGT: 20 (Servers)
  Action: Permit IP
```

SGACLs are downloaded from ISE to NADs as part of environment data. Enforcement happens at any device in the fabric that can read the SGT.

### Environment Data

When a NAD is configured for TrustSec, it downloads:
- The list of SGT names and IDs
- The SGACL matrix
- Other TrustSec configuration

This download happens periodically. Without environment data, the NAD doesn't know what SGT 10 means.

### Inline tagging vs SXP

Two ways the source's SGT travels to the enforcement point:

| Method | How | Where it works |
|---|---|---|
| **Inline tagging** | SGT carried in the L2 header (CMD field) | Cisco fabric devices that support inline TrustSec |
| **SXP** | TCP protocol carries IP-to-SGT mappings between TrustSec-aware and non-TrustSec-aware devices | Anywhere — but requires SXP peering |

Modern Cisco campus fabrics use inline tagging by default. SXP fills the gap for legacy hardware.

---

## Common exam traps

1. **SGT tags ≠ VLAN tags.** Different concepts entirely. Don't confuse 802.1Q VLAN tags with SGT.

2. **An endpoint can have both a VLAN and an SGT.** Authz profile can return both; they're complementary.

3. **SGACL enforcement is at the destination, not the source.** A switch with the destination SGT 20 (Servers) is where the SGACL "Permit Employees → Servers" gets evaluated.

4. **SXP is unidirectional** (or bidirectional if both peers are configured). One peer is the *speaker* (sends mappings), the other the *listener* (receives). Configuration mistake here is common.

5. **TrustSec needs an environment data exchange to work.** Without environment data, NADs don't know SGT names. ISE acts as the source of TrustSec env data.

---

## Next

- [`lab.md`](lab.md), [`self-test.md`](self-test.md), [`notes.md`](notes.md)
