# 08 — TrustSec — Self-Test

8 questions. Answers at the bottom.

---

### Q1
What is a Security Group Tag (SGT)?

A. A VLAN ID assigned at L2
B. An IP-based ACL classifier
C. A numeric tag attached to a session/traffic that represents identity-based group membership for use in policy enforcement
D. A username

### Q2
How is an SGT typically assigned to an endpoint's session?

A. Manually configured on the switch port
B. Returned by ISE in the Authorization Profile after successful authentication
C. Derived from the endpoint's MAC OUI
D. Set by the user in their device profile

### Q3
What is SXP (SGT Exchange Protocol) used for?

A. Encrypting RADIUS traffic
B. Conveying IP-to-SGT mappings between TrustSec-aware devices and devices that can enforce SGT policy but don't natively process inline SGT
C. Replacing 802.1X
D. Cisco's proprietary version of LLDP

### Q4
An SGACL has a source SGT of 10 (Employees) and a destination SGT of 20 (Servers). Where is this SGACL enforced?

A. On the source switch (where Employee endpoint connects)
B. On the destination side — the device with the destination SGT, typically the switch in front of the Servers
C. On the ISE PSN
D. On the firewall always

### Q5
What is "Environment Data" in the TrustSec context?

A. The temperature and humidity in the data center
B. The list of SGT names/IDs and the SGACL matrix that NADs download from ISE so they can interpret tags and enforce policy
C. The capacity of the network device
D. The state of network interfaces

### Q6
Inline tagging vs SXP — which is true?

A. Inline tagging carries the SGT in the L2 header (CMD field) on capable Cisco gear; SXP is a TCP protocol that carries IP-to-SGT mappings between devices
B. Inline tagging is for wireless; SXP is for wired
C. They're the same thing with different names
D. Inline tagging requires a separate license; SXP doesn't

### Q7
A modern Cisco campus uses inline TrustSec across the fabric. An older firewall at the edge can't process inline SGT but can enforce policy based on SGT-to-IP mappings. What's the typical solution?

A. Replace the firewall
B. Configure SXP between the access switch and the firewall, so the firewall learns IP-to-SGT mappings and can enforce SGACLs
C. Use VLAN-based segmentation only at the firewall
D. Disable TrustSec for endpoints near the firewall

### Q8
An Authorization Profile can return both a VLAN and an SGT. Why?

A. They're mutually exclusive — only one will apply
B. They're complementary — VLAN handles L2 segmentation, SGT travels with the session for fabric-wide tag-based enforcement
C. SGT is just a fallback for VLAN
D. Returning both causes ISE to log an error

---

## Answers

<details>
<summary>Click to reveal</summary>

**Q1: C.** SGT is a numeric tag representing identity-based group membership. Different from a VLAN ID. Used in fabric-wide policy enforcement.

**Q2: B.** SGT is returned by ISE in the Authorization Profile after authn — the same way VLAN, dACL, and other attributes are returned.

**Q3: B.** SXP carries IP-to-SGT mappings between devices. Used when a device can enforce SGT-based policy but can't read inline SGT in L2 headers.

**Q4: B.** Destination side. The switch (or firewall) in front of the destination is where SGACLs enforce — it knows both the source SGT (from inline tag or SXP) and the destination SGT (its own connected segment).

**Q5: B.** Environment Data is the SGT-to-name mappings and SGACL matrix that NADs download from ISE. Without it, NADs don't know what SGT 10 means.

**Q6: A.** Inline tagging carries SGT in the L2 CMD field on TrustSec-capable Cisco gear. SXP is a TCP protocol carrying IP-to-SGT mappings, used to bridge the gap to non-inline-capable devices.

**Q7: B.** Configure SXP between the access switch (or another TrustSec-aware device) and the firewall. The firewall learns mappings, applies SGACL-style policy.

**Q8: B.** Complementary. VLAN handles traditional L2 segmentation; SGT enables tag-based enforcement that travels with the session anywhere in the fabric.

</details>

---

## Scoring

- 7–8: solid; move on.
- 5–6: re-read [`README.md`](README.md), focus on what you missed.
- 0–4: needs another pass.
