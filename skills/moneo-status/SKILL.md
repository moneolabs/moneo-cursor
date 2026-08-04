---
name: moneo-status
description: >-
  Show the agent wallet's address, balance, spending limits, open positions,
  and recent activity. Use when the user asks for wallet status, balance,
  funding address, what the agent is allowed to spend, or what it has done.
---

# Moneo wallet status

All four calls below are free and settle nothing. Run them and present one
compact picture.

1. `get_balance` — report the address and the balance, and separate what is
   available from what is already committed.
2. `get_spending_limits` — report the per-transaction cap, the rolling 24-hour
   cap and how much of it is left, the velocity cap, and the escalation
   threshold.
3. `list_positions` — report open positions with cost basis and unrealized
   P&L. Say plainly if there are none.
4. `list_payments` — summarize what was sent this session, including anything
   the guard refused and why.

Present it in that order: what it holds, what it may spend, what it is in, what
it has done.

If the balance is zero, lead with the address and say it needs USDG before any
spending tool can do anything, plus a little ETH on Robinhood Chain for gas.
Do not treat a zero balance as an error — a fresh wallet starts empty.

If a refusal shows up in the payment history, state the rule it hit rather than
describing it as a failure. It is the policy working.
