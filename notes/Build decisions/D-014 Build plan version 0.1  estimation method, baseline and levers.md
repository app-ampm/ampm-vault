---
type: build-decision
id: D-014
title: "Build plan version 0.1: estimation method, baseline and levers"
date: 2026-10-26
status: accepted
area: process
relates: [D-011, D-012, D-013]
supersedes: ""
session: "[[2026-10-26 Session 6d9c1b45]]"
tags: [ampm, build-decision, area/process, status/accepted]
---
# D-014 Build plan version 0.1: estimation method, baseline and levers

## Context
On 26 October 2026, after the blueprint, the round-two decisions and the engineering standards, the user asked for the end-to-end build plan to build AMPM with Claude: schedule, effort and cost, working method, decision calendar and launch plan. Three attempts to run it as a multi-agent workflow were interrupted; the user chose to have it written directly in the session. The developer's planning inputs (K1 to K22) were not yet answered, so capacity and the commercial form had to be carried as scenarios and a formula.

## Decision
- The plan is `docs/review/build-plan.md` (Part A11) with a short version (Part A12): one critical path for one developer with Claude Code through six phases (bootstrap, foundation, wedding planning, guest operations, live execution, stabilisation) to one launch, with internal releases to staging and an AMPM acceptance gate at every phase exit, a pilot wedding run through phases 2 to 4, and four weeks of hypercare.
- Effort is estimated bottom-up from the blueprint's inventories in developer-days (eight hours of the developer's time, Claude writing most code), with the rates shown: table 0.6 day, client screen 1.0, team screen 1.1, guest step 0.8, public page 0.5, flow 2, adapters 0.5 to 4, job 0.75; offline-first 20, policy pipeline 15, infrastructure 12, bootstrap 20, stabilisation 18; acceptance support 10 percent, pilot 8, documentation 8; buffer 25 percent. Totals: about 400 low, 510 likely, 700 high developer-days; the visual design track (about 15 days) is excluded as optional.
- Calendar: about 105 weeks to launch at 40 hours a week (about 24 months), 32 months at 30 hours, 48 months at 20 hours; milestones G0 week 5, G1 week 49, G2 week 69, G3 week 82, G4 week 95, launch week 105.
- Cost: development as days times the agreed rate (illustrated at ₹10,000 a day: ₹40, 51, 70 lakh); one-off ₹2.5 to 5.5 lakh, mostly the penetration test; running ₹45,000 to 1,00,000 a month on the drafted AWS topology or ₹22,000 to 50,000 on the Supabase variant, plus 2.36 percent of subscription revenue to Razorpay. Prices verified from current sources on 26 October 2026 at ₹84 to the dollar.
- Levers written for the founder and the developer to choose before signing: discovery live after phase 1 (reopens R17), web-first client portals, tablets after launch, hosted chat, the Supabase variant, a second developer for phases 1 and 2, more hours.
- The decision calendar maps R6 to R16, R18 and R19 and the K questions to the sprint that needs them; AMPM's deliveries carry week numbers.

## Alternatives considered
- A multi-agent estimate with independent estimators, a judge, cost verifiers and adversarial critics. Prepared three times and interrupted each time; the user chose the direct route. The method and rates are shown in the plan so the numbers can be challenged, which recovers most of what the adversarial pass would have given.
- Estimating at the module level only. Rejected: the per-unit rates on 108 tables, 126 screens, 12 flows and 14 integrations make the number traceable to the blueprint, and the share of effort (half screens, a quarter data and API) is visible.
- Stating dates. Rejected until the developer's hours (K1) are known; the plan gives three capacity scenarios instead.

## Consequences
- The honest size of the single-launch plan, about two years full time for one person, is on the first page; the founder and the developer must choose levers before the plan is signed.
- After sprint 3 the measured pace replaces the assumed rates and the totals are re-run (assumption B-2).
- The plan's assumptions B-1 to B-12 and the sign-off box are the record of what was agreed; the sprint themes become sprint goals once capacity and the commercial form are chosen.
- Published as a fifth private page and two PDFs; the full package and the decision brief remain unrepublished.

## Links
- Session: [[2026-10-26 Session 6d9c1b45]]
- Related: [[D-011]], [[D-012]], [[D-013]]
