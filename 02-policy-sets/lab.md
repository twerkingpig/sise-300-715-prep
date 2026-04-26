# 02 — Policy Sets — Lab

A focused 60–90 minute lab. You'll build a custom Policy Set from scratch, apply it, watch a real auth flow against it in Live Logs, and break it on purpose to see how failures look.

This lab assumes:
- ISE is reachable at `https://<ise-fqdn>`.
- A switch is configured as a Network Access Device in ISE.
- You have a wired test client (laptop or VM) cabled to a switch port that runs IBNS 2.0 or legacy 802.1X.

If any of the above is missing, fix it before starting. The point of this section is *policy*, not bring-up.

---

## Part 1 — Read the Default policy set

Don't skip this. Knowing what's already there makes everything else easier.

1. Navigate: **Policy → Policy Sets**.
2. Click into **Default**.
3. Expand **Authentication Policy**. Note:
   - The MAB rule, condition `Wired_MAB`, identity store `Internal Endpoints`.
   - The Dot1X rule, condition `Wired_802.1X`, identity store `All_User_ID_Stores`.
4. Expand **Authorization Policy**. Find the rule called **`Basic_Authenticated_Access`**.
   - Read its conditions. Specifically `Network Access:AuthenticationStatus EQUALS AuthenticationPassed`.
   - Read its result: **`PermitAccess`**.

   This single rule is why an unconfigured ISE seems to "just work" for any MAC: any endpoint that authenticates *at all* (even via MAB if found in Internal Endpoints) gets full access. **Take a screenshot.** This is on the exam in spirit if not literally.

5. **Capture screenshots:** the Policy Sets list view + Default's authorization rules. Save under `_assets/screenshots/02-policy-sets/`.

## Part 2 — Build a custom Policy Set

Goal: a policy set named `LAB-WIRED-DOT1X` that catches wired requests from your access switches and applies tighter rules than Default.

### 2.1 — Create the set

1. **Policy → Policy Sets → "+" (top of the list)** to add a new set above Default.
2. Name: `LAB-WIRED-DOT1X`.
3. Description: `Lab access switches, wired only (802.1X and MAB)`.
4. Conditions — click into the conditions cell:
   - Click **Click to add an attribute**.
   - Build: `DEVICE:Device Type EQUALS Access-Switch` (assumes you've created a Device Type group named Access-Switch and assigned your switch to it; if not, do that first under **Administration → Network Resources → Network Device Groups**).
   - **Optional**, makes it tighter: AND `Radius:NAS-Port-Type EQUALS Ethernet`. Don't use `Service-Type EQUALS Framed` — it filters out MAB.
5. **Allowed Protocols / Server Sequence:** `Default Network Access`.
6. **Save**.

The set is created but empty. ISE will use Default until you build out the inner policies.

### 2.2 — Authentication Policy inside the set

1. From the Policy Sets list, click the `>` arrow next to `LAB-WIRED-DOT1X`.
2. Expand **Authentication Policy**. Two rules to add:
   - **MAB**:
     - Conditions: `Wired_MAB` (library condition).
     - Identity store: `Internal Endpoints`.
     - **If user not found → Continue** (so unknown MACs fall to Authorization for matching, not auto-reject).
   - **Dot1X**:
     - Conditions: `Wired_802.1X`.
     - Identity store: pick or create an Identity Source Sequence with `[AD, Internal Users]` and `If process failed → Continue`.
     - **If auth fail → Reject**.
     - **If user not found → Reject**.
3. **Save**.

### 2.3 — Authorization Profiles you'll need

Before writing authz rules, create the profiles they'll point at.

1. **Policy → Policy Elements → Results → Authorization → Authorization Profiles → +Add**.
2. Create three profiles:

| Profile name | VLAN | Notes |
|---|---|---|
| `LAB-EMPLOYEE` | A real employees VLAN ID/name | Reauth Timer = 28800 (8h). |
| `LAB-MAB-IOT` | A real IoT VLAN | Optional dACL = ACL-IOT-LIMITED if you have one. |
| `LAB-DENY` | n/a | Access Type = `ACCESS_REJECT`. Use this instead of the built-in `DenyAccess` when you want a *named* deny in your rules. |

(`PermitAccess` and `DenyAccess` are built-in profiles you can also use directly. Custom-named profiles make Live Logs easier to read.)

### 2.4 — Authorization Policy inside the set

Add these rules, top-to-bottom, to the set:

| # | Name | Conditions | Profile |
|---|---|---|---|
| 1 | `LAB-EMPLOYEES` | `Network Access:EapAuthentication EQUALS EAP-TLS` AND `Certificate:Issuer CN EQUALS <your internal CA>` AND `Radius:Service-Type EQUALS Framed` | `LAB-EMPLOYEE` |
| 2 | `LAB-IOT-CAM` | `EndPoints:IdentityGroup EQUALS <your IoT group>` AND `Radius:Service-Type EQUALS Call-Check` | `LAB-MAB-IOT` |
| Default | (default catch-all) | — | `LAB-DENY` |

**Save.** Notice the explicit `Service-Type EQUALS Framed` on rule 1 and `Service-Type EQUALS Call-Check` on rule 2 — that gates each rule to its auth method (lesson from real lab work).

## Part 3 — Test it

1. **Disable the Default policy set's `Basic_Authenticated_Access` rule** for the duration of this lab (so unknown MACs don't sneak through Default if your custom set's condition somehow doesn't match):
   - **Policy → Policy Sets → Default → Authorization Policy** → find `Basic_Authenticated_Access` → toggle Status to **Disabled**.
   - Re-enable it later if you need it for other testing.

2. **Plug your dot1x-supplicant test laptop** into the access port. Watch:
   - On the switch: `show authentication sessions interface <port> details`
   - On ISE: **Operations → RADIUS → Live Logs** — find the row for your test MAC.
   - Click the magnifying glass on the Live Log row to see the **Steps** and which rule matched.

3. **Expected result:** Live Log shows `ISEPolicySetName: LAB-WIRED-DOT1X`, `IdentityPolicyMatchedRule: Dot1X`, `AuthorizationPolicyMatchedRule: LAB-EMPLOYEES`.

4. **Capture a screenshot** of the Live Log detail. This is exam study gold.

## Part 4 — Break it on purpose

Three quick experiments. After each, Live Logs tell you exactly what happened.

### 4.1 — Wrong cert issuer

Issue your test laptop a self-signed cert (not from the internal CA). Reauth.

**Expected:** AuthN passes (EAP-TLS handshake completes), but AuthZ rule 1 misses on `Certificate:Issuer CN`. Falls through to Default rule (`LAB-DENY`).

**Lesson:** AuthN success does not mean AuthZ success.

### 4.2 — Forget the Service-Type gate

Edit `LAB-EMPLOYEES` rule, remove the `Service-Type EQUALS Framed` condition. Plug in an *unknown MAB endpoint* (a Linux VM with a MAC not in Internal Endpoints).

**Expected:** depending on the rest of your conditions, you may now see ISE evaluate `Certificate:Issuer CN` for a MAB request — which is meaningless. The rule won't match (no cert), but ISE has wasted cycles. Worse: in Live Logs you'll see ISE doing AD lookups against the MAC address.

**Lesson:** gate every authz rule with the auth method it's meant for.

### 4.3 — Reverse rule order

Move `LAB-IOT-CAM` above `LAB-EMPLOYEES`. Reauth your dot1x laptop.

**Expected:** if your laptop's MAC happens to be in the IoT identity group, you'd land in IoT VLAN — even though dot1x succeeded. This is the classic "rule order matters" bug.

**Lesson:** order rules from most-specific-condition to least.

## Part 5 — Clean up

When done:
- Re-enable `Basic_Authenticated_Access` on Default if you want.
- Either keep `LAB-WIRED-DOT1X` for the next lab (BYOD will reuse it) or delete it.
- Take final screenshots of:
  - The Policy Sets list with both sets visible
  - The full LAB-WIRED-DOT1X authz rules
  - At least one Live Log success and one Live Log failure with rules visible

---

## What you'll have memorized after this lab

- The Policy Sets navigation path (you'll have clicked it 30 times)
- The difference between authn and authz (you'll have seen authn-pass-authz-deny in Live Logs)
- Why Service-Type matters (you'll have broken a rule by leaving it out)
- Why Default matters (you'll have watched Default catch a wrong-issuer cert)
- What `Basic_Authenticated_Access` does and why it's a footgun (you'll have disabled it intentionally)

That's exactly the operational fluency the exam tests.
