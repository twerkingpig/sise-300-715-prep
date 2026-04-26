# 01 — Architecture and Deployment — Self-Test

12 questions in the shape SISE 300-715 asks. Answers at the bottom — don't peek until done.

---

### Q1
Which of the following is NOT a valid ISE persona?

A. PAN — Policy Administration Node
B. MnT — Monitoring and Troubleshooting Node
C. PSN — Policy Service Node
D. PEN — Policy Enforcement Node

### Q2
The Primary PAN dies. RADIUS authentication for endpoints continues to flow normally. What is the operational impact?

A. RADIUS stops because PSNs require Primary PAN approval for each request
B. Logging stops because PSNs send logs to PAN
C. Configuration changes (admin writes) are blocked until a Secondary PAN is manually promoted
D. The cluster automatically promotes the Secondary PAN within 60 seconds

### Q3
A small office runs ISE on a single appliance. Which personas does that node run?

A. Only PSN
B. PAN, MnT, PSN, pxGrid (all-in-one)
C. PAN and PSN only — MnT must be on a separate node
D. It depends on the licensing tier

### Q4
You have two PSNs. PSN-A authenticates an endpoint and the session is active. PSN-A then becomes unreachable. What happens to the active session?

A. The session is replicated to PSN-B and continues seamlessly
B. The session is lost; on reauth the endpoint authenticates against PSN-B (or whichever PSN is alive)
C. The endpoint is disconnected and must wait until PSN-A returns
D. PSN-B serves a cached copy of the session for 24 hours

### Q5
Which describes the relationship between Primary MnT and Secondary MnT?

A. Primary handles writes; Secondary is read-only and syncs from Primary
B. Both receive logs in parallel from PSNs
C. Secondary is only used after Primary fails over
D. Secondary acts as a backup that runs nightly

### Q6
Cisco renamed ISE license tiers around release 3.0. Which is the correct mapping?

A. Base → Essentials, Plus → Advantage, Apex → Premier
B. Base → Standard, Plus → Pro, Apex → Enterprise
C. Base → Foundation, Plus → Professional, Apex → Premium
D. Base → Core, Plus → Plus+, Apex → Apex+

### Q7
A user opens a browser to the BYOD portal at `https://psn-01.lab.example.com:8443/portal/`. Which certificate does ISE present to the browser?

A. The Admin certificate
B. The EAP certificate
C. The Portal certificate
D. The pxGrid certificate

### Q8
Which best describes how configuration changes are propagated in a distributed ISE deployment?

A. Each node manages its own config; admins must repeat changes per node
B. The Primary PAN owns the config DB and replicates changes to all other nodes
C. Secondary PAN performs writes; Primary PAN is read-only
D. PSN nodes broadcast config changes to all other PSNs via pxGrid

### Q9
You deploy a new ISE node and want it to handle TACACS+ for switch admin access. The node must run which service?

A. Profiling Service on PSN persona
B. Device Administration Service on PSN persona (with the appropriate license)
C. pxGrid Service on pxGrid persona
D. Posture Service on MnT persona

### Q10
A "node" and a "persona" are not the same thing. Which statement is correct?

A. A node is the physical/virtual appliance; a persona is a role the node plays (and one node can play multiple personas)
B. A persona is the appliance; a node is the role it plays
C. They mean the same thing in ISE; the distinction is academic
D. A persona is one of: Primary, Secondary, Standalone

### Q11
You've configured an ISE cluster with two nodes for High Availability. The Primary PAN/MnT goes offline. Which of these are TRUE? (Pick the most complete answer.)

A. RADIUS continues to work; admin writes are blocked; logs continue to flow to the surviving Secondary MnT
B. Everything stops until the Primary returns
C. The Secondary auto-promotes and admin writes resume within 60 seconds
D. Only RADIUS continues; logs and admin both stop

### Q12
pxGrid is the integration bus that lets ISE share session context with external systems (SIEM, firewalls). Which is true about pxGrid Controller as a persona?

A. It can only run on a dedicated node — never combined with PSN
B. It is the message broker; nodes running the pxGrid persona handle pub/sub between ISE and partners
C. It replaces the PSN persona for context sharing
D. It is required for ISE to function (cannot be disabled)

---

## Answers

<details>
<summary>Click to reveal</summary>

**Q1: D.** "PEN" is not an ISE persona. The four valid personas are PAN, MnT, PSN, pxGrid Controller.

**Q2: C.** RADIUS continues via PSNs. Logging continues to the Secondary MnT (PSNs send logs to *both* MnTs in parallel). What stops is admin writes — until a Secondary PAN is *manually* promoted to Primary.

**Q3: B.** A standalone deployment runs all four personas (PAN, MnT, PSN, pxGrid Controller) on the single node. Lab and small-office model.

**Q4: B.** Sessions don't replicate across PSNs. The session is lost; on reauth the endpoint authenticates against another live PSN. Switch dead-criteria handles failover so the user usually doesn't notice beyond a brief reauth.

**Q5: B.** PSNs send each log message to both MnTs in parallel. The MnTs do NOT sync between themselves; they each independently receive copies.

**Q6: A.** Base → Essentials, Plus → Advantage, Apex → Premier. The exam may use either set of names.

**Q7: C.** Portal cert. The same node may present three different certs depending on which port/path is hit: Admin (admin UI), EAP (RADIUS auth), Portal (web portals). Often these are different certs, sometimes from different CAs.

**Q8: B.** Primary PAN is the writable config interface. It replicates out to all other nodes. Other nodes are config-read-only.

**Q9: B.** TACACS+ is provided by the **Device Administration** service running on PSN persona, gated by a separate Device Admin license. It's not its own persona.

**Q10: A.** Node = appliance; persona = role. One node can play multiple personas. This precision matters for the exam.

**Q11: A.** Most complete: RADIUS continues, writes block, logs flow to Secondary MnT. The other answers either overstate impact (B, D) or invent automatic failover that ISE doesn't do (C).

**Q12: B.** pxGrid Controller is the message broker persona. It can be combined with other personas on a node. It's not required for ISE to function — many deployments don't enable it.

</details>

---

## Scoring

- 11–12: solid grasp; move on.
- 8–10: re-read [`README.md`](README.md), focus on what you missed.
- 4–7: needs another study pass; don't move on.
- 0–3: come back to this topic next week.
