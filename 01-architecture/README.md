# 01 — Architecture and Deployment

> ~10% of the SISE exam. Foundation. You can't reason about ISE behavior without knowing what the nodes are and how they relate.

This section is mostly *vocabulary and topology* — fewer hands-on tests in the labs, more "memorize the right names and what they do."

---

## What the exam tests on this topic

You should be able to answer these in your sleep:

1. **Persona vs Node Role vs Service** — three different terms that engineers conflate. The exam tests that you don't.
2. **What each persona does** — PAN, MnT, PSN, pxGrid Controller — and which can co-exist on one node.
3. **Deployment models** — Standalone, Distributed, two-node, multi-node — and the limits of each.
4. **Failover behavior** — Primary/Secondary PAN sync, MnT log handling, PSN failover from a switch's perspective.
5. **Licensing tiers** — Essential vs Advantage vs Premier (Cisco renamed Base/Plus/Apex around ISE 3.0; know both).
6. **Sizing** — small/medium/large deployments by endpoint count.
7. **Certificates** — admin cert, EAP cert, portal cert, pxGrid cert; which one is presented when.
8. **Sync and replication** — what's replicated automatically (config) vs what isn't (live sessions).

---

## Concept map

```
ISE Cluster
│
├── PAN (Policy Administration Node) — write-only config interface
│    ├── Primary PAN  — only one at a time accepts admin writes
│    └── Secondary PAN — read-only standby; manual promotion to primary
│
├── MnT (Monitoring & Troubleshooting Node) — log/event store
│    ├── Primary MnT — actively receives logs from PSNs
│    └── Secondary MnT — passively receives logs in parallel
│
├── PSN (Policy Service Node) — handles RADIUS / TACACS+ / web portal traffic
│    ├── Many PSNs allowed — load-balanced or NAD-distributed
│    └── No primary/secondary — they're equal peers
│
└── pxGrid Controller — pub/sub bus for sharing context with SIEMs/firewalls
     └── Active/Standby pair recommended
```

A "node" is a physical/virtual ISE appliance. A "persona" is a *role* it plays. **One node can run multiple personas at once** in small deployments. Large deployments dedicate nodes to single personas.

### Common deployment models

| Model | What | When to use |
|---|---|---|
| **Standalone** | One node runs all personas (PAN+MnT+PSN+pxGrid) | Lab, evaluation, very small office |
| **Two-node Distributed** | Both nodes run all personas, one is Primary PAN/MnT, the other Secondary | Small production, basic HA |
| **Distributed** | Dedicated PAN pair + dedicated MnT pair + multiple PSNs | Mid to large enterprise |
| **Multi-Geo Distributed** | Distributed deployment across multiple sites | Global enterprise; pay attention to MnT placement and replication latency |

---

## Key concepts in plain English

### Persona vs Node Role vs Service

Three words for related things; the exam tests the precision.

- **Node** = a physical/virtual appliance (e.g. `ise-psn-01.example.com`).
- **Persona** = a role this node plays. Choices: PAN, MnT, PSN, pxGrid Controller. A node can play multiple at once.
- **Node Role** = within a persona, the *primary/secondary/standalone* designation. Only PAN and MnT have primary/secondary.
- **Service** = a feature that runs on PSNs (Session, Profiling, posture, etc.). Each can be enabled/disabled per PSN.

If someone says "make this node the primary," your first question should be: *primary what?* Primary PAN? Primary MnT? Those are different.

### What each persona actually does

| Persona | Job | Where its data lives |
|---|---|---|
| **PAN** | UI for admin writes (configs, policies, profiles). Replicates config out to all other nodes. | Primary PAN's local DB. |
| **MnT** | Stores all logs (RADIUS, accounting, posture, etc.) for reporting and Live Logs. | Primary MnT's local DB; Secondary mirrors via SNMP-like push from PSNs. |
| **PSN** | Handles all *runtime* traffic — RADIUS, TACACS+, web portals, profiling. The real workhorse. | Reads config from local replica, pushed from PAN; writes session data to MnT. |
| **pxGrid Controller** | Brokers pub/sub messages between ISE and external systems (SIEM, firewalls, NAC partners). | Stateless; just a message bus. |

### Replication and sync

- **Configuration** is owned by the Primary PAN. Replicated to all other nodes automatically. Read-only on others until you promote a Secondary PAN.
- **Live sessions** are owned by the PSN that authenticated the session. NOT replicated to other PSNs. (This is why a PSN failover for a session in flight forces the endpoint to reauth.)
- **Logs** are sent by PSNs simultaneously to *both* Primary and Secondary MnT. Both MnTs see all logs in near-real-time.

### Licensing — the names changed

Cisco renamed ISE licensing around release 3.0:

| Old name | New name | Roughly what it covers |
|---|---|---|
| Base | **Essentials** | Basic AAA, MAB, 802.1X, guest |
| Plus | **Advantage** | Profiling, BYOD, posture, pxGrid |
| Apex | **Premier** | TC-NAC, Threat Centric, advanced compliance |
| Device Admin | **Device Admin** (still its own SKU) | TACACS+ |

The exam may use either old or new names. Know both.

### Certificates and which one is presented when

Each persona uses a different cert for a different purpose. **This is heavily tested.**

| Cert role | Where it's presented | Common pitfall |
|---|---|---|
| **Admin** | When you load the ISE admin UI in a browser | If invalid, you can still log in but get warnings; users see warnings on portals if portal cert is missing |
| **EAP** | During EAP-TLS / PEAP / EAP-FAST handshakes | Must be trusted by the supplicant (the *client*); usually issued from your internal CA |
| **Portal** | When an endpoint hits a guest/BYOD/portal URL | Must be trusted by *unmanaged* devices too — often a public CA cert |
| **pxGrid** | When external systems connect to pxGrid | Mutual TLS; both sides need each other's cert in their trusted store |

Same node can present 3-4 different certs depending on which port the request hits.

---

## Common exam traps

1. **PSNs do not have a primary/secondary designation.** PAN and MnT do. PSNs are equal peers. Get this wrong and other questions cascade.

2. **Promoting a Secondary PAN is a manual step.** If the Primary PAN dies, ISE does NOT auto-promote. Admins are locked out of *writes* until you manually promote. Reads, RADIUS, portals all still work — just no admin changes.

3. **Live sessions don't replicate across PSNs.** If PSN-01 dies mid-session, that session is gone — the endpoint reauths against another PSN. (Switches handle this seamlessly via dead-criteria; users may see a brief blip.)

4. **MnT replication is push from PSNs, not sync between MnTs.** Both MnTs receive logs in parallel. If one MnT goes down, the other has the same logs. They don't sync between themselves.

5. **A standalone node can be promoted to a node in a distributed deployment** — but going *from* distributed back to standalone requires a deregister step. Not just a config change.

6. **Sizing is by *concurrent active sessions*, not endpoints.** A PSN sized for 25,000 active sessions can serve a deployment with 100,000 *registered* endpoints (since not all are concurrently authenticated). Verify against current Cisco docs for the specific session counts per VM size.

7. **The PAN is required for admin writes, but not for runtime traffic.** A Primary-PAN-down event is an *operational* problem (no policy changes), not an *availability* problem (RADIUS still flows through PSNs).

8. **pxGrid Controller is its own persona — it's not the same as the pxGrid service.** Subtle distinction. The Controller is the message bus; the service is what publishes/subscribes.

---

## Next

- [`lab.md`](lab.md) — hands-on procedure (still scaffolded; fill in as you study)
- [`self-test.md`](self-test.md) — practice questions
- [`notes.md`](notes.md) — your space
