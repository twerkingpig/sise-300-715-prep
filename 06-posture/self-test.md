# 06 — Endpoint Compliance (Posture) — Self-Test

10 questions in the shape SISE 300-715 asks. Answers at the bottom.

---

### Q1
Which are the three valid PostureStatus values for an ISE session?

A. Compliant, NonCompliant, Unknown
B. Active, Inactive, Pending
C. Trusted, Untrusted, Quarantined
D. Allowed, Denied, Posture-Pending

### Q2
A new endpoint connects. ISE sees it for the first time. What's the initial PostureStatus?

A. Compliant
B. NonCompliant
C. Unknown
D. The status carries over from a previous session

### Q3
What is the AnyConnect ISE Posture module?

A. A standalone Cisco product separate from AnyConnect
B. A module within AnyConnect (or Cisco Secure Client) that runs posture checks on the endpoint and reports back to ISE
C. A web-based posture check that runs in the browser
D. A part of ISE's PSN that runs scripts on endpoints remotely

### Q4
A user's laptop is in Posture-Unknown state and the user opens a browser. The browser is redirected to ISE's Client Provisioning Portal. Which set of RADIUS attributes did ISE return to enable this redirect?

A. VLAN + dACL only
B. url-redirect-acl + url-redirect URL + dACL
C. ACL-Posture + ACL-Bypass
D. SGT only

### Q5
You configure a Posture Requirement called "Endpoint has Windows Defender running." Internally, this Requirement points to which other ISE construct?

A. A Client Provisioning Policy
B. A Posture Condition (one or more — e.g., "process WinDefend.exe is running")
C. An Authorization Profile
D. An Identity Source Sequence

### Q6
What is Periodic Reassessment (PRA) used for?

A. Initial posture check before access is granted
B. Re-checking compliance on already-Compliant sessions at a configurable interval
C. Forcing all sessions to re-authenticate
D. Updating the AnyConnect agent automatically

### Q7
A user is in stealth-mode posture. The check fails because Windows Defender is disabled. What does the user see?

A. A pop-up explaining what failed and how to fix it
B. Nothing visible — the agent runs silently. The user only knows something is wrong because their network access is restricted
C. A toast notification with a link to the IT page
D. An automatic uninstall of the offending software

### Q8
A Posture session sticks at Unknown indefinitely. The agent appears to be installed and running. What's the MOST LIKELY cause?

A. The user's AD password expired
B. The endpoint can't reach the PSN over HTTPS (typically 8443 or 8905) — a firewall or dACL is blocking
C. The Posture license is missing
D. The endpoint is on the wrong VLAN

### Q9
Client Provisioning Policies and Posture Policies are different things. Which best describes the difference?

A. They're the same thing under different names
B. Client Provisioning decides WHICH AnyConnect/Secure Client version and what posture profile to push; Posture Policies decide WHAT to check on the endpoint
C. Client Provisioning is for guests; Posture is for employees
D. Client Provisioning is for wired; Posture is for wireless

### Q10
After a Non-Compliant endpoint completes remediation (e.g., the user enabled their AV), the agent reports the new state to ISE. What does ISE do to apply new authz?

A. Nothing — the next reauth picks up the change
B. ISE fires CoA-Reauth to the NAD; the endpoint reauthenticates and the new authz rule (Compliant) matches
C. ISE sends an SNMP trap to the switch
D. The user must manually disconnect and reconnect

---

## Answers

<details>
<summary>Click to reveal</summary>

**Q1: A.** Compliant, NonCompliant, Unknown. These are the values of `Session:PostureStatus` used in authz rule conditions.

**Q2: C.** Unknown. The agent hasn't reported yet, so ISE has no compliance data. Authz rules typically detect Unknown and redirect to Client Provisioning so the agent can install/scan.

**Q3: B.** A module within AnyConnect / Cisco Secure Client. Other modules include VPN, Network Access Manager, Web Security. The Posture module runs the compliance checks the agent reports back to ISE.

**Q4: B.** Same three redirect attributes as everywhere else in ISE: url-redirect-acl, url-redirect URL, dACL. The pattern is consistent across BYOD, Web Auth, Posture.

**Q5: B.** Conditions. A Requirement groups one or more Conditions ("process X is running" + "registry key Y exists" → Requirement: AV is running properly). A Posture Policy then attaches Requirements to a target endpoint set.

**Q6: B.** PRA reassesses already-Compliant sessions periodically. Default interval is typically hours, configurable. PRA doesn't replace the initial check — that still happens via redirect-then-install.

**Q7: B.** Stealth mode runs silently. The user sees no pop-ups, no notifications. They only know something's wrong by observing limited access. Stealth is for environments where you don't want users self-remediating; trade-off is users have no visibility.

**Q8: B.** The agent must be able to reach the PSN over HTTPS (typical ports 8443 and 8905). If the dACL or upstream firewall blocks this, the agent installs but can't report results — session stays at Unknown.

**Q9: B.** Client Provisioning is "what to install" — which AnyConnect/Secure Client version, which posture profile. Posture Policies are "what to check" — which Requirements apply to the endpoint based on OS, identity group, etc.

**Q10: B.** ISE fires CoA-Reauth. This is the same CoA mechanism used in BYOD and Web Auth — reauth tells the NAD to re-evaluate the session, which now matches a different authz rule because PostureStatus changed.

</details>

---

## Scoring

- 9–10: solid; move on.
- 6–8: re-read [`README.md`](README.md), focus on what you missed.
- 3–5: needs another pass.
- 0–2: not ready.
