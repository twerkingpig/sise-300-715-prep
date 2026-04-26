# 05 — BYOD — Self-Test

10 questions in the shape SISE 300-715 asks. Answers at the bottom.

---

### Q1
The key difference between Single SSID and Dual SSID BYOD flows is:

A. Single SSID supports only iOS; Dual SSID supports all platforms
B. Single SSID uses one WiFi network for both onboarding and post-enrollment; Dual SSID uses one network for onboarding and a separate one for production
C. Single SSID is wireless-only; Dual SSID supports wired
D. Dual SSID is for guest users; Single SSID is for employees

### Q2
After successful BYOD enrollment, what authentication method does the device use to connect going forward?

A. PEAP-MSCHAPv2 with the user's AD password
B. EAP-TLS using the certificate provisioned during enrollment
C. MAB
D. Web authentication

### Q3
A user enrolls their personal iPhone via BYOD. Which protocol does ISE typically use to push the device certificate to the iPhone?

A. SNMP
B. RDP
C. SCEP or EST
D. LDAPS

### Q4
A user reports their corporate-registered laptop was stolen. They access the MyDevices portal and mark it as Lost/Stolen. What ISE construct moves the endpoint into a deny state?

A. The endpoint's MAC is added to a Network Device Group
B. The endpoint's MAC is moved into the BlackList Endpoint Identity Group, and an authz rule matches that group to DenyAccess
C. The endpoint is removed from Active Directory
D. The user's password is rotated

### Q5
During BYOD enrollment, ISE returns RADIUS attributes that include a redirect to the BYOD portal. Which set of attributes is required?

A. VLAN + dACL only
B. url-redirect-acl + url-redirect URL + dACL
C. Just the BYOD VLAN
D. SGT only

### Q6
What is Native Supplicant Provisioning (NSP)?

A. ISE's component that uses the device's native APIs to install a certificate and a WiFi profile during BYOD onboarding
B. A separate Cisco appliance that handles certificate enrollment
C. A Microsoft Intune integration
D. A way to push GPOs to AD-joined devices

### Q7
The MyDevices Portal is BEST described as:

A. The same portal as the BYOD enrollment portal
B. A separate ISE-hosted portal where end users manage their already-enrolled devices (view, mark lost, re-enroll, delete)
C. A Cisco-hosted SaaS for device management
D. A Windows-only feature

### Q8
A user starts BYOD enrollment but the certificate installation step fails. The portal shows a generic "could not provision certificate" error. Which is the MOST LIKELY cause?

A. The user's AD password is wrong
B. The endpoint is in the BlackList group
C. The dACL during the onboarding state doesn't permit traffic to the PSN's SCEP/EST endpoint
D. The endpoint's MAC isn't in Internal Endpoints

### Q9
After NSP completes and the device gets a new certificate, what triggers ISE to reauthenticate the device into a new authz rule?

A. The device automatically reauthenticates on the next reauth timer
B. ISE fires CoA-Reauth, which tells the switch/WLC to reauthenticate
C. The user must manually disconnect and reconnect to the WiFi
D. ISE schedules a reauth for the next day

### Q10
You're configuring a BYOD authz rule. You want it to match endpoints that have completed enrollment. Which condition is most appropriate?

A. EndPoints:LogicalProfile EQUALS BYOD
B. EndPoints:BYODRegistration EQUALS Yes (or Identity Group EQUALS RegisteredDevices)
C. Network Access:UseCase EQUALS BYOD
D. Radius:Service-Type EQUALS Framed

---

## Answers

<details>
<summary>Click to reveal</summary>

**Q1: B.** Single SSID = one WiFi network, used for both onboarding and post-enrollment. Dual SSID = a separate "BYOD-onboarding" SSID for enrollment and the corp SSID for production. Cisco supports both; choose based on operational preferences.

**Q2: B.** EAP-TLS using the cert that was provisioned during enrollment. The whole point of BYOD is moving the user from password-based auth to cert-based EAP-TLS.

**Q3: C.** SCEP or EST. SCEP is older and widely supported; EST is more modern (RFC 7030). ISE supports both, and can either issue from its own internal CA or proxy to your enterprise CA.

**Q4: B.** The endpoint MAC is moved to the BlackList Endpoint Identity Group. An authz rule placed near the top of the policy matches `EndPoints:IdentityGroup EQUALS BlackList` and returns DenyAccess. ISE may also fire CoA-Disconnect.

**Q5: B.** Same three attributes as Web Auth/Guest: url-redirect-acl, url-redirect URL, dACL. This is consistent across all redirect-based ISE flows.

**Q6: A.** NSP uses the device's native provisioning APIs (Apple Configuration Profiles, Windows EAP profiles, Android Wi-Fi configs) to push a certificate and WiFi profile during onboarding. It's not a Cisco appliance, not Intune, not GPO.

**Q7: B.** MyDevices is separate from BYOD Portal. BYOD Portal is for *initial enrollment*; MyDevices is for *ongoing management* by the end user. Different URLs, different purposes.

**Q8: C.** The onboarding-state dACL must permit the device to reach the PSN's SCEP/EST endpoint (typically 80, 443, and possibly 9090 depending on configuration). If the dACL blocks that, cert provisioning fails silently from the user's perspective.

**Q9: B.** ISE fires CoA-Reauth. This tells the switch (or WLC) to reauthenticate the endpoint, which now uses the new cert and matches the post-enrollment authz rule.

**Q10: B.** `EndPoints:BYODRegistration EQUALS Yes` is the canonical condition. `Identity Group EQUALS RegisteredDevices` is the equivalent group-based form. Both work. There's no built-in `LogicalProfile EQUALS BYOD`.

</details>

---

## Scoring

- 9–10: solid; move on.
- 6–8: re-read [`README.md`](README.md), focus on what you missed.
- 3–5: needs another pass.
- 0–2: not ready.
