# 07 — TACACS+ Device Administration — Self-Test

8 questions. Answers at the bottom.

---

### Q1
Which protocol is used for ISE-based network device administration (admin login to switches/routers)?

A. RADIUS
B. TACACS+
C. SNMPv3
D. SSH only

### Q2
TACACS+ uses which transport protocol and port?

A. UDP/1812
B. UDP/49
C. TCP/49
D. TCP/1812

### Q3
Which is TRUE about TACACS+ vs RADIUS encryption?

A. They both encrypt only the password
B. TACACS+ encrypts the entire payload; RADIUS encrypts only the password
C. RADIUS encrypts the entire payload; TACACS+ encrypts only the password
D. Neither encrypts; both rely on TLS at the transport layer

### Q4
Why does ISE require a separate license for TACACS+?

A. It doesn't — full Premier license covers it
B. Device Administration is sold as a separate license SKU; needs to be purchased per node where TACACS+ runs
C. It requires a Cisco DNA Center subscription
D. Only required for cloud deployments

### Q5
A network admin logs into a switch via TACACS+. They type `show running-config`. The switch sends a TACACS+ Authorization-Request to ISE. What ISE construct decides whether the command is permitted?

A. The Authorization Profile
B. The Command Set assigned to the admin's authz rule
C. The Allowed Protocols
D. The Network Device Group

### Q6
A Shell Profile in TACACS+ defines:

A. Which commands the admin can run
B. What privilege level and shell attributes (e.g., default-privilege, auto-command) the admin gets at login
C. The IP addresses the admin can connect from
D. The encryption algorithms used

### Q7
Where in the ISE UI do you configure TACACS+ device admin policies?

A. Policy → Policy Sets (the same place as RADIUS policy sets)
B. Work Centers → Device Administration
C. Operations → TACACS+ Live Logs only — there is no policy UI
D. Administration → System → Settings

### Q8
Which is a key advantage of TACACS+ over RADIUS for device admin?

A. TACACS+ supports more endpoints
B. TACACS+ allows per-command authorization — each command typed by the admin is evaluated separately
C. TACACS+ is faster than RADIUS
D. TACACS+ doesn't require a license

---

## Answers

<details>
<summary>Click to reveal</summary>

**Q1: B.** TACACS+. RADIUS is for network access (endpoints connecting to the LAN), TACACS+ is for device admin (admins logging into switches).

**Q2: C.** TCP/49. Don't confuse with RADIUS UDP ports.

**Q3: B.** TACACS+ encrypts the entire payload; RADIUS only the password. This is one of the most-quoted "TACACS+ is more secure" points.

**Q4: B.** Device Administration is sold as a separate license SKU. Even with Premier ISE, you need this additional license to enable TACACS+.

**Q5: B.** Command Set. Assigned via the device admin authz rule. Each command the admin types triggers a fresh authorization check against the Command Set.

**Q6: B.** Shell Profile = default privilege, max privilege, auto-command, etc. Different from Command Set, which controls *which commands* are allowed.

**Q7: B.** TACACS+ has its own dedicated Work Center: Work Centers → Device Administration. Separate from RADIUS Policy Sets.

**Q8: B.** Per-command authorization. Each command the admin types is evaluated against ISE in real time. RADIUS can't do this — it's session-level only.

</details>

---

## Scoring

- 7–8: solid; move on.
- 5–6: re-read [`README.md`](README.md), focus on what you missed.
- 0–4: needs another pass.
