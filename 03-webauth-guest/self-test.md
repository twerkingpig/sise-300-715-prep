# 03 — Web Auth and Guest Services — Self-Test

12 questions in the shape SISE 300-715 asks. Answers at the bottom — don't peek.

---

### Q1
What is the key architectural difference between Centralized Web Auth (CWA) and Local Web Auth (LWA)?

A. CWA uses HTTPS; LWA uses HTTP
B. CWA hosts the portal on ISE; LWA hosts the portal on the switch/WLC
C. CWA only works on wireless; LWA only on wired
D. CWA requires the Premier license; LWA requires Essentials

### Q2
A user's laptop is in a "redirect" state and the browser hits an HTTP site. The switch redirects the browser to ISE's portal URL. Which RADIUS attribute, returned by ISE, told the switch *which* traffic to intercept and redirect?

A. `url-redirect`
B. `url-redirect-acl`
C. `Filter-Id`
D. `Tunnel-Private-Group-ID`

### Q3
The url-redirect ACL on the switch contains:
```
permit tcp any any eq www
deny ip any any
```
What does this configuration tell the switch to do?

A. Allow only HTTP traffic; deny everything else
B. Redirect HTTP traffic to the portal; leave all other traffic alone
C. Block all traffic except HTTP
D. Send all traffic to ISE for inspection

### Q4
Which guest flow does NOT result in the creation of a guest user account?

A. Self-Registered Guest
B. Sponsored Guest
C. Hotspot
D. All three create accounts

### Q5
A sponsor needs to create a one-day guest account. Which configuration objects must exist?

A. A Sponsor Group containing the sponsor, and a Guest Type allowing 1-day duration that the Sponsor Group can create
B. Just a Sponsor Portal — accounts are created automatically
C. A Network Device Group only
D. An Authorization Profile named "Guest"

### Q6
After a guest successfully registers and submits credentials at the portal, ISE needs to tell the switch to reauthenticate the endpoint. Which mechanism does ISE use?

A. RADIUS Access-Accept
B. SNMP trap
C. CoA (Change of Authorization) — typically CoA-Reauth
D. The switch automatically polls ISE every 60 seconds

### Q7
A guest registers at the self-registration portal but the endpoint never moves out of the redirect state. Which is the MOST LIKELY cause?

A. The portal cert is invalid
B. The Sponsor Group lacks permission to create that guest type
C. CoA is misconfigured on the switch — `aaa server radius dynamic-author` is missing or the shared secret doesn't match
D. The guest used the wrong email address

### Q8
For Hotspot to work, which of these is required?

A. A guest account created in advance by a sponsor
B. A username and password that the user enters at the portal
C. An AUP (Acceptable Use Policy) page that the user accepts
D. A valid certificate on the endpoint

### Q9
Which RADIUS attribute does ISE NOT need to send for a successful URL redirect flow?

A. `url-redirect-acl`
B. `url-redirect`
C. dACL (Downloadable ACL)
D. `Vendor-Specific Attribute: Tunnel-Type`

### Q10
You're customizing the Self-Registration portal. Where do you make changes that affect the appearance of the login page?

A. The portal's Page Customizations section
B. The Allowed Protocols section of the Policy Set
C. The Network Device Profile
D. The pxGrid configuration

### Q11
The Sponsor Portal is itself protected. How does a sponsor authenticate to it before creating guest accounts?

A. The Sponsor Portal is open to anyone
B. Sponsors authenticate using their identity store credentials (typically AD), validated against the configured Sponsor Group's identity sources
C. Sponsors authenticate with their guest's pre-shared key
D. A separate Sponsor Identity Source must be created from scratch

### Q12
A guest account has the status "Awaiting Initial Login." What does this mean?

A. The account was created but the guest has not logged in to the portal yet for the first time
B. The account is suspended pending sponsor approval
C. The account expired without ever being used
D. The guest is currently authenticated but has not generated traffic

---

## Answers

<details>
<summary>Click to reveal</summary>

**Q1: B.** CWA = ISE hosts the portal. LWA = the switch/WLC hosts it. Most modern ISE deployments use CWA because it's more flexible (centralized customization, integrated with profiling/posture). LWA exists for legacy or niche cases.

**Q2: B.** `url-redirect-acl` references an ACL *that must already exist on the switch by name* — that ACL defines which traffic to intercept. `url-redirect` is the destination URL. They work together; neither alone is enough.

**Q3: B.** "Redirect this, leave that alone." `permit` in a redirect ACL means "redirect this traffic to the portal"; `deny` means "let it flow normally." This counterintuitive syntax is a top-five exam-and-real-world gotcha.

**Q4: C.** Hotspot does NOT create a user account. It registers the *MAC* under an Endpoint Identity Group (typically `GuestEndpoints`) after the user clicks through the AUP. Self-Registered and Sponsored both produce a user account.

**Q5: A.** Two-axis permission system: which Sponsor Group the user belongs to, and which Guest Types that Sponsor Group is permitted to create. Both must align.

**Q6: C.** CoA — specifically CoA-Reauth (or CoA-Disconnect followed by reauth, depending on configuration). Without CoA, registration succeeds but the endpoint stays in the redirect state forever.

**Q7: C.** CoA flowing from ISE → switch is the most common failure mode after registration. The switch needs `aaa server radius dynamic-author` configured with a `client` line for each ISE PSN and the matching shared secret. CoA is on UDP 1700 by default (Cisco) — port mismatch also breaks this.

**Q8: C.** Hotspot's only required interaction is the AUP page. No credentials, no sponsor, no certificate. The whole point of Hotspot is "lowest friction guest access."

**Q9: D.** `Tunnel-Type` is for VLAN assignment / dynamic VLAN, not redirect. The three redirect-essential attributes are `url-redirect-acl`, `url-redirect`, and a dACL. (VLAN can be present too, but it's not a redirect attribute.)

**Q10: A.** Each portal has Page Customizations — per-page (login page, registration page, AUP page, success page) appearance and content controls. Themes are reusable across portals; Page Customizations are per-portal.

**Q11: B.** Sponsor Portal authenticates against the identity stores configured for that Sponsor Group. Typically AD. Sponsor permissions cascade from group membership.

**Q12: A.** The account exists but the guest has never authenticated against the portal with it. The state machine for guest accounts is: Awaiting Initial Login → Active → (Suspended | Expired). The exam may show a screenshot and ask what state means.

</details>

---

## Scoring

- 11–12: solid; move on.
- 8–10: re-read [`README.md`](README.md), focus on what you missed.
- 4–7: needs another pass; don't move on.
- 0–3: not ready — re-lab and re-read.
