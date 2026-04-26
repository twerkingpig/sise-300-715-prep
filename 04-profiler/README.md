# 04 — Profiler

> ~15% of the SISE exam. Profiling is *how ISE figures out what an endpoint actually is* without trusting the endpoint to tell the truth.

This is one of the most operationally interesting topics. Authz rules constantly use `EndPoints:LogicalProfile` conditions, so profiler accuracy directly drives policy correctness.

---

## What the exam tests on this topic

1. **The full list of Probes** — RADIUS, DHCP, HTTP, SNMP (Query and Trap), NMAP, NetFlow, AD, DNS — what each one collects, what it requires.
2. **Profile Policies** — built-in vs custom, how they evaluate, certainty factor.
3. **Logical Profiles** — grouping multiple specific profiles into a single tag for use in authz.
4. **Endpoint Identity Groups** — Profiled (auto), Static (manual), system groups.
5. **Anomalous Endpoint Detection** — how ISE catches MAC spoofing.
6. **The Feed Service** — Cisco's regularly-updated profile signatures.
7. **Profile certainty and Minimum Certainty Factor** — how rules score endpoints.
8. **Where profiling integrates with policy** — using `EndPoints:LogicalProfile` and `EndPoints:IdentityGroup` in authz rules.

---

## Concept map

```
PROBES                          PROFILE POLICIES               OUTPUT
------                          ----------------               ------
RADIUS  ─┐                                                     ┌── EndPoint Profile (specific)
DHCP    ─┤                                                     │   e.g. "Cisco-IP-Phone-7945"
HTTP    ─┤                                                     │
SNMP    ─┼──► Endpoint attributes ──► Profile Policy match ──► ├── Logical Profile (grouping)
NMAP    ─┤   (DHCP fingerprint,       (rules + conditions       │   e.g. "Cisco-IP-Phone"
NetFlow ─┤    HTTP User-Agent,         + certainty factor)       │
AD      ─┤    OUI vendor, etc.)                                  └── Endpoint Identity Group
DNS     ─┘                                                          (Profiled vs Unknown)
                │
                ▼
        Used in authz rule conditions:
        EndPoints:LogicalProfile EQUALS Cisco-IP-Phone
        EndPoints:IdentityGroup EQUALS Workstation
```

---

## Key concepts in plain English

### Probes — what each one does

| Probe | What it collects | Where it gets it | Cost / Risk |
|---|---|---|---|
| **RADIUS** | MAC, basic NAD-supplied data, EAP method | RADIUS Access-Request from the switch | Free — already happening for auth |
| **DHCP** | DHCP fingerprint (vendor class, host name, options) | DHCP relay or `ip helper-address ise` | Highest-quality probe; cheap; do this first |
| **HTTP** | User-Agent header, protocol version | URL-redirect captures during portal flows | Only fires when an endpoint is redirected |
| **SNMP Query** | sysDescr, ifTable, ARP table | ISE polls switches via SNMP | Polling load on the switch; needs read-only community |
| **SNMP Trap** | linkUp/linkDown, MAC notification traps | Switch sends to ISE | Low overhead; needs trap config on switch |
| **NMAP** | Open ports, service banners | Active scan from PSN to endpoint | Loud and slow; only triggered by some profile policies |
| **NetFlow** | Traffic patterns, top talkers | NetFlow exporter to ISE | High data volume; limited use case |
| **AD** | OS info, group membership, computer object attributes | LDAP query against AD | Requires AD join; slow if AD unhealthy |
| **DNS** | Reverse DNS lookup of endpoint IP | DNS query | Cheap; some networks don't have reverse DNS for endpoints |

**Don't enable all probes by default.** Enable DHCP and RADIUS first; add others only when you need them. NMAP especially can flood the network if used carelessly.

### Profile Policies — how endpoints get classified

A Profile Policy is a set of conditions plus a *certainty factor* (a numeric score). When an endpoint's attributes match conditions in a policy, the endpoint accumulates certainty toward that profile.

Example (simplified):
```
Profile Policy: Cisco-IP-Phone-7945
  Condition: DHCP:dhcp-class-identifier CONTAINS "Cisco Systems, Inc. IP Phone CP-7945"
  Increase Certainty Factor by: 30

  Condition: SNMP:sysDescr CONTAINS "Cisco IP Phone 7945"
  Increase Certainty Factor by: 30

  Minimum Certainty Factor: 30
```

If the endpoint's DHCP fingerprint matches → certainty 30 → meets minimum → endpoint is profiled as `Cisco-IP-Phone-7945`. If SNMP later confirms → certainty 60. The **highest-certainty profile that meets minimum** is the assigned profile.

### Logical Profiles — for authz rule simplicity

Instead of writing 30 authz rules for every variant of "Cisco IP Phone", you create a Logical Profile called `Cisco-IP-Phone` that includes:
- Cisco-IP-Phone-7945
- Cisco-IP-Phone-7960
- Cisco-IP-Phone-8841
- ...etc

Then your authz rule says `EndPoints:LogicalProfile EQUALS Cisco-IP-Phone` — clean.

### Endpoint Identity Groups (vs Logical Profiles, easy to confuse)

| | Endpoint Identity Group | Logical Profile |
|---|---|---|
| **What it is** | A folder/container for endpoints | A tag derived from profile match |
| **Assignment** | Manual (Static) or automatic (Profiled) | Always automatic, follows profile |
| **Common use** | Static device classification (Printers, IoT) | Authz conditions across profile families |

A common pattern: `EndPoints:IdentityGroup EQUALS Workstation AND EndPoints:LogicalProfile EQUALS Windows-Workstation`. Belt-and-suspenders.

### The Feed Service

Cisco maintains a set of profile policies for thousands of device models. ISE downloads this Feed periodically. **Enable it.** Hand-writing profiles for every camera model, every printer, every IoT thermostat, etc. is impossible. The Feed handles ~80% of identification automatically.

Configure under **Administration → System → Profiling → Feed**. Requires internet access from PSNs.

### Anomalous Endpoint Detection — the MAC-spoofing defense

If a MAC was profiled as `Axis-Camera` last week and suddenly starts sending Windows DHCP options + browsing HTTPS — that's anomalous. ISE flags this and can fire CoA to quarantine.

Configured under **Administration → System → Settings → Profiling → Anomalous Endpoint Detection**. It's not on by default; turn it on for production.

This is the answer to the "what stops MAC spoofing on a MAB endpoint" question.

---

## Common exam traps

1. **DHCP probe needs traffic to actually reach ISE.** Either configure `ip helper-address <PSN>` on the SVI (so DHCP requests get a copy) or run an ISE Profiler service node *inline*. Forgetting this is the #1 reason DHCP probe shows nothing.

2. **HTTP probe only fires during URL-redirect flows.** It's not constantly running. So endpoints that never trigger a portal won't have HTTP attributes.

3. **NMAP probe is opt-in per profile policy.** It doesn't run unless a profile policy explicitly invokes it. Don't expect NMAP attributes on endpoints that haven't been NMAP-scanned.

4. **Profiles inherit conditions from parent profiles.** `Cisco-Device` is a parent; `Cisco-IP-Phone` is its child. Conditions that match the child also match the parent — that's why a 7945 phone is also tagged as `Cisco-Device` and `Cisco-IP-Phone`.

5. **Endpoint Identity Group "Profiled"** is the catch-all for any endpoint successfully profiled. Endpoints that didn't match any profile go into "Unknown" group. Distinguish these.

6. **A static Endpoint Identity Group assignment overrides profiling.** If you manually put a MAC in `Workstation`, it stays there regardless of what profiler thinks — until you remove it.

7. **The certainty factor is a score, not a percentage.** Maximum certainty isn't 100 — it's the sum of all matched condition weights. Some profile policies have minimum certainty 10, others 50. Read each one.

8. **Anomalous Endpoint Detection requires a baseline period.** ISE needs to have profiled the endpoint *before* it can detect that the endpoint changed. A brand-new MAC doesn't get flagged as anomalous on first connect — it just gets profiled.

---

## Next

- [`lab.md`](lab.md) — hands-on procedure (still scaffolded; fill in as you study)
- [`self-test.md`](self-test.md) — practice questions
- [`notes.md`](notes.md) — your space
