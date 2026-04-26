# 09 — Integrations — Self-Test

8 questions. Answers at the bottom.

---

### Q1
What is pxGrid in ISE?

A. A guest portal customization framework
B. A publish/subscribe bus for sharing session context with external systems (SIEM, firewalls, NAC partners)
C. A licensing tier
D. A backup/restore mechanism

### Q2
pxGrid uses what authentication mechanism between ISE and partner systems?

A. Username/password over plain HTTP
B. Mutual TLS — both ISE and partner need each other's certs trusted
C. RADIUS shared secret
D. SNMPv3 user/password

### Q3
You have a distributed ISE deployment with 3 PSNs. You add a new PSN node and want it to authenticate users against AD. What's the additional step required?

A. Nothing — joining one PSN joins them all
B. Each PSN must be joined to the AD join point individually
C. AD join is automatic when the new node is registered with the cluster
D. AD join is configured at the PAN level only

### Q4
What's the difference between ISE's Trusted Certificates store and System Certificates store?

A. They're the same store
B. Trusted Certificates store holds CA roots/intermediates that ISE trusts; System Certificates store holds certs ISE itself presents (admin, EAP, portal, pxGrid)
C. Trusted is for clients; System is for sponsors
D. System is for AD; Trusted is for everything else

### Q5
ISE sends authentication logs to a Splunk SIEM for correlation. Where in ISE is this configured?

A. Administration → External Identity Sources
B. Administration → System → Logging → Remote Logging Targets
C. Operations → RADIUS → Live Logs
D. Work Centers → Device Administration

### Q6
What does Threat-Centric NAC (TC-NAC) do?

A. Replaces RADIUS with a threat-aware protocol
B. Ingests vulnerability/threat data from external partners (Tenable, Qualys, etc.) and lets ISE authz rules use that data — e.g., quarantine endpoints with critical CVEs
C. Performs penetration testing on endpoints
D. Generates threat reports for compliance

### Q7
Which is required to use ISE's REST API (ERS)?

A. ERS is always enabled by default
B. Enable ERS under Administration → System → Settings → ERS Settings, and authenticate as an ERS Admin user
C. Premier license only
D. ISE only supports REST API on the PAN

### Q8
A pxGrid integration with a Cisco Firepower firewall fails. ISE shows the partner cert as untrusted. What's the most direct fix?

A. Reinstall ISE
B. Import the Firepower's pxGrid cert into ISE's Trusted Certificates store, AND import ISE's pxGrid cert into Firepower's trust store (mutual TLS requires both)
C. Disable TLS on pxGrid
D. Restart the PSN

---

## Answers

<details>
<summary>Click to reveal</summary>

**Q1: B.** pxGrid is a pub/sub bus. ISE publishes context (sessions, attributes); subscribers consume and act on it. Common subscribers: SIEMs, firewalls, NAC partners.

**Q2: B.** Mutual TLS. Both sides need each other's certs trusted. Asymmetric trust = broken pxGrid.

**Q3: B.** AD join is per-node. Each PSN that needs AD lookups must be joined individually. Missing this on a scale-out causes the new PSN to silently fail AD-based authz.

**Q4: B.** Trusted = "CAs we accept" (used to validate incoming certs). System = "certs ISE itself presents" (admin/EAP/portal/pxGrid). Different stores, different purposes.

**Q5: B.** Administration → System → Logging → Remote Logging Targets. ISE pushes logs to defined targets (syslog over UDP, TCP, or TLS).

**Q6: B.** TC-NAC ingests threat/vuln data from partners and exposes it in authz rule conditions. Lets ISE quarantine vulnerable endpoints automatically. Requires Premier license.

**Q7: B.** ERS is OFF by default in modern ISE. Enable explicitly + authenticate via dedicated ERS Admin role.

**Q8: B.** Mutual TLS requires both directions of trust. Importing only ISE's cert into Firepower (or only Firepower's into ISE) is half the work and pxGrid won't establish.

</details>

---

## Scoring

- 7–8: solid; move on.
- 5–6: re-read [`README.md`](README.md), focus on what you missed.
- 0–4: needs another pass.
