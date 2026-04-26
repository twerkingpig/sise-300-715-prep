# 05 — BYOD

> ~10% of the SISE exam. BYOD is "let users register their personal devices for corporate Wi-Fi access," with cert-based onboarding so they don't keep typing AD passwords.

This shares mechanics with Section 03 (Web Auth & Guest) — same redirect attributes, same CoA reauth pattern. The new ingredient: **certificate provisioning**.

---

## What the exam tests on this topic

1. **Single SSID vs Dual SSID flow** — the two ways BYOD onboarding can be structured.
2. **Native Supplicant Provisioning (NSP)** — what gets pushed to the device.
3. **MyDevices Portal** — where users manage their own enrolled devices (lost-device flagging, etc.).
4. **The cert provisioning protocols** — SCEP, EST.
5. **Endpoint Identity Group changes** during enrollment — `RegisteredDevices` group.
6. **EAP-TLS** — the post-enrollment auth method.
7. **The BYOD blacklist** — how to revoke a device.

---

## Concept map

```
Single SSID flow:                       Dual SSID flow:
─────────────────                       ───────────────
Connect to corp SSID                    Connect to "BYOD-onboarding" SSID (open or PEAP)
       │                                       │
       ▼                                       ▼
PEAP / MAB authn                        Authenticate with AD creds
       │                                       │
       ▼                                       ▼
ISE: device not registered              ISE redirects to BYOD portal
       │                                       │
       ▼                                       ▼
Redirect to BYOD portal                 User authenticates, agrees to onboard
       │                                       │
       ▼                                       ▼
NSP installs cert + WiFi profile        NSP installs cert + WiFi profile
       │                                       │
       ▼                                       ▼
CoA-Reauth                              User reconnects to corp SSID
       │                                       │
       ▼                                       ▼
Reauth with EAP-TLS using new cert      Auth with EAP-TLS using new cert
       │                                       │
       ▼                                       ▼
Lands in BYOD authz rule                Lands in BYOD authz rule
```

---

## Key concepts in plain English

### Single SSID vs Dual SSID

| | Single SSID | Dual SSID |
|---|---|---|
| **# of WiFi networks** | One | Two — onboarding and corp |
| **Initial auth on first connect** | PEAP-MSCHAPv2 with AD creds, OR MAB | Open or PEAP on the onboarding SSID |
| **Pros** | Simpler for users — one SSID name to remember | Cleaner separation of onboarding vs production traffic |
| **Cons** | Mixes onboarding and post-enrollment traffic on the same SSID | More configuration; users must remember two SSIDs |

Both work. Cisco supports both. The exam may ask about the differences and which is appropriate when.

### Native Supplicant Provisioning (NSP)

ISE's NSP service pushes a configuration package to the user's device that contains:
1. **A device certificate** issued by your internal CA (or ISE's internal CA) via SCEP or EST.
2. **A WiFi profile** that tells the device "use SSID `corp-wifi`, EAP-TLS, present this cert."
3. (For wired) A wired auth profile.

After NSP completes, the device has everything it needs to do EAP-TLS. The user no longer types passwords.

NSP supports macOS, Windows, iOS, Android, and ChromeOS — each via their native provisioning APIs.

### Cert provisioning protocols — SCEP and EST

| | SCEP | EST |
|---|---|---|
| **What** | Simple Certificate Enrollment Protocol | Enrollment over Secure Transport |
| **Transport** | HTTP-based | HTTPS-based, more modern |
| **Status** | Older but very widely supported | Newer (RFC 7030); preferred for new deployments |

ISE can issue certs from its internal CA or proxy enrollment to your enterprise CA via SCEP/EST. The exam wants you to know both names exist and what the difference is.

### MyDevices Portal

A separate ISE-hosted portal where end users can:
- View devices they've registered
- Mark a device as Lost / Stolen (which triggers blacklist)
- Re-enroll a device
- Delete a device they no longer use

URL is typically `https://<psn>:8443/mydevices/`. Authentication: AD credentials.

### Endpoint Identity Group changes during enrollment

During BYOD flow, the endpoint moves through Endpoint Identity Groups:

```
Initial connect:        Unknown
After enrollment:       RegisteredDevices  (or a custom group you specify)
After lost/stolen:      BlackList
```

Authz rules use these:
- `EndPoints:IdentityGroup EQUALS RegisteredDevices` → BYOD profile (post-enrollment)
- `EndPoints:IdentityGroup EQUALS BlackList` → DenyAccess
- `EndPoints:BYODRegistration EQUALS Yes` (alternative attribute, also useful)

### The BYOD blacklist

When a user marks a device lost in MyDevices, ISE moves the endpoint to the `BlackList` Endpoint Identity Group. An authz rule `EndPoints:IdentityGroup EQUALS BlackList → DenyAccess` (placed near the top of the policy) blocks the device immediately.

ISE typically also fires CoA-Disconnect to terminate any active session for that endpoint.

---

## Common exam traps

1. **EAP-TLS requires the supplicant to TRUST the EAP cert's issuer.** That's why NSP is configured to push the trust chain too — without it, EAP-TLS handshake fails on the *server* cert validation.

2. **The url-redirect ACL must exist on the switch** (same as Web Auth/Guest). Same gotcha, same fix.

3. **BYOD requires a cert-issuing CA.** ISE's internal CA works for lab; production deployments often use the enterprise CA (Microsoft AD CS, etc.) via SCEP/EST.

4. **MyDevices Portal is separate from BYOD Portal.** BYOD Portal is for the *initial* enrollment. MyDevices is for *managing* already-enrolled devices.

5. **Marking a device as lost in MyDevices does NOT immediately disconnect it** unless a CoA-Disconnect is configured to fire. Without that, the device stays connected until reauth time.

6. **A user can register multiple devices** — typically capped per Guest Type / BYOD policy. Default is often 5. Limit configurable per portal.

7. **The cert provisioning step requires the endpoint to talk to ISE's CA endpoint.** Make sure the dACL during onboarding allows traffic to the PSN's SCEP/EST port (typically 80 or 443 + 9090).

8. **Single SSID flow can fail silently if PEAP isn't enabled.** Single SSID typically expects users to first authenticate with AD creds via PEAP-MSCHAPv2 — that requires PEAP in the Allowed Protocols.

---

## Next

- [`lab.md`](lab.md) — hands-on procedure (still scaffolded; fill in as you study)
- [`self-test.md`](self-test.md) — practice questions
- [`notes.md`](notes.md) — your space
