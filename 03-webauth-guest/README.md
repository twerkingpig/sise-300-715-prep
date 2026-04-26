# 03 — Web Auth and Guest Services

> ~15% of the SISE exam. Where the URL-redirect mechanics from BYOD reappear, plus a layer of "who's the guest, who sponsored them, what kind of guest are they."

This and BYOD share the same underlying redirect mechanics. If you understood the *three RADIUS attributes for redirect* from your `ise-critical-auth-lab/docs/05-byod-redirect.md`, you already have the foundation.

---

## What the exam tests on this topic

1. **Centralized vs Local Web Auth** — and when each is used.
2. **Self-Registered Guest vs Sponsored Guest vs Hotspot** — three flows, three audiences.
3. **The three RADIUS redirect attributes** — `url-redirect-acl`, `url-redirect`, dACL — and what each does.
4. **Sponsor portals and sponsor groups** — who can sponsor whom.
5. **Guest Types** — daily/weekly/contractor/etc., portal-specific behavior, time bounding.
6. **Portal customization** — branding, language, AUP (acceptable use policy).
7. **CoA mechanics in the redirect-then-reauth flow** — how ISE tells the switch to reapply policy after registration.
8. **Hotspot specifically** — no auth, just AUP + MAC registration.

---

## Concept map

```
Guest types of auth, simplest to most complex:

1. Hotspot
   Endpoint plugs in / connects to SSID
   ──► ISE returns redirect to Hotspot Portal
   ──► User accepts AUP, enters nothing else
   ──► ISE registers MAC, fires CoA
   ──► Reauth → endpoint hits PermitAccess (or Guest VLAN)

2. Self-Registered Guest
   ──► Redirect to Self-Reg Portal
   ──► User fills out form (name, email, etc.)
   ──► ISE creates a Guest account, may email/SMS credentials
   ──► User logs in with those creds
   ──► CoA → reauth → hits Guest authz rule

3. Sponsored Guest
   ──► Sponsor (an internal user) creates the guest account in advance
   ──► Guest gets credentials from the sponsor
   ──► Guest connects, hits Sponsored Guest Portal, logs in
   ──► CoA → reauth → hits Guest authz rule
```

All three use the same underlying mechanism: redirect → portal → registration / login → CoA → reauth into a different authz rule.

---

## Key concepts in plain English

### Centralized vs Local Web Auth

| | Centralized Web Auth (CWA) | Local Web Auth (LWA) |
|---|---|---|
| **Who shows the portal?** | ISE | The switch / WLC |
| **How does ISE get involved?** | Returns redirect attributes to NAD; portal hosted on PSN | Switch handles the portal locally; ISE used only for AAA after credentials submitted |
| **Common use** | Most modern ISE deployments | Legacy or air-gapped scenarios |
| **Complexity** | More moving parts but more flexible | Simpler but less integrated |

The exam mostly tests CWA. Know LWA exists but don't deep-dive unless your real deployment uses it.

### The three flow types — quick reference

| Flow | Who registers? | When? | Typical audience |
|---|---|---|---|
| **Hotspot** | Nobody — just AUP click-through | At connection time | Lobby Wi-Fi, public spaces |
| **Self-Registered** | The guest themselves | At connection time | Conferences, walk-up visitors |
| **Sponsored** | An employee (sponsor) | Before the guest arrives | Scheduled meetings, contractors with a host |

### The three RADIUS attributes (same as BYOD)

When ISE's authz rule decides "this needs to go to a portal," the Access-Accept includes:

```
1. cisco-av-pair = url-redirect-acl=ACL-WEBAUTH-REDIRECT
   Tells the switch: "intercept traffic matching this ACL"
   (ACL must already exist on the switch by exact name — common gotcha)

2. cisco-av-pair = url-redirect=https://<psn>:8443/portal/...
   Tells the switch: "redirect intercepted traffic to this URL"
   ISE generates this URL with sessionId baked in for context tracking

3. dACL (downloadable ACL)
   Tells the switch: "while in the redirected state, only allow these"
   Without this, the endpoint can reach everything during the registration flow
```

Missing any one breaks the experience. URL ACL but no dACL = security hole. dACL but no URL ACL = silent black-hole. URL but no URL ACL = nothing gets intercepted.

### Sponsor groups vs guest types

These are different things; easy to confuse.

- **Sponsor Groups** = which *internal users* can sponsor guests, and which guest types they can create. E.g., "HR sponsors can create Daily and Weekly guests; receptionist can only create Daily."
- **Guest Types** = the *guest account templates*. Each defines duration, max simultaneous logins, allowed portal, AUP requirements, password complexity, etc.

A sponsor uses their *sponsor group's* permissions to create accounts of allowed *guest types*. Two-axis permission system.

### CoA in the redirect flow (this is the magic that makes it work)

After successful registration / login at the portal, ISE doesn't just say "ok, you're done." It fires a **CoA-Reauth** message to the switch. The switch reauthenticates the endpoint, ISE evaluates authz again — and *this time* the endpoint matches a different rule (the "post-registration" or "guest authenticated" rule) and gets a different VLAN/ACL.

If CoA isn't working, the user registers but stays in the redirect VLAN. Common failure: the switch's `aaa server radius dynamic-author` block isn't configured, or the wrong shared secret is used for it.

### Portal customization

ISE lets you customize portal pages — language, branding, fields, AUP text, post-login redirect. Three things worth knowing for the exam:

1. **Portal Themes** are reusable; one theme can apply to multiple portals.
2. **Page Customizations** are per-portal-page (login page, registration page, AUP page, etc.).
3. **The portal cert is presented when the browser hits the portal URL** — and *unmanaged* devices won't trust your internal CA. For BYOD this is mitigated by the device joining the trust chain after registration; for guest, you usually want a public-CA cert on the portal.

---

## Common exam traps

1. **Hotspot portal does NOT create a Guest account.** It just registers the endpoint MAC. There is no username/password. Don't confuse with Self-Registered.

2. **The url-redirect ACL must exist on the switch by exact name.** ISE doesn't push it. If misnamed, RADIUS succeeds but the redirect silently fails. Most-common BYOD/guest debugging cause.

3. **The url-redirect ACL syntax is counterintuitive.** `permit` means "redirect this," `deny` means "leave it alone." First-time engineers get this backwards constantly.

4. **CoA traffic is on UDP 1700 (Cisco default) or UDP 3799 (RFC default).** Cisco default. Both client and server must agree. Mismatch = registration appears to succeed but never reauths.

5. **Sponsor portal access is itself a portal that requires authentication.** Sponsors typically log in with AD credentials. The Sponsor Portal is configured separately from the Guest Portal.

6. **Guest accounts have a lifecycle:** Awaiting Initial Login → Active → Suspended → Expired. The exam may show a screenshot and ask "what does this status mean?"

7. **Hotspot doesn't require a guest account, but does store the registered MAC under Endpoint Identity Group `GuestEndpoints`** (or whichever group you specified). On reconnect, the endpoint is recognized and skips the AUP page (depending on settings).

8. **Local Web Auth (LWA) does NOT use the three RADIUS redirect attributes** — the switch handles the portal locally. CWA is what uses the three attributes. Know which mechanism you're talking about when answering.

---

## Next

- [`lab.md`](lab.md) — hands-on procedure (still scaffolded; fill in as you study)
- [`self-test.md`](self-test.md) — practice questions
- [`notes.md`](notes.md) — your space
