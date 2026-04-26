# SISE 300-715 — ISE Lab Prep

> Hands-on study companion for the **Cisco SISE 300-715** exam (Implementing and Configuring Cisco Identity Services Engine), structured around the official Cisco blueprint.

This is not a flashcard set. It's a **lab notebook**: nine topics, each one with a focused hands-on lab against a real ISE deployment, the exam-relevant concepts you need to know cold, and a self-test you can run on yourself.

## Why this structure

The SISE exam tests *operational* knowledge — can you actually configure ISE, not just describe it. Cisco's blueprint maps cleanly to nine functional areas; this repo is one folder per area. The pattern in each folder is identical:

| File | What it is |
|---|---|
| `README.md` | The topic in plain English. Concepts the exam tests. Why each matters. |
| `lab.md` | Step-by-step hands-on procedure against your real ISE + switch. |
| `self-test.md` | 10–15 questions in the shape the exam asks them. Answers at the bottom. |
| `notes.md` | Your space. Use it. The repo is yours. |

You can work the topics in any order. The blueprint isn't sequential — pick what's weakest first.

## Prerequisites

- A working Cisco ISE (≥ 3.1) deployment — at least one PAN/MnT/PSN node (combined for lab is fine).
- A Cisco switch capable of 802.1X (IOS-XE 17.x or 16.x ideally; older works but some screens differ).
- An AD-joined Windows test client.
- A test endpoint with no supplicant (Linux VM, IP phone, anything that can MAB).
- A working internet connection from the lab so ISE can pull profiler feeds and updates.
- Optional but useful: a syslog collector (a basic Linux VM with rsyslog is fine).

If you already worked through [`twerkingpig/ise-critical-auth-lab`](https://github.com/twerkingpig/ise-critical-auth-lab), you have most of this set up.

## Suggested study schedule

A realistic ~6 week schedule, ~5 hours/week:

| Week | Topic | Why this order |
|---|---|---|
| 1 | 01-architecture | Foundation. Get node roles and policy flow in your head before touching anything. |
| 2 | 02-policy-sets | Where ~25% of the exam lives. Get the policy evaluation flow cold. |
| 2 | 04-profiler | Pairs naturally with policy sets — most authz rules use endpoint profile conditions. |
| 3 | 05-byod | Builds on 02 + 04. Self-registration portals + EAP-TLS + provisioning. |
| 3 | 03-webauth-guest | Same portal mechanics as BYOD; do them back-to-back. |
| 4 | 06-posture | Heavy topic. Complex enough to deserve a full week. |
| 4 | 07-tacacs-device-admin | Lighter. Knock it out the same week as posture. |
| 5 | 08-trustsec | The "concepts" part is small. The "operational" part is fiddly. |
| 6 | 09-integrations | pxGrid, AD, certs. Mostly review + filling gaps. Final week. |

This isn't sacred. If you already know one area, skip it. If you spend two weeks on policy sets, that's also fine.

## How to use the labs

1. Read the topic `README.md` end-to-end. Don't skim.
2. Run the `lab.md` procedure end-to-end *in your real ISE*. Capture screenshots — they're invaluable for review.
3. Answer the `self-test.md` cold. Check answers. Re-do anything you got wrong by going back to the lab.
4. Update `notes.md` with anything that surprised you. Future-you will thank you.

The exam doesn't reward "I read about this" — it rewards "I did this and remember the menu paths." Do the labs. Write the notes.

## Repo structure

```
sise-300-715-prep/
├── README.md                       # You are here
├── blueprint.md                    # Cisco's blueprint mapped to repo folders
├── 01-architecture/
├── 02-policy-sets/
├── 03-webauth-guest/
├── 04-profiler/
├── 05-byod/
├── 06-posture/
├── 07-tacacs-device-admin/
├── 08-trustsec/
├── 09-integrations/
└── _assets/
    └── screenshots/
```

## Status

| Topic | Status |
|---|---|
| 01-architecture | ⏳ scaffolded |
| **02-policy-sets** | ✅ **complete (template for the rest)** |
| 03-webauth-guest | ⏳ scaffolded |
| 04-profiler | ⏳ scaffolded |
| 05-byod | ⏳ scaffolded |
| 06-posture | ⏳ scaffolded |
| 07-tacacs-device-admin | ⏳ scaffolded |
| 08-trustsec | ⏳ scaffolded |
| 09-integrations | ⏳ scaffolded |

Section 02 is the canonical example — every other section follows that shape. Update this table as you fill in each topic.

## License

This is study material derived from the publicly available Cisco SISE 300-715 blueprint. No proprietary Cisco content is reproduced. Lab procedures are written from operational experience.

## Disclaimer

Cisco SISE 300-715 is a registered exam of Cisco Systems. This repo is independent study material, not endorsed by or affiliated with Cisco.
