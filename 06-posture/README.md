# 06 — Endpoint Compliance (Posture)

> ~10% of the SISE exam. Posture asks: "is this endpoint compliant with our security policy *right now* — current OS, AV running, disk encrypted?" If yes, give it full access. If not, redirect to remediation or quarantine.

This is the most operationally complex topic in the exam — many moving parts that all have to align (AnyConnect agent, ISE Posture service, Client Provisioning policies, Posture Conditions/Requirements/Policies). The mechanics matter; the failure modes are subtle.

---

## What the exam tests on this topic

1. **The three posture states** — Compliant, Non-Compliant, Unknown — and what each triggers.
2. **AnyConnect ISE Posture module** — what it is, how it gets installed, how it reports back to ISE.
3. **Client Provisioning policies** — how ISE decides which version of the agent to push.
4. **Posture Conditions, Requirements, and Policies** — three layers of construct, easy to confuse.
5. **The redirect-then-posture flow** — how an endpoint enters posture state, gets remediated, and becomes Compliant.
6. **Periodic Reassessment (PRA)** — re-checking compliance on already-compliant endpoints.
7. **Stealth Mode** — invisible posture vs notification mode.
8. **Remediation actions** — automatic (run a script, install software) vs manual (show user instructions).

---

## Concept map

```
PostureStatus States
─────────────────────
Unknown ─────► (agent not installed, or hasn't reported yet)
Compliant ───► (agent reported back: passes all requirements)
NonCompliant ─► (agent reported back: failed at least one requirement)

Flow when an endpoint connects:
─────────────────────────────────
1. Authenticate (AuthN passes)
2. Authorize: ISE checks Session:PostureStatus
3a. If Unknown → return URL redirect to Client Provisioning Portal
                 + dACL allowing only ISE / remediation servers
3b. If Compliant → return full-access authz profile
3c. If NonCompliant → return remediation/quarantine authz profile

After agent installs and reports:
─────────────────────────────────
Agent runs the assigned posture policy:
  - Check requirements (AV running, OS version, registry key, etc.)
  - Each check returns Pass or Fail
  - Overall: all pass = Compliant; any fail = NonCompliant
Agent reports result to ISE → ISE updates Session:PostureStatus
ISE fires CoA-Reauth → endpoint re-evaluated → new authz applied
```

---

## Key concepts in plain English

### The three posture states

| State | Meaning | What happens |
|---|---|---|
| **Unknown** | ISE doesn't know yet — agent hasn't reported (or isn't installed) | Redirect to Client Provisioning Portal to install/run agent |
| **Compliant** | Agent reported back: all requirements met | Full access granted |
| **NonCompliant** | Agent reported back: at least one requirement failed | Remediation or quarantine |

These are values of `Session:PostureStatus` in authz rule conditions.

### AnyConnect ISE Posture module

The agent that runs *on the endpoint* and performs posture checks. Three relevant facts:

1. **It's a module of AnyConnect** — not a standalone app. Other AnyConnect modules: VPN, Network Access Manager, Web Security.
2. **It's pushed via Client Provisioning** during the redirect state. ISE has a copy of the AnyConnect installer in its Client Provisioning resources.
3. **It reports compliance results back to ISE** via HTTPS to a PSN, which updates the session's PostureStatus.

There's also **Cisco Secure Client** (the rebranded AnyConnect for newer deployments). Same idea; different name. The exam may use either.

### The three-layer construct: Conditions, Requirements, Policies

Easy to confuse — they're nested.

| Layer | Example | What it does |
|---|---|---|
| **Condition** | "ClamAV process is running" | A single check (file exists, registry key, service running, etc.) |
| **Requirement** | "Endpoint has AV running" | Bundles 1+ Conditions, plus a Remediation Action if failed |
| **Policy** | "Windows 10 endpoints in Corp identity group must meet AV-Running and OS-Updated requirements" | Maps Requirements to a target endpoint set (OS, identity group, etc.) |

A Policy points at Requirements; Requirements point at Conditions. Authz rule matches `PostureStatus` to determine what access to grant.

### Client Provisioning policies

Distinct from posture *policies* (yes, confusingly named). Client Provisioning policies decide:
- Which AnyConnect / Secure Client version to push to a given endpoint
- Which compliance module to push
- What posture configuration to attach

Match conditions are typically: OS, identity group, role. ISE selects the first matching Client Provisioning policy and pushes those resources during the redirect-then-install flow.

### Periodic Reassessment (PRA)

After an endpoint is Compliant, posture isn't a one-shot check. PRA reassesses periodically — every X minutes — and updates compliance status. If something changes (AV gets disabled, OS update reverts), PRA catches it and the endpoint may be moved to NonCompliant.

Configurable interval. Default usually around 1-4 hours. Don't make it too short — that's a load multiplier.

### Stealth Mode

Two ways the agent can run:

| Mode | What the user sees | When to use |
|---|---|---|
| **Notification** | Pop-ups during scan, results visible | Default; helps users self-remediate |
| **Stealth** | Nothing — silent scan | Locked-down environments where users shouldn't be aware |

Stealth is more rigid; users can't manually trigger remediation, so failure typically leads to admin intervention.

### Remediation actions

When a Requirement fails, ISE can apply a Remediation Action:
- **Automatic**: run a script, install software, restart a service
- **Manual**: show the user instructions, link to internal IT page
- **Mandatory vs Optional**: optional remediations let the user click through; mandatory blocks until fixed

---

## Common exam traps

1. **Posture state is per-session, not persistent.** A new RADIUS session starts as `Unknown` until the agent reports. A session that was Compliant yesterday is Unknown again on a fresh connect — until PRA or the next agent report happens.

2. **The agent must be able to reach the PSN over HTTPS.** Network blocks (firewall, dACL) between endpoint and PSN port 8443/8905 will cause posture to silently stick at Unknown.

3. **Posture redirect uses the same three RADIUS attributes** as everything else: url-redirect-acl, url-redirect URL, dACL. Same gotchas.

4. **Client Provisioning policy ≠ Posture policy.** Client Provisioning decides *what to install*. Posture decides *what to check*. Both are configured but don't confuse them.

5. **The dACL during posture redirect must allow client-to-PSN AND client-to-remediation-server**. If your remediation involves downloading an installer from an internal server, that server's IP must also be in the allowed list.

6. **AnyConnect / Secure Client naming change.** Cisco rebranded AnyConnect to Cisco Secure Client. ISE 3.x supports both names. Exam questions may use either.

7. **PRA does NOT replace the initial posture check.** PRA is for already-compliant sessions. Initial check still goes through redirect → install → scan → report.

8. **Stealth mode endpoints can't self-remediate.** If a stealth-mode posture check fails, the user has no idea why their access was downgraded. Not always a bad thing, but it shifts support load to admins.

---

## Next

- [`lab.md`](lab.md) — hands-on procedure (still scaffolded; fill in as you study)
- [`self-test.md`](self-test.md) — practice questions
- [`notes.md`](notes.md) — your space
