---
type: build-decision
id: D-077
title: "Midnight on peacock at 4.25 to 1 kept as approved, pending AMPM's call on 148F8A"
date: 2026-11-07
status: proposed
area: screens
relates: []
supersedes: ""
session: "[[2026-11-07 Session 639f78ef]]"
tags: [ampm, build-decision, area/screens, status/proposed]
---
# D-077 Midnight on peacock at 4.25 to 1 kept as approved, pending AMPM's call on 148F8A

## Context

The handoff states that "peacock with midnight ink" passes WCAG AA. Measured, midnight `#0E1534` on peacock `#138A85` is **4.25:1**. The pair carries small text: the verified/done chip (11–12px bold), the active tab and segment labels (12–13px), the highlight button ("Approve version 3", 13–15px). The engineering standards ask 4.5:1 for text; 4.25:1 passes AA only for large text and UI components (3:1).

## Decision

Keep the approved peacock for now and record the pair in the contrast gate at 3:1 with the reason (`contrastPairs` in `packages/tokens/src/colors.ts`; a test pins this as the only exception). Peacock is never used as a text colour on midnight.

**Open for AMPM:** switch peacock to `#148F8A` (every channel +3.5%, visually the same colour), which reaches **4.53:1** and removes the exception. It is a one-line token change.

## Alternatives considered

- Pearl text on peacock: 3.53:1, worse.
- Larger or bolder labels to qualify as large text (18.66px bold): breaks the approved chip and tab sizes.
- Changing the hex without asking: alters an approved brand colour; AMPM's call.

## Consequences

- Until AMPM decides, small midnight-on-peacock labels sit 0.25 below the standard's 4.5:1.
- If AMPM accepts `#148F8A`, change `palette.peacock` and the derived values (`highlight-soft`, glow), rebuild the tokens, and remove the two exception rows; the gate test then holds every pair at 4.5:1.

## Links
- Session: [[2026-11-07 Session 639f78ef]]
- Related: (none)
