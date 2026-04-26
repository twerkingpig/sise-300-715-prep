# 09 — Integrations

> Tested across the blueprint. ISE talks to a lot of other systems: AD, certificate authorities, SIEMs, firewalls, threat intel feeds. This section is the catalog.

Most of this material was touched in earlier sections. This one is reinforcement and the few corners not covered elsewhere.

---

## What the exam tests on this topic

1. **pxGrid** — the publish/subscribe bus for sharing context with external systems.
2. **Active Directory join points** — how ISE integrates with AD forests.
3. **Certificate authority chaining** — how ISE trusts external CAs, how it issues certs.
4. **MnT to SIEM** — sending logs from ISE to Splunk / QRadar / etc.
5. **Threat-Centric NAC (TC-NAC)** — feeding threat intel into authz decisions.
6. **REST API** — ERS (External RESTful Services) for programmatic ISE management.

---

## Key concepts in plain English

### pxGrid

ISE's pub/sub bus. Lets ISE share session context (who's authenticated, what device, what attributes) with **external systems** that subscribe. Common subscribers:

| Subscriber | What they do with the data |
|---|---|
| **SIEM** (Splunk, QRadar) | Correlate ISE events with other security events |
| **Firewall** (Cisco Firepower, Palo Alto, etc.) | Apply user-aware firewall rules — "deny Contractors from reaching Finance" |
| **NAC partners** (Tenable, etc.) | Trigger ISE actions (CoA-Disconnect) when a vulnerability is detected |
| **Threat intel feeds** | Feed indicators back to ISE for TC-NAC |

pxGrid uses **mutual TLS**. Both ends need each other's certs in their trust stores.

### Active Directory join points

ISE joins one or more AD forests. Each AD integration is a *Join Point*. Join Points appear in identity source sequences and as condition prefixes (`AD:ExternalGroups EQUALS ...`).

Important behaviors:

- **One ISE deployment can join multiple AD forests.** Useful for orgs with M&A history.
- **Each PSN must be joined separately** — joining is per-node, not cluster-wide.
- **AD connectivity is monitored** — `Administration → External Identity Sources → Active Directory → (join point) → Connection`. Status `Operational` means the join is healthy.

When AD is unhealthy, identity source sequences fall through (if "If process failed → Continue" is set), as covered in Section 02.

### Certificate authority chaining

ISE trusts external CAs by importing their root and intermediate certs into the **Trusted Certificates** store. ISE issues internal certs from its own internal CA (or proxies enrollment to your enterprise CA via SCEP/EST as covered in Section 05).

Three cert stores you should know:

| Store | Purpose |
|---|---|
| **Trusted Certificates** | Roots and intermediates of CAs ISE trusts — used for validating client certs in EAP-TLS |
| **System Certificates** | Certs ISE itself presents (admin cert, EAP cert, portal cert, pxGrid cert) |
| **Endpoint Certificates** | Certs issued to endpoints by ISE's internal CA via BYOD/posture |

### MnT-to-SIEM

ISE's MnT can stream logs to external syslog/SIEM systems. Configured under **Administration → System → Logging → Remote Logging Targets**. Two common formats:

- **Syslog (RFC 5424 / 3164)** — most SIEMs accept this
- **Cisco-specific formats** — preserve more attributes; require SIEM-side parsing

Big deployments often centralize this — e.g., MnT logs go to Splunk, where they're correlated with firewall logs, AD logs, etc.

### Threat-Centric NAC (TC-NAC)

ISE can ingest **vulnerability / threat data** from external partners (Tenable, Qualys, Cisco AMP, etc.) and use that data in authz decisions. Example:

- Tenable identifies endpoint X as having a critical CVE
- ISE TC-NAC ingests this via pxGrid
- ISE authz rule: `Threat:CVSS-Score GREATER_THAN 8.0 → Quarantine VLAN`
- ISE fires CoA-Reauth → endpoint is quarantined

Requires Premier license and enabled TC-NAC service.

### ERS — External RESTful Services

ISE's REST API for programmatic management. Use cases:
- Add/delete endpoints from Internal Endpoints store
- Manage guest accounts
- Pull report data

Enabled at **Administration → System → Settings → ERS Settings**. Authentication via dedicated ERS Admin user.

---

## Common exam traps

1. **pxGrid uses mutual TLS.** Both pxGrid Controller and the partner system need each other's certs trusted. Classic deployment failure: one side trusts the other but not vice versa.

2. **AD join is per-node, not per-cluster.** When you add a new PSN, you must join it to AD separately. Adding nodes without joining them to AD is a common mistake during scale-out.

3. **Trusted Certificates ≠ System Certificates.** The first is "CAs we trust." The second is "certs ISE itself presents." Don't confuse.

4. **TC-NAC requires Premier license.** Standard Advantage doesn't include it.

5. **ERS is disabled by default** in newer ISE versions for security reasons. Forget to enable it and your automation script silently fails.

6. **pxGrid Controller persona vs pxGrid service** — Controller is the persona/role; the service is what runs on it. Subtle distinction; the exam tests it.

---

## Next

- [`lab.md`](lab.md), [`self-test.md`](self-test.md), [`notes.md`](notes.md)
