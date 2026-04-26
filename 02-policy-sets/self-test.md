# 02 — Policy Sets — Self-Test

12 questions in the shape SISE 300-715 asks. Answer them cold without referring to your notes. Score yourself, then re-read what you missed.

Answers at the bottom. **Do not peek until you've answered all 12.**

---

### Q1
What is the order of evaluation when a RADIUS Access-Request arrives at ISE?

A. Authorization Policy → Authentication Policy → Allowed Protocols → Policy Set
B. Policy Set → Authentication Policy → Authorization Policy
C. Authentication Policy → Policy Set → Authorization Policy
D. Allowed Protocols → Authentication Policy → Authorization Policy → Policy Set

### Q2
A device sends a RADIUS Access-Request with `Service-Type = Call Check`. What does this indicate to ISE?

A. The endpoint is performing 802.1X
B. The endpoint is being MAB-authenticated
C. The endpoint is connecting to a wireless SSID
D. The endpoint is initiating an admin session via TACACS+

### Q3
You configure a Policy Set with the condition `DEVICE:Device Type EQUALS Access-Switch AND Radius:Service-Type EQUALS Framed`. Which RADIUS request will FAIL to match this set?

A. An 802.1X request from a switch in Device Type group `Access-Switch`
B. A MAB request from a switch in Device Type group `Access-Switch`
C. An 802.1X request from a wireless controller in Device Type group `Access-Switch`
D. A request from a Cisco IP phone via 802.1X

### Q4
Which best describes the difference between an Authorization Profile and an Authorization Rule?

A. They are the same thing with different names
B. A Profile is the if-then rule; a Rule is the bundle of attributes returned to the NAD
C. A Rule is the if-then logic; a Profile is the bundle of RADIUS attributes returned on match
D. Profiles are used for AAA; Rules are used for posture

### Q5
The default `Default Network Access` policy set contains a rule named `Basic_Authenticated_Access` that returns `PermitAccess` for any endpoint where `AuthenticationStatus EQUALS AuthenticationPassed`. What is the security implication of leaving this rule enabled in production?

A. None — it only applies during initial deployment
B. Any endpoint with a valid certificate gets full access
C. Any endpoint that successfully MAB authenticates against Internal Endpoints gets full network access, including endpoints that ISE auto-profiled
D. Only AD-joined users can match the rule

### Q6
You have an Identity Source Sequence configured as `[AD-CORP, LDAP-FALLBACK, Internal Users]` with the option **"If process failed → Continue"**. Active Directory's connector is broken. A user authenticates with valid AD credentials. What happens?

A. Authentication fails because AD is the first store
B. ISE waits 60 seconds and retries AD
C. ISE proceeds to LDAP-FALLBACK, attempts authentication against the same directory via LDAP, and may succeed
D. ISE skips authentication and proceeds directly to authorization

### Q7
In an Authorization Profile, which attribute combination is REQUIRED for a successful URL redirect to a portal (e.g., for BYOD onboarding)?

A. VLAN + dACL only
B. URL Redirect ACL + URL Redirect URL only
C. URL Redirect ACL + URL Redirect URL + dACL
D. URL Redirect URL + Web Authentication = Centralized only

### Q8
A switch port is configured with IBNS 2.0. An endpoint plugs in. ISE returns an authorization profile with VLAN `EMPLOYEES (100)` but the endpoint is a Cisco IP phone. The phone fails to register. Why?

A. The switch port is not configured with `voice vlan` enabled
B. The Authorization Profile is missing the `device-traffic-class=voice` Cisco-AV-Pair, so the switch placed the phone on the data VLAN
C. ISE does not support voice authentication
D. The phone's MAC was not added to Internal Endpoints

### Q9
You configured a custom Policy Set with a strict condition. Live Logs show that requests for an endpoint are matching the **Default** policy set instead of your custom set. What is the FIRST place to check?

A. The Authorization Profile assigned to the endpoint
B. The Identity Source Sequence
C. The Policy Set's match condition — it does not actually match the incoming RADIUS attributes
D. The endpoint's certificate validity

### Q10
Which of these is a valid CONDITION for an Authorization Rule, but not a valid condition for a Policy Set?

A. `Radius:Service-Type EQUALS Framed`
B. `DEVICE:Device Type EQUALS Access-Switch`
C. `Network Access:AuthenticationStatus EQUALS AuthenticationPassed`
D. `DEVICE:Location EQUALS Campus-Building-A`

### Q11
You build an Authorization Rule with conditions: `Certificate:Issuer CN EQUALS Internal-CA`. A laptop authenticates with a valid cert from a *different* CA you trust. ISE's Authentication policy passes (the cert chains to a trusted root). What happens at Authorization?

A. The endpoint matches the rule because authentication passed
B. The endpoint matches the rule because the cert is from a trusted CA
C. The endpoint fails to match this rule and falls through to the next authz rule (or Default)
D. ISE rejects the request at the authorization stage with `5400 Authentication failed`

### Q12
A common compound condition for a corporate workstation might be:
`Certificate:Issuer CN EQUALS Internal-CA AND AD:ExternalGroups EQUALS Domain Computers AND EndPoints:LogicalProfile EQUALS Windows-Workstation`

Why use a compound condition like this instead of just `Certificate:Issuer CN EQUALS Internal-CA`?

A. ISE evaluates compound conditions faster than single conditions
B. The compound condition catches stolen certs by also requiring AD group membership and endpoint profile match — defense in depth
C. Cisco best practice mandates at least three conditions per rule
D. The single-condition rule fails to evaluate at runtime

---

## Answers

<details>
<summary>Click to reveal answers</summary>

**Q1: B.** Policy Set first (which one applies?), then Authentication Policy (who is this?), then Authorization Policy (what can they do?). The order matters because each step gates the next.

**Q2: B.** `Service-Type = Call Check` is the RADIUS attribute the switch sends for MAB. 802.1X uses `Service-Type = Framed`. This distinction is heavily tested.

**Q3: B.** MAB sends `Service-Type = Call Check`, not `Framed`, so it cannot match a policy set keyed on `Framed`. The MAB request would fall through to whichever set matches next (probably Default). This is the real-world bug from the lab repo.

**Q4: C.** The Rule is the `if-then` logic ("if conditions match, return profile X"). The Profile is the bundle of RADIUS attributes (VLAN, dACL, reauth timer, etc.) that gets returned.

**Q5: C.** `Basic_Authenticated_Access` returns `PermitAccess` for any successful authn — including MAB matches in Internal Endpoints, which can be auto-populated by profiling. Net effect: any MAC ISE has ever seen gets PermitAccess. This is the most important policy set anti-pattern to recognize.

**Q6: C.** With `If process failed → Continue`, ISE moves to the next store. LDAP-FALLBACK pointing at the same directory often still works because the AD *connector* failed, not the directory itself.

**Q7: C.** Three RADIUS attributes are required: `url-redirect-acl=<ACL_NAME>` (must exist on the switch by exact name), `url-redirect=<portal URL>`, and a dACL to lock down what the endpoint can reach during the redirect. Missing any one breaks the flow.

**Q8: B.** Without `device-traffic-class=voice` in the AV-Pair, the switch treats the returned VLAN as the data VLAN, not the voice VLAN. This is one of the most common dot1x-with-IP-phone bugs.

**Q9: C.** If your custom set isn't being matched, the *condition* on the Policy Set isn't actually matching the RADIUS request. Check what attributes the request actually carries (Live Log → Other Attributes), and compare to your set's condition.

**Q10: C.** Policy Set conditions can only use attributes available *before* authentication completes — typically `DEVICE:`, `Radius:`, basic attributes. `Network Access:AuthenticationStatus` is only known *after* authn, so it can only be used in Authorization Policy.

**Q11: C.** The condition `Issuer CN EQUALS Internal-CA` literally requires the issuer to be Internal-CA. A different (even trusted) CA's cert won't match. The endpoint falls through to the next authz rule.

**Q12: B.** Defense in depth. Any one of those signals can be compromised: certs can be stolen, AD groups can be misconfigured, endpoint profiles can be spoofed. Requiring all three together raises the bar significantly. This is the policy design pattern Cisco recommends and the exam tests.

</details>

---

## Scoring

- 11–12 correct: solid grasp. Move on to the next section.
- 8–10: re-read [`README.md`](README.md), focus on what you missed.
- 4–7: redo [`lab.md`](lab.md) with the README open. Don't move on.
- 0–3: this topic isn't ready. Re-read everything and lab again.
