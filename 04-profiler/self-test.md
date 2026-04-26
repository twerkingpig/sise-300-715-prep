# 04 — Profiler — Self-Test

12 questions in the shape SISE 300-715 asks. Answers at the bottom.

---

### Q1
You enable the DHCP probe on ISE but no DHCP attributes show up for any endpoint. The endpoints DO get DHCP addresses successfully from a separate DHCP server. What is the most likely missing piece?

A. The DHCP probe is bound to the wrong network interface
B. ISE doesn't see the DHCP traffic — needs `ip helper-address <PSN>` on the SVI, or DHCP relay configured
C. The DHCP server is not licensed for ISE integration
D. DHCP probe only works on wireless

### Q2
Which probe should you enable FIRST when starting profiling deployment?

A. NMAP — for active fingerprinting
B. NetFlow — for traffic patterns
C. RADIUS and DHCP — they're cheap, high-quality, and require minimal configuration
D. SNMP Trap — most accurate

### Q3
A profile policy has Minimum Certainty Factor = 30. An endpoint matches three conditions in that profile, each adding 10 to certainty. Will the endpoint be profiled?

A. No — minimum is 30 but only 30 was reached, so it's borderline
B. Yes — certainty 30 meets minimum 30, so the profile is assigned
C. No — minimum is exclusive (>30 required)
D. Only if the endpoint matches at least one parent profile

### Q4
The HTTP probe captures User-Agent strings. When does it actually run?

A. Constantly — ISE inspects all HTTP traffic
B. Only when the endpoint is in a URL-redirect state and traverses a PSN-hosted portal
C. Only on wireless endpoints
D. Only after the AnyConnect ISE module is installed

### Q5
You have several Cisco IP Phone models on the network. You want one authz rule that catches all of them rather than one rule per model. What ISE construct should you use?

A. Endpoint Identity Group with all the MAC addresses listed
B. A Logical Profile that groups Cisco-IP-Phone-7945, Cisco-IP-Phone-8841, etc.
C. A Network Device Group
D. A Sponsor Group

### Q6
What is the difference between an Endpoint Identity Group and a Logical Profile?

A. They're the same thing with different names
B. Endpoint Identity Group is a container (manual or auto-assigned); Logical Profile is a tag derived from profile matching
C. Endpoint Identity Group is for printers only; Logical Profile is for everything else
D. Logical Profile is for wireless; Endpoint Identity Group is for wired

### Q7
Your authz rule has the condition `EndPoints:LogicalProfile EQUALS IP-Camera`. An attacker spoofs a known camera's MAC onto their laptop and connects. Initially the laptop matches the IP-Camera profile (because the MAC is associated with that profile in ISE's history) and gets the IoT VLAN. Eventually ISE notices the laptop is sending Windows DHCP fingerprints and HTTP traffic. Which ISE feature can detect this MAC-spoofing scenario?

A. Posture
B. pxGrid
C. Anomalous Endpoint Detection
D. CoA-Disconnect

### Q8
The Cisco Feed Service (Profiler Feed) does which of the following?

A. Updates ISE with new policy sets
B. Updates ISE with new built-in profile policies for newly-released devices, and refines existing ones
C. Pushes guest portal templates from Cisco
D. Provides a list of vulnerable devices

### Q9
A device is in the "Unknown" Endpoint Identity Group. What does this mean?

A. The endpoint failed authentication
B. The endpoint has been profiled, but its profile result is ambiguous
C. The endpoint has not yet matched any Profile Policy with sufficient certainty
D. The endpoint was manually placed there by an administrator

### Q10
You manually move an endpoint MAC into the static Endpoint Identity Group "Workstation." Profiler later detects strong DHCP attributes that match "Axis-IP-Camera" with high certainty. What happens?

A. ISE moves the endpoint to "Axis-IP-Camera"
B. The endpoint stays in "Workstation" because static assignments override profiling
C. ISE creates a duplicate endpoint
D. ISE rejects authentication for the endpoint

### Q11
The NMAP probe in ISE is BEST described as:

A. Always running, scanning every endpoint that connects
B. Only invoked by specific profile policies that need active fingerprinting (e.g., to confirm a device's open ports)
C. Required for HTTP probe to work
D. Used only for wireless endpoints

### Q12
A profile hierarchy has `Cisco-Device` as a parent, with children `Cisco-IP-Phone` and `Cisco-Switch`. An endpoint matches conditions on the `Cisco-IP-Phone` profile. Which of these is TRUE?

A. The endpoint is tagged ONLY as Cisco-IP-Phone
B. The endpoint is tagged as Cisco-IP-Phone, Cisco-Device (the parent), and any matching grandparents
C. The endpoint is tagged as Cisco-IP-Phone and Cisco-Switch (peer profiles)
D. Profile hierarchy doesn't affect tagging

---

## Answers

<details>
<summary>Click to reveal</summary>

**Q1: B.** DHCP probe needs the DHCP traffic to physically reach the PSN. Either `ip helper-address <PSN>` on the SVI (most common) or a DHCP relay configured to forward to ISE. Without the traffic, the probe sees nothing.

**Q2: C.** RADIUS is automatic (you're already doing AAA). DHCP is the highest-quality cheap probe. Both should be on day one. Other probes can layer on as needed; NMAP especially should be deferred until you understand what it triggers.

**Q3: B.** Minimum Certainty Factor is inclusive — if the endpoint reaches the minimum exactly, it matches. The profile is assigned.

**Q4: B.** HTTP probe captures User-Agent only when the endpoint is being redirected to an ISE-hosted portal. Endpoints that never trigger a redirect have no HTTP attributes.

**Q5: B.** Logical Profile. Group multiple specific profiles into one tag, reference that tag from authz rules. Cleaner than enumerating every model.

**Q6: B.** Endpoint Identity Group is a container (Profiled vs Unknown vs static groups like Printers). Logical Profile is a tag derived from profile matching. Authz rules can reference either, often both.

**Q7: C.** Anomalous Endpoint Detection. ISE notices that an endpoint's profile attributes have shifted from "Axis-Camera" to "Windows-Workstation" — that's anomalous, and ISE can fire a CoA to re-evaluate or quarantine.

**Q8: B.** Feed Service brings new built-in profile policies and refines existing ones, regularly. It also brings updated OUI mappings. It does NOT push policy sets, portals, or vulnerability data.

**Q9: C.** "Unknown" means ISE has profiled the endpoint but it didn't match any Profile Policy with enough certainty. Different from authn failure (which doesn't depend on profiling at all).

**Q10: B.** Static Endpoint Identity Group assignment overrides automatic profiling. The endpoint stays in "Workstation" until you manually remove it. This is intentional — it lets you override profiler decisions you don't trust.

**Q11: B.** NMAP is invoked selectively by profile policies that explicitly call for it (e.g., "if device looks like X, scan ports to confirm"). It's loud and slow; running it on every endpoint would flood the network.

**Q12: B.** Profile hierarchy means matching a child also satisfies the parent. A 7945 phone is tagged as `Cisco-IP-Phone-7945`, `Cisco-IP-Phone`, AND `Cisco-Device`. Sibling profiles (`Cisco-Switch`) don't apply.

</details>

---

## Scoring

- 11–12: solid; move on.
- 8–10: re-read [`README.md`](README.md), focus on what you missed.
- 4–7: needs another pass; don't move on.
- 0–3: not ready.
