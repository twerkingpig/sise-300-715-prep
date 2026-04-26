# RETAKE-PLAN.md — SISE 300-715

> An accountability artifact, not a study guide. The study guide is the rest of this repo.

This file exists to make a comeback visible. The first attempt at SISE 300-715 didn't pass. This document is the plan for the next 5 months.

---

## Context

| | |
|---|---|
| **First attempt** | April 5, 2026 — Did not pass |
| **Cisco retake policy** | 180 days from a *passed* exam (no waiting period after a fail; can retake after 5 days) |
| **Target retake date** | Early September 2026 (~5 months out) |
| **Why September not April** | 4 months of structured lab work + 1 month buffer for sample exams and final review. Don't race; do it right. |

## Honest read of the first attempt

A passing score requires balanced performance across all sections. The first attempt was **strong in two areas, weak in three.**

### Strengths to preserve

- **Profiler** — solid grasp of probes, profile policies, endpoint identity groups. Don't lose this.
- **Network Access Device Administration (TACACS+)** — solid. Smaller section, but a strength.

### Critical weak spots

Three sections shared a structural pattern: **multi-step flows with multiple components that all have to align.**

- **Web Auth and Guest Services** — the redirect mechanics, portal types, sponsor groups vs guest types
- **BYOD** — Single vs Dual SSID flows, NSP, cert provisioning, lifecycle states
- **Endpoint Compliance (Posture)** — PostureStatus state machine, AnyConnect / Client Provisioning, requirements vs conditions vs policies

### Borderline

- **Policy Enforcement** — the highest-weight section (~25%). Solid foundation, but middling performance translates to a big point loss. Even moving from middling to confident here is the single biggest leverage point.
- **Architecture and Deployment** — covered the basics; can be tightened.

## The pattern (this is the lesson)

The strong sections were *concrete*: "what command, what attribute, what protocol." Either you know it or you don't. Studying these by reading worked.

The weak sections were *operational*: "what RADIUS attributes return for a redirect; what does the portal user click; what triggers the CoA-Reauth; how does the agent report back." These can be *read* a hundred times and still feel abstract until you've built one with your own hands and watched it work.

**The comeback is built on lab work, not more reading.**

---

## 5-month plan

| Month | Focus | Deliverable |
|---|---|---|
| 1 (May) | Web Auth & Guest — three flow types built end-to-end | `03-webauth-guest/lab.md` filled in with actual procedure; screenshots in `_assets/screenshots/03-webauth-guest/`; `notes.md` with surprises |
| 2 (June) | BYOD — Single + Dual SSID, real cert provisioning | `05-byod/lab.md` filled in; iPhone or iPad enrollment captured; lost-device flow demonstrated |
| 3 (July) | Posture — full Compliant/NonCompliant/Unknown lifecycle | `06-posture/lab.md` filled in; AnyConnect ISE module deployed; PRA tested |
| 4 (August) | Policy Enforcement deep dive + revisit weak sections | `02-policy-sets/lab.md` rerun; build novel policy sets; self-test all weak sections to 95%+ |
| 5 (Early September) | Sample exams + final review + retake | Boson / Cisco practice exams; gap analysis; book the retake |

Each month ends with the corresponding section's self-test scored cold. Track results in the table below.

## Lab completion tracker

Check off when the section's `lab.md` has been built out **AND** run end-to-end at least once against real ISE. Update as you go.

- [ ] 01-architecture — read & understood
- [ ] 02-policy-sets — lab.md run end-to-end (already a canonical doc; needs *doing*, not writing)
- [ ] 03-webauth-guest — lab.md filled in + run
- [ ] 04-profiler — lab.md filled in + run
- [ ] 05-byod — lab.md filled in + run
- [ ] 06-posture — lab.md filled in + run
- [ ] 07-tacacs-device-admin — lab.md filled in + run (lighter; skip if confident)
- [ ] 08-trustsec — lab.md filled in + run
- [ ] 09-integrations — lab.md filled in + run

## Self-test score tracker

Take the self-test cold at the start of each month, then again at month-end after the lab. Track delta.

| Section | Self-test 1 (date / score) | Self-test 2 (date / score) | Goal |
|---|---|---|---|
| 01-architecture | | | 11/12 |
| 02-policy-sets | | | 11/12 |
| 03-webauth-guest | | | 11/12 |
| 04-profiler | | | 11/12 |
| 05-byod | | | 9/10 |
| 06-posture | | | 9/10 |
| 07-tacacs-device-admin | | | 7/8 |
| 08-trustsec | | | 7/8 |
| 09-integrations | | | 7/8 |

When all sections are at goal, you are ready to retake.

## Session log

Append-only. One line per study session. The discipline of writing a line at the end of every session is half the battle.

| Date | Time spent | What I did | What I learned / what tripped me |
|---|---|---|---|
| 2026-04-26 | n/a | Wrote this plan; published cert prep repo | Strong sections were concrete; weak sections were operational. Need lab work, not more reading. |
| | | | |
| | | | |

## Ground rules for this comeback

1. **Don't open a new study book.** This repo + real ISE + real switch is the curriculum.
2. **No exam-dumps.** They're a shortcut to passing one specific question pool, not understanding. Cisco rotates pools and an exam-dump-trained engineer can't do the actual work.
3. **One section at a time.** Finish a section's lab + self-test before moving on. Don't dabble across three sections in one week.
4. **Honest scoring.** Self-tests cold, no peeking. A 4/10 honestly faced is more useful than an 8/10 with the answers half-visible.
5. **Update this file every session.** A 30-second log entry beats a fancy tracker you abandon. The session log table above is the canonical source.
6. **No retake until all sections are at goal.** Don't book the exam to pressure yourself into studying — book it after the data shows you're ready.

---

## Why this file is public

Two reasons:

1. **Accountability.** Hidden plans don't get followed.
2. **Other people study for SISE.** A real "I failed once, here's how I came back" record is more useful to other engineers than a polished study guide. If even one person finds it via search and recognizes themselves, that's worth the visibility.

Specific scores from the first attempt are intentionally not in this file. The pattern is the lesson; the numbers are noise.
