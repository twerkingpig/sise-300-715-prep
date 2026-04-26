# 02 — Policy Sets

> ~25% of the SISE exam. The most important topic in the blueprint. If you only deeply learn one section, learn this one.

This is where ISE makes its decisions: *should this endpoint be authenticated, and if so, what authorization should it get?* Understanding the **policy evaluation flow** cold is non-negotiable for the exam.

---

## What the exam tests on this topic

You should be able to answer these in your sleep:

1. **The order of evaluation** — what does ISE evaluate first when a RADIUS request arrives?
2. **What a policy set is** — and why you'd have more than one.
3. **The difference between Authentication and Authorization** — these are *different* policies, evaluated separately, and they fail differently.
4. **What happens when a rule matches** vs. doesn't match — including the role of the Default rule.
5. **Conditions you can use** — `Radius:`, `Network Access:`, `EndPoints:`, `AD:`, `Certificate:`, `InternalUser:`, etc., and what each means.
6. **Identity Source Sequences** — what they are, why use them, "if user not found" vs "if process failed."
7. **Authorization Profiles vs. Authorization Rules** — what's the difference and how do they relate.
8. **Compound conditions** — AND vs. OR, library vs. custom, why compound.

If any of those make you hesitate, that's where to study.

---

## Concept map

```
Endpoint plugs in
        │
        ▼
Switch sends RADIUS Access-Request to ISE
        │
        ▼
┌─────────────────────────────────────────┐
│  Policy Set (top-level container)       │
│   - Conditions: which set to use        │
│      (e.g. Wired_802.1X, Wireless_MAB)  │
│                                         │
│   - Allowed Protocols                   │
│      (which EAP types are accepted)     │
│                                         │
│   ┌─────────────────────────────────┐   │
│   │  Authentication Policy          │   │
│   │   - Identity Source Sequence    │   │
│   │   - Auth fail / user-not-found  │   │
│   │     / process-fail behaviors    │   │
│   └─────────────────────────────────┘   │
│                                         │
│   ┌─────────────────────────────────┐   │
│   │  Authorization Policy           │   │
│   │   - Top-down rule evaluation    │   │
│   │   - First match wins            │   │
│   │   - Result = Authorization      │   │
│   │     Profile (VLAN, dACL, SGT,   │   │
│   │     URL redirect, etc.)         │   │
│   └─────────────────────────────────┘   │
└─────────────────────────────────────────┘
        │
        ▼
ISE returns Access-Accept (with RADIUS attributes from authz profile)
or Access-Reject (DenyAccess)
```

---

## Key concepts in plain English

### Policy Sets

A **policy set** is a top-level container that holds:
- A **condition** that decides whether incoming RADIUS hits this set (e.g. *Device Type EQUALS Access-Switch*).
- An **Allowed Protocols** definition (which EAP methods/PAP/CHAP are allowed).
- One **Authentication Policy** with one or more rules.
- One **Authorization Policy** with one or more rules.

Policy sets are evaluated **top-to-bottom**. The first set whose condition matches is the one that handles the request. Default Set always matches at the bottom.

### Authentication vs Authorization (don't confuse these)

| | Authentication | Authorization |
|---|---|---|
| **Question it answers** | "Who is this endpoint?" | "What can this endpoint do?" |
| **Inputs** | Credentials (cert, password, MAC) | Identity (the result of authn) plus context (group, profile, time, location) |
| **Identity source** | Internal Users, AD, LDAP, Internal Endpoints, Certificate Store | Already known by this point |
| **Result** | Pass / Fail | Authorization Profile (e.g. EMPLOYEE-FULL-ACCESS) or DenyAccess |

You can **authenticate successfully and still get authorization=DenyAccess.** That's a common exam trap. "Auth passed but Default authz hit DenyAccess" is a real outcome.

### Identity Source Sequences (ISS)

An **Identity Source Sequence** is a list of identity stores ISE will try in order. Two important advanced settings:

| Setting | Meaning |
|---|---|
| **If user not found** | "Reject" or "Continue to next store" |
| **If process failed** | "Reject" or "Continue to next store" |

The second matters for **AD-down resilience.** If your ISS is `[AD, Internal Users]` and `If process failed = Continue`, then when AD's connector breaks, ISE falls through to Internal Users instead of rejecting.

### Authorization Profiles

An **Authorization Profile** is a *named bundle of RADIUS attributes* that ISE returns to the switch on success. Common contents:

- **Access Type** — `ACCESS_ACCEPT` or `ACCESS_REJECT`
- **VLAN** — `[T] Tagged or [U] Untagged, VLAN ID or Name`
- **dACL** — a downloadable ACL ISE pushes
- **Reauth Timer** — how long until ISE wants the switch to reauth
- **URL Redirect** + **URL Redirect ACL** — for portal flows
- **Web Authentication** — type (Centralized / Local / Native)
- **Voice Domain Permission** — sets `device-traffic-class=voice`
- **SGT** — TrustSec Security Group Tag
- **Common Tasks** — friendly UI for the above

Authorization Profiles are *referenced* by Authorization Rules. One profile, many rules.

### Conditions (the building blocks)

| Condition Family | Common Examples | Used For |
|---|---|---|
| `Radius:` | `Service-Type EQUALS Framed`, `NAS-Port-Type EQUALS Ethernet`, `Calling-Station-ID MATCHES <regex>` | Distinguishing 802.1X (Framed) from MAB (Call Check), wired from wireless |
| `Network Access:` | `AuthenticationStatus EQUALS AuthenticationPassed`, `EapAuthentication EQUALS EAP-TLS` | What happened during authn |
| `EndPoints:` | `LogicalProfile EQUALS Cisco-IP-Phone`, `BYODRegistration EQUALS Yes`, `IdentityGroup EQUALS Workstation` | Profiling outcomes |
| `AD:` | `ExternalGroups EQUALS Domain Computers`, `<attribute> EQUALS <value>` | Group memberships and AD attributes |
| `Certificate:` | `Issuer CN EQUALS Internal-CA`, `Template Name EQUALS Workstation-Auth` | Cert-based identity |
| `InternalUser:` | `Name EQUALS ise-probe`, `Identity Group EQUALS Employees` | Local user accounts |
| `DEVICE:` | `Device Type EQUALS Access-Switch`, `Location EQUALS Campus-Building-A` | NAD attributes — used in policy set conditions |
| `Session:` | `PostureStatus EQUALS Compliant`, `AgentNotInstalled EQUALS True` | Posture results |

Mix and match with **AND** / **OR** to build *compound* conditions. Compound conditions are how you avoid the "anyone with a cert from our internal CA gets in" trap.

### The Default rule

Every Authorization Policy ends with a **Default** rule. This is your fail-closed safety net. Make it `DenyAccess` in production. If the Default ends up matching for an endpoint that *should* have matched something else, that's a misconfiguration to investigate — never make Default permissive to "fix" a missing rule.

---

## Common exam traps

1. **`Service-Type EQUALS Framed` will silently exclude MAB.** MAB sends `Service-Type = Call Check`, not `Framed`. If your Policy Set is keyed on Framed, MAB requests fall through to Default. (Real-world bug from `ise-critical-auth-lab` — your design doc fixed it.)

2. **Default policy set is `Default Network Access` — it has a rule called `Basic_Authenticated_Access`** that grants PermitAccess to any endpoint found in Internal Endpoints. That rule is on by default and turns MAB into "any MAC ever seen by ISE = full access." A common interview question is "why is this insecure" — be ready to answer.

3. **Compound conditions** in the new UI live under **Policy → Policy Elements → Conditions → Authorization → Compound Conditions** (or Library Conditions in newer versions). The exam tests that you know where to find them, not just the syntax.

4. **First match wins, top-down**, but there are *two* policies in a set: Authentication and Authorization. They evaluate in order: AuthN first, then AuthZ. AuthZ runs **only** if AuthN passed. If AuthN is "Continue" on user-not-found, AuthZ still runs — but the identity will be empty.

5. **Authorization Profile vs. Authorization Rule** — easy to confuse. Profile = the result (the bundle of attributes). Rule = the if-then logic that selects a profile.

6. **`Permit Access` vs. `PermitAccess`** — those are two different things in the UI. `PermitAccess` (one word) is the built-in profile that returns just `Access-Accept` with no other attributes. `Permit Access` (two words) is sometimes used as a friendly label for the *outcome* of a custom profile.

7. **Rule order matters more than rule conditions.** A correct rule in the wrong order produces wrong results. Reorder by dragging in the UI; verify with Live Logs which rule matched.

---

## Next

- [`lab.md`](lab.md) — hands-on procedure
- [`self-test.md`](self-test.md) — questions in the shape the exam asks them
- [`notes.md`](notes.md) — your space
